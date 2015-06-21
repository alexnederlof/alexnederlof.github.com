---
layout: post
title: "Running multiple versions of a micro service"
date: 2015-06-21 13:24:31 +0200
comments: true
categories: 
---
Last week I visited [GOTO Amsterdam](http://gotocon.com/amsterdam-2015). There were some fantastic speakers talking about micro services. At [Magnet.me](https://magnet.me) we're spending a lot of effort in transforming our monolithic application into micro services so we were curious to see if we were on the right track. Turns out we are, for which I'm very proud of the team. One talk in in particular stood out and gave me a new insight, which I'd like to share here.

The talk was by [Fred George](https://twitter.com/fgeorge52) who presented this definition of micro services. I thought it was pretty accurate: 

<blockquote class="twitter-tweet tw-align-center" lang="en"><p lang="en" dir="ltr">Principles of micro services according to <a href="https://twitter.com/fgeorge52">@fgeorge52</a> <a href="https://twitter.com/hashtag/gotoams?src=hash">#gotoams</a> <a href="http://t.co/89SvBAePXz">pic.twitter.com/89SvBAePXz</a></p>&mdash; Alex Nederlof (@alexnederlof) <a href="https://twitter.com/alexnederlof/status/611453606718513152">June 18, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Fred told a very interesting story about rolling out new versions of micro services. He gave the example of a car rental company that had a micro service that was responsible for generating ads to show customers on the website. The system was connected via a message bus. Because the advertisement service was so small, it was easy for programmers to duplicate the micro service (literally just copy/paste it), optimize it, and then run the new version *along side* the older version. This sounded very wrong and horrible to me at first, because you're duplicating code, and you'd have to "maintain" two versions of a service. But then Fred shed some light on the potentially huge benefits.

<!--more-->

When a customer would log-in, an event would fire that the web server needs an advertisement. Both the old and the new add service would react to the event, and send their advertisement back over the bus. The one that would that was received first, was sent to the client. Because the new version of the advertisement service was supposed to be faster, so those advertisements were often the ones to win the race to the client, and thus got served.

Getting comfortable with running multiple versions of the same services has other benefits too: 

Because just listening on an message bus is so easy you can compare how they perform. Even if you just ignore on of the newer versions. This is great to try out new stuff. For example, you can define a fitness function for the accuracy of the advertisement, run a new version for a week, and compare the broadcasted advertisements from the old and the new version, to see which one behaves better.

If you have compatibility issues, you can let the old and the new version of a service run in parallel, and let the newer services consume the newer version of the messages. If you do this right, and the services are small enough, this actually much easier than refactoring your API. 

I can already think of a bunch of services where we're going to try this technique at [Magnet.me](https://magnet.me). So Fred if you're reading this. Thank you!

You can find the slides of Fred's talk [on the conference site](http://gotocon.com/amsterdam-2015/presentation/Challenges%20in%20Implementing%20MicroServices). 