---
layout: tutorial
title: Setup
topic: reactors
logoname: reactress-mini-logo-flat.png
projectname: Reactors.IO
homepage: http://reactors.io
permalink: /reactors/setup/index.html
pagenum: 2
pagetot: 40
section: guide-intro
---

## Setup

This section contains instructions on how to get Reactors working in your project.
Reactors.IO has multiple language frontends, and works on multiple platforms.
Currently, Reactors can be used with Scala and Java as a library for the JVM,
or alternatively on NodeJS or inside the browser if you are using the Scala.js frontend.


### SBT

If you are developing using the [sbt](http://www.scala-sbt.org/) build tool,
the easiest is to include Reactors into your project as a library depedency.

To get started with Reactors.IO, you should grab the latest snapshot version distributed
on Maven. If you are using SBT, add the following to your project definition:

```scala
resolvers ++= Seq(
  "Sonatype OSS Snapshots" at
    "https://oss.sonatype.org/content/repositories/snapshots",
  "Sonatype OSS Releases" at
    "https://oss.sonatype.org/content/repositories/releases"
)

libraryDependencies ++= Seq(
  "io.reactors" %% "reactors" % "0.8-SNAPSHOT")
```

If you are using Scala.js, use the following dependency:

```scala
libraryDependencies ++= Seq(
  "io.reactors" %%% "reactors" % "0.8-SNAPSHOT")
```

Alternatively, you can download these dependencies manually,
and keep them in your folder for managed libraries.

