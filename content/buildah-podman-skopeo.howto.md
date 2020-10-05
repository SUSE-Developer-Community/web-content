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

Buildah uses the standard CRI set of tooling and configuration which means that it runs very nicely along side CRI-O and Podman making for a full (mostly drop in) replacement for Docker.

# What is a Container (simply)

A container is a collection of filesystem layers that when joined together provide everything needed to run the application in a self-contained way. It also includes a manifest that describes how to use what’s inside the container.
The process for building a container is to build a layer, save it, build the next layer, save it, etc... until you have everything you want in there. The reason for splitting out layers is to ease both the building and splitting out of layers. 
If a certain part of your build step always builds the same thing, why repeat that work? The same is for transport, if there’s a large baseline that each container needs, you don’t need to download it every time.


[Some graphic here?](!/img/asdas )

As these are really just a collection of bits, there's no magic to building one that requires a centralized service to do the build. Also, removing the 


# Toolset

There a little bit of overlap between the tools but they each do a different part of the puzzle.

Buildah builds OCI compliant images. It can allow you to run commands in the container’s context as well as mount the container’s filesystem to move files in from elsewhere 

Podman runs images in a CRI compatible runtime (such as CRI-O). It is a drop in replacement for Docker and supports all of the same subcommands.


# Installation 

To install on OpenSUSE, just run:

```bash
zypper in podman buildah
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

NOTE: Most times these scripts would be written into a bash script. So, while this looks complicated, it’s only moderately more work than using a dockerfile and the increase in security, flexibility, and transfer speeds make up for the extra complexity.

## Base Layer

Create the base layer and hold on to the name for later steps.

For ease of instruction we will start with a OpenSUSE specific base layer (which I choose to trust). For containers without dependencies (where the application is a compiled binary), you can use `buildah from scratch` to start with an empty layer.

```bash
ctr=$(buildah from "docker.io/opensuse/leap:15.2")
```

NOTE: For most scripts it’s easier to just capture it in the first step and just use it as a variable. If you are doing this manually,the value can be found as the Container ID in the output of:

```bash
buildah containers
```

## Installing packages

Now that we have the base layers of the container, let’s add to it. There are two ways to go about this: 
- Mount the container filesystem and use zypper (or any package manager) to install into it
- Run zypper (or any package manager) from inside the container
- Copy only relevant files into the right directory

Each of these options have their time and place. As we are running rootless, we cannot run our package manager from the host so the first option is out. Since we want to use our package manager to install nginx and all of it's needed dependencies, we will run zypper from inside the container.

This is done with:

```bash
buildah run $ctr zypper in nginx
```

This will need to update the registry metadata then will install nginx into the container.

## Committing the layer

It's good practice to split your images into logical layers. This does two things: You don't have to continually rebuild your image from scratch, and different nodes can share any common layers between them instead of having to re-download full images each time.

We can commit the layer and then continue building onto it using:

```bash
image=$(buildah commit --rm $ctr leap_with_nginx)
ctr=$(buildah from $image)
echo $ctr
```

Note: The `--rm` flag instructs buildah to remove the previous working container after committing. This way we don't waste storage.


## Writing Nginx Config

Next we will write the nginx config needed to tell it how to host our files.

To do this we will mount the container filesystem to a place we can access from the host. There is a little weirdness in this step sicne we are running as a non-root user and we will need to run in a shared user namespace. To do this, run:

```bash
podman unshare 
```

This will start a shell in a separate linux namespace where we have more permissions but are still safely segmented out from the rest of the system. Please note that the variables we were using don't come with us (which is the reason for echoing it out in the previous step).

To mount the container we are working on we can use:

```bash
ctr="leap_with_nginx-working-container" # or the output of `echo $ctr` above
ctr_wd=$(buildah mount $ctr)
```

The stored output tells you where to find the root of the container's filesystem.

Using your favorite editor, create a new file at $ctr_wd/app/nginx.conf and add the following content. This will instruct nginx to listen on port 8080, run as an executable, and host files from /app/srv. Note that you will need to create the containing folder with `mkdir -p $ctr_wd/app`.

```nginx.conf
worker_processes 1;
error_log stderr;
daemon off;
pid nginx.pid;

events {
  worker_connections 1024;
}

http {
  include /etc/nginx/mime.types;
  default_type application/octet-stream;
  sendfile on;
  keepalive_timeout 65;

  server {
    listen 8080;
  
    location / {
      root /app/srv/;
      index index.html index.htm;
    }
  }
}
```

## Adding Manifest

We need to add some config into the container manifest so it knows what to run when started. This is done with:

```bash
buildah config --cmd '' $ctr
buildah config --entrypoint '["/usr/sbin/nginx","-c","/app/nginx.conf","-p","/app/"]' $ctr
```

This will set up our container to start nginx with our newly created configuration when the container starts and to not attempt to run the default bash command.

## Committing Layer

With those changes, we should commit again and then remount it with the following sequence of commands:

```bash
image=$(buildah commit --rm $ctr leap_with_nginx_configured)
ctr=$(buildah from $image)
ctr_wd=$(buildah mount $ctr)
```

## Adding content

Now, with the new layer built and mounted, let’s actually add content to host into it:

```bash
mkdir $ctr_wd/app/srv
echo "Hello, World! I'm stuck in a container!" > $ctr_wd/app/srv/index.html
```

While this is a trivial example so I just echo in some content. From this shared namespace, I can copy over files from the host as well.

## Committing Final Layer

Lastly, let’s commit that container for us to run:

```bash
buildah commit --rm $ctr my_hello_world
exit
```


## Running Container Locally

Now that the container is created, we can run it locally using `podman` with:

```bash
podman run -p 8080:8080 my_hello_world
```

This will run with the terminal attached so you can see any output from the command. When you browse to `http://127.0.0.1:8080/` you should see the hello world in the browse and will likely see an error about a missing favicon in the command line. We didn't add one so that's expected.

## Pushing Your Container Image

Let's publish our new container to a registry!

We can use either `buildah push` or `podman push` to do this. Since we've been on a buildah kick, let's use that!

Let's login to dockerhub so we can actually push first: (create a dockerhub account first if you haven't already!)

```bash
buildah login docker.io
```

Then we can push the images with:

```bash
buildah push my_hello_world docker.io/<username>/my_hello_world
```

# Next Steps

With these basics, you can build all sorts of applications into a container. Since we aren't using Docker, we can do this without needing access to the Daemon or root access. 
