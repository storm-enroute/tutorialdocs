---
layout: tutorial
title: Schedulers and Reactor Lifecycle
topic: reactors
logoname: reactress-mini-logo-flat.png
projectname: Reactors.IO
homepage: http://reactors.io
permalink: /reactors/schedulers-and-lifecycle/index.html
pagenum: 3
pagetot: 40
section: guide
---

## Schedulers

Each reactor template can be used to start multiple reactor instances,
and each reactor instance can be started with a different reactor scheduler.
Different schedulers have different characteristics in terms of execution priority,
frequency, latency and throughput.
In this section, we'll take a look at how to use a non-default scheduler,
and how to define custom schedulers when necessary.

We start with the import of the standard Reactors.IO package:

```scala
import io.reactors._
```

<div class='panel-group' id='acc-1'>
  <div class='panel panel-default'>
    <div class='panel-heading'>
      <h4 class='panel-title'>
        <a data-toggle='collapse' data-parent='#acc-1'
          href='#clps-2'>
          Java
        </a>
      </h4>
    </div>
    <div id='clps-2' class='panel-collapse collapse'>
      <div class='panel-body'>
{% capture s %}
{% include reactors-java-schedulers-import.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
We then define a reactor that logs incoming events,
reports every time it gets scheduled,
and ends after being scheduled three times.
We will use the `sysEvents` stream of the reactor,
which will be explained shortly -
for now, all you need to know is that this stream produces
events when the reactor gets some execution time (i.e. gets scheduled),
and pauses its execution (i.e. gets preempted).

```scala
class Logger extends Reactor[String] {
  var count = 3
  sysEvents onMatch {
    case ReactorScheduled =>
      println("scheduled")
    case ReactorPreempted =>
      count -= 1
      if (count == 0) {
        main.seal()
        println("terminating")
      }
  }
  main.events.onEvent(println)
}
```

<div class='panel-group' id='acc-3'>
  <div class='panel panel-default'>
    <div class='panel-heading'>
      <h4 class='panel-title'>
        <a data-toggle='collapse' data-parent='#acc-3'
          href='#clps-4'>
          Java
        </a>
      </h4>
    </div>
    <div id='clps-4' class='panel-collapse collapse'>
      <div class='panel-body'>
{% capture s %}
{% include reactors-java-schedulers-logger.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
Before starting, we need to create a reactor system,
as we learned in the previous sections:

```scala
val system = new ReactorSystem("test-system")
```

<div class='panel-group' id='acc-5'>
  <div class='panel panel-default'>
    <div class='panel-heading'>
      <h4 class='panel-title'>
        <a data-toggle='collapse' data-parent='#acc-5'
          href='#clps-6'>
          Java
        </a>
      </h4>
    </div>
    <div id='clps-6' class='panel-collapse collapse'>
      <div class='panel-body'>
{% capture s %}
{% include reactors-java-schedulers-system.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
Every reactor system is bundled a default scheduler
and some additional predefined schedulers.
When a reactor is started, it uses the default scheduler,
unless specified otherwise.
In the following, we override the default scheduler with the one using Scala's
global execution context, i.e. Scala's own default thread pool:

```scala
val proto = Proto[Logger].withScheduler(JvmScheduler.Key.globalExecutionContext)
val ch = system.spawn(proto)
```

<div class='panel-group' id='acc-7'>
  <div class='panel panel-default'>
    <div class='panel-heading'>
      <h4 class='panel-title'>
        <a data-toggle='collapse' data-parent='#acc-7'
          href='#clps-8'>
          Java
        </a>
      </h4>
    </div>
    <div id='clps-8' class='panel-collapse collapse'>
      <div class='panel-body'>
{% capture s %}
{% include reactors-java-schedulers-global-ec.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
In Scala.js, there is no multi-threading - executions inside a single JavaScript
runtime must execute in a single thread. For this reason, you will need to use
a special `JsScheduler.Key.default` instance.

<div class='panel-group' id='acc-9'>
  <div class='panel panel-default'>
    <div class='panel-heading'>
      <h4 class='panel-title'>
        <a data-toggle='collapse' data-parent='#acc-9'
          href='#clps-10'>
          Scala.js
        </a>
      </h4>
    </div>
    <div id='clps-10' class='panel-collapse collapse'>
      <div class='panel-body'>
{% capture s %}
{% include reactors-scala-js-custom-scheduler.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
Running the snippet above should start the `Logger` reactor and print `scheduled`
once, because starting a reactor schedules it even before any event arrives.
If we now send an event to the main channel, we will see `scheduled` printed again,
followed by the event itself.

```scala
ch ! "event 1"
```

<div class='panel-group' id='acc-11'>
  <div class='panel panel-default'>
    <div class='panel-heading'>
      <h4 class='panel-title'>
        <a data-toggle='collapse' data-parent='#acc-11'
          href='#clps-12'>
          Java
        </a>
      </h4>
    </div>
    <div id='clps-12' class='panel-collapse collapse'>
      <div class='panel-body'>
{% capture s %}
{% include reactors-java-schedulers-global-ec-send.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
Sending the event again decrements the reactor's counter.
The main channel gets sealed, leaving the reactor in a state without non-daemon
channels, and the reactor terminates:

```scala
ch ! "event 2"
```

<div class='panel-group' id='acc-13'>
  <div class='panel panel-default'>
    <div class='panel-heading'>
      <h4 class='panel-title'>
        <a data-toggle='collapse' data-parent='#acc-13'
          href='#clps-14'>
          Java
        </a>
      </h4>
    </div>
    <div id='clps-14' class='panel-collapse collapse'>
      <div class='panel-body'>
{% capture s %}
{% include reactors-java-schedulers-global-ec-send-again.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>

## Reactor Lifecycle

Every reactor goes through a certain number of stages during its lifetime,
jointly called a **reactor lifecycle**.
When the reactor enters a specific stage, it emits a lifecycle event.
All lifecycle events are dispatched on a special daemon event stream called `sysEvents`.
Every reactor is created with this event stream.

The reactor lifecycle is as follows:

- After calling `spawn`,
  the reactor is scheduled for execution.
  Its constructor is started asynchronously,
  and immediately after that,
  a `ReactorStarted` event is dispatched.
- Then, whenever the reactor gets execution time,
  the `ReactorScheduled` event is dispatched.
  After that, some number of events are dispatched on normal event streams.
-  When the scheduling system decides to preempt the reactor,
  the `ReactorPreempted` event is dispatched.
  This scheduling cycle can be repeated any number of times.
- Eventually, the reactor terminates,
  either by normal execution or exceptionally.
  If a user code exception terminates execution,
  a `ReactorDied` event is dispatched.
- In either normal or exceptional execution,
  `ReactorTerminated` event gets emitted.

This reactor lifecycle is shown in the following diagram:

```
    |
    V
ReactorStarted
    |
    V
ReactorScheduled <----
    |                 \
    V                 /
ReactorPreempted -----
    |                 \
    |            ReactorDied
    V                 /
ReactorTerminated <---
    |
    x
```

To test this, we define the following reactor:

```scala
class LifecycleReactor extends Reactor[String] {
  var first = true
  sysEvents onMatch {
    case ReactorStarted =>
      println("started")
    case ReactorScheduled =>
      println("scheduled")
    case ReactorPreempted =>
      println("preempted")
      if (first) first = false
      else throw new Exception
    case ReactorDied(_) =>
      println("died")
    case ReactorTerminated =>
      println("terminated")
  }
}
```

<div class='panel-group' id='acc-15'>
  <div class='panel panel-default'>
    <div class='panel-heading'>
      <h4 class='panel-title'>
        <a data-toggle='collapse' data-parent='#acc-15'
          href='#clps-16'>
          Java
        </a>
      </h4>
    </div>
    <div id='clps-16' class='panel-collapse collapse'>
      <div class='panel-body'>
{% capture s %}
{% include reactors-java-schedulers-lifecycle.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
Upon creating the lifecycle reactor,
the reactor gets the `ReactorStarted` event,
and then `ReactorStarted` and `ReactorScheduled` events.
The reactor then gets suspended,
and remains that way until the scheduler gives it more execution time.

```scala
val ch = system.spawn(Proto[LifecycleReactor])
```

<div class='panel-group' id='acc-17'>
  <div class='panel panel-default'>
    <div class='panel-heading'>
      <h4 class='panel-title'>
        <a data-toggle='collapse' data-parent='#acc-17'
          href='#clps-18'>
          Java
        </a>
      </h4>
    </div>
    <div id='clps-18' class='panel-collapse collapse'>
      <div class='panel-body'>
{% capture s %}
{% include reactors-java-schedulers-lifecycle-spawn.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
The scheduler executes the reactor again
when it detects that there are pending messages.
If we send an event to the reactor now,
we can see the same cycle of `ReactorScheduled` and `ReactorPreempted`
on the standard output.
However, the `ReactorPreempted` handler this time throws an exception.
The exception is caught, `ReactorDied` event is emitted,
followed by the mandatory `ReactorTerminated` event.

```scala
ch ! "event"
```

<div class='panel-group' id='acc-19'>
  <div class='panel panel-default'>
    <div class='panel-heading'>
      <h4 class='panel-title'>
        <a data-toggle='collapse' data-parent='#acc-19'
          href='#clps-20'>
          Java
        </a>
      </h4>
    </div>
    <div id='clps-20' class='panel-collapse collapse'>
      <div class='panel-body'>
{% capture s %}
{% include reactors-java-schedulers-lifecycle-send.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
At this point, the reactor is fully removed from the reactor system.

