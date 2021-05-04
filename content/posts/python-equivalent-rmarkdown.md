+++ 
draft = false
date = 2021-05-04T20:20:00
title = "A python equivalent for R markdown"
description = "R markdown is a powerful tool for sharing insights with stakeholders. In this post I talk about what I use when I need an equivalent solution in python."
slug = "python-equivalent-rmarkdown"
authors = ["Jack Baker"]
tags = ["data science", "development", "insights", "python", "R"]
+++

[R markdown](https://rmarkdown.rstudio.com/) is a powerful tool for sharing insights with stakeholders. You can write snippets of R code that generate plots. This can then be compiled to a HTML or pdf file that you can share with non-technical stakeholders.

This is not as straightforward in python. Yes, Jupyter notebooks are a great way of sharing analysis with other developers. But compiling to HTML/pdf, with code snippets removed, that looks nice enough for a non-technical stakeholder, I've found clunky using Jupyter notebooks. R markdown also has great tools to generate [nice looking tables](https://bookdown.org/yihui/rmarkdown-cookbook/kable.html), not just plots.

I've been working with a python heavy team though, so have been trying to figure out how to generate R markdown style documents. In this post I'll outline what I've been using to generate HTML reports in python, that look nice enough to share with non-technical stakeholders. The process uses a few tools.


## Step 1: Embedding Plots in HTML

The first step is to embed plots into a static HTML that you can then share with others. A great tool for this is [plotly](https://plotly.com/python/). Plotly has a [`to_html` function](https://plotly.com/python-api-reference/generated/plotly.io.to_html.html) (one of my amazing colleagues found this) which will write the plots as a HTML string, which you can then write to a file.

Plotly graphs look the part, and they allow the user to hover over the points to see what the values are; I've found this goes down well with customers.

Sometimes users need plots they can copy-paste though. In this case I recommend using python's more standard plotting libraries, like [seaborn](https://seaborn.pydata.org/) or [matplotlib](https://matplotlib.org/). To embed the image into HTML without needing to have a separate image file the HTML references is a bit more involved. But you can do it by encoding the image as base64 and writing directly to your HTML. Just follow the instructions in this [stackoverflow post](https://stackoverflow.com/questions/48717794/matplotlib-embed-figures-in-auto-generated-html).


## Step 2: Setting the Layout of the HTML Document

Great, now you can embed plots in HTML, and actually if you turn off the `full_html` option in plotly's `to_html` function, you can write as many plotly plots as you like to a html file by just appending the strings.

But what about layout, commentary, and tables of data? This is where I use python's HTML templating libraries -- which allow you to use python to generate static HTML files from templates. This technique is powerful, and used in python backend development libraries like [flask](https://flask.palletsprojects.com/en/1.1.x/) and [django](https://www.djangoproject.com/). It's quick to learn, but does require some knowledge of HTML.

I use the [Jinja](https://jinja.palletsprojects.com/en/2.11.x/) library to generate my HTML reports (it's one of the most popular HTML templating libraries in python).


## Step 3: Making the Report Look Nice

For anyone that's worked with HTML before you'll know it takes a long time to make anything which looks presentable.

I didn't want to spend much time on this because I wanted to generate the report as quick as possible. So I used the [bootstrap](https://getbootstrap.com/) CSS library. This tool allows you to make your report look nice in a short amount of time (all I tend to do is wrap everything in a `<div class="container">` and maybe add some padding).

Bootstrap is basically a set of prebuilt HTML classes you can use to format your HTML. For example to make an HTML table look nice in bootstrap it's as simple as `<table class="table">` (tables look quite ugly in standard HTML). You can add padding at the top of a title element by adding `<h1 class="pt-1">`.


## Thoughts

As you can see, this is a fiddlier process than with R markdown (if anyone has a better way, please get in touch!). But the tools are useful to learn, especially Jinja and bootstrap which are standard tools for web development. Once you've learnt the libraries, and have some pre-made templates ready to go, the process gets quite quick.