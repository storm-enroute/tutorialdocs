---
layout: tutorial
title: The Hello World Program
topic: reactors
logoname: reactress-mini-logo-flat.png
projectname: Reactors.IO
homepage: http://reactors.io
permalink: /reactors/hello-world/index.html
pagenum: 3
pagetot: 40
section: guide-intro
---

## The Hello World Program

This section contains a simple, working Hello World program,
without getting into too much details - you can read more in the subsequent sections.
The following is a minimal example of a working reactor program.

```scala
val welcomeReactor = Reactor[String] {
  self =>
  self.main.events onEvent { name =>
    println(s"Welcome, $name!")
    self.main.seal()
  }
}
val system = ReactorSystem.default("test-system")
val ch = system.spawn(welcomeReactor)
ch ! "Alan"
```

The program above declares an anonymous reactor called `welcomeReactor`,
which waits for a name to arrive on its main event stream,
prints that name, and then seals its main channel, therefore terminating itself.
The main program then creates a new reactor system,
uses the reactor template to start a new running instance of the reactor,
and sends an event to it.

The subsequent sections explain the features used in this program
in more detail.

