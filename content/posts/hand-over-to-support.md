+++ 
draft = false
date = 2021-04-17T15:36:34+01:00
title = "How to move your data science solution into a hands off state"
description = "Once you've developed a data science solution it's often easy to tinker with it indefinitely. This is not ideal - you want to free up your time to develop new solutions and give further value to a business. In this post I give some tips to make a data science solution easier to leave in a hands off state."
slug = "data-science-solution-to-hands-off-state"
authors = ["Jack Baker"]
tags = ["data science", "development"]
+++


{{< image src="/post_images/hands-off/relaxed.jpg" alt="A person relaxing next to a lake" caption="Photo by " attrlink="https://unsplash.com/@simonmigaj?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText" attr="S Migaj on Unsplash.">}}


Once you've developed a data science solution it's easy to tinker with it indefinitely: running it manually, making small changes to the output, iterating on the model. This is not ideal - once you have a solution you're happy with ideally you should be able to leave it running with minimal input from you, unless something falls over. 

Whether you're handing over to a support team, or will be supporting the tool yourself, this post gives some tips when you're trying to move your solution into a hands-off state - or putting it into production.

Firstly you want to be logging. Logging is where output from your code is printed to a file so you (or the support team) can refer to it later. To move your code into a hands-off state you want to have logs that save to a file, so you can inspect these later if something goes wrong with the tool. 

What should you be logging? Any obvious things that might go wrong with the tool, such as data validation input, or the tool not being able to connect to a web service, etc. should be flagged as a warning or error and logged. Another good rule of thumb is to flag any 'transaction'. What I mean by this is things like reading files, connecting to a database or calling an API. These are all things that can fail and help serve as a rough guide for where your code failed and what likely went wrong. It's a good idea to have a summary at the end of a log indicating if the tool finished successfully, and if it failed, what went wrong. A log summary is really appreciated by support teams.

Another useful tool is alerting. This is where your tool notifies you (or the support team) when something went wrong or failed. One of the most effective ways of doing this is to get your tool to message you on slack or MS teams when something's gone wrong. This is very easy to do using webhooks. You can also use email for this e.g. using AWS SNS. It's a good idea to notify you when something's gone wrong, as well as when the tool is started and finished; then absence of these alerts will also tell you something is wrong.

Triage tables are invaluable, especially if you are handing over to a support team or someone is keeping an eye on the tool while you're away. Triage tables list the most common things that can go wrong with the tool, as well as the team that should be contacted to resolve this issue. For example if your tool relies on a data feed that's managed by another team; if this breaks you can request support to contact that team directly. That way the issue can be resolved without you acting as an intermediary which wastes time for everyone. The act of doing this also forces you to think carefully about your tool and what error handling should be in place, to make sure it's clear from the logs what went wrong.

Finally, you have to make sure you have a business's best interests at heart. It's easy as a data scientist to make constant iterations to a model, or to agree to every request a customer asks for. But at the end of the day, your job is to provide as much value to a business as possible. You want a model to be effective, but you don't need it to be perfect, after a point you can probably provide more value working on a different solution. Likewise, it's imperative to listen to what the customer wants, but after a time of ensuring you have met customer requirements, you might be able to provide better value by building the customer a different tool.