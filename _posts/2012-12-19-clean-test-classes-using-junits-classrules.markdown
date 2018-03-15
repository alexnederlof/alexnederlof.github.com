---
layout: single
title: "Clean test classes using JUnit's rules"
date: 2012-12-19 21:23
comments: true
categories: [Testing, Java]
---
A couple of days ago I discovered the beauty of JUnit's [TestRules](http://kentbeck.github.com/junit/javadoc/4.10/org/junit/rules/TestRule.html) while searching for an easy way to set a time-out on all tests in a testcase. JUnit has a built-in rule for this called [`Timeout`](http://kentbeck.github.com/junit/javadoc/4.10/org/junit/rules/Timeout.html). You can set this rule for every test in your class by setting the timeout in a field like this:

```java Setting a Timeout Rule http://kentbeck.github.com/junit/javadoc/4.10/org/junit/rules/Timeout.html View the Javadoc

public class MyTest {
	
	@Rule
	public MethodRule globalTimeout= new Timeout(20);
	
	@Test
	public void someTest() {
		...
	}
	
```
Another gem is the `ExpectedException` rule, which allows you to inspect a thrown exception in several ways.

```java Inspecting excptions http://kentbeck.github.com/junit/javadoc/4.10/org/junit/rules/ExpectedException.html View the Javadoc
public static class HasExpectedException {
	@Rule
	public ExpectedException thrown= ExpectedException.none();

	@Test
	public void throwsNothing() {
	// no exception expected, none thrown: passes.
	}
 
 	@Test
    public void throwsNullPointerException() {
		thrown.expect(NullPointerException.class);
		throw new NullPointerException();
	}
 
	@Test
	public void throwsNullPointerExceptionWithMessage() {
		thrown.expect(NullPointerException.class);
		thrown.expectMessage("happened?");
		thrown.expectMessage(startsWith("What"));
		throw new NullPointerException("What happened?");
	}
 }
```

The great thing is, it's super easy to extend one of these rules. 

<!--more-->

In [Crawljax](https://github.com/crawljax/crawljax), another project I'm currently working on, I wanted a Jetty server to start before I the tests run, and to shut it down afterwards. I could do this using a `@BeforeClass` method and then clean it up in the `@AfterClass` method but that doesn't make it reusable in other classes. To make it reusable I could put it in an abstract class that just has the setup and teardown methods and inherit that class in the classes where I need the server. However, that can lead to weird class hierarchies that don't make any sense. Again, JUnit's rules come to the rescue. There's the [`ExternalResource`](http://kentbeck.github.com/junit/javadoc/4.10/org/junit/rules/ExternalResource.html) that allows you to setup resources before tests, and tear them down afterwards. I inherited this class to provide my Jetty server.

```java Rule to start a Jetty Server https://github.com/crawljax/crawljax/blob/4b3a3f44c946b32c1dee5fa14960764c90393666/src/test/java/com/crawljax/demo/RunWithWebServer.java View on GitHub
public class RunWithWebServer extends ExternalResource {

	private final Resource resource;

	private int port;
	private URL demoSite;
	private Server server;
	private boolean started;

	/**
	 * @param classPathResource
	 *            The name of the resource. This resource must be on the test or regular classpath.
	 */
	public RunWithWebServer(String classPathResource) {
		resource = Resource.newClassPathResource(classPathResource);
	}

	@Override
	protected void before() throws Throwable {
		server = new Server(0);
		ResourceHandler handler = new ResourceHandler();
		handler.setBaseResource(resource);
		server.setHandler(handler);
		server.start();
		this.port = server.getConnectors()[0].getLocalPort();
		this.demoSite = new URL("http", "localhost", port, "/");
		this.started = true;
	}

	@Override
	protected void after() {
		try {
			if (server != null) {
				server.stop();
			}
		} catch (Exception e) {
			throw new RuntimeException("Could not stop the server", e);
		}
	}

	// Some getters for the fields
}
```

The class starts the Jetty server using the given resource as the web folder. Tests can then use this rule to obtain a real URL to test with:

```java Using the webserver. https://github.com/crawljax/crawljax/blob/4b3a3f44c946b32c1dee5fa14960764c90393666/src/test/java/com/crawljax/core/CandidateElementExtractorTest.java Usage example
public void SomeWebTest {

	@ClassRule
	public static final RunWithWebServer SERVER = new RunWithWebServer("/site/crawler");
	
	@Test
	public void test() {
		URL url = SERVER.getSiteUrl();
		testStuffUsingThe(url);	
	}
	
}
```

There are more built-in rules in JUnit like:

* The [TemporaryFolder](http://kentbeck.github.com/junit/javadoc/4.10/org/junit/rules/TemporaryFolder.html) That creates a temporary folder for you,
* the [Verifier](http://kentbeck.github.com/junit/javadoc/4.10/org/junit/rules/Verifier.html) that can verify some invariant after each test method,
* and more! Checkout the [JUnit wiki](https://github.com/KentBeck/junit/wiki/Rules) to learn more of them.

Have fun!