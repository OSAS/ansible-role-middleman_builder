**This repository is obsolete**, this project moved to https://gitlab.com/osci/ansible-role-web_builder

Ansible module used by OSAS to deploy a static website builder. While
working, the role is still being written, as seen by the TODO list at the end
of this document.

Currently this role supports the following web generators:
* Ascii Binder (http://asciibinder.org/)
* [Jekyll](https://jekyllrb.com/)
* [Middleman](https://middlemanapp.com/)
* [Nikola](https://getnikola.com/)
* Planet (using Planet Venus)

# Example

The role requires a working web server somewhere, if possible managed by ansible
as well. In the simplest invocation, this should be something like this:

```
$ cat deploy_builder.yml
- hosts: webbuilder
  roles:
  - role: builder
    builder_name: website_example_org
    git_url: "https://git.example.org/website.git"
    rsync_location: /var/www/html
    rsync_server: www.example.org
    rsync_user: rsync_user
```

The role defaults to using Middleman, but you can choose the web generator you need
by setting the `builder` variable to either `middleman` or `jekyll`.

This will deploy the build of https://git.example.org/website.git by
using rsync as rsync_user on www.example.org, copying in /var/www/html. If you do
not provide it, then no deployment the default builder deployment method is used
instead, if any. The OpenShift deployment has been removed as version 2 is EOL.
It is also possible to only provide a rsync_location without the other parameters
for a local sync.

The script will not sync if build failed, and will not send email (that's on the
TODO list, see end of the file). Nevertheless, failures caught by Cron can be
sent to an email address if specified by `cron_error_email`; if you setup
multiple builders using the same UNIX user, then beware only the latest email
defined will be taken into account.

# Handling multiple repositories

In order to support multiple repositories, the script detect if
submodules are used and track the latest commit for each and force update
if needed. While that's not how submodules are supposed to be used, people
often forget to update submodules and so we need to use this as a workaround.

This cannot be turned off for now, but will be added as a option if needed.

# Regular rebuild

The role will rebuild the website on a regular basis, every 6h
by default. This can be changed with the parameter `rebuild_interval`, which express
the time between automated rebuild attempts if nothing changed, expressed in hours.

This feature is disabled on containers.

# Builder deployment specific parameters

Sometimes a builder deployment really needs to be adapted because there is no way to do so later in the build configuration.

- **nikola_theme**: theme name to install using Nikola's repository
                    (the configuration file is not parsed, you need to maintain both parameters in sync)

# Debug the build

In order to debug a non working build, the easiest is to connect to the
server as the non privileged user used to build (with ssh + sudo or su most of the time).

Each website to build will have a directory serving as workspace for the build, that
contains a status file of the form `status_$NAME.yml`

```
$ cat status_website_example_org.yml
last_build: '1467869767'
last_build_commit: 819bcaef161706441f9a4477d664b67a3616049a
last_build_human: Tue Jul  5 03:27:57 2016
submodule_commits: {}
```

Then, the build script can be run with `/usr/local/bin/build_deploy.py -d -f -n ~/website_example_org.yml`,
which would force a build (-f) without pushing (-n) with debug turned on (-d).

This feature is disabled on containers.

# Containers

If running in a container this role will not create crontabs but instead build the website at once. You may
use the rsync feature to locally install the resulting pages in the proper place. In this case the publishing
space may need to be created in between the builder is setup and the website built, so you may set
`builder_container_build_now` to False and use the `build` entrypoint manually when convenient.

# Jenkins integration

If you wish to use the role with a external system to trigger such as Jenkins, you will need to disable
the cronjob use to run the build job on a regular basis. This can be done by setting the variable `external_trigger`
to True. This will also likely requires to use the `--no-refresh` option for the build script to not mess with
Jenkins git updating, depending on the setup you want.

# Missing features

While being already used in production, several options are missing
- notification of failure by sending a email
- notification of the start of a build
  - either by publishing the logs
  - or by pushing a status file on the website
- proper logging of error
- handling automatically some errors (like rebuilding gems)
- change the schedule of automated rebuild
- make result directory available to subsequent roles to avoid
  copying files when unnecessary

