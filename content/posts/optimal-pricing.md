+++ 
draft = true
date = 2021-09-19T18:18:31
title = "Reinforcement learning for optimal pricing"
description = "How to optimise pricing using reinforcement learning."
slug = "optimal-pricing"
authors = ["Jack Baker"]
tags = ["data science", "machine learning", "optimisation", "reinforcement learning"]
+++


# Elasticity

Lowering a product's price is an obvious way to make more sales. But the amount more that you sell given a lowering in price varies between products. For example, even if the price of water was raised to extortionate amounts, people would still pay, because they need it to survive and there's no alternative. On the other hand, if there was a world shortage of toastie makers, people would probably switch to using the grill.

This concept is known in economics as elasticity. Elasticity is defined as
\\[
    \frac{\text{% change in demand}}{\text{% change in price}},
\\]
or letting \\(x\\) be our price and \\(y\\) be our demand, we define it as
\\[
    \frac{dy / y}{dx / x}.
\\]

Elasticity measures how sensitive your demand is to a change in price. Products with low elasticity will not react much to a change in price; while products with high elasticity will react a lot.

{{< image src="/post_images/optimal-pricing/constant-elasticity.png" alt="A demand-price curve with constant elasticity." caption="By Namtranhoang1992 - Wikimedia Commons, CC BY-SA 4.0. " attrlink="https://commons.wikimedia.org/w/index.php?curid=76870640" attr="Link.">}}

Price elasticity is a good start for our pricing problem: in order to optimise how much profit is affected by price, we need to know how sensitive it is to it. Elasticity does not have to be fixed, it can be different as the price is varied. But for simplicity we assume it is a fixed parameter that needs to be estimated -- this is a common assumption for economic models.


## Modelling Elasticity

A natural model to use when assumming elasticity is fixed, is the [iso-elastic](https://en.wikipedia.org/wiki/Isoelastic_function) model. Keeping \\(x\\) as our price, and \\(y\\) as our demand, this is defined as
\\[
    y = Ax^e.
\\]
Here \\(A\\) is a constant, and \\(e\\) is the elasticity (this can be shown using the definition above).

This is all well and good, but we don't know what the elasticity of our good is -- we need to model it. By applying logs to both sides, this should start to look in a familiar form
\\[
    \log(y) = e \log(x) + \log(A).
\\]
Set \\(\beta_0 = \log(A)\\) and add an error term \\(\epsilon\\) and we have a linear regression model with two parameters: \\(e, \beta_0\\).
\\[
    \log(y) = e \log(x) + \beta_0 + \epsilon
\\]

Now we have a linear model which will allow us to predict the elasticity of our good given data. In practice, we may want to add more regressors to this model (for example the day of the week to model seasonality). In fact we might even choose to use a full forecasting model with price as a regressor. This would try to ensure as much variability not due to price-elasticity is captured.

For simplicity though, we will continue with this simple model of elasticity; adding the extensions mentioned does not change the procedure too much, and I will talk about how to add these later.


# Bayesian Linear Bandits


For our pricing procedure, we essentially need to 