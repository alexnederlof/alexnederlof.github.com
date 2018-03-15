---
layout: single
title: "LaTeX build server"
date: 2013-02-22 10:58
comments: true
categories: [Automate everything, Research]
---
I recently had to work with [LaTeX](http://www.LaTeX-project.org/) again. Although LaTeX has its perks like a proper equation editor and BibTex I don't like working with it for several reasons: 

1. there is no proper [WYSIWYG](http://en.wikipedia.org/wiki/WYSIWYG) editor that compiles while you type for OS X and the source files are hard to read by themselves
2. you have to manually configure that it compiles LaTeX, then bibtex, and then LaTeX twice
3. the horrors of positioning and loading images with all the different compilers, bounding boxes, etc.
4. the fact that they have their own [Stack exchange Q&A](http://tex.stackexchange.com/) is an indication of how arcane it is.

Because the source files are so unpleasant to read, you always have to make sure you send the compiled latest version of your document to the right people. What I like about some MarkDown related projects like [Jekyll](https://github.com/mojombo/jekyll) and [Octopress](octopress.org) is the way you can push the source to for example GitHub, and they build the site for you. I wanted that for LaTeX as well. Using a simple script I now have my own LaTeX build server. This way my colleagues and I can always see the latest version of the document in any browser so I don't have to think about distributing the latest version to the right people or devices.

The end result is nice and simple listing of the PDF, diff and log in a folder that indicates the build date: 

![End result](/assets/images/latex-build.png)

<!--more-->

### Setting it up
I host my own Git server so I hooked the script as a `post-receive` script in the Git hooks directory. If you are running elsewhere you could also do it via a WebHook I guess. You need a workspace folder to checkout the project and a web folder to dump the results in. Here is the script: 

{% gist 5015614  %}

I use [Pygments](http://pygments.org/) to format the diff file and `pdflatex` to parse the LaTeX. Make sure you have these packages installed on you system.

I'm running Apache which gives a nice directory listing. However, it sorts the files in the wrong order. I want the most recent to be on top. Luckily this can be configured in Apache using the `IndexOrderDefault Descending Date` in your configuration.


### Potential improvements:
* I use a static location for the work folder but you could also create a temporary folder and delete it afterwards.
* This script only builds the master branch, you could also create a per-branch output folder system.
* Adding [git-discribe](http://kernel.org/pub/software/scm/git/docs/git-describe.html) to the output would be nice.