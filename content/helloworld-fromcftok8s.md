# Moving a Cloud Foundry hello world app to Kubernetes - how hard can it be?

Essentially, I've spent most of last year figuring out Cloud Foundry and then telling others about it. And, believe it or not, I sort of fell in love with the famous cf push experience (add a link?). The step from running things locally to running things in the cloud was just one additional file and running one command. It was so low threshold that I actually started not testing changes locally anymore. 

Recently, I have become more interested in Kubernetes though. And instead of taking a course, watching a bunch of youtube videos or reading a book, I kind of got the idea of just taking one of the example Cloud Foundry applications I had written and see what it takes to move them over to Kubernetes. Just a few lines of python code, how hard can it be?

And it was surprisingly challenging. Why so? Well, of course the main reason is my ignorance. I just didn't have much clue about what I was trying to accomplish. Which, no matter what you actually try, kind of guarantees a certain level of frustration along the way, right? 

But I assume I'm not alone in this. No matter how proficient you are with Kubernetes, you've probably been at this point of not really knowing how it works except that Kubernetes, can "run your stuff" somhow. But when you check out the most basic examples you'll find with a simple web search, you'll find that they have one thing in common: they mostly explain how you can run someone else's code that has been packed by someone else into someone else's container. That is not what I wanted to do. I wanted to find out how I could run my own code, in my own container that contains exactly what I want. And that is what took me some time to figure out. 

So let me take you with me on this journey. 

# Where am I and where do I want to go?

Python hello world app from CAP Sandbox GSG. Can push to CF with a simple cf push. 

Goal: build it into a container, push to docker hub, deploy to Kubernetes. See what it takes and what obstacles I need to overcome. 

Started to search for articles on this but only found ones that describe how to put something into a docker container and also run it via docker, or I found examples how to pull down an image from docker hub and run it on k8s. I did not find a good article that put these two steps together. 

# Step 1: build a container and push to Docker hub

Dockerfile. 

Which base image? How do I choose the right base image for me? What options do I have?

Building container. tag it. Upload it. How the latest tag works. 

# Step 2: deploying to kubernetes

create a deployment yaml. apply/create it. Explain what happens. Why this is not sufficient. Further reading resources for those who want to understand the details of what a pod, replicaset, deployment is. 

# Step 3: making it reachable

Create service. reconfigure VM so that it becomes reachable (Private network). 

Some future work pointers (MetalLB). 