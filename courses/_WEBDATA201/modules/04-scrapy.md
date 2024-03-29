---
layout: module
show-in-nav-bar: true
short-title: "Mod. 4"
title: "Web scraping using Python and Scrapy"
questions:
- "How can scraping a web site be automated?"
- "How can I setup a scraping project using the Scrapy framework for Python?"
- "How do I tell Scrapy what elements to scrape from a webpage?"
- "How do I tell Scrapy to follow URLs and scrape their contents?"
- "What to do with the data extracted with Scrapy?"
objectives:
- "Setting up a Scrapy project."
- "Understanding the various elements of a Scrapy projects."
- "Creating a spider to scrape a website and extract specific elements."
- "Creating a two-step spider to first extract URLs, visit them, and scrape their contents."
- "Storing the extracted data."
keypoints:
- "Scrapy is a Python framework that can be use to scrape content from the web."
- "A Scrapy project is a set of configuration files and pieces of code that tell Scrapy what to do."
- "In Scrapy, a \"Spider\" is the code that tells it what to do on a specific website."
- "A Scrapy project can have more than one spider but needs at least one."
- "With Scrapy, we can use XPath, CSS selectors and Regular Expressions to define what elements to scrape from a page."
- "Extracted data can be stored in \"Item\" objects. Such objects must be defined before they can be used."
- "Scrapy will automatically stored extracted data in CSS, JSON or XML format based on the file extension given in the -o option."
show-in-outline: true
outline-number: 4
---

## Recap
Here is what we have learned so far:

* We can use XPath queries to select what elements on a page to scrape.
* We can look at the HTML source code of a page to find how target elements are structured and
  how to select them.
* We can use the browser console and the `$x(...)` function to try out XPath queries on a live site.
* We can use the Scraper browser extension to scrape data from a single web page. Its interface even
  tries to guess the XPath query to target the elements we are interested in.

This is quite a toolset already, and it's probably sufficient for a number of use cases, but there are
limitations in using the tools we have seen so far. Scraper requires manual intervention and only scrapes
one page at a time. Even though it is possible to save a query for later, it still requires us to operate
the extension.

## Introducing Scrapy

