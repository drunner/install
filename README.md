## Under development. Try it, don't trust it!

# Docker Runner

Docker Runner (dr) is a script and a set of conventions to make it easy to install, 
configure and use Docker containers. 

Docker Runner eliminates the need to separately store and manage scripts to use the Docker container, 
or deal with long docker run commands.

Features:
* dr compatible Docker Images are self contained - everything dr needs is inside
* Simple discoverable commands for using compatible services (no manual needed)
* Flexible configuration for each service, stored in Docker Volume containers that are managed for you
* Services can consist of any number of containers
* Backup an entire service to a single file, trivially restore on another machine
* Destroying a service leaves the machine in a clean state, no cruft left
* Everything in containers is run as a non-root user
* Trivial to install a service multiple times with different configurations (e.g. mulitple minecraft servers)
* Ansible friendly for automation (see [Exit Codes](https://github.com/j842/dr#exit-codes) below).

# Usage

## Example

### First time installation

#### Dependencies

Docker Runner needs docker. You can install it with:
```
   wget -nv -O /tmp/install_docker.sh https://goo.gl/2cxobx ; bash /tmp/install_docker.sh
```
#### Installing Docker Runner

Install dr on the host by downloading the install script:
```
    wget https://raw.githubusercontent.com/j842/dr/master/dr-install
    bash dr-install /opt/dr
```
Now you're ready to try things.

### Running some containers

#### Helloworld

Install and try the [helloworld](https://github.com/j842/docker-dr-helloworld) example:
```
    dr hi install j842/dr-helloworld
    hi run
```

Back up helloworld to an encrypted archive (including all settings and local data), 
then destroy it, leaving the machine clean:
```
   PASS=shh dr hi backup hi.backup
   hi destroy
```
Restore the backup as hithere, and run it:
```   
   PASS=ssh dr hithere restore hi.backup
   hithere run
```

Other images to try:
* [simplesecrets](https://github.com/j842/docker-simplesecrets) - store low sercurity secrets in S3.

## General Use

Install a container (e.g. from DockerHub) that supports dr and call the service 'SERVICENAME'.
```
    dr SERVICENAME install IMAGENAME 
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
SERVICENAME update                        -- update service scripts from container (e.g. after docker pull)
SERVICENAME destroy                       -- destroy service and ALL data! Leaves host in clean state.

PASS=? dr SERVICENAME backup BACKUPFILE   -- backup container, configuration and local data.
PASS=? dr SERVICENAME restore BACKUPFILE  -- restore container, configuration and local data.
```
   

# Docker Runner Compatibility

## Example

For an example/template see: https://github.com/j842/docker-dr-helloworld

## User

dr runs as root on the host.
dr requires the Dockerfile to create a non-root user and siwtch to it with the USER command. You
can set up and use sudo in the container, but you can't run as root (for security). Use different UIDs for
different services to help with container isolation.

## Service Configuration File

The container must include the file 
```
/dr/service.cfg     -- define VOLUMES and EXTRACONTAINERS
```
This file is read by bash and defines an array ofthe paths to map as volume containers. It also
defines any additional containers that should be pulled on update (using the Docker Hub name).
See [helloworld](https://github.com/j842/dr-helloworld/blob/master/dr/volumes) for an example.

See below for how to mount these containers when you run commands.

## Backup/Restore 
You can backup and restore services. The backup is generally self contained, so can be restored to a different host.
(There may be external resources needed by the service, but for many its got everything. See the specific container for what it supports.).

Backup and restore of the volumes defined by /dr/volumes are managed by dr. The backup/restore scripts provided by the image
handle any other actions needed (such as dumping a database to a file).

## Files Required

In additiont to /dr/volumes, the container image must include a path /dr containing the following scripts that can be run on the host:
```
/dr/install        -- automatically run on host when installed
/dr/destroy        -- automatically run on host when destroyed
/dr/help           -- show help for commands available
/dr/backup  PATH   -- backup to files in PATH
/dr/restore PATH   -- restore from files in PATH
/dr/enter          -- get bash shell in container
```

Each of these bash scripts should begin by sourcing a _variables file in the same directory, i.e.
```
#!/bin/bash
source "$( dirname "$(readlink -f "$0")" )/_variables"
```

This file is created by dr when the service is installed and it sets several variables:
```
VOLUMES      Array of paths (as defined in the volumes file)
DOCKERVOLS   Array of the corresponding docker volume names
DOCKEROPTS   Array of the -v options to pass to docker to mount the volumes
SERVICENAME  Name of the service
IMAGENAME    Name of the docker image (e.g. j842/dr-helloworld)
INSTALLTIME  Datestamp for when the service was installed on the host
```

Using DOCKEROPTS, you can then run Docker like this from your /dr/ scripts:
```
docker run -i -t --name="mydocker" ${DOCKEROPTS[@]} ${IMAGENAME} echo "whee!"
docker rm "mydocker"
```

### Additional commands

You can also add any other command (bash script) that would be useful, e.g. run, configure etc.
Also source _variables from these if you wish.
```
/dr/ANOTHERCMD [ARGS...]  -- any other command needed.
```

Those commands are invoked from the host with
```
SERVICENAME ANOTHERCMD [ARGS...]
```

## Exit Codes

The convention for exit codes is:
* 0 for success,
* 1 for error and 
* 3 for no change 

This is to aid Ansible use.
