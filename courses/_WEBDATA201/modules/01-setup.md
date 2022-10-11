---
layout: module
show-in-nav-bar: true
short-title: "Mod. 1"
title:  "Setup Instructions"
show-objectives: false
objectives:
- How to setup for the course
show-in-outline: true
outline-number: 1
---

We will be working from VM's to ensure everyone is in the same environment. We will be using browsers.

## Part one: Understanding the web.

For the first half of the lesson, we will use browser capabilities to investigate how web pages are built.

* I will be working in Firefox, but similar capabilities are available in Chrome and Safari.

## Part two: Write Python programs to automatically scrape data

### Shell and Python

We will be using VM's, but if you wanted to continue practicing after this week there are some instruction for how to get running below.

The second part of the lesson requires the Python programming language (Version 3.5 or greater) and access to a command-line interface (shell) on your computer. Please refer to the [Software Carpentry setup instructions](http://swcarpentry.github.io/workshop-template/#setup) for the Bash shell and Python if you need guidance.

## Prerequisites

This part of the lesson requires some prior knowledge of Python and how to use a shell. If you need help getting started on those topics, we suggest going through the following lessons first (during a workshop or on your own):

* [The Unix Shell](http://swcarpentry.github.io/shell-novice/)
* [Programming with Python](http://swcarpentry.github.io/python-novice-inflammation/)

### Scrapy

Once you have a working installation of Python 3, the next step is to install [Scrapy](https://scrapy.org/).

If you have installed Python using the Anaconda framework as suggested by the Software Carpentry setup instructions, you can easilly install Scrapy by doing the following:

1. Open a new shell (e.g. Terminal on Mac, or the Anaconda command-line tool on Windows)
2. Type the following:

> conda install -c conda-forge scrapy

Alternatively, if you have another distribution of Python, you can try using pip (or pip3 if you’re on ubuntu):

> pip install scrapy

If you run into issues while installing Scrapy, refer to the [official Scrapy install guide](https://doc.scrapy.org/en/latest/intro/install.html#intro-install) or get in touch with your eRA.
