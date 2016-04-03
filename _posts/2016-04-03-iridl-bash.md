---
title: "Make writeups great again with bash, make, and pandoc"
author: "Alexey Shiklomanov"
date: 2016-04-03
layout: post
comments: true
---

# Introduction

This semester, I'm taking a graduate level climate class ([ES 520 Modes of Climate Variability](https://sites.google.com/site/es520climatemodes/)), a big part of which are data exercises that require the use of the [IRI/LDEO Climate Data Library](iridl.ldeo.columbia.edu) -- a powerful, web-based tool for climate data visualization and analysis. Data in this library can be manipulated and analyzed "point-and-click" style, or, preferably, through its special scripting language (which it calls "expert mode"). Although it has a bit of a learning curve and is not ideal for data exploration, the programmatic approach is much better for repeatibility and, as I'll show here, for automation.

Our data exercise assignments usually require us to perform several analyses on certain data sets, generate some figures, and answer some questions concerning the figures. We turn in a write-up containing the figures and answers. In other classes, where we use R to accomplish similar tasks, my go-to method was to use R-markdown, allowing me to integrate text and code into a single file that I can run to keep everything up to date and neatly formatted. However, with the IRIDL language, there is no analog. So for the first few data exercises, my workflow was basically this:

1. Draft the "expert mode" code in a local text file.
2. Copy-paste it into the browser.
3. Download the resulting figure.
4. Link to the figure in my markdown document containing the writeup.
5. Compile the markdown document to a PDF using Pandoc.

But I quickly got fed up with steps 1-3, because I had to repeat them manually for each figure, and was never entirely sure if the figures I currently had copies of reflected the latest draft of my expert mode code. Instead, I wanted a more automated solution. In a nutshell, here's what I came up with (described in more detail below):

* I still write the expert mode code in a local text file, but I have a simple shell script that parses this code into a IRIDL URL and downloads the corresponding figure.
* I use a makefile to track dependencies -- every time I change code or text, I redownload the appropriate image and re-compile the PDF.

# Code to GIF: A Bash script

Here's what IRIDL expert mode code looks like:

{% highlight bash %}
SOURCES .NOAA .NCEP-NCAR .CDAS-1 .MONTHLY .Intrinsic .MSL .pressure
    (mb) unitconvert
    yearly-anomalies
    T (Jun-Aug) seasonalAverage
    Y -40 VALUES
    X AVERAGE
    T fig: line :fig
{% endhighlight %}

When you punch these into IRIDL's expert mode, [this is what you get](http://iridl.ldeo.columbia.edu/SOURCES/.NOAA/.NCEP-NCAR/.CDAS-1/.MONTHLY/.Intrinsic/.MSL/.pressure/%28mb%29unitconvert/yearly-anomalies/T/%28Jun-Aug%29seasonalAverage/Y/-40/VALUES/X/AVERAGE/T/fig:/line/:fig/). I want to grab just the figure, but there isn't a clean way to programmatically replicate clicking through a few menus to get to it.

Fortunately, if you inspect the URL from the link above, you should notice that it's actually the exact "expert mode" command, except with all spaces replaced by backslashes. This means that generating a URL from code like the above is just a few lines of bash. For instance, the `tr` command performs character substitution; for instance, to replace all spaces with backslashes in `file`, I just run:

{% highlight bash %}
tr ' ' '/' < file
{% endhighlight %}

With that figured out, everything else is pretty straightforward. I can download any file from the web with the `wget` command (here `wget -O filename` forces the downloaded file to have the name `filename`). When I need to stack two images on top of each other (which I do because IRIDL returns figures and their legends as separate images), I use the lightweight but powerful [GraphicsMagick](www.graphicsmagick.org/) (`gm`) tool:

{% highlight bash %}
gm convert img1.gif img2.gif -append img12.gif
{% endhighlight %}

Everything else is just standard bash syntax (logic, string concatenation, etc.). If you have questions, ask them in the comments! Here's the final result:

{% gist ashiklom/412af086383a333b44f9fa1398f560a7 download.iridl.sh %}

# Building the document with make

Anytime you have a web of dependencies among your files, you'd be hard-pressed to find a better tool than GNU `make` for sorting out those dependencies to build your project correctly and efficiently. The basic idea is that you provide a "recipe" for each file that, like an actual cooking recipe, describes the "ingredients" (dependent files) you need as well as instructions (in the form of bash code) for how to combine them to make the resultant file. Here's my makefile for this project:

{% gist ashiklom/412af086383a333b44f9fa1398f560a7 makefile %}

A few things to note about this. First of all, you'll notice that I don't have a single real filename anywhere in here. Instead, I use wildcards and substitution to determine all of the files I need to make. For example, `$(wildcard *.md)` returns a list of all files in the directory ending in `*.md`, and `$(patsubst, %.md, %.pdf, ...)` replaces every `md` extension with `pdf`, thus resulting in a list of PDFs that I need to compile. Similarly, I don't define any file-specific rules, and instead tell `make` how to deal with specific file extensions. For instance, the `%.gif : %.code` syntax says that all `.code` files are to be converted to `.gif` files by running the `code.to.url.sh` script on them (the strange-looking `$<` argument just refers to whatever the first dependency is -- here, whatever `.code` file make is currently working on). The end result is that my makefile works as follows: First, it finds every file with the `.code` extension and generates a `.gif` from it. Then, it finds every `.md` file and compiles it to a PDF using `pandoc`.

Now, with this setup in hand, I can write my ES520 homework assignments without ever having to open up a browser, and can sleep soundly knowing that the PDF I'm generating corresponds to the latest version of my code and text!
