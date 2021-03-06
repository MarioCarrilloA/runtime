# Developers Clear Containers 3.0 Install

This guide is not intended for end-users. Instead, this guide provides
instructions for any developers eager to try Clear Containers 3.0 and who
want to build Clear Containers from the source code and are familiar with the
process.

## Requirements

  * [go 1.8.3](https://golang.org/)
  * [glibc-static](https://www.gnu.org/software/libc/libc.html)
  * [gcc](https://gcc.gnu.org/)

## Clear Containers 3.0 components

  * [Runtime](https://github.com/clearcontainers/runtime)
  * [Proxy](https://github.com/clearcontainers/proxy)
  * [Shim](https://github.com/clearcontainers/shim)

**IMPORTANT:** Do not combine [Clear Containers 2.1](https://github.com/01org/cc-oci-runtime) and [Clear Containers 3.0](https://github.com/clearcontainers).
Both projects ship ``cc-proxy`` and they are not compatible with each other.

## Setup the environment

1. Define GOPATH

```bash
$ export GOPATH=$HOME/go
```

2. Create GOPATH Directory

```bash
$ mkdir -p $GOPATH
```

3. Get the code

```bash
$ go get -d github.com/clearcontainers/runtime
$ go get -d github.com/clearcontainers/proxy
$ git clone https://github.com/clearcontainers/shim $GOPATH/src/github.com/clearcontainers/shim
$ go get -d github.com/clearcontainers/tests
```

## Build and install components

1. Proxy

```bash
$ cd $GOPATH/src/github.com/clearcontainers/proxy
$ make
$ sudo make install
```

2. Shim

```bash
$ cd $GOPATH/src/github.com/clearcontainers/shim
$ ./autogen.sh
$ make
$ sudo make install
```

3. Runtime

```bash
$ cd $GOPATH/src/github.com/clearcontainers/runtime
$ make build-cc-system
$ sudo -E PATH=$PATH make install-cc-system
```

For more details on the runtime's build system, run:

```bash
$ make help
```

4. Qemu-Lite, Clear Containers image and kernel

In Fedora:
```bash
$ source /etc/os-release
$ sudo -E VERSION_ID=$VERSION_ID dnf config-manager --add-repo \
http://download.opensuse.org/repositories/home:/clearcontainers:/clear-containers-3/Fedora\_$VERSION_ID/home:clearcontainers:clear-containers-3.repo
$ sudo -E dnf install -y qemu-lite clear-containers-image linux-container
```

In Ubuntu 16.04 or newer:
```bash
$ sudo -E apt-get install -y apt-transport-https ca-certificates curl software-properties-common
$ sudo sh -c "echo 'deb http://download.opensuse.org/repositories/home:/clearcontainers:/clear-containers-3/xUbuntu_$(lsb_release -rs)/ /' >> /etc/apt/sources.list.d/clear-containers.list"
$ curl -fsSL http://download.opensuse.org/repositories/home:/clearcontainers:/clear-containers-3/xUbuntu_$(lsb_release -rs)/Release.key | sudo apt-key add -
$ sudo -E apt-get update
$ sudo -E apt-get install -y qemu-lite clear-containers-image linux-container

```

## Enable Clear Containers 3.0 for Docker

1. Clear Containers configuration file

Edit `$SYSCONFDIR/clear-containers/configuration.toml` according to your needs.

Refer to [https://github.com/clearcontainers/runtime#debugging](https://github.com/clearcontainers/runtime#debugging)
for additional information how to debug the runtime.

2. Configure Docker for Clear Containers 3.0

```bash
$ sudo mkdir -p /etc/systemd/system/docker.service.d/
$ cat << EOF | sudo tee /etc/systemd/system/docker.service.d/clear-containers.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -D --add-runtime cc-runtime=/usr/local/bin/cc-runtime --default-runtime=cc-runtime

[Service]
# Allow maximum number of containers to run.
TasksMax=infinity

EOF
```

3. Restart Docker and Clear Containers systemd services

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
$ sudo systemctl enable cc-proxy.socket
$ sudo systemctl start cc-proxy.socket
```

## Run Clear Containers 3.0

```bash
$ docker run -it ubuntu bash
root@6adfa8386497732d78468a19da6365602e96e95c401bec2c74ea1af14c672635:/#
```
