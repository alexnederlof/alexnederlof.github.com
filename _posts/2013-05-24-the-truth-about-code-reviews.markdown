---
layout: single
title: "The truth about code reviews"
date: 2013-05-24 16:31
comments: true
categories: [Research]
---
During the [International Conference of Software Engineering 2013 ](http://2013.icse-conferences.org/) I visited a great talk by [Alberto Bacchelli](https://twitter.com/sback_) on modern code reviews. It resonated with my experience with code reviews and I'd like to share the highlights of [his research](http://www.inf.usi.ch/phd/bacchelli/publications/icse2013.pdf) conducted among 17 developers from 16 different teams at Microsoft.

Most developers have to do some kind of code reviewing at some point. When you use GitHub for example, every pull request can be seen as a code review. At some companies like Microsoft, Google and Facebook, code reviews are part of the job. To make it easy for developers they work with specialized tools like [Google's Rietveld](https://developers.google.com/appengine/articles/rietveld), [Facebook's Phabricator](http://phabricator.org/) or the Open Source tool [Gerrit](https://code.google.com/p/gerrit/). We do code reviews because it is well known that [code reviews offer many advantages](http://www.youtube.com/watch?v=CKjRt48rZGk). When I have to explain to someone why code reviews are important, I come up with the same arguments most developers come up with (in order of importance):

1. finding defects
2. code quality improvement
3. alternative Solutions
4. knowledge Transfer
5. shared code ownership

However, research shows a different result:

> Although the top motivation driving code reviews is still finding defects, the practice and the actual outcomes are less about finding errors than expected: Defect related comments comprise a small proportion and mainly cover small logical low-level issues

<!--more-->

So what does research show us? Here's the overview of what developers indicate is their motivation for code reviews:

![Motivations](/assets/images/code-reviews-motivations.png)

By observing developers doing code reviews, Bacchelli has found quite a different outcome. In order of preference, the actual use of code reviews turns out to be:

1. __Code improvements:__ Taking up 29% of the comments are things like better code practices, removing unnecessary code, and improving readability.
2. __Understanding:__ Discussions about what the code does take up a little over 20%
3. __Social communication:__ Messages such as thanking reviewers for a comment, and informing them that a proposed change has been implemented.
4. __Defects:__ In contrast to what developers believe is the most important reason, defect finding comes fourth with 14% of the comments. These include logical issues, high-level issues, security issues and exception handling.

### Code Reviewing Is Understanding

So why is finding defects in fourth place instead of first? The problem is that most of the code in a review is hard to understand if you are not the author. The study reports that most defects found in a review were not complicated errors:

>  …most of the comments on “defects” regard uncomplicated logical errors, e.g., corner cases, common configuration values, or operator precedence.

To this extend, the researchers asked the developers and testers what activities require most understanding when giving a code review. The majority votes for a _complete_ or _high_ understanding to find defects or offer alternative solutions. Sharing code ownership and knowledge require _low_ or _high_ understanding of the code. The lesson here is that when you ask for a code review, (1) make sure you ask the right person, and (2) don't assume the reviewer will find any bugs.

### Can we improve the way we review?
Most of the code reviews I do go via GitHub pull requests. Although they give an excellent textual presentation of the diff, the code is ordered alphabetically and it isn't interactive. As far as I know, the same goes for the other review tools mentioned before. There are a couple of ways this could be improved in my opinion:

* By showing something like a class/module/file diagram diff. This would make it easier for a developer to see in which modules most changes occurred and how those propagate through the code.
* Allow method jumping in a diff. Especially in a large diff you quickly lose the overview of where methods lead to or come from. Being able to jump from the method you're reading to the method that is being called would be a huge improvement in code diff navigation.
* As mentioned in the research, it is best the review is performed by another author of the given file. It would be great if the review tool would choose the best reviewer for a given changeset based on the code history.

### Conclusions
Code reviews are still very helpful however we should understand that their are not as useful for finding bugs as we might have thought. The tools we use for reviewing can still be improved. Until they do, make sure you appoint your reviews to the right person.

*N.B. Copyright notice: All data and graphs used in this post are from the [original paper](http://www.inf.usi.ch/phd/bacchelli/publications/icse2013.pdf).*