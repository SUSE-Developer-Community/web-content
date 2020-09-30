---
title: "Building Containers safely without Docker (Look Ma! No Docker!)"
date: 2020-09-29T13:42:36-07:00 
draft: true
---

If you want to create a container, the traditional way to go about it is to use just `docker build .` and let docker take care of it for you.

One problem with this is that it requires access to the Docker daemon which then runs as root. It also tends to lead to more bloated images as the typical Dockerfile starts with pulling in someone elses' base container (not always bad, just be aware of where your base containers come from).

These problems become more important when you start automating container builds as someone could use a CI/CD pipeline to execute their own code which would get run with elevated privileges! Removing the dependency on the Daemon removes this path of exploits.


My preferred way to build my container images is with [Buildah](https://buildah.io)!

Note: I'll probably be using "container" and "container image" interchangeably. It should be obvious from context what I mean.

Bulidah uses the standard CRI set of tooling and configuration which means that it runs very nicely along side CRI-O, Podman, and Skopeo making for a full (mostly drop in) replacement for Docker.

# What is a Container (simply)

A container is a collection of filesystem layers that when joined together provide everything needed to run the application in a self-contained way. It also includes a manifest that describes how to use what’s inside the container.
The process for building a container is to build a layer, save it, build the next layer, save it, etc... until you have everything you want in there. The reason for splitting out layers is to ease both the building and splitting out of layers. 
If a certain part of your build step always builds the same thing, why repeat that work? The same is for transport, if there’s a large baseline that each container needs, you don’t need to download it every time.


[Some graphic here?](!/img/asdas )

As these are really just a collection of bits, there's no magic to building one that requires a centralized service to do the build. Also, removing the 


# Toolset

There a little bit of overlap between the tools but they each do a different part of the puzzle.]

Buildah builds OCI compliant images. It can allow you to run commands in the container’s context as well as mount the container’s filesystem to move files in from elsewhere 

Podman runs images in a CRI compatible runtime (such as CRI-O). It is a drop in replacement for Docker and supports all of the same subcommands.

Skopeo copies images from one location to another and can handle switching formats if needed. 


# Installation 

To install on OpenSUSE, just run:

```bash
zypper in podman buildah skopeo
```

RedHat and Ubuntu (as well as most major distros) have these tools in their repos as well.

This will install the tools needed and all of their dependencies. 

To run as a non-root user, you will need to map your user to a list of user and group ids that can be 

This can be done with:

```bash
sudo sh -c 'echo `whoami`:100000:100001 >> /etc/subuid'
sudo sh -c 'echo `whoami`:100000:100001 >> /etc/subgid'
```

# Building the Container

The first thing we will do is build a new container to deploy. This is done in a few steps.
NOTE: Most times these scripts would be written into a bash script. So, while this looks complicated, it’s only moderately
more work than using a dockerfile and the increase in security, flexibility, and transfer speeds make up for the extra
complexity.
Create the base layer and hold on to the name for later steps
$ ctr=$(buildah from "${1:-registry.suse.com/suse/sle15}")
NOTE: For most scripts it’s easier to just capture it in the first step and just use it as a variable but the value stored here
can be found as the Container ID in the output of:
$ buildah containers
Now that we have the base layers of the container, let’s add to it. There are two ways to go about this: mount the
container filesystem and use zypper to install into it, or run zypper from inside the container.
Both of these options have their time and place. To me, dealing with zypper from inside the container comes with a lot of
extra bloat due to the need for copying in your service and repo files from the host. In my opinion, mounting the filesystem
can give a much more streamlined experience.
Note: Read this blog by Jason Evans about how to build images with zypper inside the container.
This is done with:
$ ctr_wd=$(buildah mount $ctr)
The stored output tells you where this container filesystem is located.
This location can be used by zypper as an alternate install root. Let’s install nginx (and it’s dependencies) to this container
with:
$ zypper --installroot $ctr_wd -n install nginx
We need to do a little config to run this securely in a container.
Create a new file at $ctr_wd/app/nginx.conf and add the following content. This listens on port 8080 as an executable
hosting files from /app/srv.
worker_processes 1;
error_log stderr;
daemon off;
pid
 nginx.pid;
events {
worker_connections 1024;
}
http {
include
 /etc/nginx/mime.types;
default_type application/octet-stream;
sendfile
 on;
keepalive_timeout 65;
server {
listen
 8080;
location / {
root /app/srv/;
index index.html index.htm;
}
}
}
Lastly for this step, we need to add some config into the container manifest so it knows what to run when started. This is
done with:
$ buildah config --entrypoint '["/usr/sbin/nginx","-c","/app/nginx.conf",”-p”,”/app/”]' $ctr
With those changes, we can commit, create a new layer, and remount it with the following sequence of commands:
$ image=$(buildah commit --rm $ctr my_nginx)
$ ctr=$(buildah from $image)
$ ctr_wd=$(buildah mount $ctr)
Now, with the new layer built and mounted, let’s add some content to host into it:
$ mkdir $ctr_wd/app/srv
$ echo “Hello, World!” > $ctr_wd/app/srv/index.html
Let’s commit that container for us to run:
$ buildah commit $ctr my_nginx
