---
title: "Don't Forget your Linux Fundamentals in your Cloud Native World"
date: 2020-18-11
draft: true
menu:
  nav:
    name: help
    url: "/linux-in-cn/"
    weight: 100
---

Note: I feel like this title might suggest that I'm going to be the old guy yelling at cloud and talking about how all you "whipper-snappers" (does that translate to other countries other than the US?) are moving too fast for me. I hope I'm way too young for that kinda talk and it's not that way at all. 

Even in the modernest of modern IT stacks, most everything is still based on Linux as the primary Operating System running application binaries and abstrating away the nitty gritty of hardware and scheduling. Containers are just linux processes but with some extra context (or lack of) surrounding them and the building of these applications are the same. 

What has changed, is our expectations of how they are managed once built. 


# What is Cloud Native?

From the usage I'm seeing, the term cloud native has mostly come to mean "running in Kubernetes". This means that there's likely a processes for the applications build cycle, and deployment. 


# What should we not forget?


## Security! 

Just because the application is walled off in a container, doesn't mean there's no threat. There have been many SVEs found where an rouge process can break out of it's confinement like Tai Lung from Kung Fu Panda. This means that we still need to think about security. 



## Build Process

The processes for building applications don't really change much. Most CI/CD pipelines are really just directed graphs of fancy scripts. This means that the tooling shouldn't change much other than the way it gets called. 

The act of packaging an application into a container is just creating a tarfile containing a root filesystem. 


## Optimization

Sadly, spinning up new instances and scaling horizontally to infinity doesn't fix the physics of how long it takes electrons to move around. 



