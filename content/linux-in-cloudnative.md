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

Even in the modernest of modern IT stacks, most everything is still based on Linux as the primary Operating System running application binaries and abstracting away the nitty gritty of hardware and scheduling. Containers are just linux processes but with some extra context (or lack of) surrounding them and the building of these applications are the same. 

What has changed, is our expectations of how they are managed once built. 


While writing this article, an extremely interesting example came up in the form of AWS Kinesis failing and taking down us-east-1 for a fairly large amount of time. The root cause of this failure was a innocuous-seeming increase in backend capacity. This caused a over-run of the linux processes allocated in the front-end which cascaded into a widespread failure. 

One main take away from this failure is that if the AWS operations team can make this mistake, anyone can. (It definitely makes me feel better about having some odd instability in our developer sandbox due to inotify-limits being met!)


We likely don't need to know or keep track of all of these different things, in our day to day work in the cloud, but we do need to remember that Kubernetes is just a nice set of abstractions over Linux. 


I am not claiming to be a Linux expert but have put together a list of places where I've run into issues or seen others struggle. This is by no means exhaustive, and likely doesn't even scratch the surface of where you could potentially run into "legacy" issues while running in the cloud! 



# Resource Allocations and Limits

Linux has a lot of resource constraints built into the OS and these can cause failures in odd and unexpected ways.

As we saw in the AWS example, process limits exist!

There are also file descriptor limits, inotify limits, port allocation limits, and similar. And for more high performance data centers, we should also keep in mind physical constraints as well (such as disk speed, network latency, heat dissipation, etc...)

For a full list of tuning parameters on SUSE Enterprise Linux and OpenSUSE, check out https://documentation.suse.com/sles/15-SP2/single-html/SLES-tuning/#book-sle-tuning

*TODO* Find more human readable list of parameters?

# Build Process

The processes for building applications don't really change much. Most CI/CD pipelines are really just directed graphs of fancy scripts. This means that the build tooling shouldn't change much other than the way it gets triggered.

We still need to be concerned with aspects like artifact size, build time, governance, and reproducability to be able to best help the Ops team do their job! For example, knowing the dependencies that go into a build and being able to track them to the final product lets us know where potential issues might pop up before they do.


# Security! 

NOTE: I'm also not a security expert so the main point is that nothing has changed, and there's just more work to do as Kubernetes is insecure by default. 

Just because the application is walled off in a container doesn't mean there's no threat. There have been many SVEs found where a rouge process can break out of it's confinement like Tai Lung from Kung Fu Panda. This means that we still need to think about security on the host and in the applications as well as security of the cluster data plane and network. 

We need to still be concerned with host and physical security if running locally. Setting up App Armor profiles, correct firewall rules, and minimal privileges are still needed to keep the control plane and applications secure. This also means not running privileged pods in your cluster as those can get root access to your node and mess with kernel parameter (ask me how I know that it's possible...). 

There's also security concerns with the build pipeline. Too many tools take a shortcut and ask for the container to mount the docker socket. This would allows a ci/cd script to make changes to your cluster. One thing that can help this (in my opinion) is moving from the docker daemon to cri-o and using tools that are runtime agnostic.

# Optimization

Sadly, spinning up new instances and scaling horizontally to infinity doesn't fix the physics of how long it takes electrons to move around. This means that we should still be concerned with both computational complexity *and* resource constraints. These aren't normally important early in the process but can definitely start to be costly when you get to push the system to scale to larger sizes.

For an example that I've seen play out in my systems, knowing when to use different storage types can dramatically change the end user's experience of a system. The same is true for knowing when to break up single services or components into multiple pieces to allow faster or more flexible scaling.



# All is not lost!

Luckily, you likely have people in your organization with all of these skills! 