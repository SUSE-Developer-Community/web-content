# Moving a Cloud Foundry hello world app to Kubernetes - how hard can it be?

Essentially, I've spent most of last year figuring out Cloud Foundry and then telling others about it. And, believe it or not, I sort of fell in love with the simplicity of pushing a changed code base to the platform with one simple command (aka the "cf push experience" (add a link?)). The step from running things locally to running things in the cloud was just one additional file: the Cloud Foundry [manifest](https://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html)). It was so low threshold that I actually started not testing changes locally anymore. Instead, I would just push changes to my deployed app immediately. If I had to, I could always hook up a debugger to the deployed app running in Cloud Foundry. 

Recently however, I have become more interested in Kubernetes. And instead of taking a course, watching a bunch of youtube videos or reading a book (which I'll eventually have to do anyway as we'll find out further down), I kind of got the idea of just taking one of the example Cloud Foundry applications I had written and see what it takes to move them over to Kubernetes. Just a few lines of python code, how hard can it be?

And it was surprisingly challenging. Why so? Well, of course the main reason is my ignorance. I just didn't have much clue about what I was trying to accomplish. Which, no matter what you actually try, kind of guarantees a certain level of frustration along the way, right? 

But I assume I'm not alone in this. No matter how proficient you are with Kubernetes, you've probably been at this point of not really knowing how it works, except that Kubernetes can "run your stuff" somehow. But when you check out the most basic examples you'll find with a simple web search, you'll find that they have one thing in common: they mostly explain how you can run someone else's code that has been packed by someone else into someone else's container. That is not what I wanted to do. I wanted to find out how I could run my own code, in my own container that contains exactly what I want. And that is what took me some time to figure out. 

So let me take you with me on this journey. 

# Where am I and where do I want to go?

