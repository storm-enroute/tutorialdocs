---
layout: tutorial
title: Event Streams
topic: reactors
logoname: reactress-mini-logo-flat.png
projectname: Reactors.IO
homepage: http://reactors.io
permalink: /reactors/event-streams/index.html
pagenum: 1
pagetot: 40
section: guide-main
---

## Event streams

Event streams are objects that may *asynchronously* publish events.
Clients can subscribe to event streams to listen to these events.

An important thing to realize here is that asynchronous execution
does not imply concurrency.
In fact, if we look up the dictionary definition of *concurrent*, we find:

> Happening at the same time, side-by-side.

If we now look at the definition of *asynchronous*, we find:

> Not occurring at the same time.

Despite the fact that we, programmers, sometimes conflate these two terms,
in english language they mean almost opposite things.
But this is not just a matter of language purism,
asynchronous and concurrent also have separate meanings in programming!
When a computation is asynchronous,
it means that it does necessarily happen right away --
it might happen **later in the future**, possibly **on the same thread**.
When a computation is concurrent,
it means that the computation might happen **at the same time**,
but **on a different thread**.

Although event streams publish events to their clients asynchronously,
they are effectively **single-threaded**.
Event streams created within the same concurrency unit
always emit events in that concurrency unit.
This concurrency unit may be the current thread,
if we're using event streams without reactors (as we do in this section),
or a reactor otherwise (we cover reactors in the next section).

The fact that two event streams never activate concurrently is beneficial.
If two listeners to an event stream both access the same shared state,
then they will never be invoked at the same time.
Thus, there is no need for expensive and error-prone techniques
traditionally used to protect shared state.

