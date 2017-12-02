---
layout: post
title: Install Anaconda in Win10
categories: []
tags: [Python]
comments: true
---

If you want to use NumPy on Windows, Anaconda, which is one of the most popular python data science platforms, is a good choice.

Download it from [Anaconda's website](https://www.anaconda.com/download/).

Install it by default.

Set the environment variables. Add `[your install path of Anaconda]` and `[your install path of Anaconda]\Scripts` to your *path*, in order to run python and conda in your command line. 

Update the environment. Run `conda update --prefix [your install path of Anaconda] anaconda` in command line.

You can check if it's right with script below
```python
import numpy
numpy.__version__
``` 