I am using the Python hello world example from our CAP Developer Sandbox [Getting Started Guide](https://gettingstarted.cap.explore.suse.dev/cli/#pushing-applications). The code can be found at [https://github.com/SUSE-Developer-Community/helloworld-python](https://github.com/SUSE-Developer-Community/helloworld-python). 

My goal is to build a container that runs the exact same app, push the image to docker hub and to finally deploy it to a Kubernetes cluster running in a local VM on my laptop. So let's see what it takes and which obstacles I need to overcome. 

The first thing I did was obviously to search for articles on this. But what I found either described how to put something into a docker container and also run it via docker, or how to pull down an already existing image from docker hub and run it on Kubernetes. I did not find a good article that put these two steps together. So, to some extent, I had to work this out for myself. 

# Step 1: Build a container and push to Docker hub

The first thing I obviously had to do was build a container image that packages the app and is set up to start it correctly. The web told me I needed to write a Dockerfile to do this. 

Essentially, a dockerfile follows the format

```
# Comment
INSTRUCTION arguments
```

The most important instructions are
- FROM: start from an already existing image as a base (more on that later) 
- RUN: run a command at build time
- COPY: copy a file from the local work directory (the place where you run the docker build command)
- EXPOSE: specify a port the container should open at runtime
- CMD: the command that should be executed when the container starts

There's a complete [reference documentation](https://docs.docker.com/engine/reference/builder/) on dockerfile syntax, but for my simple use case I was able to work it out from a couple of examples I found on the web. 

## Choosing the base image

When building an application specific container image, the first decision you'll need to take is which base image you want to start from. It took me a while to figure out how to go about this, since there's a confusingly large amount of choice. I could have gone for a generic operating system image like Ubuntu or [Alpine Linux](https://alpinelinux.org). In the end it seemed best to start from a Python image for my case, so that I didn't have to worry about setting up a proper Python runtime environment first. Not that this would have been particularly difficult, but you know, I'm lazy. 

So I just went to [hub.docker.com](http://hub.docker.com) and searched for "Python". This way I found a [Python image](https://hub.docker.com/_/python) that claims to be an "Official Docker Image" - sounds legit, right? 

Later on I found this nice [article](https://medium.com/swlh/alpine-slim-stretch-buster-jessie-bullseye-bookworm-what-are-the-differences-in-docker-62171ed4531d) explaining what all the different tag names on docker hub mean, which would have helped me a ton at the time. (Spoiler alert: most of them refer to Debian releases the respective images are based on.) 

## Building the image

Next, I needed to make sure all python packages required by my application were installed, so I had to tell Docker to copy my requirements.txt into the container and run pip install. 

After that, it was time to copy the app itself, which consists of server.py and index.html. At second thought, a more elegant way would have been to use a RUN instruction to clone the GitHub repository so that I always get the latest version. In this case however, I just used my local clone. 

All in all, this is the dockerfile I ended up with:
```
    FROM python:3.7

    RUN mkdir /app
    WORKDIR /app

    COPY ./requirements.txt requirements.txt
    RUN pip install -r requirements.txt

    COPY ./index.html index.html
    COPY ./server.py server.py

    EXPOSE 8080

    CMD [ "python", "/app/server.py" ]                                                                               
```
The next step was to build the container by running "docker build" in the app directory:

```
    > docker build .
    Sending build context to Docker daemon   68.1kB
    Step 1/9 : FROM python:3.7
    ---> 11c6e5fd966a
    Step 2/9 : RUN mkdir /app
    ---> Using cache
    ---> a8c6a7422a68
    Step 3/9 : WORKDIR /app
    ---> Using cache
    ---> 15b8199aebb3
    Step 4/9 : COPY ./requirements.txt requirements.txt
    ---> Using cache
    ---> b08660f8fa00
    Step 5/9 : RUN pip install -r requirements.txt
    ---> Using cache
    ---> 7775be938d98
    Step 6/9 : COPY ./index.html index.html
    ---> Using cache
    ---> 968251f8d14a
    Step 7/9 : COPY ./server.py server.py
    ---> Using cache
    ---> 28f522c40101
    Step 8/9 : EXPOSE 8080
    ---> Using cache
    ---> 2bf22d5e4046
    Step 9/9 : CMD [ "python", "/app/server.py" ]
    ---> Using cache
    ---> 9f5e24e9052a
    Successfully built 9f5e24e9052a
```

Now, it was time to upload the container to docker hub. To do so, I first had to give it an official name, a so-called "tag". 

TODO: explain tags and docker hub repositories a bit more

```
    > docker tag 9f5e24e9052a timirnich/timhelloworld:latest
```

Finally, I was able to push the image up to my personal docker hub account. 

```
    > docker push timirnich/timhelloworld:latest
```

# Step 2: Deploying to Kubernetes

Now that I had a container image built and uploaded to a place Kubernetes can get it from, it was time to work out how I can tell Kubernetes what to do. With Cloud Foundry, what I'm used to doing is writing a manifest.yaml. With Kubernetes, things are a little more complex. Ok, that was an understatement. Things are way more complex in fact. 

## Choosing the right workload resource: Pod, ReplicaSet or Deployment?

The first thing I ran into was a concept called "pods". According to the [Kubernetes documentation](https://kubernetes.io/docs/concepts/workloads/pods/), a pod is "a group of one or more containers, with shared storage/network resources, and a specification for how to run the containers". Sounds about right for what I'm trying to do, doesn't it? However, the documentation also said I shouldn't be using pods, since, in a nutshell, I would be missing out on what Kubernetes is all about - making sure that my app is always available even when something goes wrong (e.g. a container dies). Not that resilience would be of any importance to my use case here, but hey. Let's figure out the proper way then. 

One of the things suggested by the documentation as well as many of the examples I found is the use of a resource called "Deployment". Citing the [Kubernetes documentation for Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) again, a Deployment "provides declarative updates for Pods and ReplicaSets". What?

In dramatically simplified terms, the point of a Deployment is to provide control over the process of updating a number of replicated instances of a container or a set of containers. 

Updating one container to a newer version of the underlying container image requires the container to shut down and restart, which causes downtime for my app. Nobody wants downtime. To avoid this, you deploy a number of parallel instances of your container and make sure that incoming traffic is automatically load balanced between them (i.e. each incoming request is randomly sent to one particular container instance). To do this, Kubernetes has a workload resource called [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/), which does exactly what the name suggests: it manages a specified number of identical copies of a pod. If a pod dies, the ReplicaSet will re-create it. If I created an extra instance of a pod managed by a ReplicaSet for funsies, the ReplicaSet would nuke it immediately. 

Now, a Deployment provides a way to control the state of Pods and ReplicaSets, by defining the *desired state* of the Deployment. For example, if you want to roll out a new version of a container, you can tell the Deployment to update all running instances at a controlled rate (e.g. one every 5 seconds). If you later on realize that you shouldn't have done this because the new version contains an error, you can simply roll the entire thing back to its previous state. Fancy, isn't it?

## Writing the Deployment yaml

The easiest way to get started with writing yaml files for Kubernetes is to use kubectl's dry-run feature, which is invoked by adding the --dry-run flag:

```
    > kubectl create deploy timhelloworld --image=timirnich/timhelloworld:latest --dry-run -o yaml > timhelloworld.yaml
```

This results in the following file

```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    creationTimestamp: null
    labels:
        app: timhelloworld
    name: timhelloworld
    spec:
    replicas: 1
    selector:
        matchLabels:
        app: timhelloworld
    strategy: {}
    template:
        metadata:
        creationTimestamp: null
        labels:
            app: timhelloworld
        spec:
        containers:
        - image: timirnich/timhelloworld:latest
            name: timhelloworld
            resources: {}
    status: {}
```

Now I can deploy this with 

```
    > kubectl apply -f timhelloworld.yaml
```

TODO: Show the result. 

I could have run the create command directly, but the advantage of doing it this way is that I now have a file on my local disk that I can use to make changes to the deployment easily. For example, if I wanted to pin the container image to a specific version, I could just do

```
       - image: timirnich/timhelloworld:0.5
```

instead. 

# Step 3: Making it reachable

Now I have my container running on Kubernetes, but how do I talk to it? By default, the port exposed by the container is only reachable from *within* the cluster. To make it accessible from the outside world, I need to create a Service. The [Kubernetes documentation on Services](https://kubernetes.io/docs/concepts/services-networking/service/) says a Service is "An abstract way to expose an application running on a set of Pods as a network service." So a Service is somewhat equivalent to a Cloud Foundry Route (which, confusingly, has nothing to do with an IP rooute - it's a URL that exposes an HTTP service endpoint). 

The way I make sense out of Kubernetes Services is that pods, particularly when created by ReplicaSets, are perishable. They may get destroyed or re-created any time, and hence the IP address I was talking to a second ago may not be the one I need to talk to now. The Service resource is a way to establish a stable endpoint to talk to while the underlying backend and all the Kubernetes container orchestration shebang does what it does. 

To create the service I just run

```
    kubectl create svc nodeport timhelloworld --tcp=8080:80 --dry-run -o yaml > timhelloworld-service.yaml
```

The result is 

```
    > kubectl get services
    NAME                    TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    kubernetes              ClusterIP      10.43.0.1       <none>        443/TCP          35d
    timhelloworld           NodePort       10.43.251.28    <none>        8080:30683/TCP   1m
```

Now I can curl my Kubernetes VM's external IP address on port 80 and see the index.html page being returned:




Create service. reconfigure VM so that it becomes reachable (Private network). 

Some future work pointers (MetalLB). 