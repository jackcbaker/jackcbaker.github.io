+++ 
draft = false
date = 2021-04-11T15:36:34+01:00
title = "How to tell your data science pipeline is actually working?"
description = "After developing a data science pipeline it's notoriously difficult to check it's working as expected. This post details a tactic to solve this problem."
slug = "check-data-science-pipeline-working"
authors = ["Jack Baker"]
tags = ["data science", "development", "testing"]
+++

I've been there - studying model output for hours to check a model is working. There's some better ways than this. For example designing backtests to check a model does better than a baseline. The trouble with these methods is they don't categorically show that a model doing exactly as it is designed to do; they're also not automated, or (as with a backtest) are expensive to run; and sometimes they're harder to design than the model itself! Tests that are not automated mean as soon as you make changes you need to do another lengthy check or risk the model being invalid. Other options are unit and integration tests, which are automated, but these don't explicitly check the model is working correctly.

The way I solve this problem is test datasets. We often spend a lot of time as data scientists waiting for data to arrive. Why not create a test dataset during that time (it only takes about an hour for simple pipelines, which is a lot less than typical data lead time). This test dataset should: be modelled off the schema of your source data; have output that, once processed through your modelling pipeline, is known exactly. For example:
* If you have a regression pipeline, you could generate random linearly related data and check held-out predictions match your actuals within a desired level of tolerance.
* If you have a forecasting pipline, you could check it can predict on timeseries data generated from an ARIMA process within a desired level of tolerance.

This does not replace the function of a backtest: it is not checking if the model actually works well on the problem at hand. But what it does is check that your modelling pipeline is doing exactly what it's designed to do. This is especially important when you have quite a complex pipeline which is doing multiple data transformations before applying the model. It's easy to make a mistake with these transformations and not notice. The test can also be automated, and is cheap to run (if you keep your test dataset small), so you get a free integration test on your full pipeline. That way if you make any changes you can just run the test again to check things are still working as expected. You can also add some tests in between different stages of the pipeline to get integration tests on different parts of the pipeline for free.