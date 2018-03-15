---
layout: single
title: "Using Git submodules for Maven artifacts not in central"
date: 2013-07-08 12:04
comments: true
categories: [Git, Maven, Java]
---
Sometimes you come across a project (or a branch of a project) that you need and isn't in [Maven central](http://search.maven.org). If you own your own Maven repository that's no problem: you simply deploy the project to your own Maven repository and you are done. However, when you don't have your own Maven repository things get complicated. 

You could install the other project in you local Maven repository using `mvn install`. After that you tell other developers via the README that they need to do the same. However, this requires manual effort, which is always more error-prone. You'd like to have Maven build that project automatically.

An alternative is to copy/paste the code from that project into your project. This is also not the best idea: you'll loose track of the version you've imported, you'll have code that doesn't belong to you in your repository, chances are that no one will update the code after that initial import because it might break, and finally there might be some licensing issues.

There is a third option which is a bit more elegant (albeit not perfect). You can import the third party project as a [Git submodule](http://git-scm.com/book/en/Git-Tools-Submodules) or using [Git subtree merging](http://git-scm.com/book/ch6-7.html). Here are the main differences:

- You can use *submodules* if you want to use the code but not import their repository in yours. This is useful if you want to contribute back to the original repository, or the original repository is very large and you don't want to bloat your repository. In other words: this will only link to the other repository.
- You can use *subtree merging* if you just want to import the code into your repository. This is useful if you just want read-only access to the other repository and you're not planning to contribute back to that repository. 

For more info about the differences read [this blog](http://blogs.atlassian.com/2013/05/alternatives-to-git-submodule-git-subtree/) by Atlasssian. 

I'm going to focus on using submodules, but you can do the same trick with Maven using subtree merging. I recommend [this excellent guide](http://nuclearsquid.com/writings/subtree-merging-and-you/) for subtree merging if you decided to go that way. Here's how it works for *submodules*:

<!--more-->

### Setting it up

__1.__ You setup your Maven project to have a parent pom and your own project as a [Maven module](http://maven.apache.org/guides/mini/guide-multiple-modules.html) of that project.

__2.__ You import the project you want into your project. For example [this legacy jsocks project](https://github.com/ravn/jsocks).

```sh
git submodule add -b master git@github.com:ravn/jsocks.git
git submodule update --remote
```
	
The `git submodule` command will now have cloned the master branch of the remote repository in a folder called *jsocks*. Git will have added a `.gitmodule` file which looks like this:

```plain .gitmodule
submodule ["jsocks"]
        path = jsocks
        url = git@github.com:ravn/jsocks.git
        branch = master
```

You directory structure should look like this

```plain Directory layout
yourParentProject
- pom.xml 
- .git
- .gitmodule
- yourproject
  \- pom.xml
- jsocks
  \- pom.xml
```

__3.__ In you parent pom.xml you add the *jsocks* folder as a [module](http://maven.apache.org/guides/mini/guide-multiple-modules.html).

```xml
<modules>
	<module>yourproject</module>
	<module>jsocks</module>
</modules>
```

And in your project, you add *jsocks* as a dependency:

```xml
<groupId>com.github.ravn.jsocks</groupId>
<artifactId>jsocks</artifactId>
<version>0.0.1-SNAPSHOT</version>
``` 

__4.__ Now everything is set-up. If you call `mvm test` on the parent project you will see that it first builds *jsocks* and then your project.

### Using it

When other developers clone your project, they need to install the module as well using two git commands:

```sh 
git submodule init
git submodule update --remote
```

### Caveats

When the submodule changes everyone using the project has to manually do a `git submodule update --remote`. This means that it is not guaranteed that all developers are working with the same version of the submodule. Checking out a certain tag or commit is not possible at the moment as far as I know. This means that everyone either has to try to stay up to date, or you might have to reconsider using *subtree merging*.

_Inspired by [this great tutorial @ vogella.com](http://www.vogella.com/articles/Git/article.html#submodules). Read [the official documentation](http://git-scm.com/book/en/Git-Tools-Submodules) for more details._