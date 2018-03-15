---
layout: single
title: "Cache-Control using annotations with Jersey"
date: 2013-07-28 12:02
comments: true
categories: [Jersey, Java]
---
Building RESTful services with Jax-RX is awesome, but there's no annotation based notation to set your `Cache-Control` headers. You can either set the Cache-Control headers using a filter based on a URL pattern and map your responses through there, or you can use the [ResponseBuiler](http://jersey.java.net/nonav/apidocs/1.12/jersey/javax/ws/rs/core/Response.ResponseBuilder.html) with the [CacheControl](http://jersey.java.net/nonav/apidocs/1.12/jersey/javax/ws/rs/core/CacheControl.html) class like this:

```java
CacheControl control = new CacheControl();
control.setMaxAge(60);
Response.ok(myEntity).cacheControl(control).build();
```

However, I would rather have something like this:

```java
@GET
@CacheMaxAge(time = 10, unit = TimeUnit.MINUTES)
@Path("/awesome")
public String returnSomethingAwesome() {
	...
}
```

It turns out that's pretty easy to set-up. I mostly use either no caching at all or caching for a certain period. 

<!--more-->

First lets define those two annotations:

{% gist 6098121 CacheAnnotations.java %}

As you can see I can either set `@NoCache` or I can set `@CacheMaxAge` with a certain time.

Now we need to configure these annotations using a 'ResourceFilterFactory': 

{% gist 6098121 CacheFilterFactory.java %}

Finally we need to install these filters. Using the `web.xml` file you'll can add the class like so:

```xml
<init-param>
    <param-name>com.sun.jersey.spi.container.ResourceFilters</param-name>
    <param-value>package.name.CacheFilterFactory</param-value>
</init-param>
```

...or if you are using Guice you can add it to the parameters like this:

```java
Map<String, String> params = new HashMap<>();
params.put(ResourceConfig.PROPERTY_RESOURCE_FILTER_FACTORIES, CacheFilterFactory.class.getName());
filter("/api/*").through(GuiceContainer.class, params);
```

You can no add these annotations to any resource method you like. As you can see it is easy to extend to add any other headers you like. With a little tweaking you might also be able to configure it so that it can use ETags by intercepting the content but that was outside the scope for me. 

This solution was inspired by [this StackOverflow post](http://stackoverflow.com/questions/10934316/jersey-default-cache-control-to-no-cache).