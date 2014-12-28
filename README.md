snappy-swarm
============

The following steps/scripts will build a "snappy" [Docker swarm](https://github.com/docker/swarm) using Ubuntu's
[Snappy](http://www.ubuntu.com/cloud/tools/snappy) core release.

If being run on a host with a decent number of cores and ram, then
start from the beginning and install all of the pieces on that
host. Otherwise you can create vm's running snappy which then are
added to the swarm.

## Preparing the Host

> The `bin/prep-host` script will perform the steps listed below. The
> steps are listed below with commentary for reference.

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

1. Do the "easy" install:

```
    curl -sSL https://get.docker.com/ubuntu/ | sudo sh
```

2. To eliminate the need to `sudo` every time you run `docker` do the
   following:

> Note: there is a potential security issue; see
> [Docker Daemon Attack Surface](https://docs.docker.com/articles/security/#dockersecurity-daemon)
> for details.

```
    sudo groupadd docker
    sudo gpasswd -a ${USER} docker
    sudo service docker restart
```

5. Modify the Docker start script to bind to a port on the private
   network in addition to the file socket:

```
   sudo service docker stop
   sudo sh -c 'echo DOCKER_OPTS=\"-H tcp://192.168.122.1:4243 -H unix:///var/run/docker.sock\" >> /etc/default/docker'
   sudo service docker start
``` 

### Install and Configure go

1. Install the go binaries:

```
    sudo apt-get install -y golang-go golang-go.tools
```

2. Configure the environment.  We're going to set up some global
   variables and ensure that they're loaded into subsequent shell
   sessions:

```
	sudo sh -c "cat > /etc/profile.d/golang-env.sh" <<EOF
	export GOROOT=/usr/share/go
	export GOPATH=\${GOROOT}
	export PATH=\${PATH}:\${GOROOT}/bin
	EOF

	. /etc/profile.d/golang-env.sh
```

### Install swarm

We're installing it to the global position; in order to do this we
need to "cheat" a little bit & specify an environmental variable to
tell it where to install:

```
sudo GOPATH=$GOPATH go get -u github.com/docker/swarm
```

### Logout

Logout and log back in to ensure all the environment settings are
captured.

## Prepare the swarm
