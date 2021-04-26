+++ 
draft = false
date = 2021-04-26T21:00:00
title = "Three ways to speed up developing your data science solutions"
description = "Customers want results quickly. In this post I'll share some ways I use to try and develop solutions faster."
slug = "speed-up-data-science-development"
authors = ["Jack Baker"]
tags = ["data science", "development", "ai", "reusability"]
+++


Your customers want results and quickly. This can be stressful as a data scientist: solutions are often experimental and results not guaranteed; you need time to think about the problem.

In this post I'll share three ways I use to develop data science solutions faster.

I find there are a few time sinks when developing data science solutions: going down a rabbit hole or dead end, productionising, checking your solution actually beats the current business process, and ensuring your code is free of bugs. How do I try to avoid these pitfalls?


## 1. Start as simple as possible

When I start a new problem, I implement the simplest possible solution I can think of. Once this is developed and tested, I iteratively improve on it. What this gets you:

* A baseline you can compare all your models against. This helps stop you going down a rabbit hole: I talk about this more later.
* A quick, quality start -- I regularly find a baseline beats a more complex model.
* A chance to develop the pipeline without worrying about intricate model headaches. I find this makes a solution easier to productionise.

This also applies to iterations on your baseline. Break these down so they're as small as possible. If you follow the third point this becomes powerful.


## 2. Modularise

Often a data science solution consists of lots of mini problems you need to solve. I recommend you break down your problem as much as possible into components. What this gets you:

* You can write each modular part as separate code modules. Set these to take relatively general input and output, and your code will be much easier to productionise: all you need to write are interfaces to your particular infrastructure (which is reusable code). 
* Your code will be easier to test, which helps ensure it's bug free. For example using the technique I talk about [in this post](https://jackcbaker.github.io/posts/check-data-science-pipeline-working/).
* If you follow the first point, your code will be reusable for future solutions.
* Your baseline becomes even simpler :).

For example, say you're developing a churn model, you might have: one or two modules handling data transformation, a module that handles the model build, one that handles serving predictions, and one that handles customer visualisations.


## 3. Test each iteration against your baseline, the current business process and your best model so far

If you follow what I mentioned earlier and keep your model iterations small, this can really keep you on track:

* You catch rabbit holes early by seeing that an iteration made no improvement.
* You can hone in on which module needs most improvement.
* You can fail fast -- if something isn't working, you'll know it quickly and can manage customer expectations.
* You have regular tangible results to share with customers.