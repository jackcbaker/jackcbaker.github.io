+++ 
draft = false
date = 2021-05-08T13:17:00
title = "Upweighting your recent observations in regression and classification"
description = "COVID-19 has changed many established patterns. We're all having to adjust to a new normal: how do we get our data science models to adjust as well? This post describes how to build regression and classification models that upweight more recent observations."
slug = "upweight-recent-observations-regression-classification"
authors = ["Jack Baker"]
tags = ["data science", "machine learning", "python", "R"]
+++

COVID-19 has changed many established patterns. We're all having to adjust to a new normal: how do we get our data science models to adjust as well?

COVID aside, in any regression or classification problem where observations have a time element, old patterns can become stale. For this reason, I'm often asking myself -- how do I upweight my most recent observations? In this post I explain how to do this.

All the code in this tutorial is available as a [jupyter notebook](https://github.com/jackcbaker/blog-notebooks/blob/main/regression-forgetting.ipynb) on my [github](https://github.com/jackcbaker/).

## Step 1: Fetch the data

For this post I'll be using data from the great [UK government COVID dashboard](https://coronavirus.data.gov.uk/). The data and plots from this website are readily available as part of the [Open Government Licence v3.0](https://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/).

For this tutorial, we'll be using two measurements from the dashboard: ['People tested positive'](https://coronavirus.data.gov.uk/details/cases) (I'll call this positive tests) and ['Deaths within 28 days of positive test'](https://coronavirus.data.gov.uk/details/deaths) (I'll call this deaths).

The plotted data as it appeared on the dashboard at the time of writing is below. We can see that for the recent data, positive tests seems to have a strong relationship with deaths; but that at the start of the pandemic - when less testing was being performed - the relationship does not look as strong.

{{< image src="/post_images/upweight-recent-observations/dashboard_snip.png" alt="Image of government dashboard of positive tests and deaths" caption="Government Dashboard of positive tests and deaths OGLv3.0." attrlink="https://coronavirus.data.gov.uk/" attr="Link to dashboard.">}}


This suggests a pattern that has changed through time: this is something where we want to give more weight to new observations.

To investigate this, let's first fetch both the datasets and combine them based on the date:


{{< highlight python >}}
import pandas as pd
# We'll need this later...
from sklearn.linear_model import LinearRegression

positive_tests = pd.read_csv("https://coronavirus.data.gov.uk/api/v1/data?filters=areaType=overview&structure=%7B%22areaType%22:%22areaType%22,%22areaName%22:%22areaName%22,%22areaCode%22:%22areaCode%22,%22date%22:%22date%22,%22newCasesBySpecimenDate%22:%22newCasesBySpecimenDate%22,%22cumCasesBySpecimenDate%22:%22cumCasesBySpecimenDate%22%7D&format=csv")
num_deaths = pd.read_csv("https://coronavirus.data.gov.uk/api/v1/data?filters=areaType=overview&structure=%7B%22areaType%22:%22areaType%22,%22areaName%22:%22areaName%22,%22areaCode%22:%22areaCode%22,%22date%22:%22date%22,%22newDeaths28DaysByDeathDate%22:%22newDeaths28DaysByDeathDate%22,%22cumDeaths28DaysByDeathDate%22:%22cumDeaths28DaysByDeathDate%22%7D&format=csv")
positive_tests['positive_tests'] = positive_tests['newCasesBySpecimenDate']
num_deaths['num_deaths'] = num_deaths['newDeaths28DaysByDeathDate']
covid_uk_df = num_deaths[['date', 'num_deaths']].merge(positive_tests[['date', 'positive_tests']], on='date')
covid_uk_df['date'] = pd.to_datetime(covid_uk_df['date'])
# Check there's not duplicates
assert len(set(covid_uk_df['date'])) == len(covid_uk_df['date'])
covid_uk_df
{{< / highlight >}}




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>num_deaths</th>
      <th>positive_tests</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2021-05-05</td>
      <td>8</td>
      <td>2229</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2021-05-04</td>
      <td>9</td>
      <td>2524</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2021-05-03</td>
      <td>6</td>
      <td>2145</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2021-05-02</td>
      <td>9</td>
      <td>1502</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2021-05-01</td>
      <td>8</td>
      <td>1388</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>425</th>
      <td>2020-03-06</td>
      <td>0</td>
      <td>81</td>
    </tr>
    <tr>
      <th>426</th>
      <td>2020-03-05</td>
      <td>3</td>
      <td>50</td>
    </tr>
    <tr>
      <th>427</th>
      <td>2020-03-04</td>
      <td>0</td>
      <td>56</td>
    </tr>
    <tr>
      <th>428</th>
      <td>2020-03-03</td>
      <td>2</td>
      <td>56</td>
    </tr>
    <tr>
      <th>429</th>
      <td>2020-03-02</td>
      <td>1</td>
      <td>39</td>
    </tr>
  </tbody>
</table>
<p>430 rows × 3 columns</p>
</div>



## Step 2: Adding weights

Now we need to decide how to weight the observations. There are lots of options here, which is a bit confusing.

I tend to use the same weights as used in [exponential smoothing models](https://otexts.com/fpp3/ses.html). Why? Exponential smoothing is a popular, simple forecasting method that has been around for over 60 years! It's tried and tested, and has not gone away.

Let's set $T$ to be the time of the most recent observation, $t$ to be the time of the observation we're interested in, and $\gamma$ to be a hyperparameter we pick that's between 0 and 1. Then I set my weights $w$ to be

\begin{equation}
w = \gamma^{[T-t]}.
\end{equation}

What does this mean? Suppose we set $\gamma = 0.95$. Then if my observation is made at the most recent time point, its weight will be 1. If it's made at the second most recent time point, its weight will be 0.95. If it's made at the 10th most recent time point, its weight will be $\gamma^{10} = 0.95^{10} \approx 0.6$. Essentially our weight smoothly decreases to nothing as the observation gets older and older.

An unfortunate side effect to this is we've added a hyperparameter to tune. I often just quickly do this by eye looking at the data, but you can also tune this in the normal way using cross-validation or AIC/BIC. For this tutorial, I'll just set it to 0.9.

Let's add weights to our data now:


```python
# Set hyperparam
gamma = 0.9
most_recent_date = covid_uk_df['date'].max()
days_before_recent_date = (most_recent_date - covid_uk_df['date']).dt.days
covid_uk_df['weight'] = gamma ** days_before_recent_date.values
covid_uk_df
```



{{< rawhtml >}}
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>num_deaths</th>
      <th>positive_tests</th>
      <th>weight</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2021-05-05</td>
      <td>8</td>
      <td>2229</td>
      <td>1.000000e+00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2021-05-04</td>
      <td>9</td>
      <td>2524</td>
      <td>9.000000e-01</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2021-05-03</td>
      <td>6</td>
      <td>2145</td>
      <td>8.100000e-01</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2021-05-02</td>
      <td>9</td>
      <td>1502</td>
      <td>7.290000e-01</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2021-05-01</td>
      <td>8</td>
      <td>1388</td>
      <td>6.561000e-01</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>425</th>
      <td>2020-03-06</td>
      <td>0</td>
      <td>81</td>
      <td>3.573276e-20</td>
    </tr>
    <tr>
      <th>426</th>
      <td>2020-03-05</td>
      <td>3</td>
      <td>50</td>
      <td>3.215948e-20</td>
    </tr>
    <tr>
      <th>427</th>
      <td>2020-03-04</td>
      <td>0</td>
      <td>56</td>
      <td>2.894353e-20</td>
    </tr>
    <tr>
      <th>428</th>
      <td>2020-03-03</td>
      <td>2</td>
      <td>56</td>
      <td>2.604918e-20</td>
    </tr>
    <tr>
      <th>429</th>
      <td>2020-03-02</td>
      <td>1</td>
      <td>39</td>
      <td>2.344426e-20</td>
    </tr>
  </tbody>
</table>
<p>430 rows × 4 columns</p>
</div>
{{< /rawhtml >}}



## Step 3: Fit your model

You might think that fitting our model will become a challenge now we've added weights. It's actually dead easy.

Most regression and classification algorithms allow you to provide a dataset weight: for tree based methods (sklearn random forest, xgboost, lightgbm), you just set the `sample_weight` in the `fit` function; for linear regression R's `lm` function has a `weights` argument, sklearn's `LinearRegression` has a `sample_weight` argument in the `fit` function.

If your algorithm does not allow you to set a weight, you can borrow from the class imbalance techniques and [oversample/undersample](https://machinelearningmastery.com/random-oversampling-and-undersampling-for-imbalanced-classification/) your observations based on your weights.

Let's fit our model using sklearn's linear regression:


```python
weighted_model = LinearRegression()
weighted_results = weighted_model.fit(
    X=covid_uk_df[['positive_tests']],
    y=covid_uk_df['num_deaths'], 
    sample_weight=covid_uk_df['weight']
)
# Return the R^2 score for our model
r2_weighted = weighted_results.score(
    X=covid_uk_df[['positive_tests']],
    y=covid_uk_df['num_deaths'], 
    sample_weight=covid_uk_df['weight']
)
f"R^2 score for the weighted model is {r2_weighted}"
```




    'R^2 score for the weighted model is 0.5247115256308429'



Let's compare this to an unweighted model:


```python
unweighted_model = LinearRegression()
unweighted_results = weighted_model.fit(
    X=covid_uk_df[['positive_tests']],
    y=covid_uk_df['num_deaths']
)
# Return the R^2 score for our model
r2_unweighted = unweighted_results.score(
    X=covid_uk_df[['positive_tests']],
    y=covid_uk_df['num_deaths']
)
f"R^2 score for the unweighted model is {r2_unweighted}"
```




    'R^2 score for the unweighted model is 0.39299060569674615'



In this case we've improved our in-sample fit significantly. Since regular testing is likely to continue in the UK, our weighted model should provide better out-of-sample predictions as well.
