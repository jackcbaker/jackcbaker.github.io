+++ 
draft = true
date = 2021-04-25T15:36:34+01:00
title = "Speed up developing your data science solutions"
description = ""
slug = "check-data-science-pipeline-working"
authors = ["Jack Baker"]
tags = ["data science", "development", "testing"]
+++


When I started in my first data science job, I thought making a data science tool reusable meant building one big data science tool that solves the original problem at hand and can be reapplied many times. While this is great when it's possible, there's many hurdles: business constraints might be different, the data sources might vary, or the objective might be different. These can often hamper the ability to make reusable tools.

Luckily through some great mentors, I've learnt this is not the only way to make reusable tools. In this post I'll offer some tips for building a data science solution that is easier to reuse.


## Break a problem down into its smallest component part

While the whole data science solution may not be the same, often there are small component parts that are the same. For example, in a pricing model, you'll always need to approximate a demand curve; in an inventory optimisation, you'll always need to calculate safety stocks. Often these component parts can be tiny, such as combining data sources in a particular way.


## Write your code data source agnostic

If you're working with large datasets, databases are unavoidable. But often data science pipelines will pull code into memory and run everything in python/R. If that's the case it's a good idea to separate your transforms as much as possible from the specifics of where your data is stored (e.g. is it local, S3, etc.). For example, once you've pulled your dataset into memory, you can write most of your code as functions/classes that just require a dataframe as input and return a dataframe as output.

This means that provided you get another dataset in the correct schema, you can reuse that piece of code. Doing this also helps clarify exactly the data you require for that particular component, which helps you define data requirements more quickly.

Combining this point with the point above, and you should have a library of small pieces of modular code that is data source agnostic. Already your code should be much easier to reuse.

If you need to work with databases, you can still use this tip, just writing your database transforms in as reusable way as possible. This means separating out your data transforms as much as possible from the database specifics themselves (e.g. the connection information and database type). Another useful tool for writing reusable database tools is using an [object relational mapper](https://en.wikipedia.org/wiki/Object%E2%80%93relational_mapping).