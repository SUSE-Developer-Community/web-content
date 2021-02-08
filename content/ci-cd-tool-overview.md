---
title: "Overview of Ci/CD tools"
date: 2021-02-03
draft: true
---

# Intro

Continuous Integration (CI) and Continuous Delivery (CD) are hot topics right now with the move to the DevOps model of software development for so many organizations.


# Options

There is an [ever growing list](https://landscape.cncf.io/card-mode?category=continuous-integration-delivery&grouping=category) of tools to help you build and automate CI/CD pipelines. Here are a few of the most popular or interesting in my opinion.

## [Argo](https://argoproj.github.io/)

Argo is built around splitting CI from CD but only supporting the deployment of already built applications. It has a variety of ways to trigger deployments from your CI system as well as monitoring when the application definitions themselves change in git. 

It also comes with some pretty nice visualizations of the components being deployed and how they change over time! 

## Jenkins X

This is a fully featured GitOps solution that allows your developers to know less about Kubernetes that other options. It's actually built on top of Tekton (below) and adds a ton of usability enhancements and abstractions to the pipelines. 

## Gitlab CI

As part of their full set up, Gitlab offers an integrated way to automate pipelines directly from a project's Gitlab repo in an easy to do way. Pipelines are tied to the repositories themselves and can be extended to make developer's lives easier with CD options like Argo. 

If you are using Gitlab, definitely check this out!

## Tekton

Tekton is becoming the go-to for other projects to build on top of. It uses [Custom Resource Definitions](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) to give a very expressive way to build pipeline definitions with the existing RBAC in Kubernetes. 

One major distinction is that pipelines are built outside of the project's git. This allows for a way to reuse portions of pipelines across any and all projects.  


## Spinnaker

Spinnaker is an continuous deployment specific option from Netflix. As such, it seems to be pretty optimized for huge deployments. It gives a lot of control over deployment strategies to allow for safer rolling deployments with many places for customization.

## Drone

Drone gives you a way (similar to CircleCI or GitlabCI) to create full pipelines in a configuration in git with your application code. It supports several git providers to give a lot of flexibility in where you store your code (something that Gitlab CI and Github Actions obviously don't allow). 



# How do I choose?

As with every infrastructure choice, there are trade offs to be made. In CI/CD, the tradeoffs are based around power vs usability; complete CI/CD vs CD specific; pipelines coupled with code vs higher order abstractions, etc.

Most of the options can be made to do what you need, but obviously you will get more out of the tool if it's used in conjunction.

