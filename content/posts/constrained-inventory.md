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

We assume a loading amount \\(a_{ti}\\) is chosen for the period, and after that a random amount \\(Y_{ti}\\) is demanded. This is fulfilled from the inventory \\(x_{ti} + a_{ti}\\), with reward \\(r_i\\) for each unit of item \\(i\\) successfully fulfilled, but any demand over the inventory is lost sales.

The rest of the problem setup is as follows

* \\(C_i\\) -- cost of loading one unit of item \\(i\\)
* \\(K\\) -- capacity of items that brought into the warehouse in any period, i.e. for all \\(t\\), \\(\sum_{i=1}^I a_{ti} \leq K\\)


## Possible heuristic procedure

Because starting inventory \\(x_{1i}\\) is known, we can solve the problem exactly using dynamic programming. We would do this by proceeding backwards from the last horizon, which we solve using a single period problem. Then each of the possible cases for \\(T-1\\) (depending on inventory levels at that point) can be solved using a single period problem, and the value function for the expected remaining inventory at \\(T\\), etc.

The big problem with this approach is we have to consider all possible inventory levels for each item, which will quickly explode as the number of items increases.

We attempt to get around this by noticing that we are only interested in \\(a_{1i}\\) at each period, i.e. the loading we have to do today. To do this we have to consider the future states in case there's a particularly busy day. But we suggest instead of having it as a state, we replace it with its expected value, determined by \\(x_{ti}\\), \\(\hat a_{ti}\\) and \\(\mathbb E(Y_{ti})\\). Once again we defined \\(\hat a_{ti}\\) to be the loadings chosen by the algorithm so far.

We can initialise the problem using a greedy algorithm. Set \\(\hat a_{1i}\\) by solving the single period problem for period 1. Now initialise the inventory for period 2 using expected values
\\[
    x_{2i} = \max\left[x_{1i} + \hat a_{1i} - \mathbb E(Y_{1i}), 0 \right].
\\]

We can proceed initialisation iteratively using a similar procedure: solving a single period problem for horizon \\(t\\) to get an initial \\(\hat a_{ti}\\), then initialising the inventory as
\\[
    x_{t+1,i} = \max\left[x_{1i} + \sum_{s=1}^t \left(\hat a_{si} - \mathbb E(Y_{si})\right), 0 \right].
\\]

Now we've intialised the problem, we can proceed to the heuristic algorithm. For this, the inventory \\(x_{ti}\\) and loading amounts \\(\hat a_{ti}\\), need to be constantly updated as things are changed by the algorithm.

Similar to dynamic programming, we proceed backwards, but now with our fixed inventory states. For period \\(t=T\\), we are at the end of the horizon, so have no horizons ahead to consider. So it's safe to assume that the single period problem will be a good result. Therefore we do nothing for this period, and move back to the next period.

At period \\(t=T-1\\), we need to look at both the current period, and the period \\(t=T\\). First set \\(b_{t,i} = x_{ti} + \hat a_{ti}\\). Applying our inventory assumption, similar to the single period problem, we can write down the marginal reward of adding another unit of item \\(i\\) to the loading in period \\(T-1\\):
\\[
    N_{T-1,i} = \mathbb P(Y_{T-1,i} > b_{T-1, i}) R_i - C_i +
    \mathbb P(Y_{T-1,i} \leq b_{T-1, i}) \mathbb P(Y_{T,i} > b_{T,i}) R_i.
\\]
We can break down the parts of the marginal reward as follows: the additional unit of product $i$ will either be ordered at time $T-1$, not ordered at time $T-1$ but ordered at time $T$; or not ordered at all because we've reached the end of the horizon.

Because we've initialised this problem with the single period algorithm though, our loadings might be full. In this case we can't just add the item with the best marginal reward, we need to swap it with an item in the loading.

