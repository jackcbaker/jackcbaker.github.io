+++ 
draft = true
date = 2022-07-22T07:50:31
title = "Kubernetes python client: programatically creating a job"
description = "I found a few things I wanted to do in the python kubernetes client not covered in the examples and documentation. This set of tutorials aims to cover those. This first tutorial is on programatically creating a job, which I will build on in subsequent tutorials."
slug = "kubernetes-python-client-job"
authors = ["Jack Baker"]
tags = ["data science", "mlops", "software engineering"]
+++


# What is the python kubernetes client

The [python kubernetes client](https://github.com/kubernetes-client/python) is the official python client for kubernetes. It acts a lot like kubectl, but is accessed through python rather than your CLI.

## Tips for using the python kubernetes client

Some useful things to know about the python kubernetes client:

- It is designed to completely mirror the kubernetes API. Therefore if you want to get the reference for a particular function or kubernetes building block, I found the [kubernetes API docs](https://kubernetes.io/docs/reference/#api-reference) to be an excellent place to do this. I found the API reference in python kubernetes itself to be a bit limited for this... but maybe I just couldn't find it!


# Setting up

## Setting up a local version of kubernetes

## Fetching the API objects

The first step after installing `python-kubernetes` is to call the API objects and save them to variables. Kubernetes has a few different APIs for different resources. This [resource](https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-apiversion-definition-guide.html) explains those differences. We need core, which holds many of the core objects, and batch which holds many of the objects required for batch processing and jobs.

```python
from kubernetes import client, config

# Load config from kubernetes cluster.
# This command will check for kubernetes config in default places,
# and if that fails will assume in-cluster config.
config.load_config()

core_v1_api = client.CoreV1Api()
batch_v1_api = client.BatchV1Api()
```