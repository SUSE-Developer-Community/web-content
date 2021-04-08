---
title: "Overview of CI/CD tools"
date: 2021-02-03
draft: true
---

## Intro 

Continuous Integration (CI) and Continuous Delivery or Deployment (CD) are hot topics right now as so many organizations move to the DevOps model of software development. 

We normally talk about the process as a whole, but each term can be thought of separately:

- Continuous Integration: All of code changes get merged into the main git branch with minimal delay
- Continuous Delivery: Instead of releasing on a cadence, every build that passes tests gets pushed out
- Continuous Deployment: Expands the delivery into having the builds that pass being deployed automatically (and rolled back automatically when needed)


All together, we can gain a substantial amount of agility in our teams. By keeping changes as small as possible, there's less risk overall and any bugs can be found sooner. Also, with the right automation (or augmentation), we can focus more on delivering value instead of "babysitting" builds.

With all of this additional complexity in the development process, we really need to automate massive chunks of the work. It would otherwise be too hard to do all of the steps manually.


## Options

There is an [ever growing list](https://landscape.cncf.io/card-mode?category=continuous-integration-delivery&grouping=category) of tools to help you build and automate CI/CD pipelines. Here are a few of the most popular and, in my opinion, interesting tools in this space.

### [Argo CD](https://argoproj.github.io/)

Argo CD is built around splitting CD from CI by focusing on the deployment of already built applications into a Kubernetes cluster. It has a variety of ways to trigger deployments from your CI system as well as monitoring when the declarative definitions themselves change in git. 

It also comes with some nice visualizations of the components being deployed and shows how they change over time. 

Argo is likely the right choice for projects where having the CI portion of your pipeline decoupled is useful.

### [Jenkins X](https://jenkins-x.io)


Jenkins X is a fully featured GitOps solution that requires developers to know less about Kubernetes than other options. It's built on top of Tekton (below) and adds a ton of usability enhancements and abstractions to the pipelines. 

It's focus is on removing complexity while still allowing for insight into how the system is working. If you want a fully featured pipeline on premise, Jenkins X should be on your short list to try.

### [GitLab CI/CD](https://docs.gitlab.com/ee/ci/)

As part of their full setup, GitLab offers an integrated and easy way to automate pipelines directly from a project's GitLab repo. Pipelines are tied to the repositories themselves and can be extended to make developer's lives easier with CD options like Argo CD. 

There is also their Auto DevOps offering that will make your pipelines act like a PaaS of sorts by building and deploying the containers in an easy to configure way. 

If you are using GitLab, definitely check this out! 

### [Tekton](https://tekton.dev)

Tekton is becoming the go-to for other projects to build on. It uses [Custom Resource Definitions](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) to provide an expressive way to build pipeline definitions with the existing RBAC in Kubernetes. 

One major distinction is that pipelines are built outside of the project's git. This allows for a way to reuse portions of pipelines across any and all projects.

If you are looking for a way to write abstract pipelines that can be reused in different projects, Tekton might be your best bet!


### [Spinnaker](https://spinnaker.io/)

Spinnaker is a continuous deployment specific option from Netflix. As such, it is optimized for huge deployments. It gives a lot of control over deployment strategies to allow for safer rolling deployments with many places for customization.

It's designed around sane defaults and best practices and would be a good choice for larger organizations that have a ton of different applications working together.

### [Drone](https://www.drone.io)

Drone gives you a way (similar to CircleCI or GitLab CI/CD) to create full pipelines in a configuration in git with your application code. It supports several git providers, giving you flexibility in where you store your code (something that GitLab CI/CD and Github Actions obviously don't allow). 

On this list, Drone is likely the simplest option to use. This simplicity makes it a good choice for people that are working by themselves or working on a small team!

## How Do I Choose?

As with any infrastructure choice, there are tradeoffs. In CI/CD, they are based around power versus usability, complete CI/CD versus CD specific, pipelines coupled with code versus higher order abstractions, etc.

You can make most of these options do what you need. But obviously you'll get more out of the tool if it's used in conjunction with processes and architectures that work in the same way the creators intended! Definitely try out a few and match them up with how you think about building applications to give the best experience.
