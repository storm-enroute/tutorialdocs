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

This section contains a simple, working Hello World program.
We won't get into too much details - you can find deeper information
in the subsequent sections.
We will define a reactor that waits for one incoming event,
prints a message to the standard output,
and then terminates.

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

Above, we note the following:

- A reactor is defined using the `Reactor[T]` constructor.
- A reactor reacts to incoming events as specified in the callback function passed
  to the `onEvent` method.
- Calling `main.seal()` terminates the reactor.
- A reactor with a specific definition is started with the `spawn` method, which
  returns the reactor's default channel.
- Events are sent to the reactor by calling the `!` operator on its channel.

The subsequent sections will explain these features in depth.

