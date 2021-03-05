# Moving a Cloud Foundry Hello World App to Kubernetes - How Hard Can It Be?

Essentially, I've spent most of last year figuring out Cloud Foundry and then telling others about it. And, believe it or not, I sort of fell in love with the simplicity of pushing a changed code base to the platform with one simple command (aka the ["cf push experience"] (https://www.cloudfoundry.org/blog/cloud-foundrys-proven-developer-experience-comes-to-kubernetes-with-cf-for-k8s-1-0/)). The step from running things locally to running things in the cloud was just one additional file: the Cloud Foundry [manifest](https://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html)). It was so low threshold that I actually stopped testing changes locally. Instead, I just pushed changes to my deployed app immediately. If I had to, I could always hook up a debugger to the deployed app running in Cloud Foundry. 

Recently, however, I have become more interested in Kubernetes. And instead of taking a course, watching a bunch of YouTube videos or reading a book (which I'll eventually have to do anyway as we'll find out further down), I got the idea to take an example Cloud Foundry application I had written and move it over to Kubernetes. Just a few lines of Python code -- how hard can it be?

It was surprisingly challenging. Why? Well, of course, the main reason is my ignorance. I just didn't have much clue about what I was trying to accomplish. Which, no matter what you actually try, kind of guarantees a certain level of frustration along the way, right? 

But I assume I'm not alone in this. No matter how proficient you are with Kubernetes, you've probably been at the point of not really knowing how it works, except that Kubernetes can "run your stuff" somehow. But when you check out the most basic examples found with a simple web search, you'll see that they have one thing in common: they mostly explain how you can run someone else's code, packed by someone else into someone else's container. That is not what I wanted to do. I wanted to find out how I could run my own code, in my own container that contains exactly what I want. And that is what took me some time to figure out. 

So let me take you on my journey. 

# Where am I and Where Do I Want to Go?

