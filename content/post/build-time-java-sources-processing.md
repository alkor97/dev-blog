+++
title = "Build-time Java sources processing"
draft = false
date = "2016-10-30T20:03:42+01:00"
tags = ["java", "maven"]
description = "How to generate build-related constants in Java sources during Maven build"

[Params]
Author = "MAciej Stachurski"

+++

In one of projects I was involved there was a need to ensure consistency between Maven-level properties and Java sources. We had names of some EE resources (e.g. JMS topic, REST paths) defined as Maven properties in order to be able to generate installation scripts during the build (using Maven filtering). Of course, it was possible to just use static content and ensure consistency manually, but it would rather clumsy solution.

Having resource names defined in Maven properties I decided to use them also in Java sources, e.g. for defining REST paths in JAX-RS annotations. Defining paths as String constants was the only solution here. The solution is slightly more complicated, as Maven does not allow filtering of Java sources out-of-the-box. The key points are:

- store to-be-filtered sources in different directory than regular sources: `src/main/java/filtered`
- sources should contain Maven properties references (in form `${name}`, eg. `${project.version}`) that will be replaced with values during filtering.
- add to-be-filtered sources as filtered resources in POM:

```xml
...
<build>
 ...
 <resources>
  <resource>
   <directory>src/main/filtered</directory>
   <filtering>true</filtering>
   <includes>
    <include>**/*.java</include>
   </includes>
   <targetPath>${project.build.directory}/generated-sources/filtered</targetPath>
  </resource>
 </resources>
 ...
</build>
...
```
- add filtered sources to compilation path with `add-source` goal of [build-helper-maven-plugin](http://mojo.codehaus.org/build-helper-maven-plugin/) (will be treated as generated sources):


```xml
...
<build>
 ...
 <plugins>
  ...
  <plugin>
   <groupId>org.codehaus.mojo</groupId>
   <artifactId>build-helper-maven-plugin</artifactId>
   <version>1.8</version>
   <executions>
    <execution>
     <id>add-filtered-sources</id>
     <phase>process-sources</phase>
     <goals>
      <goal>add-source</goal>
     </goals>
     <configuration>
      <sources>
       <source>${project.build.directory}/generated-sources/filtered</source>
      </sources>
     </configuration>
    </execution>
   </executions>
  </plugin>
  ...
 <plugins>
 ...
</build>
...
```

Then running regular `maven build` will first process the sources and then compile them. Please note that before project gets built you may notice compilation errors in IDE when you reference filtered classes (they will become visible once build finishes).

That's it. Simple yet powerful. Full example you can fetch from [here](/downloads/JavaFiltering-1.0-src.zip).