Enter [Scrapy](https://scrapy.org/)! Scrapy is a _framework_ for the [Python](https://swcarpentry.github.io/python-novice-inflammation/)
programming language.

>
> A framework is a reusable, "semi-complete" application that can be specialized to produce custom applications.
> (Source: [Johnson & Foote, 1988](http://www1.cse.wustl.edu/~schmidt/CACM-frameworks.html))
>

In other words, the Scrapy framework provides a set of Python scripts that contain most of the code required
to use Python for web scraping. We need only to add the last bit of code required to tell Python what
pages to visit, what information to extract from those pages, and what to do with it. Scrapy also
comes with a set of scripts to setup a new project and to control the scrapers that we will create.

It also means that Scrapy doesn't work on its own. It requires a working Python installation
(Python 2.7 and higher or 3.4 and higher - it should work in both Python 2 and 3), and a series of
libraries to work. If you haven't installed Python or Scrapy on your machine, you can refer to the
[setup instructions](/lc-webscraping/setup). If you install Scrapy as suggested there, it should take care to install all
required libraries as well.

You can verify that you have the latest version of Scrapy installed by typing

~~~
scrapy version
~~~
{: .source}

in a shell. If all is good, you should get the following back (as of February 2017):

~~~
Scrapy 2.3.0
~~~
{: .output}

If you have a newer version, you should be fine as well.

To introduce the use of Scrapy, we will use the a new example to what we used in the previous
section. We will start by scraping a list of URLs from [the list of senators from the Australian Parliament](https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results) and then visit those URLs to
scrape [detailed information](https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=R36)
about those ministers.

## Setup a new Scrapy project

The first thing to do is to create a new Scrapy project.

Let's navigate first to a folder on our drive where we want to create our project (refer to Software
Carpentry's lesson about the [UNIX shell](http://swcarpentry.github.io/shell-novice/) if you are
unsure about how to do that). Then, type the following

~~~
scrapy startproject au_gov
~~~
{: .source}

where `au_gov` is the name of our project.

Scrapy should respond will something similar to (the paths will reflect your own file structure)

~~~
New Scrapy project 'au_gov', using template directory '/Users/thomas/anaconda/lib/python3.5/site-packages/scrapy/templates/project', created in:
    /Users/thomas/Documents/Computing/python-projects/scrapy/au_gov


You can start your first spider with:
    cd au_gov

    scrapy genspider example example.com
~~~
{: .output}

If we list the files in the directory we ran the previous command

~~~
ls -F
~~~
{: .source}

we should see that a new directory was created:

~~~
au_gov/
~~~
{: .output}

(alongside any other files and directories you had lying around previously). Moving into that new
directory

~~~
cd au_gov
~~~
{: .source}

we can see that it contains two items:

~~~
ls -F
~~~
{: .source}

~~~
au_gov/	scrapy.cfg
~~~
{: .output}

Yes, confusingly, Scrapy creates a subdirectory called `au_gov` within the `au_gov` project
directory. Inside that _second_ directory, we see a bunch of additional files:

~~~
ls -F au_gov
~~~
{: .source}

~~~
__init__.py	items.py	settings.py
__pycache__	pipelines.py	spiders/
~~~
{: .output}

To recap, here is the structure that `scrapy startproject` created:

~~~
au_gov/			# the root project directory
	scrapy.cfg		# deploy configuration file

	au_gov
/		# project's Python module, you'll import your code from here
		__init__.py

		items.py		# project items file

		pipelines.py	# project pipelines file

		settings.py	# project settings file

		spiders/		# a directory where you'll later put your spiders
			__init__.py
			...
~~~
{: .output}

We will introduce what those files are for in the next paragraphs. The most important item is the
`spiders` directory: this is where we will write the scripts that will scrape the pages we
are interested in. Scrapy calls such scripts _spiders_.

## Creating a spider

Spiders are the business end of the scraper. It's the bit of code that combs through a website and harvests data.
Their general structure is as follows:
* One or more _start URLs_, where the spider will start crawling
* A list of _allowed domains_ to constrain the pages we allow our spider to crawl (this is a good way to
  avoid mistakenly writing an out-of-hand spider that mistakenly starts crawling the entire Internet...)
* A method called `parse` in which we will write what data the spider should be looking for on the pages
  it visits, what links to follow and how to parse found data.

To create a spider, Scrapy provides a handy command-line tool:

~~~
scrapy genspider <SCRAPER NAME> <START URL>
~~~
{: .source}

We just need to replace `<SCRAPER NAME>` with the name we want to give our spider and `<START URL>` with
the URL we want to spider to crawl. In our case, we can type:

~~~
scrapy genspider au_gov_sen www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results
~~~
{: .source}

This will create a file called `au_gov_sen.py` inside the `spiders` directory of our project.
Using our favourite text editor, let's open that file. It should look something like this:

~~~
import scrapy

class AuGovSenSpider(scrapy.Spider):
    name = "au_gov_sen"  # The name of this spider
	
    # The allowed domain and the URLs where the spider should start crawling:
    allowed_domains = ["www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results"]
    start_urls = ['http://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results/']
	
    # And a 'parse' function, which is the main method of the spider. The content of the scraped
    # URL is passed on as the 'response' object:
    def parse(self, response):
        pass
~~~
{: .source}

Note that here some comments have been added for extra clarity, they will not be there upon
first creating a spider.

> ## Don't include `http://` when running `scrapy genspider`
>
> The current version of Scrapy (1.3.2 - February 2017) apparently only expects URLs without
> `http://` when running `scrapy genspider`. If you do include the `http` prefix, you might
> see that the value in `start_url` in the generated spider will have that prefix twice, because
> Scrapy appends it by default. This will cause your spider to fail. Either run `scrapy genspider`
> without `http://` or check the resulting spider so that it looks like the code above.
>
{: .callout}

> ## Object-oriented programming and Python classes
>
> You might be unfamiliar with the `class AuGovSenSpider(scrapy.Spider)` syntax used above.
> This is an example of [Object-oriented programming](https://en.wikipedia.org/wiki/Object-oriented_programming).
>
> All elements of a piece of Python code are __objects__: functions, variables, strings, integers, etc.
> Objects of a certain type have certain things in common. For example, it is possible to apply special
> functions to all strings in Python by using syntax such as `mystring.upper()` (this will make the contents
> of `mystring` all uppercase).
>
> We call these types of objects __classes__. A class defines the components of an object (called __attributes__),
> as well as specific functions, called __methods__, we might want to run on those objects.
> For example, we could define a class called `Pet` that would contain the attributes `name`, `colour`, `age` etc.
> as well as the methods `run()` or `cuddle()`. Those are common to all pets.
>
> We can use the Object-oriented paradigm to describe a specific type of pet: `Dog` would __inherit__ the
> attributes and methods of `Pet` (dogs have names and can run and cuddle) but would __extend__ the `Pet` class
> by adding dog-specific things like a `pedigree` attribute and a `bark()` method.
>
> The code in the example above is defining a __class__ called `AuGovSenSpider` that __inherits__ the `Spider` class
> defined by Scrapy (hence the `scrapy.Spider` syntax). We are __extending__ the default `Spider` class by defining
> the `name`, `allowed_domains` and `start_urls` attributes, as well as the `parse()` method.
>
{: .discussion}

> ## The `Spider` class
>
> A `Spider` class will define how a certain site (or a group of sites, defined in `start_urls`) will be scraped,
> including how to perform the crawl (i.e. follow links) and how to extract structured data from their pages
> (i.e. scraping items) in the `parse()` method.
>
> In other words, Spiders are the place where we define the custom behaviour for crawling and parsing
> pages for a particular site (or, in some cases, a group of sites).
>
{: .callout}

Once we have the spider open in a text editor, we can start by cleaning up a little the code that Scrapy
has automatically generated.

> ## Paying attention to `allowed_domains`
>
> Looking at the code that was generated by `genspider`, we see that by default the entire start URL
> has ended up in the `allowed_domains` attribute.
>
> Is this desired? What do you think would happen
> if later in our code we wanted to scrape a page living at the address `www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=R36`?
> > ## Solution
> >
> > `allowed_domains` is a safeguard for our spider, it will restrict its ability to scrape pages
> > outside of a certain realm. An URL is structured as a path to a resource, with the root directory
> > at the beginning and a set of "subdirectories" after that. In `www.mydomain.ca/house/dog.html`,
> > `http://www.mydomain.ca/` is the root, `house/` is a first level directory and `dog.html` is a file
> > sitting inside the `house/` directory.
> >
> > If we restrict a Scrapy spider with `allowed_domains = ["www.mydomain.ca/house"]`, it means
> > that the spider will be able to scrape everything that's inside the `www.mydomain.ca/house/` directory (including
> > subdirectories), but not, say, pages that would be in `www.mydomain.ca/garage/`. However,
> > if we set `allowed_domains = ["www.mydomain.ca/"]`, the spider can scrape both the contents of
> > the `house/` and `garage/` directories.
> >
> > To answer the question, leaving `allowed_domains = ["www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results"]`
> > would restrict the spider to pages with URLs of the same pattern, and
> > `www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=R36`
> > if of a different pattern, so Scrapy would prevent the spider from scraping it.
> >
> {: .solution}
>
> How should `allowed_domains` be set to prevent this from happening?
>
> > ## Solution
> >
> > We should let the spider scrape all pages inside the `www.aph.gov.au` domain by editing
> > it so that it reads:
> >
> > ~~~
> > allowed_domains = ["www.aph.gov.au"]
> > ~~~
> > {: .source}
> >
> {: .solution}
>
{: .challenge}

Here is what the spider looks like after cleaning the code a little:

(editing `au_gov/au_gov/spiders/au_gov_sen.py`)

~~~
import scrapy

class AuGovSenSpider(scrapy.Spider):
    name = "au_gov_sen"
	
    allowed_domains = ["www.aph.gov.au"]
    start_urls = ['http://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results/']

    def parse(self, response):
        pass
~~~
{: .source}

Don't forget to save the file once changes have been applied.

## Running the spider

Now that we have a first spider setup, we can try running it. Going back to the Terminal, we first make sure
we are located in the project's top level directory (where the `scrapy.cfg` file is) by using `ls`, `pwd` and
`cd` as required, then we can run:

~~~
scrapy crawl au_gov_sen
~~~
{: .source}

Note that we can now use the name we have chosen for our spider (`au_gov_sen`, as specified in the `name` attribute)
to call it. This should produce the following result

~~~
2016-11-07 22:28:51 [scrapy] INFO: Scrapy 1.3.2 started (bot: au_gov_sen)

(followed by a bunch of debugging output ending with:)

2022-10-12 05:18:58 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results/> (referer: None)
2022-10-12 05:18:59 [scrapy.core.engine] INFO: Closing spider (finished)
2022-10-12 05:18:59 [scrapy.statscollectors] INFO: Dumping Scrapy stats:
{'downloader/request_bytes': 980,
 'downloader/request_count': 4,
 'downloader/request_method_count/GET': 4,
 'downloader/response_bytes': 68447,
 'downloader/response_count': 4,
 'downloader/response_status_count/200': 2,
 'downloader/response_status_count/301': 2,
 'elapsed_time_seconds': 1.306556,
 'finish_reason': 'finished',
 'finish_time': datetime.datetime(2022, 10, 12, 5, 18, 59, 6296),
 'log_count/DEBUG': 4,
 'log_count/INFO': 10,
 'log_count/WARNING': 1,
 'memusage/max': 52572160,
 'memusage/startup': 52572160,
 'response_received_count': 2,
 'robotstxt/request_count': 1,
 'robotstxt/response_count': 1,
 'robotstxt/response_status_count/200': 1,
 'scheduler/dequeued': 2,
 'scheduler/dequeued/memory': 2,
 'scheduler/enqueued': 2,
 'scheduler/enqueued/memory': 2,
 'start_time': datetime.datetime(2022, 10, 12, 5, 18, 57, 699740)}
2022-10-12 05:18:59 [scrapy.core.engine] INFO: Spider closed (finished)

~~~
{: .output}

The line that starts with `DEBUG: Crawled (200)` is good news, as it tells us that the spider was
able to crawl the website we were after. The number in parentheses is the _HTTP status code_ that
Scrapy received in response of its request to access that page. 200 means that the request was successful
and that data (the actual HTML content of that page) was sent back in response.

However, we didn't do anything with it, because the `parse` method in our spider is currently empty.
Let's change that by editing the spider as follows (note the contents of the `parse` method):

(editing `au_gov/au_gov/spiders/au_gov_sen.py`)

~~~
import scrapy

class AuGovSenSpider(scrapy.Spider):
    name = "au_gov_sen"
    allowed_domains = ["www.aph.gov.au"]
    start_urls = ['http://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results/']

    def parse(self, response):
        with open("test.html", 'wb') as file:
            file.write(response.body)
~~~
{: .source}

Now, if we go back to the command line and run our spider again

~~~
scrapy crawl au_gov_sen
~~~
{: .source}

we should get similar debugging output as before, but there should also now be a file called
`test.html` in our project's root directory:

~~~
ls -F
~~~
{: .source}

~~~
au_gov/	scrapy.cfg	test.html
~~~
{: .output}

We can check that it contains the HTML from our target URL:

~~~
less test.html
~~~
{: .source}

~~~
<!doctype html>
(...)
<html class="no-js" lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<!--<![endif]-->

<head id="Head1">
(...)
<title>
        Senators & Members Search Results
        
     &ndash; Parliament of Australia
</title>
(...)_
~~~
{: .output}

## Defining which elements to scrape using XPath

Now that we know how to access the content of the [web page with the list of Australian Senators](https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results),
the next step is to extract the information we are interested in, in that case the URLs pointing
to the detail pages for each politician.

Using the techniques we have [learned earlier](/02-xpath), we can start by looking at
the source code for our [target page](https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results)
by using either the "View Source" or "Inspect" functions of our browser.
Here is an excerpt of that page:

~~~
(...)
<div class="search-filter-results search-filter-results-snm row">
        <div>
            <div class="row border-bottom padding-top">
                <div class="medium-push-2 medium-7 large-8  columns">
                    <h4 class="title">
                        <a href="/Senators_and_Members/Parliamentarian?MPID=R36">Hon Anthony Albanese MP</a>
                    </h4>
                    <dl class="dl--inline__result text-small">
                        <dt>For</dt>
                        <dd>Grayndler, New South Wales</dd>

                            <dt>Positions</dt>
<dd>Prime Minister</dd><dt>Party</dt>
                            <dd>Australian Labor Party</dd>

                            <dt class="snm-hide-links-content">Connect</dt>
                            <dd class="snm-hide-links-content">
(...)
~~~
{: .output}

There are different strategies to target the data we are interested in. One of them is to identify
that the URLs are inside `h4` elements of the class `title`. From there the URL is inside a `a` element.

We recall that the XPath syntax to access all such elements is `$x("//h4[@class='title']/a")`, which we can
try out in the browser console:

~~~
> $x("//h4[@class='title']/a")
~~~
{: .source}

> ## Selecting elements assigned to multiple classes
>
> The above XPath works in this case because the target `h4` elements are only assigned to the
> `title` class. It wouldn't work if those elements had more than one class, for example
> `<h4 class="title larger">`. The more general syntax to select elements that belong to
> the `title` class and potentially other classes as well is
>
> ~~~
> `//h4[contains(concat(" ", normalize-space(@class), " "), " title ")]`
> ~~~
> {: .source}
>
> This [comment on StackOverflow](http://stackoverflow.com/a/9133579) has more details on
> this issue.
>
{: .discussion}

Once we were able to confirm that we are targeting the right cells, we can expand our XPath query
to only select the `href` attribute of the URL:

~~~
> $x("//h4[@class='title']/a/@href")
~~~
{: .source}

This returns an array of objects:

~~~
<- Array(12) [ href="/Senators_and_Members/Parliamentarian?MPID=R36", href="/Senators_and_Members/Parliamentarian?MPID=298839", href="/Senators_and_Members/Parliamentarian?MPID=13050", href="/Senators_and_Members/Parliamentarian?MPID=290544", href="/Senators_and_Members/Parliamentarian?MPID=230886", href="/Senators_and_Members/Parliamentarian?MPID=269375", href="/Senators_and_Members/Parliamentarian?MPID=282237", href="/Senators_and_Members/Parliamentarian?MPID=281558", href="/Senators_and_Members/Parliamentarian?MPID=16913", href="/Senators_and_Members/Parliamentarian?MPID=300706", … ]
~~~
{: .output}

### Debugging using the Scrapy shell

As we learned in the previous section, using the browser console and the `$x()` syntax can be useful to make sure
we are selecting the desired elements using our XPath queries. But it is not the only way. Scrapy provides a similar
way to test out XPath queries, with the added benefit that we can then also debug how to further work on those
queries from within Scrapy.

This is achieved by calling the _Scrapy shell_ from the command line:

~~~
scrapy shell https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results
~~~
{: .source}

which launches a Python console that allows us to type live Python and Scrapy code to
interact with the page which Scrapy just downloaded from the provided URL. We can see that we are inside an
interactive python console because the prompt will have changed to `>>>`:

~~~
(similar Scrapy debug text as before)

2017-02-26 22:31:04 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.ontla.on.ca/web/members/members_current.do?locale=en/> (referer: None)
[s] Available Scrapy objects:
[s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
[s]   crawler    <scrapy.crawler.Crawler object at 0x7fab12ca5320>
[s]   item       {}
[s]   request    <GET https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results>
[s]   response   <200 https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results>
[s]   settings   <scrapy.settings.Settings object at 0x7fab12a8c7b8>
[s]   spider     <AuGovSenSpider 'au_gov_sen' at 0x7fab120b73c8>
[s] Useful shortcuts:
[s]   fetch(url[, redirect=True]) Fetch URL and update local objects (by default, redirects are followed)
[s]   fetch(req)                  Fetch a scrapy.Request and update local objects
[s]   shelp()           Shell help (print this help)
[s]   view(response)    View response in a browser
>>>
~~~
{: .output}

We can now try running the XPath query we just devised against the `response` object, which in Scrapy
contains the downloaded web page:

~~~
>>> response.xpath("//h4[@class='title']/a/@href")
~~~
{: .source}

This will return a bunch of `Selector` objects (one for each URL found):

~~~
[<Selector xpath="//h4[@class='title']/a/@href" data='/Senators_and_Members/Parliamentarian...'>,
 <Selector xpath="//h4[@class='title']/a/@href" data='/Senators_and_Members/Parliamentarian...'>,
 ...]
>>>
~~~
{: .output}

Those objects are pointers to the different element in the scraped page (`href` attributes) as
defined by our XPath query. To get to the actual content of those elements (the text of the URLs),
we can use the `extract()` method. A variant of that method is `extract_first()` which does the
same thing as `extract()` but only returns the first element if there are more than one:

~~~
>>> response.xpath("//h4[@class='title']/a/@href").extract_first()
~~~
{: .source}

returns

~~~
'/Senators_and_Members/Parliamentarian?MPID=R36'
>>>
~~~
{: .output}

> ## Dealing with relative URLs
>
> Looking at this result and at the source code of the page, we realize that the URLs are all
> _relative_ to that page. They are all missing part of the URL to become _absolute_ URLs, which
> we will need if we want to ask our spider to visit those URLs to scrape more data. We could
> prefix all those URLs with `https://www.aph.gov.au/` to make them absolute, but
> since this is a common occurence when scraping web pages, Scrapy provides a built-in function
> to deal with this issue.
>
> To try it out, still in the Scrapy shell, let's first store the first returned URL into a
> variable:
>
> ~~~
> >>> testurl = response.xpath("//h4[@class='title']/a/@href").extract_first()
> ~~~
> {: .source}
>
> Then, we can try passing it on to the `urljoin()` method:
>
> ~~~
> >>> response.urljoin(testurl)
> ~~~
> {: .source}
>
> which returns
>
> ~~~
> 'https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=R36'
> ~~~
> {: .output}
>
> We see that Scrapy was able to reconstruct the absolute URL by combining the URL of the current page context
> (the page in the `response` object) and the relative link we had stored in `testurl`.
>
{: .callout}


## Extracting URLs using the spider

Armed with the correct query, we can now update our spider accordingly. The `parse`
methods returns the contents of the scraped page inside the `response` object. The `response`
object supports a variety of methods to act on its contents:

|Method|Description|
|-----------------|:-------------|
|`xpath()`| Returns a list of selectors, each of which points to the nodes selected by the XPath query given as argument|
|`css()`| Works similarly to the `xpath()` method, but uses CSS expressions to select elements.|

Those methods will return objects of a different type, called `selectors`. As their name implies,
these objects are "pointers" to the elements we are looking for inside the scraped page. In order
to get the "content" that the `selectors` are pointing to, the following methods should be used:

|Method|Description|
|-----------------|:-------------|
|`extract()`| Returns the entire contents of the element(s) selected by the `selector` object, as a list of strings.|
|`extract_first()`| Returns the content of the first element selected by the `selector` object.|
|`re()`| Returns a list of unicode strings within the element(s) selected by the `selector` object by applying the regular expression given as argument.|
|`re_first()`| Returns the first match of the regular expression|

> ## Know when to use `extract()`
> The important thing to remember is that `xpath()` and `css()` return `selector` objects, on which it
> is then possible to apply the `xpath()` and `css()` methods a second time in order to further refine
> a query. Once you've reached the elements you're interested in, you need to call `extract()` or
> `extract_first()` to get to their contents as string(s).
>
> Whereas `re()` returns a list of strings, and therefore it is no longer possible to apply
> `xpath()` or `css()` to the results of `re()`. Since it returns a string, you don't need to
> use `extract()` there.
>
{: .callout}

Since we have an XPath query we know will extract the URLs we are looking for, we can now use
the `xpath()` method and update the spider accordingly:

(editing `au_gov/au_gov/spiders/au_gov_sen.py`)

~~~
import scrapy

class AuGovSenSpider(scrapy.Spider):
    name = "au_gov_sen"
    allowed_domains = ["www.ontla.on.ca"]
    start\_urls = ['http://www.ontla.on.ca/web/members/members_current.do?locale=en/']

    def parse(self, response):
        for url in response.xpath("//h4[@class='title']/a/@href").extract():
            print(response.urljoin(url))
~~~
{: .source}

> ## Looping through results
>
> Why are we using `extract()` instead of `extract_first()` in the code above?
> Why is the `print` statement inside a `for` clause?
> > ## Solution
> >
> > We are not only interested in the first extracted URL but in all of them.
> > `extract_first()` only returns the content of the first in a series of
> > selected elements, while `extract()` will return all of them in the form of an
> > array.
> >
> > The `for` syntax allows us to loop through each of the returned elements one by one.
> >
> {: .solution}
{: .challenge}

We can now run our new spider:

~~~
scrapy crawl au_gov_sen
~~~
{: .source}

which produces a result similar to:

~~~
2022-10-12 23:38:01 [scrapy.utils.log] INFO: Scrapy 2.3.0 started (bot: au_gov)
(...)
https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=R36
https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=298839
https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=13050
https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=290544
(...)
2022-10-12 23:38:03 [scrapy.core.engine] INFO: Closing spider (finished)
~~~
{: .output}

We can now pat ourselves on the back, as we have successfully completed the first stage
of our project by successfully extracing all URLs leading to the minister profiles!

> ## Limit the number of URL to scrape through while debugging
>
> We've seen by testing the code above that we are able to successfully gather all URLs from
> the list of Parliament Members. But while we're working through to the final code that will allow us
> the extract the data we want from those pages, it's probably a good idea to only run it
> on a handful of pages at a time.
>
> This will not only run faster and allow us to iterate more quickly between different
> revisions of our code, it will also not burden the server too much while we're debugging.
> This is probably not such an issue for a couple of hundred of pages, but it's good
> practice, as it can make a difference for larger scraping projects. If you are planning
> to scrape a massive website with thousands of pages, it's better to start small. Other
> visitors to that site will thank you for respecting their legitimate desire to access
> it while you're debugging your scraper...
>
> An easy way to limit the number of URLs we want to send our spider to is to
> take advantage of the fact that the `extract()` method returns a list of matching elements.
> In Python, lists can be _sliced_ using the `list[start:end]` syntax and we can leave out
> either the `start` or `end` delimiters:
>
> ~~~
> list[start:end] # items from start through end-1
> list[start:]    # items from start through the rest of the array
> list[:end]      # items from the beginning through end-1
> list[:]         # all items
> ~~~
>
> We can therefore edit our spider thusly to only scrape the first five URLs:
>
> ~~~
> import scrapy
>
>    class AuGovSenSpider(scrapy.Spider):
>        name = "au_gov_sen"
>        allowed_domains = ["www.ontla.on.ca"]
>        start_urls = ['http://www.ontla.on.ca/web/members/members_current.do?locale=en/']
>
>        def parse(self, response):
>            for url in response.xpath("//*[@class='mppcell']/a/@href").extract()[:5]:
>                print(response.urljoin(url))
> ~~~
> {: .source}
>
> Note that this only works if there are at least five URLs that are being returned, which
> is the case here.
>
{: .callout}

## Recursive scraping

Now that we were successful in harvesting the URLs to the detail pages, let's begin by editing
our spider to instruct it to visit those pages one by one.

For this, let's begin by defining a new method `get_details` that we want to run on the detail pages:


(editing `au_gov/au_gov/spiders/au_gov_sen.py`)

~~~
import scrapy

class AuGovSenSpider(scrapy.Spider):
    name = "au_gov_sen" # The name of this spider

    # The allowed domain and the URLs where the spider should start crawling:
    allowed_domains = ["www.aph.gov.au"]
    start_urls = ['http://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results/']

    def parse(self, response):
        # The main method of the spider. It scrapes the URL(s) specified in the
        # 'start_url' argument above. The content of the scraped URL is passed on
        # as the 'response' object.

        for url in response.xpath("//h4[@class='title']/a/@href").extract()[:5]:
            # This loops through all the URLs found inside an element of class 'mppcell'
			
            # Constructs an absolute URL by combining the response’s URL with a possible relative URL:
            full_url = response.urljoin(url)
            print("Found URL: "+full_url)

            # The following tells Scrapy to scrape the URL in the 'full_url' variable
            # and calls the 'get_details() method below with the content of this
            # URL:
            yield scrapy.Request(full_url, callback=self.get_details)

    def get_details(self, response):
        # This method is called on by the 'parse' method above. It scrapes the URLs
        # that have been extracted in the previous step.
        print("Visited URL: "+response.url)
~~~
{: .source}

We've also added some comments to the code to make it easier to read and understand.

If we now run our spider again:

~~~
scrapy crawl au_gov_sen
~~~
{: .source}

We should see the result of our `print` statements intersped with the regular Scrapy
debugging output, something like:

~~~
(...)
Found URL: https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=13050
Found URL: https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=290544
Found URL: https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=230886
(...)
Visited URL: https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=R36
Visited URL: https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=13050
(...)
Visited URL: https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=290544
Visited URL: https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=298839
(...)
~~~
{: .output}

We've truncated the results above to make it easier to read, but on your console
you should see that all 5 URLs (remember, we are limiting the number of URLs to scrape
for now) have been first "found" (by the `parse()` method) and then "visited"
(by the `get_details()` method).

> ## Asynchronous requests
>
> If you look closely at the output of the code we've just run, you might be surprised
> to see that the "Found URL" and "Visited URL" statements didn't necessarily get
> printed out one after the other, as we might expect.
>
> The reason this is so is that Scrapy requests are [scheduled and processed asynchronously](http://stackoverflow.com/questions/748175/asynchronous-vs-synchronous-execution-what-does-it-really-mean).
> This means that Scrapy doesn’t need to wait for a request to be finished and processed
> before it runs another or do other things in the meantime. This is more efficient
> than running each request one after the other, and it also allows for Scrapy to keep
> working away even if some requests fails for whatever reason.
>
> This is especially advantageous when scraping large websites. Depending on the resources
> of the computer on which Scrapy runs, it can scrape hundreds or thousands of pages
> simultaneously.
>
> If you want to know more, the Scrapy documentation
> [has a page detailing how the data flows between Scrapy's components ](https://doc.scrapy.org/en/latest/topics/architecture.html#topics-architecture).
>
{: .callout}

## Scrape the detail pages

Now that we are able to visit each one of the detail pages, we should work on getting the
data that we want out of them. In our example, we are primarily looking
to extract the following details:

* Phone number(s)
* Mailing address(es)

Unfortunately, it looks like the content of those pages is not consistent. Sometimes, only
one email address is displayed, sometimes more than one. Some Parliament Members have one Constituency
address, others have more than one, etc.

To simplify, we are going to stop at the first phone number and the first
mailing address we find on those pages, although in a real life scenario we might be interested
in writing more precise queries to make sure we are collecting the right information.

> ## Scrape phone number and mailing address
> Write XPath queries to scrape the first phone number and the first mailing address
> displayed on each of the detail pages that are linked from
> the [Australian Parliament Members list](https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results).
>
> Try out your queries on a handful of detail pages to make sure you are getting
> consistent results.
>
> Tips:
>
> * Look at the source code and try out XPath your queries until you find what
>   you are looking for.
> * You can either use the browser console or the Scrapy shell mode (see above)
>   to try out your queries.
> * The syntax for selecting an element like `<div class="mytarget">` is `div[@class = 'mytarget']`.
> * The syntax to select the value of an attribute of the type `<element attribute="value">`
>   is `element/@attribute`.
>
> > ## Solution
> >
> > This returns an array of phone (and fax) numbers (using the Scrapy shell):
> >
> > ~~~
> > scrapy shell "https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=R36"
> > >>> response.xpath("//dd/a").extract()
> > ~~~
> > {: .source}
> >
> > ~~~
> > ['<a href="tel:+61295643588">(02) 9564 3588</a>',
> >  '<a href="tel:+61262777700">(02) 6277 7700</a>',
> >  '<a aria-label="Open personal website" href="http://www.anthonyalbanese.com.au" target="_blank" rel="noopener noreferrer">Personal website<i aria-hidden="true" class="fa fa-external-link"></i></a>',
> >  '<a aria-label="Open party website" href="http://www.alp.org.au/" target="_blank" rel="noopener noreferrer">Party website <i aria-hidden="true" class="fa fa-external-link"></i></a>',
> >  '<a aria-label="Open other website" href="http://www.pm.gov.au/" target="_blank" rel="noopener noreferrer">Prime Minister\'s website <i aria-hidden="true" class="fa fa-external-link"></i></a>',
> >  '<a aria-label="Open other website" href="http://www.peo.gov.au" target="_blank" rel="noopener noreferrer">Parliamentary Education Office <i aria-hidden="true" class="fa fa-external-link"></i></a>']
> > ~~~
> > {: .output}
> >
> > And this returns an array of mailing addresses:
> >
> > ~~~
> > >>> response.xpath("//*[contains(text(),'Postal')]/following-sibling::p").extract()
> > ~~~
> > {: .source}
> >
> > ~~~
> > ['<p>\r\n                                        PO Box 5100<br>\r\n                                        Marrickville, NSW, 2204\r\n                                        \r\n                                    </p>']
> > ~~~
> > {: .output}
> >
> {: .solution}
>
{: .challenge}

> ## Scraping using Regular Expressions
> In combination with XPath queries, it is also possible to use [Regular Expressions](https://en.wikipedia.org/wiki/Regular_expression)
> to scrape the contents of a web page.
>
> This is done by using the `re()` method. That method behaves a bit differently
> than the `xpath()` method in that it has to be applied on a `selector` object
> and returns an array of unicode strings (it is therefore not necessary to
> use `extract()` on its results).
>
> Using the Scrapy shell, try writing a query that selects all phone numbers found on
> a politician's detail page regardless of where they are located, using Regular Expressions.
>
> You might find the [Regex 101](https://regex101.com/) interactive Regular Expressions
> tester useful to get to the proper syntax.
>
> Tips:
>
> * We are looking for phone numbers following the North American syntax: ###-###-####
> * `re()` expects a regular expression string which should be prefixed by `r` as in `re(r'Name:\s*(.*)')`.
> * Remember that `re()` is run on a `selector` object, so you can't do `response.re(r'...')`. Instead you may
>   want to try doing something like `response.xpath('//body').re(r'...')`.
>
> > ## Solution
> >
> > This returns an array of phone (and fax) numbers (using the Scrapy shell):
> >
> > ~~~
> > scrapy shell "http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=7085"
> > >>> response.xpath('//body').re(r'\S\d{2}\S\s\d{4}\s\d{4}')
> > ~~~
> > {: .source}
> >
> > ~~~
> > ['(02) 9564 3588', '(02) 9564 1734', '(02) 6277 7700', '(02) 6273 4100']
> > ~~~
> > {: .output}
> >
> {: .solution}
>
{: .challenge}


Once we have found XPath queries to run on the detail pages and are happy with the result,
we can add them to the `get_details()` method of our spider:


(editing `au_gov/au_gov/spiders/au_gov_sen.py`)

~~~
import scrapy

class AuGovSenSpider(scrapy.Spider):
    name = "au_gov_sen" # The name of this spider

    # The allowed domain and the URLs where the spider should start crawling:
    allowed_domains = ["www.ontla.on.ca"]
    start_urls = ['http://www.ontla.on.ca/web/members/members_current.do?locale=en/']

    def parse(self, response):
        # The main method of the spider. It scrapes the URL(s) specified in the
        # 'start_url' argument above. The content of the scraped URL is passed on
        # as the 'response' object.

        for url in response.xpath("//*[@class='mppcell']/a/@href").extract()[:5]:
            # This loops through all the URLs found inside an element of class 'mppcell'
			
            # Constructs an absolute URL by combining the response’s URL with a possible relative URL:
            full_url = response.urljoin(url)
            print("Found URL: "+full_url)

            # The following tells Scrapy to scrape the URL in the 'full_url' variable
            # and calls the 'get_details() method below with the content of this
            # URL:
            yield scrapy.Request(full_url, callback=self.get_details)

    def get_details(self, response):
        # This method is called on by the 'parse' method above. It scrapes the URLs
        # that have been extracted in the previous step.
        name_detail = response.xpath("//h1/text()").extract_first()
        phone_detail = response.xpath("//dd/a/text()").extract_first()
		addr_lines = response.xpath("//*[contains(text(),'Postal')]/following-sibling::p/text()").extract()
		mail_detail = ', '.join(line.strip() for line in addr_lines)
        print("Found details: " + name_detail + ', ' + phone_detail + ', ' + mail_detail)
~~~
{: .source}

Running our scraper again

~~~
scrapy crawl au_gov_sen
~~~
{: .source}

produces something like

~~~
2022-10-13 01:55:08 [scrapy.utils.log] INFO: Scrapy 2.3.0 started (bot: au_gov)
(...)
2022-10-13 01:55:10 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results/> (referer: None)
Found URL: https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=R36
Found URL: https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=298839
(...)
Found details: Hon Karen Andrews MP, (07) 5580 9111, PO Box 409, Varsity Lakes, QLD, 4227
Found details: Senator Penny Allman-Payne, (07) 3001 8170, GPO Box 228, Brisbane, QLD, 4001
(...)
2022-10-13 01:55:11 [scrapy.core.engine] INFO: Closing spider (finished)
~~~
{: .output}

We appear to be getting somewhere! The last step is doing something useful with the
scraped data instead of printing it out on the terminal. Enter the Scrapy Items.

## Using Items to store scraped data

Scrapy conveniently includes a mechanism to collect scraped data and output it
in several different useful ways. It uses objects called `Items`. Those are akin
to Python dictionaries in that each Item can contain one or more fields to
store individual data element. Another way to put it is, if you visualize the
data as a spreadsheet, each Item represents a row of data, and the fields within
each item are columns.

Before we can begin using Items, we need to define their structure. Using our editor,
let's navigate and edit the following file that Scrapy has created for us when we
first created our project: `au_gov/au_gov/items.py`

Scrapy has pre-populated this file with an empty "AuGovItem" class:

(editing `au_gov/au_gov/items.py`)

~~~
import scrapy

class AuGovItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    pass
~~~
{: .output}

Let's add a few fields to store the data we aim to extract from the detail pages
for each politician:

~~~
import scrapy

class AuGovItem(scrapy.Item):
    # define the fields for your item here like:
    name = scrapy.Field()
    phone = scrapy.Field()
    email = scrapy.Field()
~~~
{: .source}

Then save this file. We can then edit our spider one more time:

(editing `au_gov/au_gov/spiders/au_gov_sen.py`)

~~~
import scrapy
from au_gov.items import AuGovItem # We need this so that Python knows about the item object

class AuGovSenSpider(scrapy.Spider):
    name = "au_gov_sen" # The name of this spider

    # The allowed domain and the URLs where the spider should start crawling:
    allowed_domains = ["www.ontla.on.ca"]
    start_urls = ['http://www.ontla.on.ca/web/members/members_current.do?locale=en/']

    def parse(self, response):
        # The main method of the spider. It scrapes the URL(s) specified in the
        # 'start_url' argument above. The content of the scraped URL is passed on
        # as the 'response' object.
		
        for url in response.xpath("//*[@class='mppcell']/a/@href").extract()[:5]:
            # This loops through all the URLs found inside an element of class 'mppcell'

            # Constructs an absolute URL by combining the response’s URL with a possible relative URL:
            full_url = response.urljoin(url)
            print("Found URL: "+full_url)
			
            # The following tells Scrapy to scrape the URL in the 'full_url' variable
            # and calls the 'get_details() method below with the content of this
            # URL:
            yield scrapy.Request(full_url, callback=self.get_details)

    def get_details(self, response):
        # This method is called on by the 'parse' method above. It scrapes the URLs
        # that have been extracted in the previous step.

        item = AuGovItem() # Creating a new Item object
        # Store scraped data into that item:
        item['name'] = response.xpath("normalize-space(//div[@class='mppdetails']/h1/text())").extract_first()
        item['phone'] = response.xpath("normalize-space(//div[@class='phone']/text())").extract_first()
        item['email'] = response.xpath("normalize-space(//div[@class='email']/a/text())").extract_first()

        # Return that item to the main spider method:
        yield item

~~~
{: .source}

We made two significant changes to the file above:
* We've included the line `from au_gov.items import AuGovItem` at the top. This is required
  so that our spider knows about the `AuGovItem` object we've just defined.
* We've also replaced the `print` statements in `get_details()` with the creation of an `AuGovItem`
  object, in which fields we are now storing the scraped data. The item is then passed back to the
  main spider method using the `yield` statement.

If we now run our spider again:

~~~
scrapy crawl au_gov_sen
~~~
{: .source}

we see something like

~~~
2022-10-13 02:03:52 [scrapy.utils.log] INFO: Scrapy 2.3.0 started (bot: au_gov)
(...)
2022-10-13 02:03:57 [scrapy.core.scraper] DEBUG: Scraped from <200 https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=R36>
{'mail': 'PO Box 5100, Marrickville, NSW, 2204',
 'name': 'Hon Anthony Albanese MP',
 'phone': '(02) 9564 3588'}
2022-10-13 02:03:57 [scrapy.core.scraper] DEBUG: Scraped from <200 https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=230886>
{'mail': 'PO Box 409, Varsity Lakes, QLD, 4227',
 'name': 'Hon Karen Andrews MP',
 'phone': '(07) 5580 9111'}
2022-10-13 02:03:57 [scrapy.core.scraper] DEBUG: Scraped from <200 https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=290544>
{'mail': '', 'name': 'Dr Michelle Ananda-Rajah MP', 'phone': '(03) 9822 4422'}
(...)
2017-02-27 21:53:54 [scrapy.core.engine] INFO: Spider closed (finished)
~~~
{: .output}

We see that Scrapy is dumping the contents of the items within the debugging output using
a syntax that looks a lot like JSON.

But let's now try running the spider with an extra `-o` ('o' for 'output') argument that
specifies the name of an output file with a `.csv` file extension:

~~~
scrapy crawl au_gov_sen -o output.csv
~~~
{: .source}

This produces similar debugging output as the previous run, but now let's look inside the
directory in which we just ran Scrapy and we'll see that it has created a file called
`output.csv`, and when we try looking inside that file, we see that it contains the
scraped data, conveniently arranged using the Comma-Separated Values (CSV) format, ready
to be imported into our favourite spreadsheet!

~~~
cat output.csv
~~~
{: .source}

Returns

~~~
phone,mail,name
(03) 9822 4422,,Dr Michelle Ananda-Rajah MP
(07) 3001 8170,"GPO Box 228, Brisbane, QLD, 4001",Senator Penny Allman-Payne
(02) 9564 3588,"PO Box 5100, Marrickville, NSW, 2204",Hon Anthony Albanese MP
(08) 9409 4517,"PO Box 219, Kingsway, WA, 6065",Hon Dr Anne Aly MP
(07) 5580 9111,"PO Box 409, Varsity Lakes, QLD, 4227",Hon Karen Andrews MP
~~~
{: .output}

By changing the file extension to `.json` or `.xml` we can output the same data
in JSON or XML format.
Refer to the [Scrapy documentation](http://doc.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-exports)
for a full list of supported formats.

Now that everything looks to be in place, we can finally remove our limit to the number
of scraped elements...

(editing `au_gov/au_gov/spiders/au_gov_sen.py`)

~~~
import scrapy
from au_gov.items import AuGovItem # We need this so that Python knows about the item object

class AuGovSenSpider(scrapy.Spider):
    name = 'au_gov_sen'
    allowed_domains = ['www.aph.gov.au']
    start_urls = ['http://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results/']

    def parse(self, response):
        # The main method of the spider. It scrapes the URL(s) specified in the
        # 'start_url' argument above. The content of the scraped URL is passed on
        # as the 'response' object.

        for url in response.xpath("//h4[@class='title']/a/@href").extract():
            # This loops through all the URLs found inside an element of class 'mppcell'

            # Constructs an absolute URL by combining the response’s URL with a possible relative URL:
            full_url = response.urljoin(url)
            print("Found URL: "+full_url)

            # The following tells Scrapy to scrape the URL in the 'full_url' variable
            # and calls the 'get_details() method below with the content of this
            # URL:
            yield scrapy.Request(full_url, callback=self.get_details)

    def get_details(self, response):
        # This method is called on by the 'parse' method above. It scrapes the URLs
        # that have been extracted in the previous step.
        item = ActmpsItem()
        item['name'] = response.xpath("//h1/text()").extract_first()
        item['phone'] = response.xpath("//dd/a/text()").extract_first()
        addr_lines = response.xpath("//*[contains(text(),'Postal')]/following-sibling::p/text()").extract()
        item['mail'] = ', '.join(line.strip() for line in addr_lines)
        yield item
~~~
{: .source}

(we've removed the `[:5]` at the end of the for loop on line 16 of the above code)

... and run our spider one last time:

~~~
scrapy crawl au_gov_sen -o au_gov_sen.csv
~~~
{: .source}

> ## Add other data elements to the spider
>
> Try modifying the spider code to add more data extracted from the Austrlian Parliament pages.
> Remember to edit the Item definition to allow for all extracted fields to be taken
> care of.
>
{: .challenge}

You are now ready to write your own spiders!

# Reference

* [Scrapy documentation](https://doc.scrapy.org/en/latest/index.html)


