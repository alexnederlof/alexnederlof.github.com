---
layout: single
title: "Installing Selenium with Jenkins on Ubuntu"
date: 2012-11-19 16:01
comments: true
categories: [Testing]
---
Last week I fell in love with [Selenium](http://seleniumhq.org/) and started to create some tests using the [Firefox Selenium IDE](http://seleniumhq.org/projects/ide/). If you’ve never done this I highly recommend trying it out. It’s really easy and great fun.

Now that I’ve created some great test suites I wanted to hook the tests to my Jenkins build server. There was no tutorial that covered everything online so here’s what I did:

**This will install Jenkins, Selenium standalone, Chrome and FireFox on a headless Ubuntu server.**

<!--more-->

I’m running Ubuntu 12.04. I assume you have Jenkins installed. If not, the Jenkins installation documentation is an excellent resource.

To download Chrome we need to add the Google key to for apt:


	wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
	sudo sh -c 'echo deb http://dl.google.com/linux/chrome/deb/ stable main > /etc/apt/sources.list.d/google.list'


Let’s install the required packages. The fonts aren’t really necessary but installing them prevents you from getting annoying warnings. [xvfb](http://en.wikipedia.org/wiki/Xvfb) can start a virtual X window on a server where Selenium can start the browser in to run your tests.

	sudo apt-get update && sudo apt-get install -y xfonts-100dpi xfonts-75dpi xfonts-scalable xfonts-cyrillic xvfb x11-apps  imagemagick firefox google-chrome-stable

Now we need to download the [Chrome driver](https://www.google.com/search?q=selenium+chrome+driver) binary for Ubuntu and we need the latest [Selenium Server jar](http://seleniumhq.org/download/). I placed the Chrome driver in `/var/lib/chrome-driver/chrome-driver` and the selenium jar in `/var/lib/selenium/selenium.jar` but you can put them anywhere you like.

To start xvfb I like to have a little start/stop script like this one:

```bash
	
	#!/bin/bash
	if [ -z "$1" ]; then
		echo "`basename $0` {start|stop}"
		exit
	fi
	
	case "$1" in
	start)
		/usr/bin/Xvfb :99 -ac -screen 0 1024x768x8 &
	;;
	stop)
		killall Xvfb
	;;
	esac
```

This will start a Window on display :99. This is important later on. Place the script in `/etc/init.d/`. To start it automatically on boot, run:  `sudo update-rc.d xvfb defaults`.

Now it’s time to test if Selenium works. I presume you hava a Selenium HTML Suite plus a site you can test with.

```bash
	# First lets start xvfb
	sudo /etc/init.d/xvfb start

	# Now we have to set the DISPLAY env variable so Firefox and Chrome know where to open the browser.
	export DISPLAY=:99

	# Let's see if Selenium works for firefox:
	java -jar /var/lib/selenium/selenium-server.jar -htmlSuite *firefox http://yoursite.com "/path/to/your/tests/Suite.html" "/tmp/firefox-results.html"

	# For chrome we also need to specify the Chrome driver location.:
	java -jar -Dwebdriver.chrome.driver=/var/lib/chrome-driver/chromedriver /var/lib/selenium/selenium-server.jar -htmlSuite *googlechrome http://yoursite.com "/path/to/your/tests/Suite.html" "/tmp/chrome-results.html"
```

These commands should now work without any errors and show you the results in the specified files in `/tmp/`.

## Now let’s hook it onto Jenkins!

Make sure you have [the SeleniumHQ plugin](http://wiki.hudson-ci.org/display/HUDSON/Seleniumhq+Plugin) installed for Jenkins.

In Jenkins global configuration add the environment variable that specifies the display:

![Selenium with Jenkins](/assets/images/installing-selenium-with-jenkins-on-ubuntu/env.png)

Then select *“New Job”* and then select *“Build a free-style software project”* and give it a name.

Fill in your SCM provider, assuming you have one so the workspace can be filled. I have my tests stored in a folder named tests in the repo.

In the *“Build Steps”* section, add an entry for the Chrome test and for the Firefox test. Note that in the Chrome test you have to specify the path to your Chrome driver.

Then as a post-build action add the selenium reports.

![Config](/assets/images/installing-selenium-with-jenkins-on-ubuntu/config.png)

If everything went well you should see nice test graphs in your Jenkins like this:

![Test result](/assets/images/installing-selenium-with-jenkins-on-ubuntu/test-result.png)
