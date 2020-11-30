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


While writing this article, an extremely interesting example came up in the form of AWS Kinesis failing and taking down us-east-1 for a fairly large amount of time. The root cause of this failure was a innocuous-seeming increase in backend capacity. This caused a over-run of the linux processes allocated in the front-end which cascaded into a widespread failure. 

One main take away from this failure is that if the AWS operations team can make this mistake, anyone can. (It definitely makes me feel better about having some odd instability in our developer sandbox due to inotify-limits being met!)



There are a lot of places where we need to remember our linux fundamentals. This is by no means an exhaustive list, just a few that I've run in to or seen others run in to.


# Resource Allocations and Limits

Linux has a lot of resource constraints built into the OS. 

As we saw in the AWS example, process limits exist!

There are also file descriptor limits, inotify limits, port allocation limits.

For more high performance datacenters, we should keep in mind physical constraints as well. Such as disk speed, network latency, heat dissipation, etc. 

For a full list of tuning parameters, check out https://documentation.suse.com/sles/15-SP2/single-html/SLES-tuning/#book-sle-tuning

*TODO* Find more human readable list of parameters?

# Build Process

The processes for building applications don't really change much. Most CI/CD pipelines are really just directed graphs of fancy scripts. This means that the tooling shouldn't change much other than the way it gets called. 

We still need to be concerned with aspects like artifact size, build time, governance, and reproducability to be able to best help the Ops team do their job! For example, knowing the dependencies that go into a build and being able to track them to the final product lets us know where potential issues might pop up before they do.


# Security! 

NOTE: I'm not a security expert so the main point is that nothing has changed, and there's just more work to do...

Just because the application is walled off in a container doesn't mean there's no threat. There have been many SVEs found where an rouge process can break out of it's confinement like Tai Lung from Kung Fu Panda. This means that we still need to think about security on the host and in the applications as well as security of the cluster data plane and network. 

We need to still be concerned with host and physical security if running locally. This also means not running privileged pods in your cluster as those can get root access to your node and mess with kernel parameter (ask me how I know...). 

Setting up app armor profiles, correct firewalld rules, and privileges 

There's also security concerns with the build pipeline. Too many tools take a shortcut and ask for the container to mount the docker socket. This would allows a ci/cd script to make changes to your cluster. One thing that can help this (in my opinion) is moving from the docker daemon to cri-o and using tools that are runtime agnostic.

# Optimization

Sadly, spinning up new instances and scaling horizontally to infinity doesn't fix the physics of how long it takes electrons to move around. This means that we should still be concerned with both computational complexity *and* resource constraints. 

For example, knowing when to use different storage types or when it's appropriate to split processing across the network can dramatically change the end user's experience of a system. 

All of these optimization problems tend to be pretty specialized skills (of which, I likely have none). They liekly aren't important in the first stages of building and scaling an app, but can mean the difference between success and failure when things start to grow and get hectic. 


# All is not lost!

Luckily, you likely have people in your organization with all of these skills! 