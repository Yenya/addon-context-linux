# Linux VM Contextualization

## Description

These are the source of the contextualization packages used by VM to be configured with the information generated by OpenNebula.

## Development

To contribute bug patches or new features, you can use the github Pull Request model. It is assumed that code and documentation are contributed under the Apache License 2.0.

More info:
* [How to Contribute](http://opennebula.org/addons/contribute/)
* Support: [OpenNebula user forum](https://forum.opennebula.org/c/support)
* Development: [OpenNebula developers forum](https://forum.opennebula.org/c/development)
* Issues Tracking: Github issues (https://github.com/OpenNebula/addon-context-linux/issues)

## Authors

* Leader: Javier Fontan (jfontan@opennebula.org)

## Compatibility

This add-on is compatible with OpenNebula >= 4.6.

## Requirements

  * Ruby >= 1.8.7
  * gem fpm
  * dpkg utils for deb package creation
  * rpm utils for rpm package creation

On Ubuntu/Debian you can install the package `rpm` and you will be able to generate both rpm and deb packages.

## Use

### Package Description

Here are located the files needed to generate OpenNebula contextualization packages. The packages generated contain these files:

* `/etc/udev/rules.d/*`     These files disable the udev network an cdrom
                            generation
* `/etc/init.d/vmcontext`   This is the startup script that will try to mount
                            context cdrom, load contextualizaton variables,
                            call scripts in the contextualization scripts
                            directory and call init.sh if it exists in the
                            context cd.
* `/etc/one-context.d/*`    This directory holds the scripts that will be
                            called by vmcontext script. They should be named
                            starting with a number so they are called in order.

By default only the network configuration context script is included in the
packages. These scripts are different for rpm and deb based distributions and
are located in `base_<deb|rpm>` directories.

The packages also have a post-install script that does these steps:

  * Delete persistent cd and net rules from /etc/udev/rules.d
  * Links vmcontext script to /etc/rc<runlevel>.d
  * Deletes network configuration files

### Package Generation

The script `generator.sh` generates both deb and rpm packages and can be configured to include more files in the package or change some of its parameters.

On start it creates a temporary directory and copies there:

  * `base` directory
  * `base_<deb|rpm>` directory
  * Any file or directory from the arguments.

Then these files are included in the package.

The default parameters to create a package are as follows:

    VERSION=1.0.1
    MAINTAINER=OpenNebula Systems <support@opennebula.systems>
    LICENSE=Apache
    PACKAGE_NAME=one-context
    VENDOR=OpenNebula Systems
    DESCRIPTION="
    This package prepares a VM image for OpenNebula:
      * Disables udev net and cd persistent rules
      * Deletes udev net and cd persistent rules
      * Unconfigures the network
      * Adds OpenNebula contextualization scripts to startup

    To get support use the OpenNebula mailing list:
      http://opennebula.org/community:mailinglists
    "
    PACKAGE_TYPE=deb
    URL=http://opennebula.org

You can change any parameter setting an environment variable with the same name. For example, to generate an rpm package with a different package name:

    $ PACKAGE_TYPE=rpm PACKAGE_NAME=my-context ./generate.sh

You can also include new files. This is handy to, for example, include new scripts executed to contextualize an image. For example, we can have an script that install a user ssh key. We will create the file hierarchy that will go inside the package in a directory:

    $ mkdir -p ssh/etc/one-context.d
    $ cp <our-ssh-script> ssh/etc/one-context.d/01-ssh-key
    $ ./generate.sh ssh/etc

NOTE: The generator must be executed from the same directory it resides.

