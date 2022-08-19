+++ 
draft = true
date = 2022-08-11T07:50:31
title = "Practical kubernetes for data scientists: 3. deploying a job"
description = "Learn to deploy your data science solutions using the most popular open-source framework kubernetes. This tutorial focuses on deploying the simplest form of data science tool: a batch job."
slug = "kubernetes-for-data-scientists-deploying-jobs"
authors = ["Jack Baker"]
tags = ["data science", "mlops", "machine learning engineer", "software engineering"]
+++


# What is a job?

The simplest type of data science tool we can deploy is a 'job'. This is a tool that automatically runs at a set time or given an event. For example:

* Every Monday at 9am, we run a churn model that is passed onto the customer support team.
* Every time we get a batch of credit data through, we run a credit scoring model to predict likelihood of default.

In this tutorial, we focus on the very simplest option, which is the first one – getting a model to run regularly at a specified time in production.


# Setting up the job

Before we can build the kubernetes job, we need something that we want to run! For this, we have a very simple script that trains a random forest on a standard scikit-learn dataset. I've put this script into the `src` subdirectory of the code.

```python
import logging

from sklearn.ensemble import RandomForestRegressor
from sklearn.model_selection import train_test_split
from sklearn import datasets
import pandas as pd


logging.basicConfig(
    level=logging.INFO
)
logger = logging.getLogger()


def main():
    dataset = datasets.fetch_california_housing()
    X = pd.DataFrame(
        data=dataset['data'],
        columns=dataset['feature_names']
    )
    y = pd.Series(
        data=dataset['target'],
        name=dataset['target_names'][0]
    )
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=.1)
    model = RandomForestRegressor()
    model.fit(X=X_train, y=y_train)
    logger.info(
        "Model on test set has R^2 score of %s",
        model.score(X=X_test, y=y_test)
    )


if __name__ == '__main__':
    main()
```

Find the full template code for this example [HERE – TODO]().


## Containerising the job

Kubernetes is designed to work with containers/docker. These are basically mini-operating systems surrounding an installed app. This allows Kubernetes to deploy the job without worrying about dependencies (e.g. what version of this do I need?) – the container handles that. 

Therefore the first step to successfully deploying this example on kubernetes is to build a docker container that can run the code. I will assume a basic understanding of docker, if you need to build this knowledge I recommend [docker's official tutorials](https://docs.docker.com/get-started/).

Here is a Dockerfile that will install the python dependencies we need, which have been listed in `requirements.txt`.

```docker
FROM python:3.9

WORKDIR /app

COPY requirements.txt ./
RUN pip install -r requirements.txt

COPY src ./src/

CMD python src/train.py
```

This grabs the python3.9 default image, installs the dependencies, copies the python code across from `src/`, and then when it is run will run our python file.

To build it run `docker build -t train .` in the root of the package. To check it works run `docker run -t train`. It should return the R^2 score for our model.


## Run containerised job in kubernetes

Now we have a containerised job, we have everything we need to build and run our first job in kubernetes!

First we need to tell kubernetes how to run the job we want. We do this by creating a *kubernetes manifest*, which is a yaml file – a standard file often used for configuration like this.