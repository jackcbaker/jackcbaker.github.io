+++ 
draft = true
date = 2022-07-22T07:50:31
title = "Practical kubernetes for data scientists: 1. what is kubernetes?"
description = "Learn to deploy your data science solutions using the most popular open-source framework kubernetes. This tutorial focuses on what kubernetes is, and what it can do."
slug = "what-is-kubernetes"
authors = ["Jack Baker"]
tags = ["data science", "mlops", "machine learning engineer", "software engineering"]
+++

# What you'll get out of these tutorials?

I've been deploying data science solutions on kubernetes for a while now. But found some of the tutorials online a little difficult to get my head around sometimes. I also found many of the tutorials did not have a focus on data science tools.

This set of tutorials documents what I've learnt, and aims to provide a resource that is data science focused and completely practical – allowing you to dive in and learn by doing.

# Why use kubernetes?

Kubernetes has a few advantages over other methods:

#### Low-cost, cloud agnostic

Kubernetes is free and open-source, and comes with a lot of powerful features. Many of these features enable cost management of your infrastructure. Also certain things that would require yet another cloud service (with associated costs) can be done on kubernetes for free:

* Trigger jobs (i.e. use the resources you need at the time, rather than keeping a big server running all the time)
* Create and serve APIs
* Autoscaling

You can install a kubernetes cluster on the infrastructure you have available, or get a managed server for the cloud. Any setup you create for the cluster can then easily be moved across to a different provider.

#### A standard tool across IT

Many data science teams face hurdles when trying to deploy tools because of IT security or architecture concerns. This is especially true of platforms that help with data science deployment, as they might require sensitive data to sit on third party storage, or the platform itself needs to be setup on company infrastructure. The great thing about kubernetes is it is a standard for deployment across many areas of IT. 

This means many companies will already have a kubernetes solution in place, or may be more receptive to deployment using kubernetes.

#### Local testing

You can easily install kubernetes on your laptop (docker desktop even comes with it already). This means that it is easy to test stuff locally for free. This is also great for learning. This fact allows this tutorial to be completely practical.

## Downsides

The main downside to kubernetes flexibility is it has quite a steep learning curve, and some stuff can take longer than you want. However once you have built up templates for how to do things (this blog series will link to many templates), you will be able to do stuff faster.

# What data science tools can kubernetes deploy?

There are plenty of kubernetes resources depicting kubernetes features. But here I want to map those features back to what data science tools kubernetes can help you deploy:

* Simple batch processing jobs: these are jobs that run regularly at a specific time period (e.g. a tool that runs at 9am every Monday).
* Dashboards such as plotly dash or RShiny.
* APIs.
* Data science jobs that get triggered by an API call or a user clicking something in a dashboard.
* Full-stack apps.


<div style="text-align: right">**[Next: deploying a data science job](kubernetes-deploy-job.md)**</div>