+++ 
draft = false
date = 2021-04-06T18:29:47+01:00
title = "How to include libraries in your project while also editing them"
description = "How to include libraries in your codebase when you also want to make changes to that library."
authors = ["Jack Baker"]
tags = ["development", "coding", "python"]
+++

I came across this issue the other day: I wanted to include a library in a project, but to make it work in my code I would have to edit it. I then needed to write those changes back to the original library, with minimal faff. This post explains the best way I found of doing this.

An obvious solution is to include the whole codebase of the library into your project. This will allow you to make the required changes. This can be quite messy though: your repo can become very large if you're working with a big library, and it's a pain to push your changes back to the original project.

I found a better way of doing this: [`git submodule`](https://git-scm.com/book/en/v2/Git-Tools-Submodules). `git submodule` allows you to manage additional repos within your project. Using it, the submodule repo is contained in a directory in your own project. By going into the directory that contains that repo you can make changes to that module using the standard git workflow, and push them back to the original library. 

When you push your own project repo, it will just contain a reference to the libraries repo location, and a commit-id. This keeps it small and concise. A great tutorial on how `git submodule` works is [here](https://git-scm.com/book/en/v2/Git-Tools-Submodules).

There's a big push for reusability in code at the moment. This is a great way to keep your code as small reusable components rather than a big monolith.