---
layout: tutorial
title: Schedulers and Reactor Lifecycle
topic: reactors
logoname: reactress-mini-logo-flat.png
projectname: Reactors.IO
homepage: http://reactors.io
permalink: /reactors/schedulers/index.html
pagenum: 3
pagetot: 40
section: guide
---

## Schedulers

TODO

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
and ends after being scheduled three times:

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
{% include reactors-java-schedulers-system.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
We then create a reactor system, as we saw in the previous sections,

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
val proto = Proto[Logger].withScheduler(
  ReactorSystem.Bundle.schedulers.globalExecutionContext)
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
Running the snippet above should start the `Logger` reactor and print `scheduled`
once, because starting a reactor schedules it even before any event arrives.
If we now send an event to the main channel, we will see `scheduled` printed again,
followed by the event itself.

```scala
ch ! "event 1"
```

<div class='panel-group' id='acc-9'>
  <div class='panel panel-default'>
    <div class='panel-heading'>
      <h4 class='panel-title'>
        <a data-toggle='collapse' data-parent='#acc-9'
          href='#clps-10'>
          Java
        </a>
      </h4>
    </div>
    <div id='clps-10' class='panel-collapse collapse'>
      <div class='panel-body'>
{% capture s %}
{% include reactors-java-schedulers-global-ec-send.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
Sending the event again decrements the reactor's counter and seals the main channel,
therefore terminating the reactor:

```scala
ch ! "event 2"
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
{% include reactors-java-schedulers-global-ec-send-again.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