The first step in any program using reactors is to import the contents of the
`io.reactors` package. The following line will allow us to declare event streams,
channels and reactors:

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
{% include reactors-java-event-streams-import.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
So far, so good!
Now, let's study the basic data-type that drives most computations in the Reactors.IO
framework -- an *event stream*. Event streams represent special program values that
can occasionally produce events. Event streams are represented by the `Event[T]` type.
Here is an event stream `myEvents`, which produces string events:

```scala
val myEvents: Events[String] = new Events.Emitter[String]
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
{% include reactors-java-event-streams-create.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
To be useful, an event stream must allow the users to somehow manipulate the events it
produces. For this purpose, every event stream has a method called `onEvent`, which
takes a user callback function and invokes it every time an event arrives:

```scala
myEvents.onEvent(x => println(x))
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
{% include reactors-java-event-streams-on-event.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
The `onEvent` method is similar to what most callback-based frameworks offer -- a way
to provide an executable snippet of code that is invoked later, once an event becomes
available. However, the receiver of the `onEvent` method, the event stream, is a
*first-class* value. This subtle difference allows passing the event stream as an
argument to other methods, and consequently write more general abstractions. For
example, we can implement a reusable `trace` method as follows:

```scala
def trace[T](events: Events[T]) {
  events.onEvent(println)
}
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
{% include reactors-java-event-streams-trace.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
Before we continue, we note that event streams are entirely a single-threaded entity.
The same event stream will never concurrently produce two events at the same time, so
the `onEvent` method will never be invoked by two different threads at the same time
on the same event stream. As we will see, this property simplifies the programming
model and makes event-based programs easier to reason about.

To understand this better, let's study a concrete event stream called *an emitter*,
represented by the `Events.Emitter[T]` type. In the following, we instantiate an
emitter:

```scala
val emitter = new Events.Emitter[Int]
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
{% include reactors-java-event-streams-emitter.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
An emitter is simultaneously an event stream and an *event source*. We can
imperatively "tell" it to produce an event by calling its `react` method. When we do
that, emitter invokes the callbacks previously registered with the `onEvent` method.

```scala
var luckyNumber = 0
emitter.onEvent(luckyNumber = _)
emitter.react(7)
assert(luckyNumber == 7)
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
{% include reactors-java-event-streams-lucky-number.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
By running the above snippet, we convince ourselves that `react` really forces the
emitter to produce an event.

## Lifecycle of an event stream  

We now take a closer look at the events that an event stream can produce. An event
stream of type `Events[T]` usually emits events of type `T`. However, `T` is not the
only type of events that an event stream can produce. Some event streams are finite.
When they emit all their events, they can `unreact` -- emit a special event that
denotes that there will be no further events. And sometimes, event streams cause
exceptional situations, and emit exceptions instead of normal events.

The `onEvent` method that we saw earlier can only react to normal events. To listen to
other event kinds, event streams have the more general `onReaction` method. The
`onReaction` method takes an `Observer` object as an argument -- an `Observer` has
three different methods used to react to different event types. In the following, we
instantiate an emitter and listen to all its events:

```scala
var seen = List[Int]()
var errors = List[String]()
var done = 0
val e = new Events.Emitter[Int]
e.onReaction(new Observer[Int] {
  def react(x: Int, hint: Any) = seen ::= x
  def except(t: Throwable) = errors ::= t.getMessage
  def unreact() = done += 1
})
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
{% include reactors-java-event-streams-observer.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
The `Observer[T]` type has three methods:

- `react`: Invoked when a normal event is emitted. The second, optional `hint`
  argument may contain an additional value, but is usually set to `null`.
- `except`: Invoked when the event stream produces an exception. An event stream can
  produce multiple exceptions -- an exception does not terminated the stream.
- `unreact`: Invoked when the event stream unreacts, i.e. stops producing events.
  After this method is invoked on the observer, no further events nor exceptions
  will be produced by the event stream.

Let's assert that this contract is correct for `Events.Emitter`. We already learned
that we can produce events with emitters by calling `react`. We can similarly call
`except` to produce exceptions, or `unreact` to signal that there will be no more
events. For example:

```scala
e.react(1)
assert(seen == 1 :: Nil)
e.react(2)
assert(seen == 2 :: 1 :: Nil)
assert(done == 0)
e.except(new Exception("^_^"))
assert(errors == "^_^" :: Nil)
e.react(3)
assert(seen == 3 :: 2 :: 1 :: Nil)
assert(done == 0)
e.unreact()
assert(done == 1)
e.react(4)
e.except(new Exception("o_O"))
assert(seen == 3 :: 2 :: 1 :: Nil)
assert(errors == "^_^" :: Nil)
assert(done == 1)
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
{% include reactors-java-event-streams-observer-test.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
As you can see above, after `unreact`, subsequent calls to `react` or `except` have
no effect -- `unreact` effectively terminates the emitter. Furthermore, we can see
that emitters are simultaneoulsy event streams and observers. Not all event streams
are as imperative as emitters, however. Most other event streams are created by
composing different event streams.

## Functional composition of event streams

In the previous sections, we saw how `onEvent` and `onReaction` methods work. In
addition to these two, there are a few additional ways to add a callback to an event
stream, as shown in the following example:

```scala
val e = new Events.Emitter[String]
e.onEventOrDone(x => println(x))(println("done, she's unreacted!"))
e.onDone(println("event stream done!"))
e onMatch {
  case "ok" => println("a-ok!")
}
e on {
  println("some event, but I don't care which!")
}
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
{% include reactors-java-event-streams-on-x.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
However, using only the `onX` family of event stream methods can easily result in a
*callback hell* -- a program composed of a large number of unstructured `onX` calls,
which is hard to understand and maintain. Having first-class event streams is a step
in the right direction, but it is not sufficient.

Event streams support *functional composition* -- a programming pattern that allows
forming complex values by composing simpler ones. Consider the following example:

```scala
var squareSum = 0
val e = new Events.Emitter[Int]
e.onEvent(x => squareSum += x * x)
for (i <- 0 until 5) e react i
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
{% include reactors-java-event-streams-sum-var.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
The example is fairly straightforward, but what if we want to make `squareSum` an
event stream so that another part of the program can react to its changes? We would
have to create another emitter and have our `onEvent` callback invoke `react` on
that new emitter, passing it the value of `squareSum`. This could work, but it's
ugly:

```scala
val ne = new Events.Emitter[Int]
e onEvent { x =>
  squareSum += x * x
  ne react squareSum
}
```

<div class='panel-group' id='acc-21'>
  <div class='panel panel-default'>
    <div class='panel-heading'>
      <h4 class='panel-title'>
        <a data-toggle='collapse' data-parent='#acc-21'
          href='#clps-22'>
          Java
        </a>
      </h4>
    </div>
    <div id='clps-22' class='panel-collapse collapse'>
      <div class='panel-body'>
{% capture s %}
{% include reactors-java-event-streams-sum-ugly.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
Let's rewrite the previous snippet using event stream combinators.
We use the `map` and `scanPast` combinators.
The `map` combinator transforms events in one event stream into events for a derived
event stream -- we use it to square each integer event.
The `scanPast` combinator combines the last and the current event to produce a new
event for the derived event stream -- we use it to add the previous value of the sum
to the current one.
For example, if an input event stream produces numbers `0`, `1` and `2`, the
event stream returned by `scanPast(0)(_ + _)` produces numbers `0`, `1` and `3`.

Here is how we can rewrite the previous example:

```scala
val e = new Events.Emitter[Int]
val sum = e.map(x => x * x).scanPast(0)(_ + _)
for (i <- 0 until 5) e react i
```

<div class='panel-group' id='acc-23'>
  <div class='panel panel-default'>
    <div class='panel-heading'>
      <h4 class='panel-title'>
        <a data-toggle='collapse' data-parent='#acc-23'
          href='#clps-24'>
          Java
        </a>
      </h4>
    </div>
    <div id='clps-24' class='panel-collapse collapse'>
      <div class='panel-body'>
{% capture s %}
{% include reactors-java-event-streams-sum-fun.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
The `Events[T]` type comes with a large number of predefined combinators.
You can inspect all of them in the online API documentation.
A set of event streams composed using functional combinators forms a
**dataflow graph**. Emitters are usually source nodes in this graph, event streams
created by various combinators are inner nodes, and callback methods like `onEvent`
are sink nodes. Combinators such as `union` take several input event streams.
Such event streams correspond to graph nodes with multiple input edges. Here is one
example:

```scala
val numbers = new Events.Emitter[Int]
val even = numbers.filter(_ % 2 == 0)
val odd = numbers.filter(_ % 2 == 1)
val numbersAgain = even union odd
```

<div class='panel-group' id='acc-25'>
  <div class='panel panel-default'>
    <div class='panel-heading'>
      <h4 class='panel-title'>
        <a data-toggle='collapse' data-parent='#acc-25'
          href='#clps-26'>
          Java
        </a>
      </h4>
    </div>
    <div id='clps-26' class='panel-collapse collapse'>
      <div class='panel-body'>
{% capture s %}
{% include reactors-java-event-streams-filter-union.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
The example above induces the following dataflow graph:

```
       /---filter---> [even] ----\
[numbers]                         union---> [numbersAgain]
       \---filter---> [odd ] ----/
```

### Higher-order event streams

In some cases event streams produce events that are themselves event streams -- we
call these **higher-order event streams**. A higher-order event stream can have a type
such as this one:

```scala
Events[Events[T]]
```

Calling `onEvent` on such an event stream is not always useful, because it gives us
access to nested event streams, and not their events.
There is more than one way to access events of type `T` from the inner event streams.
We might be interested in the events from the last `Events[T]` produced in the
higher-order event stream. To access those events, we use the `mux` operator. This
operator multiplexes events from the last `Events[T]` -- whenever a new nested event
stream is emitted, events from previously emitted nested event streams are ignored,
and only the events from the latest nested event stream get forwarded.

Consider the following example.

```scala
var seen = List[Int]()
val higherOrder = new Events.Emitter[Events[Int]]
val evens = new Events.Emitter[Int]
val odds = new Events.Emitter[Int]
higherOrder.mux.onEvent(seen ::= _)

evens react 2
odds react 1
higherOrder react evens
assert(seen == Nil)
odds react 3
evens react 4
assert(seen == 4 :: Nil)
higherOrder react odds
evens react 6
odds react 5
assert(seen == 5 :: 4 :: Nil)
```

<div class='panel-group' id='acc-27'>
  <div class='panel panel-default'>
    <div class='panel-heading'>
      <h4 class='panel-title'>
        <a data-toggle='collapse' data-parent='#acc-27'
          href='#clps-28'>
          Java
        </a>
      </h4>
    </div>
    <div id='clps-28' class='panel-collapse collapse'>
      <div class='panel-body'>
{% capture s %}
{% include reactors-java-event-streams-mux.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
In some cases we want to obtain all the events from all the even streams produced
by the higher-order event stream.
To achieve this, we use the postfix `union` combinator:

```scala
var seen2 = List[Int]()
val higherOrder2 = new Events.Emitter[Events[Int]]
higherOrder2.union.onEvent(seen2 ::= _)

higherOrder2 react evens
odds react 3
evens react 4
assert(seen2 == 4 :: Nil)
higherOrder2 react odds
evens react 6
assert(seen2 == 6 :: 4 :: Nil)
odds react 5
assert(seen2 == 5 :: 6 :: 4 :: Nil)
```

<div class='panel-group' id='acc-29'>
  <div class='panel panel-default'>
    <div class='panel-heading'>
      <h4 class='panel-title'>
        <a data-toggle='collapse' data-parent='#acc-29'
          href='#clps-30'>
          Java
        </a>
      </h4>
    </div>
    <div id='clps-30' class='panel-collapse collapse'>
      <div class='panel-body'>
{% capture s %}
{% include reactors-java-event-streams-union.html %}
{% endcapture %}
{{ s | markdownify }}
      </div>
    </div>
  </div>
</div>
For more examples of higher-order event stream combinators, please refer to the
online API.

