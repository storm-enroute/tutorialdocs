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
import io.reactors._

object HelloWorld {
  def main(args: Array[String]) {
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
  }
}
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
- Events are sent to the reactor by calling the `!` operator on its channels.

The subsequent sections will explain these features in depth.

**Note:** If you are running the above example in IntelliJ or another IDE that runs
your Scala programs in a separate JVM process, you need to ensure that this new
JVM process does not die when the `main` function ends. Reactors run on daemon
threads by default, so they will not prevent the JVM from terminating. There are
several ways to fix this, and the easiest is to add a `Thread.sleep` at the end of
the `main` function. A more sophisticated approach is to start your `welcomeReactor`
on a dedicated thread instead of a thread pool:

<div class='panel-group' id='acc-1'>
  <div class='panel panel-default'>
    <div class='panel-heading'>
      <h4 class='panel-title'>
        <a data-toggle='collapse' data-parent='#acc-1'
          href='#clps-2'>
          JVM
        </a>
      </h4>
    </div>
    <div id='clps-2' class='panel-collapse collapse'>
      <div class='panel-body'>
{% capture s %}
{% include reactors-scala-jvm-spawn-thread.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
