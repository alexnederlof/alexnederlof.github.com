---
layout: single
title: "Separating the fast from the slow JUnit tests"
date: 2012-11-21 16:16
comments: true
categories: [Testing, Java]
---
For some time now I was looking for a good way to do real integration testing with JUnit. These tests tests tend to be slow because the whole stack has to be build up and shut down. Furthermore, some tests also need a specific environment like a database connection which is not available to any developer. That’s why you probably want to split up your test suite in fast and slow (or dependent) tests. JUnit has a technique to split up these tests using [Categories](http://kentbeck.github.com/junit/javadoc/4.10/org/junit/experimental/categories/Categories.html). This allows you to specify the category your tests belongs to and then skip those tests in your Suite like so:

```java
	@RunWith(Categories.class)
	@IncludeCategory(SlowTests.class)
	@SuiteClasses( { A.class, B.class })
	public static class SlowTestSuite { }
```

The downside here is that you have to specify all the tests in the test suite. As far as I know, JUnit has no mechanism to do this for this for you. You can work with the [Maven Surefire plugin](http://maven.apache.org/plugins/maven-surefire-plugin/examples/junit.html#Using_JUnit_Categories) to filter stuff in and out, but I think there’s a better way. A man by the name of [Johannes Link](http://www.johanneslink.net/) built a [great little library](http://www.johanneslink.net/projects/cpsuite.jsp) which does just what I want. I allows us to specify in a Suite to run all JUnit tests it can find. It can also exclude and include certain tests. Unfortunately  it doesn’t work with JUnit’s Categories system but it uses type inheritance. This makes it easy to filter out types of tests. For example I define all the fast tests like this:

```java
@RunWith(ClasspathSuite.class) // Loads all unit tests it finds on the classpath
@ExcludeBaseTypeFilter(SlowTest.class) // Excludes tests that inherit SlowTest
public class FastTests {}
```

<!--more-->

This will run every test that doesn’t implement the SlowTest inferface. The slow test suite needs a tearing up of the server, and once the suite is done the server should be shut down. It could also be that you need to set up a database connection or connect to another server. In my case, the suite should only run subtypes of MultiBrowserTest. Here’s how that looks:

```java
@RunWith(ClasspathSuite.class)
@BaseTypeFilter(MultiBrowserTest.class)
public class IntegrationTestSuite {

	private static MyWebServer server;

	@BeforeClass
	public static void startServer() throws Exception {
		server = new MyWebServer();
		server.start();
	}

	@AfterClass
	public static void shutDownServer() throws Exception {
		server.stop();
	}
}
```

Now that we’ve split up the tests we want Maven to run the fast tests always and do the integration tests only when we ask for it. This can easily be done using Maven profiles. I define two profiles default and integrationtests. They set the include property for the surefire plugin.

``` xml
<profiles>
		<profile>
			<id>default</id>
			<activation>
				<activeByDefault>true</activeByDefault>
			</activation>
			<properties>
				<tests.include>com/alexnederlof/seleniumtest/suites/FastTests.java</tests.include>
			</properties>
		</profile>
		<profile>
			<id>integrationtests</id>
			<properties>
				<tests.include>com/alexnederlof/seleniumtest/suites/*</tests.include>
			</properties>
		</profile>
	</profiles>

	<build>
		<plugins>
			<plugin>
				<artifactId>maven-surefire-plugin</artifactId>
				<groupId>org.apache.maven.plugins</groupId>
				<version>2.12.4</version>
				<configuration>
					<includes>
						<include>${tests.include}</include>
					</includes>
				</configuration>
			</plugin>
		</plugins>
	</build>
```

Now running `mvn test` will run only the FastTests and `mvn test -P integrationtests` will run all the tests.

You can find the code for this example [on GitHub](https://github.com/alexnederlof/integration-testing-example/). This post is actually part of [another blog post of mine](/blog/2012/11/21/integration-testing-with-jetty/) where I explain how to do integration testing using Jetty and Selenium.