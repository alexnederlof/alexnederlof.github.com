---
layout: single
title: "Integration testing with Jetty"
date: 2012-11-21 16:23
comments: true
categories: [Testing, Java]
---
This is a followup after [my previous post about separating JUnit tests into fast tests and integration tests](/blog/2012/11/21/separating-the-fast-from-the-slow-junit-tests/). The sample code is [available on GitHub](https://github.com/alexnederlof/integration-testing-example).

When building a web application I like to have an integration test suite that resembles the real life situation as best as possible. The code should be able to run without too much effort from a build server like [Jenkins](http://jenkins-ci.org/) and it should be fairly easy to maintain. In this post I will explain how I achieved these goals.

To see what this example app does run the server by running the main method in `com.alexnederlof.inttesting.MyWebServer.java` and brows to [http://localhost:9090](). You can do this from your favorite IDE.

<!--more-->

### Installing Selenium server
Before we get started we need to install Selenium as a service on the build server. [In a previous post I wrote](/blog/2012/11/19/installing-selenium-with-jenkins-on-ubuntu/) how to install Selenium on a headless Ubuntu server. To install Selenium for this example, follow that post until the part where you hook it onto Jenkins. We don’t need that here because we will use JUnit instead of the HTML scripts I used in that post. To start and stop Selenium we need a script to wrap the jar. You can find my version of this script [here](https://gist.github.com/4120566). Adopt the script to your settings and install it in `/etc/init.d/selenium`. Make sure the display port matches the one you defined in `/etc/init.d/xvfb`. Now start Selenium and check the log to see if it went without any errors.

### Creating the tests
I created the tests for this example using the Firefox Selenium IDE. I exported them to Java and imported them in my project. They test two very simple pages included in the sample project that refer to each other. Here’s what the test looks like:

```java A simple web test https://github.com/alexnederlof/integration-testing-example/blob/master/src/test/java/com/alexnederlof/inttesting/SimpleSeleniumTest.java View on Github
	@Test
	public void testTheWebApp() throws Exception {
		Selenium selenium = getSelenium();
		selenium.open("/");
		assertThat(selenium.getText("css=h1"), is("This is the main page"));
		assertThat(selenium.getTitle(), is("Welcome"));
		selenium.click("id=otherLink");
		selenium.waitForPageToLoad("30000");
		assertThat(selenium.getTitle(), is("Other page"));
		assertThat(selenium.getText("css=h1"), is("This is the other page"));
		selenium.click("id=main");
		selenium.waitForPageToLoad("30000");
		assertThat(selenium.getTitle(), is("Welcome"));
	}
```

I want my tests to run on multiple browsers. To do this I use one of JUnit’s latest cool features: [@Parameterized](http://junit.sourceforge.net/javadoc/org/junit/runners/Parameterized.html). In the superclass of all my Selenium tests (`MultiBrowserTest.java`) the @Parameterized passes the browser as an argument to the constructor. Selenium is then started using that browser. The name parameter gives a nice name to the individual tests instances. Here’s what the test’s superclass looks like:

```java MultiBrowserTest https://github.com/alexnederlof/integration-testing-example/blob/master/src/test/java/com/alexnederlof/inttesting/MultiBrowserTest.java View it on GitHub
public abstract class MultiBrowserTest implements SlowTest {

	private static final String BASE_URL = MyWebServer.BASE_URL;
	private static final String SELENIUM_HOST = "127.0.0.1";
	private static final int SELENIUM_PORT = 4443;

	@Parameters(name = "browser={0}")
	public static List<Object[]> data() {
		return Arrays.asList(new Object[][] { { "*firefox" },
				{ "*googlechrome" } });
	}

	private final String browser;

	private Selenium selenium;

	public MultiBrowserTest(String browser) {
		this.browser = browser;
	}

	@Before
	public void setUp() throws Exception {
		selenium = new DefaultSelenium(SELENIUM_HOST, SELENIUM_PORT,
				getBrowser(), BASE_URL);
		selenium.start();
	}

	public String getBrowser() {
		return browser;
	}

	public Selenium getSelenium() {
		return selenium;
	}

	@After
	public void tearDown() throws Exception {
		selenium.stop();
	}
}
```

Now we need to setup the server to serve the pages Selenium will test. As shown in [the previous post](/blog/2012/11/21/separating-the-fast-from-the-slow-junit-tests/), I have a test suite that starts and stops  `MyServer.java`. This class wraps Jetty with the most basic configuration. It looks like this:

```java Simple Jetty Wrapper
	public static WebAppContext buildWebAppContext() {
		WebAppContext webAppContext = new WebAppContext();
		webAppContext.setWar(new File("src/main/webapp/").getAbsolutePath());
		return webAppContext;
	}

	private final Server server;

	public MyWebServer() {
		this.server = new Server(PORT);
		server.setHandler(buildWebAppContext());
	}

	public void start() throws Exception {
		server.start();
	}

	public void stop() throws Exception {
		server.stop();
	}
```

All in all this is what happens when you run `mvn test -P integrationtests`:

* The IntegrationTestSuit.java is run. This loads the Jetty server before it continues with the tests.
* SimpleSelenium test is loaded twice, once for Chrome and once for FireFox.
* The tests connect to the Selenium server running on localhost.
* After the tests have ran, either successful or not, the Suite shuts down the Jetty server.
* If you run this procedure from a continues integration server like Jenkins, you get nice statistics on tests. Hurray you have automated your integration tests!

Even cooler is that if you have a pretty decent Selenium test suite, you can also check your code coverage to see how much code you reach with your integration tests.

### Whats next?

I want to find out how Selenium Grid works and set it up on a Windows VM so I can also test Safari and Internet Explorer.