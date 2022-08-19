+++ 
draft = true
date = 2022-07-22T07:50:31
title = "Practical kubernetes for data scientists: 2. Setup a local kubernetes cluster"
description = "Learn to deploy your data science solutions using the most popular open-source framework kubernetes. This tutorial shows you how to setup a local kubernetes cluster to run all the tutorials yourself."
slug = "kubernetes-for-ds-local-kubernetes"
authors = ["Jack Baker"]
tags = ["data science", "mlops", "machine learning engineer", "software engineering"]
+++

As I mentioned before, these tutorials aim to be completely practical. To do so you need to be able to run the tutorials we have!

Luckily for us, docker desktop preinstalls a simple, local version of kubernetes that we can use to play around with it.


# Installing docker desktop

Go [here](https://docs.docker.com/desktop/#download-and-install) for instructions on how to download and install docker desktop for your operating system – there are instructions for all major operating systems there.

While for commercial applications docker desktop is not free, for educational purposes it is. If you are interested in going a completely open-source route, one option is to use [minikube](https://minikube.sigs.k8s.io/docs/tutorials/docker_desktop_replacement/#:~:text=You%20need%20to%20start%20minikube,docker%2Denv%20and%20docker%20build%20.). Docker desktop is an excellent, and standard tool across data science and software engineering though, so most companies will be happy to purchase a licence.

## 2. Setup kubernetes within docker desktop

If you opted for minikube, you can skip this step. For docker desktop users, it's dead easy to enable kubernetes. Once you've opened docker desktop, navigate to Preferences > Kubernetes. Then tick the box `Enable Kubernetes`. See the screenshot below.

{{< image src="/post_images/kubernetes/local-kubernetes/docker-desktop-enable-kubernetes.png" alt="Screenshot of enabling kubernetes in docker desktop." >}}