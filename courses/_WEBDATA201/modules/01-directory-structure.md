---
layout: module
show-in-nav-bar: true
short-title: "Mod. 1"
title:  "Setup"
show-objectives: false
objectives:
- How to setup for the course
show-in-outline: true
outline-number: 1
---

# Setup Instructions

In order to follow this lesson, you will need to make sure the following software is installed on your computer.

## Part one: Manually scrape data using browser extensions

For the first half of the lesson, we will use a Chrome browser extension to get started with web scraping.

* Please ensure you have a working copy of the Chrome browser.
* Using Chrome, download and enable the Scraper extension.

## Part two: Write Python programs to automatically scrape data

### Shell and Python

The second part of the lesson requires the Python programming language (Version 3.5 or greater) and access to a command-line interface (shell) on your computer. Please refer to the Software Carpentry setup instructions for the Bash shell and Python if you need guidance.

## Prerequisites

    This part of the lesson requires some prior knowledge of Python and how to use a shell. If you need help getting started on those topics, we suggest going through the following lessons first (during a workshop or on your own):

        The Unix Shell
        Programming with Python

### Scrapy

Once you have a working installation of Python 3, the next step is to install Scrapy.

If you have installed Python using the Anaconda framework as suggested by the Software Carpentry setup instructions, you can easilly install Scrapy by doing the following:

    Open a new shell (e.g. Terminal on Mac, or the Anaconda command-line tool on Windows)
    Type the following:

    conda install -c conda-forge scrapy

Alternatively, if you have another distribution of Python, you can try using pip (or pip3 if you’re on ubuntu):

    pip install scrapy

If you run into issues while installing Scrapy, refer to the official Scrapy install guide or get in touch with your lesson instructor.
