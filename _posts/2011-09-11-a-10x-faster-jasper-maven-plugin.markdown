---
layout: single
title: "A 10x faster Jasper maven plugin"
date: 2011-09-11 15:57
comments: true
categories: Tech
---
For some months now I had to work with Java projects that use [Jasper](http://jasperforge.org/projects/jasperreports) to generate PDF files. These reports have to be compiled from a kind xml-like document to a compiled ‘.jasper’ file. The [Jasperreports-maven-plug-in](http://mojo.codehaus.org/jasperreports-maven-plugin/) does this for me whenever I make a release of the software or when I’m working on the reports. However, the plug-in takes considerable time to compile these reports. Compiling 52 reports takes about 49 seconds. That’s a bit too long for me if I have to make several builds. Since I wanted to learn how to make my own Maven plug-ins anyway I decided to try to make a faster plug-in on my own.

The original plug-in is created in Java 4, works single-threaded and the last time any committed to the repo was (at time of writing) 31st of August, 2009. Not really an active project it seems. For that reason I decided to just create a new project on Github.

I created the plug-in using Java 6, presuming most people will have Java 6 installed by now. Because the compiler has to read and write to a disk, doing so concurrently should make it faster. Finally, the code looks a better because the collections now have generics.

It turned out my plug-in was indeed a lot faster. While the original plug-in takes 48 seconds to compile 52 files, my plug-in only takes 4.7 seconds. Almost ten times faster!

Since it is my first Maven project there’s probably more work to be done. However it seems to work fine on Maven 2 and 3 on my Mac using Java 6.

You can try it yourself and download the source from [the Github repo](https://github.com/alexnederlof/Jasper-report-maven-plugin) I created. Issues can be filed there as well.