---
layout: default
title: Scalatra | The anatomy of a Scalatra app
---

<div class="page-header">
  <h1>The anatomy
    <small>of a Scalatra app</small>
  </h1>
</div>

## Project structure

The recommended way of structuring a Scalatra project is as follows:

    project

    src
    |_ main
       |_ scala
       |  |   |_Scalatra.scala <= see note below!
       |  |
       |  |_org
       |      |_yourdomain
       |        |
       |        |_projectname
       |          |_ ArticlesServlet.scala
       |          |_ UsersServlet.scala
       |
       |_ webapp
          |
          |_ WEB-INF
             |
             |_ views
             |  |_ default.jade
             |
             |_ layouts
             |  |_ default.jade
             |
             |_ web.xml


## Serving static files

Static files can be served out of the `webapp` folder, which acts as the ROOT
directory. As with any servlet based application, the contents of this directory
are all public, with the exception of files in the WEB-INF directory.

An example structure may help in understanding this.

    src
    |_ main
       |_ scala
       |  |_ Web.scala
       |_ webapp
          |_ WEB-INF
          |  |_ secret.txt
          |  |_ views
          |  |  |_ default.jade
          |  |
          |  |_ layouts
          |  |  |_ default.jade
          |  |
          |  |_ web.xml
          |- stylesheets
          |    |_ default.css
          |- images
               |_ foo.jpg


In this application, the only publicly accessible files will be at
`stylesheets/default.css` and `images/foo.jpg`. Everything else will be
protected by the web application container.


## ScalatraServlet vs. ScalatraFilter

There are two base classes you can inherit from in order to make a
Scalatra application: `ScalatraServlet` and `ScalatraFilter`.

{% highlight scala %}

  class YourServlet extends ScalatraServlet with ScalateSupport {
    // your class here
  }

{% endhighlight %}

vs.

{% highlight scala %}

  class YourFilter extends ScalatraFilter with ScalateSupport {
    // your class here
  }

{% endhighlight %}

The main difference is the default behavior when a route is not found.
A `ScalatraFilter` will delegate to the next filter or servlet in the chain (as
configured by web.xml), whereas a `ScalatraServlet` will return a 404
response.

Another difference is that `ScalatraFilter` matches routes relative to
the WAR's context path. `ScalatraServlet` matches routes relative to the
servlet path. This allows you to mount multiple servlets in different namespaces
in the same WAR.

### Use ScalatraFilter if:

* You are migrating a legacy application inside the same URL space
* You want to serve static content from the WAR rather than a
  dedicated web server

### Use ScalatraServlet if:

* You want to match routes with a prefix deeper than the context path.


## Scalatra's SBT dependencies

Scalatra uses Scala's [Simple Build Tool][sbt-site], or `sbt`, as a build system.

[sbt-site]: http://www.scala-sbt.org/

The `build.sbt` file defines the libraries which your application will depend on,
so that `sbt` can download them for you and build your Scalatra project.

Here's an example Scalatra sbt file:

{% highlight scala %}

    organization := "org.example"

    name := "yourapp"

    version := "0.1.0-SNAPSHOT"

    scalaVersion := "2.9.1"

    seq(webSettings :_*)

    classpathTypes ~= (_ + "orbit")

    libraryDependencies ++= Seq(
      "org.scalatra" % "scalatra" % "2.1.0",
      "org.scalatra" % "scalatra-scalate" % "2.1.0",
      "org.scalatra" % "scalatra-specs2" % "2.1.0" % "test",
      "ch.qos.logback" % "logback-classic" % "1.0.6" % "runtime",
      "org.eclipse.jetty"        % "jetty-webapp"           % "8.1.5.v20120716"     % "container",
      "org.eclipse.jetty"        % "test-jetty-servlet"     % "8.1.5.v20120716"     % "test",
      "org.eclipse.jetty.orbit"  % "javax.servlet"          % "3.0.0.v201112011016" % "container;provided;test" artifacts (Artifact("javax.servlet", "jar", "jar"))

    )

    resolvers += "Sonatype OSS Snapshots" at "http://oss.sonatype.org/content/repositories/snapshots/"

{% endhighlight %}

<div class="alert alert-info">
<span class="badge badge-info"><i class="icon-flag icon-white"></i></span>
If your project depends on any additional libraries, you can add them to the
<code>libraryDependencies</code> section. Doing that and running <code>sbt</code>
again will download the dependency jar libraries and make them available to
your application. If you don't know what the dependency details are, you can
find out on <a href="http://search.maven.org">http://search.maven.org</a>.
</div>

The default dependencies are:

<dl class="dl-horizontal">
  <dt>scalatra</dt>
  <dd>This is the core Scalatra module, and is required to run the framework.</dd>
  <dt>scalatra-scalate</dt>
  <dd>This integrates with <a href="http://scalate.fusesource.org">Scalate</a>,
  a template engine supporting multiple template formats. Scalate is optional, but
  highly recommended for any app requiring templating.</dd>
  <dt>scalatra-specs2</dt>
  <dd>This integrates the <a href="https://github.com/etorreborre/specs2">Specs2</a>
      testing libraries.</dd>
  <dt>logback-classic</dt>
  <dd>Basic logging functionality, courtesy of <a href="http://logback.qos.ch/">Logback</a>.</dd>
  <dt>jetty-webapp</dt>
  <dd>This is the embedded servlet container used by the web plugin. Your application should be portable to any servlet container supporting at least the 2.5 specification.</dd>
  <dt>servlet-api</dt>
  <dd>Required for building your app.  It is placed in the `provided`
  configuration so that it is not bundled with your application.  Your servlet
  container will provide this at deployment time.</dd>
</dl>