To do this we can check the marginal cost of removing one unit of one item from the knapsack. This is equivalant to the marginal reward  above replacing \\(b_{T-1, i}\\) with \\(b_{T-1, i} - 1\\); i.e.
\\[
    M_{T-1,i} = \mathbb P(Y_{T-1,i} > b_{T-1, i} - 1) R_i - C_i +
    \mathbb P(Y_{T-1,i} \leq b_{T-1, i} - 1) \mathbb P(Y_{T,i} > b_{T,i} - 1) R_i.
\\]

A single step of the algorithm then proceeds as follows:

1. Check if \\(N^* = \max_{i} N_{T-1,i}\\) is positive. If it's \\(\leq 0\\), STOP.

2. Set \\(i_{best} = \text{argmax}_{i} N_{T-1,i}\\). Set \\(M_* = \min M_{T-1, i}\\), and \\(j_{worst} = \text{argmin}_{i} M_{T-1,i}\\).

3. If the capacity \\(K\\) is not reached (i.e. \\(\sum_{i=1}^I a_{T-1,i} < K\\)), then add \\(i_{best}\\) to the loading. Set \\(a_{T-1,i_{best}} = a_{T-1, i_{best}} + 1\\).

4. If the capacity \\(K\\) is reached, then check if \\(N^* > M_*\\). If it isn't STOP. Otherwise load \\(i_{best}\\) and unload \\(j_{worst}\\); i.e. set \\(\hat a_{T-1, i_{best}} = \hat a_{T-1, i_{best}} + 1\\) and \\(\hat a_{T-1, j_{worst}} = \hat a_{T-1, j_{worst}} - 1\\)


After each iteration of this algorithm, if \\(N^* \leq 0\\), or \\(N^* \leq M_*\\), we stop completely and move to the next period.

Otherwise we update the inventory values \\(x_{t,i_{best}}\\) and \\(x_{t, j_{worst}}\\) for \\(t = T-1, T\\) using the new loadings \\(\hat a_{T-1, i_{best}}\\) and \\(\hat a_{T-1, j_{worst}}\\). Also update \\(N_{T-1, i_{best}}\\), \\(N_{T-1, j_{worst}}\\), \\(M_{T-1, i_{best}}\\) and \\(M_{T-1, i_{worst}}\\) given the updated loadings and inventory values. Then we repeat the algorithm above, until either \\(N^* \leq 0\\), or \\(N^* \leq M_*\\).


Moving to time period $t = T-2$ we can follow the exact logic as for $t = T-1$. The difference here will be our marginal rewards. We have
\\[
    N_{T-2,i} = \mathbb P(Y_{T-2,i} > b_{T-2, i}) R_i - C_i 
    + \mathbb P(Y_{T-2,i} \leq b_{T-2, i}) \mathbb P(Y_{T-1,i} > b_{T-1,i}) R_i
    + \mathbb P(Y_{T-2,i} \leq b_{T-2, i}) \mathbb P(Y_{T-1,i} \leq b_{T-1, i}) \mathbb P(Y_{T,i} > b_{T,i}) R_i.
\\]
Breaking this down: the additional unit of product \\(i\\) will either be ordered at time \\(T-2\\), not ordered at time \\(T-2\\) but ordered at time \\(T-1\\), not ordered at time \\(T-2, T-1\\) but ordered at time \\(T\\); or not ordered at all because we've reached the end of the horizon. Similar logic can be used to calculate \\(M_{T-2, i}\\).

This procedure can be repeated to time period $t=1$, which will be our required loading for today.


## Intuition

The intuition for this heuristic is that the uncertainty in demand has two effects: the uncertainty for that period; the uncertainty of how much inventory will be available in later periods. This heuristic captures how demand affects the uncertainty in each period, but ignores knock-on uncertainty effects on the inventory levels -- replacing it with a deterministic expected value. This should work because we are mainly interested in the first period, so not capturing the inventory uncertainty would hopefully not have a massive effect.