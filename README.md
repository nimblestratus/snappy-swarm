snappy-swarm
============

The following steps/scripts will build a "snappy" [Docker swarm](https://github.com/docker/swarm) using Ubuntu's
[Snappy](http://www.ubuntu.com/cloud/tools/snappy) core release.

If being run on a host with a decent number of cores and ram, then
start from the beginning and install all of the pieces on that
host. Otherwise you can create vm's running snappy which then are
added to the swarm.

## Preparing the Host

This assumes a 64-bit server which is running Ubuntu 14.04. Verify
that updates are installed:

```
    sudo apt-get update
	
    sudo apt-get upgrade -y
```

### uv-tool, kvm, and Ubuntu core

1. Install uv-tool (from directions located at
   [Snappy Ubuntu Core and uvtool](http://ubuntu-smoser.blogspot.com/2014/12/snappy-ubuntu-core-and-uvtool.html)):

```
    sudo apt-add-repository ppa:snappy-dev/tools
    sudo apt-get update
    sudo apt-get -y install uvtool
    newgrp libvirtd
```

2. If you don't have a `~/.ssh/id_rsa.pub` then:

```
    ssh-keygen
```

3. Download the images

```
    uvt-simplestreams-libvirt sync --snappy flavor=core release=devel
```

### Install Docker

We want a recent/new version of Docker, so we need to use a different
set of repositories than the "default". These instructions follow the
Docker on Ubuntu [installation instructions](https://docs.docker.com/installation/ubuntulinux/).

1. Verify that `apt` can handle `https` urls (and fix it if it can't!)

```
    [ -e /usr/lib/apt/methods/https ] || {
      apt-get update
      apt-get install -y apt-transport-https
    }
```

2. Add the Docker repository to your keychain

```
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys
```

3. Add the Docker repositiory and install docker:

```
    sudo sh -c "echo deb https://get.docker.com/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
    sudo apt-get update
    sudo apt-get install -y lxc-docker
```

4. To eliminate the need to `sudo` every time you run `docker` do the
   following:

> Note: there is a potential security issue; see
> [Docker Daemon Attack Surface](https://docs.docker.com/articles/security/#dockersecurity-daemon)
> for details.

```
    sudo groupadd docker
    sudo gpasswd -a ${USER} docker
    sudo service docker restart
```

5.
