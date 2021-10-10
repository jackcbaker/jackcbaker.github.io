+++ 
draft = true
date = 2021-09-18T18:18:31
title = "Optimising supply chain networks using data science"
description = "Most inventory optimisation methods tell you how much stock to bring in. But often at the warehouse level there is not enough space, or enough trucks to move it all. In this case you need to prioritise what to bring in, this article explains how."
slug = "inventory-prioritisation"
authors = ["Jack Baker"]
tags = ["data science", "machine learning", "optimisation"]
+++


# Why supply chain networks are often strained?

Most demand planning is performed at the highest level of granularity (e.g. how much will be demanded in the UK over the next week). This reduces the volatility of demand and so means less unnecessary stock is produced or bought. But this is only part of the supply chain. A big part is then distributing the stock around the supply network to ensure it is as close to the customer as possible. 

Since there is only enough stock at the highest granularity, this causes a strain on the network because:

### 1. The demand at lower granularities will be more volatile

To fill each warehouse with enough stock to account for these volatilities would be more stock than is available in the network, since demand planning performed at the highest level of granularity.

### 2. Warehouse capacities

Often individual warehouse capacities are tight: for example they might be full of a product that hasn't had as much demand as expected etc.

### 3. Capacity for moving stock

The capacity to move stock around the network is limited (e.g. the number of trucks available). Because the network is strained stock might be put in the wrong warehouse and then has to be moved. But this is a waste that is bad for profitability and the environment, so should be minimised if possible.

### 4. Seasonality

Warehouses will tend to have highly busy days followed by quieter days. Often the capacity for moving stock will be enough for a day of 'average demand'. This adds a big problem: when should I start bringing in stock for my busiest day? Because if you don't, you'll miss a lot of sales as you didn't bring enough stock in on time.


# Why traditional inventory optimisation methods don't work at the network level?

The classic inventory optimisation methods such as the _reorder point_ method, and the _(s, S)_ method answer the question:

#### 'Given my (uncertain) demand how much inventory should I bring into my warehouse to make sure I'm unlikely to miss a sale?'

But at the network level, this is probably the wrong question to ask. Because your network is strained, you're unlikely to be able to bring in the stock these methods recommend at the warehouse level. The methods also focus on one item at a time, but at the network level, bringing in one item may mean you can't bring in enough of another. This means you need to consider all items at the same time.

In reality, the question often morphs into, what is the most valuable stock to bring into the warehouse today? This is the question I'll be solving in this article.


# What is the most valuable stock to bring into the warehouse today?

In this article I'll focus on a way to solve this problem. For now I'll focus on what to bring into one warehouse with multiple items, random demand, and a constraint on how much can be bought in (e.g. the number of trucks available to move the product). 

This problem is not too bad to solve on its own, but typically supply networks will see seasonal demand (see point 4 above). This means we don't just need to think about today, we need to look ahead to see if stock desperately needs to be brought in for the busiest day. This adds a lot of complexity, and the solution took a lot of thought; but I cover a heuristic for this case after the single day case.

The solution can easily be extended to multiple warehouses competing for stock, using the same logic. I'll leave this to a future post.

It should also be extendable to the case where the warehouse capacity itself is the constraint.


# Single period problem

We'll start with the single period problem (e.g. solving the problem for just one day), where we can derive an optimal solution in terms of expected profit. This sets up the notation for the multi-period problem.

## Setup

Assume we have \\(i= 1,...,I\\) SKUs. At the end of the period a random number \\(Y_i\\) of each item is demanded. Let's assume we have a reasonable handle on \\(\mathbb P(Y_i \geq y)\\), i.e. the demand probabilities. This can be achieved through a probabilistic forecast.

We assume starting inventory \\(x_i\\) for each item is known. The amount of inventory of item \\(i\\) we choose to bring in we label \\(a_i\\).

With this in place, we can label the remaining variables, and define the problem:

* \\(r_i\\) -- the revenue gained if one unit of item \\(i\\) is demanded.
* \\(C_i\\) -- the cost of bringing one further unit of item \\(i\\) into the warehouse. This could be due to e.g. transport or purchase costs.
* \\(K\\) -- maximum number of items that can be brought into the warehouse: i.e., we required \\(\sum_{i=1}^I a_i \leq K\\). This could be due to movement capacity constraints (e.g. number of trucks). There might be an explicit warehouse capacity constraint, which could be easily added to the single period problem, but requires more work for the multiple period problem.

We can define the problem as bringing in \\(a_i\\) in order to maximize profit, defined as
\\[
    \sum_{i=1}^I \left(r_i Y_i - a_i C_i \right),
\\] 
while staying below the movement constraint \\(\sum_{i=1}^I a_i \leq K\\).

Bring in too much stock, and we are paying unnecessary cost \\(C_i\\). But not enough stock will lead to unmet demand, and lost sales. Sometimes a penalty for missed sales gets added to this problem, to indicate impact to customer loyalty. This can easily be done in this formulation.


We can solve the single period problem to maximise expected reward using a simple iterative algorithm. First define \\(\hat a_i\\) to be the amount of item \\(i\\) scheduled to be moved so far by the algorithm, and initialise by setting equal to 0, \\(a_i = 0\\).


We can calculate the marginal reward of loading an extra unit of item \\(i\\) exactly as
\\[
    r_i \mathbb P(Y_i > x_i + \hat a_i) - C_i.
\\]
The first term is the revenue multiplied by the probability that additional unit would be ordered. The second term is the cost of loading that item.

This marginal reward immediately gives rise to an iterative algorithm:


### Single period algorithm

1. Initialise \\(a_i = 0\\)
2. Until STOP do

    i. 
        \\[
            i_{best} = \text{argmax}_i \left[ r_i \mathbb P(Y_i > x_i + \hat a_i) - C_i \right]
        \\]
    ii. 
        If for \\(i_{best}\\) \\(r_i P(Y_i > x_i + \hat a_i) - C_i < 0\\), or \\(\sum_{i=1}^I \hat a_i \geq K\\): STOP

    iii. 
        Load item \\(i_{best}\\): set \\(\hat a_{i_{best}} = \hat a_{i_{best}} + 1\\)


# Multi-period problem

This problem is more complex but if we want to handle seasonality it's essential. Now we add in time \\(t = 1, ..., T\\). We set \\(T\\), the horizon, to be the seasonal period (e.g. 7 if we're looking at daily data). This ensures the busiest days are taken into account when loading.


We assume at the start of each period \\(t\\), we have inventory for item \\(i\\) of \\(x_{ti}\\). We assume \\(x_{1i}\\) is known, but for all later periods it depends on the demand and loading choice. So is unknown.