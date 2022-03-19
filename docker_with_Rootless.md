## Introduction 

When you install Docker, it will need full permissions on the system (root), and this causes a security problem because containers and the (daemon) Docker service will work as root.

If it is hacked, the hacker will have full access to the system, and there is no real isolation of the containers.

After the release of the Podman project, which is primarily designed to run without root, the pressure on Docker to support the feature increased, and it has supported it for a long time, so that containers run as normal users but the Docker service (daemon) works as root,

But the feature to run Docker without permission completely was released as a beta in 2019, and it was fully released in 2020.

And in this article, I will explain how to install Docker without root access.


##  Why Docker instead of Podman

The biggest reason to use Docker instead of Podman is interoperability.

You don't need to deal with Podman and adjust things because it is different from Podman. Here is a Docker without root.

If you don't care about compatibility, you may find Podman's infrastructure and not being directly affiliated with any company excellent. Also, it makes it easier for you to play k8s files, so if you use k8s a lot, you may like podman.

##  Disadvantages of running Docker in rootless mode

The biggest downside to this mode is the network, and these problems are also present in podman.

**Rootlesskit**

By default, Docker uses a rootless network.

Because it is the fastest, with a speed of up to 30 Gbitps and supports IPv4 and IPv6.

But it has very big problems.

Containers will not have the external IP of the request, and all requests will appear from 127.0.0.1.

This is a big problem, especially if, for example, you want to put in protection that limits distributed denial-of-service (DDOS) attacks because all requests will come from the same address.

**Slirp4netns**

This mode solves this problem, and shows the original address of the request.
But it has two problems.
  - ipv6 not supported.
  - The speed is much slower, about 7gbitps.

Prerequisites : 

Ensure that the **newuidmap** **newgidmap** packages are installed and that there are 65,536 child ids.

**newuidmap** verifies that the caller is the owner of the process indicated by **pid**.
```
id -u
1001
```

```
whoami
testuset
```

```
grep ^$(whoami): /etc/subuid
testuser:231072:65536
```

```
grep ^$(whoami): /etc/subgid
testuser:231072:65536
```

What do these numbers mean?

The numbers after the username mean this : 
  
  - The first number is the first id allowed to use.

  - Numbers after **:** How many id do you have.

For example, it starts with 231072, id 0 means 231072 and id 1000 means 241072.

Install the **dbus-user-session** and **fuse-overlayfs** packages.

For Debian :
Install **dbus-user-session**

```bash
sudo apt-get install -y dbus-user-session
```
Install **fuse-overlayfs**

```bash
sudo apt-get install -y fuse-overlayfs
```

Arch Linux / CentOS 8 /RHEL 8 : 

Installing **fuse-overlayfs**

```bash
sudo pacman -S fuse-overlayfs
```

> It is recommended to use Kernel 5.11 or later

## install docker rootless

Follow the normal Docker installation 

Let's take Ubuntu as an example.

**Install Docker Engine on Ubuntu**

To get started with Docker Engine on Ubuntu, make sure you meet the prerequisites, then install Docker.

Your Ubuntu version should be 64-bit and at least 18.04 (LTS).
To get information about your distro version, run :
```bash
lsb_release -a 
```
In my case 


![sysinfo](https://github.com/Codingforhackers/Linuxhandbook/blob/main/sysinfo.png)


To uninstall old versions run :

```bash
 sudo apt-get remove docker docker-engine docker.io containerd runc
```
Install Docker Engine : 


```bash
 sudo apt-get update
 sudo apt-get install docker-ce docker-ce-cli containerd.io
```
Verify that Docker Engine is installed correctly by running the **hello-world** image.


```bash
sudo docker run hello-world
```

Then install the **docker-ce-rootless-extras** package via your distribution's package manager.

Then install it via **dockerd-rootless-setuptool.sh install**
Finally, there will be two things written: **export=xxx**
Copy and paste them into the last .bashrc file or if you are using ZSH, the .zshrc  file . 

Sign out and then sign in.

## After installation

Run daemon docker rootless

```bash
 systemctl --user start docker

```
Run rootless docker when booting

```bash
systemctl --user enable docker
sudo loginctl enable-linger $(whoami)
```

## Using the docker socket api with docker rootless


In case you have any container that uses socket api (meaning it uses /var/run/docker.sock
Use the user socket, for example, for the **watchtowerr** container:

```
version: "3"
services:
    watchtower:
        image: containrrr/watchtower 
        volumes:
            - $XDG_RUNTIME_DIR/docker.sock:/var/run/docker.sock

```

The **$XDG_RUNTIME_DIR** variable is the location of the home directory, and since Docker is running as a normal user the socket will be inside the same directory.


Ressources :
https://docs.docker.com/engine/security/rootless/
