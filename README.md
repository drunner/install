# Status

DEPRECATED. See http://drunner.io

dRunner is under development. Try it, don't trust it!

Current status (22/02/2016):
* Main dRunner scripts are functional but unoptimised (e.g. sometimes will pull twice in one operation).
* Core services are okay (helloworld, dtest, ddev, simplesecrets)
* Extra services (e.g. minecraft) are broken. Minecraft in particular is very broken - don't leave it running.

Next steps:
* dRunner is being re-written in C++ with becoming production ready in mind. The new version should be compatible with existing dServices without modification. ETA for completion is Wednesday.
* Extra services (minecraft, Ansible, samba) will be fixed.
* Support will be added for templating with Docker Compose rather than creating a custom file for volumes and extra containers.

# dRunner

dRunner is a script and a set of conventions to make it easy to install,
configure and use Docker containers on a Debian host via the command line interface.

dRunner eliminates the need to separately store and manage scripts to use the Docker container,
or deal with long docker run commands.

Features:
* dRunner compatible Docker Images are self contained - everything dRunner needs is inside
* Simple discoverable commands for using compatible services (no manual needed)
* Flexible configuration for each service, stored in Docker Volume containers that are managed for you
* Services can consist of any number of containers
* Backup an entire service to a single file, trivially restore on another machine
* Obliterating a service leaves the machine in a clean state, no cruft left
* Everything in containers is run as a non-root user, drunner on host runs as non-root
* Trivial to install a service multiple times with different configurations (e.g. mulitple minecraft servers)
* Ansible friendly for automation (see [Exit Codes](https://github.com/j842/dr#exit-codes) below).
* Small footprint: /opt/drunner, /etc/drunner and one symbolic link per service in /usr/local/bin

# Usage

## Example

### First time installation

We assume here you have a standard user account called testuser which you'll use for drunner.

#### Dependencies

dRunner needs docker. You can install it as root with:
```
wget -nv -O /tmp/install_docker.sh https://goo.gl/2cxobx ; bash /tmp/install_docker.sh
```
and give testuser permissions to run docker with:
```
adduser testuser docker
```


#### Installing Docker Runner

Now switch to a normal user (not root!). Install dRunner on the host by downloading the install script:
```
 wget https://raw.githubusercontent.com/drunner/install/master/drunner-install
 bash drunner-install ~/drunner
 source ~/.profile
```
Now you're ready to try things.

### Running some containers

#### Helloworld

Install and try the [helloworld](https://github.com/j842/docker-dr-helloworld) example:
```
 drunner install drunner/helloworld
 helloworld run
```
helloworld is now in your path, you can run it directly, e.g. with no arguments
to see the help.

Back up helloworld to an encrypted archive (including all settings and local data),
then obliterate it, leaving the machine clean:
```
PASS=shh drunner backup helloworld hw.b
drunner obliterate helloworld
```
Restore the backup as hithere, and run it:
```   
PASS=shh drrunner restore hw.b hi
hi run
```

### Running some tests

dRunner can test containers for compatibility and functionality. Try it out with:
```
drunner install drunner/dtest
dtest test drunner/helloworld
```
### dRunner Images to play with

Other images to try:
* [minecraft](https://github.com/j842/drunner-minecraft) - really easy minecraft server.
* [simplesecrets](https://github.com/j842/drunner-simplesecrets) - store low security secrets in S3.
* [samba](https://github.com/drunner/samba) - run samba for development work.

(more coming soon!)

## General Use

Install a container (e.g. from DockerHub) that supports dr and call the service 'SERVICENAME'.
```
drunner install IMAGENAME SERVICENAME
```

Manage that service:
```
SERVICENAME COMMAND ARGS
```
The available COMMANDs depend on the service; they can be things like run and configure. You can get help on the service
which also lists the available COMMANDs with
```
SERVICENAME
```

Other commands that work on all services:
```
drunner obliterate SERVICENAME                 -- uninstalls service and removes ALL data! Leaves host in clean state.
drunner update SERVICENAME                     -- update service scripts from container (and docker pull)

PASS=? drunner backup SERVICENAME BACKUPFILE   -- backup container, configuration and local data.
PASS=? drunner restore BACKUPFILE SERVICENAME  -- restore container, configuration and local data.
```


# dRunner Compatibility

To see how to make a dRunner compatible docker image [read the documentation](https://github.com/j842/dRunner/blob/master/MAKECOMPATIBLE.md).

## Exit Codes

The convention for exit codes is:
* 0 for success,
* 1 for error and
* 3 for no change

This is to aid Ansible use.

## Security
See [Docker's Security Statement](https://docs.docker.com/engine/security/security) for information on security and docker.