I am using the Python hello world example from our CAP Developer Sandbox [Getting Started Guide](https://gettingstarted.cap.explore.suse.dev/cli/#pushing-applications). You can find the code can be found [here](https://github.com/SUSE-Developer-Community/helloworld-python](https://github.com/SUSE-Developer-Community/helloworld-python). 

My goal is to build a container that runs the exact same app, push the image to Docker Hub and deploy it to a Kubernetes cluster running in a local VM on my laptop. So let's see what it takes and which obstacles I need to overcome. 

The first thing I did was obviously to search for articles to help me. But what I found either described how to put something into a Docker container and also run it via Docker, or how to pull down an already existing image from Docker Hub and run it on Kubernetes. I did not find a good article that put these two steps together. So, to some extent, I had to work this out for myself. 

# Step 1: Build a Container and Push it to Docker Hub

The first thing I had to do was build a container image that packages the app and is set up to start it correctly. According to the web, I needed to write a Dockerfile to do this. 

Essentially, a Dockerfile follows the format

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

There's a complete [reference documentation](https://docs.docker.com/engine/reference/builder/) on dockerfile syntax, but for my simple use case I worked it out based on examples I found on the web. 

## Choosing the base image

When building an application-specific container image, the first decision you'll need to take is which base image you want to start from. It took me a while to figure out how to go about this, since there are a confusingly large amount of choices. I could have gone for a generic operating system image like Ubuntu or [Alpine Linux](https://alpinelinux.org). In the end it seemed best to start from a Python image, so that I didn't have to worry about setting up a proper Python runtime environment first. Not that this would have been particularly difficult, but you know, I'm lazy. 

So I just went to [Docker Hub](http://hub.docker.com) and searched for "Python". I found a [Python image](https://hub.docker.com/_/python) that claims to be an "Official Docker Image" - sounds legit, right? 

Later I found this nice [article](https://medium.com/swlh/alpine-slim-stretch-buster-jessie-bullseye-bookworm-what-are-the-differences-in-docker-62171ed4531d) explaining what the different tag names on Docker Hub mean, which would have helped me a ton at the time. (Spoiler alert: most of them refer to Debian releases the respective images are based on.) 

## Building the image

Next, I needed to make sure I had all the Python packages required by my application were installed, so I told Docker to copy my requirements.txt into the container and run `pip install`. 

After that, it was time to copy the app itself, which consists of server.py and index.html. On second thought, a more elegant way would have been to use a RUN instruction to clone the GitHub repository so that I always get the latest version. In this case however, I just used my local clone. 

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

Then it was time to upload the container to Docker Hub. First I had to give it an official name, a so-called "tag." 

TODO: explain tags and docker hub repositories a bit more

```
    > docker tag 9f5e24e9052a timirnich/timhelloworld:latest
```

Finally, I  pushed the image to my personal Docker Hub account. 

```
    > docker push timirnich/timhelloworld:latest
```

# Step 2: Deploying to Kubernetes

Now that I had a container image built and uploaded to a place Kubernetes access it, I had to  work out how to tell Kubernetes what to do. With Cloud Foundry, what I'm used to doing is writing a manifest.yaml. With Kubernetes, things are a little more complex. Ok, that was an understatement. Things are way more complex. 

## Choosing the Right Workload Resource: Pod, ReplicaSet or Deployment?

The first thing I ran into was a concept called "pods." According to the [Kubernetes documentation](https://kubernetes.io/docs/concepts/workloads/pods/), a pod is "a group of one or more containers, with shared storage/network resources, and a specification for how to run the containers." Sounds about right for what I'm trying to do, doesn't it? However, the documentation also said I shouldn't be using pods, since, in a nutshell, I would be missing out on what Kubernetes is all about - making sure that my app is always available even when something goes wrong (e.g. a container dies). Not that resilience would be of any importance to my use case here. So, let's figure out the proper way. 

The documentation, as well as many of the examples I found, suggested the use of a resource called "Deployment." Citing the [Kubernetes documentation for Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) again, a Deployment "provides declarative updates for Pods and ReplicaSets." What?

In dramatically simplified terms, the point of a Deployment is to provide control over the process of updating a number of replicated instances of a container or a set of containers. 

Updating one container to a newer version of the underlying container image requires the container to shut down and restart, which causes downtime for my app. Nobody wants downtime. To avoid this, you deploy a number of parallel instances of your container and make sure that incoming traffic is automatically load balanced between them (i.e. each incoming request is randomly sent to one particular container instance). Kubernetes has a workload resource called [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/), which does exactly what the name suggests: it manages a specified number of identical copies of a pod. If a pod dies, the ReplicaSet re-creates it. If I created an extra instance of a pod managed by a ReplicaSet (for fun), the ReplicaSet would nuke it immediately. 

A Deployment provides a way to control the state of Pods and ReplicaSets by defining the *desired state* of the Deployment. For example, if you want to roll out a new version of a container, you can tell the Deployment to update all running instances at a controlled rate (e.g. one every 5 seconds). If you later  realize that you shouldn't have done this because the new version contains an error, you can simply roll the entire thing back to its previous state. Fancy, isn't it?

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

While I could have run the create command directly, the advantage of doing it this way is that I now have a file on my local disk that I can use to make changes to the deployment easily. For example, if I wanted to pin the container image to a specific version, I could just do

```
       - image: timirnich/timhelloworld:0.5
```

instead. 

# Step 3: Making it Reachable

Now my container is running on Kubernetes, but how do I talk to it? In Cloud Foundry, each app automatically gets a [route](https://docs.cloudfoundry.org/devguide/deploy-apps/routes-domains.html), which is an external http endpoint that hits the default route specified by my app. If I don't specify it, CF assigns a so-called random route when I push my app for the first time. Every app in CF has a route, unless you actively make sure it doesn't get one. Yes, there are cases where that makes sense but that's off topic for now. 

In Kubernetes, things are a bit more complicated. The Kubernetes deployment resource I used above doesn't care about making things reachable from the outside world - it cares about deploying stuff and making sure that the actual state of the deployment matches the wanted state expressed by it's yaml specification. Nothing else. As a result, the port exposed by the container (the one I specified in the Dockerfile) is only reachable from *within* the cluster - other containers and other pods can (theoretically) use it to talk to my app, but nobody else. 

Btw, a good article that helped me wrap my head around all this is ["Kubernetes Ingress for Beginners"](https://thenewstack.io/kubernetes-ingress-for-beginners/) by Nick Ramirez. Thanks Nick!

As an aside, it is not a good idea to do it this way either, since pods are ephemeral - they come and go. One key assumption Kubernetes makes about containers is that they die and have to be re-created all the time - as we all know, developers can't be trusted, right? o the deployment's main concern is to make sure that the actual state of the deployment matches the wanted state expressed by its yaml specification. If a container dies, it will recreate it (well, in fact that's what the underlying ReplicaSet does but hey).

Fair enough, so what else have we got? 

The first thing you'll run across in many examples is the concept of a Kubernetes *Service*. The [Kubernetes documentation on Services](https://kubernetes.io/docs/concepts/services-networking/service/) says a Service is "an abstract way to expose an application running on a set of Pods as a network service." The key idea of a Service is that it hides the ephemeral nature of a pod - no matter how often pods get restarted or scaled up and down, the service endpoint remains accessible and doesn't change. It also performs basic (round-robin) load balancing, i.e. it makes sure that incoming requests are distributed evenly across a number of identical containers in a pod or a set of pods in a ReplicaSet. 

So is a Service the Kubernetes counterpart of a Cloud Foundry application route? Not necessarily, since there are different service types you can specify. To make our service externally reachable, we need to make it be of the type "NodePort." This essentially makes Kubernetes assign a random TCP port in the port range between 30000 and 32767 and makes this port reachable outside the cluster. There's another Service type called "LoadBalancer" which essentially implements the service by requesting a load balancer resource from the underlying cloud provider - but let's leave that aside for now. 

To create a service of type "NodePort" for my hello world app I just run

```
    kubectl create svc nodeport timhelloworld --tcp=8080:80
```

The result is 

```
    > kubectl get services
    NAME                    TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    kubernetes              ClusterIP      10.43.0.1       <none>        443/TCP          35d
    timhelloworld           NodePort       10.43.251.28    <none>        8080:30683/TCP   1m
```
which means that port 8080 of the container (where the web server listens) is now mapped to port 30683 of the Kubernetes VM's public IP address. 

Now I can curl my Kubernetes VM's external IP address on port 80 and see the index.html page being returned (192.168.33.11 is the hosts-de IP address of my Kubernetes VM):

```
    > curl 192.168.33.11:30683
    <html>
        <head>
            <title>Python says Hello World!</title>
        </head>
        <body>
            <h1>Python says Hello World!</h1>
            <p>And this is running on the SUSE Cloud Application Platform developer sandbox...</p>
            <p>Update this page and simply run cf push to get your changes online within seconds!</p>
        </body>
    </html>
```
Close but no cigar yet - I don't have a an actual domain name for my app like I used to have in Cloud Foundry. 

To get things lifted up to the level of domains and hostnames as opposed to IP addresses and ports, I need yet another Kubernetes resource, called Ingress. An Ingress basically specifies how incoming requests should be routed to services. For example, if your app consists of multiple services, you can use an ingress to make sure requests to http://my-app.my-domain/service1 go to service 1, and http://my-app.my-domain/service2 go to service 2.  

For the simple hello world example we're discussing here, it is easy to define an Ingress that does the job of assigning a domain name to my app:

```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: timhelloworld-ingress
spec:
  rules:
  - host: timhelloworld.192.168.33.11.xip.io
    http:
      paths:
      - path: /
        backend:
          serviceName: timhelloworld
          servicePort: 8080
```

Note that I'm using the xip.io service to cheat around the fact that I do not have a public DNS entry that resolves to my K3s VM's IP address. I could have entered this into the /etc/hosts file on my laptop but this way is just simpler and quicker. In case you don't know what xip.io does, it simple cuts out the IP address from the URL and sends it back as the IP address this name resolves to. Why didn't I have this idea myself ages ago? Don't say it...

And now, finally, we are back to where we started. What a journey. All just to reproduce what a single 'cf push' command does. 

Am I the first one to note that there's a bit of a complexity problem here? Not at all. If there's one thing that's certain, when something looks like a really good idea, most of the time someone else has already thought of it. Same here. There are tons of tools out there that try to make things simpler for developers when developing apps that only run properly once deployed to Kubernetes. More on that in a future post: stay tuned. 

If you've made it this far, thank you very much for bearing with me. Remember, the only stupid question is the unasked one. Everybody else is just Googling for stuff too. 
