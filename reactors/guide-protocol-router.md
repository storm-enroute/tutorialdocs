---
layout: tutorial
title: Router Protocol
topic: reactors
logoname: reactress-mini-logo-flat.png
projectname: Reactors.IO
homepage: http://reactors.io
permalink: /reactors/protocol-router/index.html
pagenum: 2
pagetot: 40
section: guide-protocol
---

## Router Protocol

In this section,
we take a look at a simple router protocol.
Here, events coming to a specific channel are routed between a set of target channels,
according to some user-specified policy.
In practice, there are a number of applications of this protocol,
ranging from data replication and sharding, to load-balancing and multicasting.
The protocol is illustrated in the following figure:

```
           #----------------#
           |                |
           |              /-+-- channel 1
           |             /  |
router  ---o--> router -----+-- channel 2
channel    |             \  |
           |              \-+-- channel 3
           |                |
           #----------------#
```

In our first example,
we will instantiate a master reactor that will route the incoming requests
between two workers.
For simplicity, requests will be just strings,
and the workers will just print those strings to the standard output.

There are several ways to instantiate the router protocol.
First, the protocol can be started within an existing reactor,
in which case it is just one of the protocols running inside that reactor.
Second, the protocol can be started as a standalone reactor,
in which case that reactor is dedicated to the router protocol.

We will start by creating an instance of the router protocol in an existing reactor.
We first import the contents of the `io.reactors`
and the `io.reactors.protocol` packages,
and then instantiate a default reactor system.

```scala
import io.reactors._
import io.reactors.protocol._

val system = ReactorSystem.default("test-system")
```

We can now use the reactor system to start two workers, `worker1` and `worker2`.
We use a shorthand method `spawnLocal`, to concisely start the reactors
without first creating the `Proto` object:

```scala
val worker1 = system.spawnLocal[String] { self =>
  self.main.events.onEvent(x => println(s"1: ${x}"))
}
val worker2 = system.spawnLocal[String] { self =>
  self.main.events.onEvent(x => println(s"2: ${x}"))
}
```

Next, we declare a reactor whose main channel takes `Unit` events,
since we will not be using the main channel for anything special.
Inside that reactor, we first call `router[String]` on the `channels` service
to open a connector for the router.
Just calling the `router` method does not start the protocol - we need to call
`route` on the connector to actually start routing.

The `route` method expects a `Router.Policy` object as an argument.
The policy object contains the routing logic for the router protocol.
In this example, we will use the simple round-robin policy.
The `Router.roundRobin` factory method expects a list of channels
for the round-robin policy, so we will pass a list with `worker1` and `worker2`:

```scala
system.spawnLocal[Unit] { self =>
  val router = system.channels.daemon.router[String]
    .route(Router.roundRobin(Seq(worker1, worker2)))
  router.channel ! "one"
  router.channel ! "two"
}
```

Having instantiated the `router` protocol, we use the input `channel`
associated with the router to send two events to the workers.

After starting the router protocol and sending the events `"one"` and `"two"`
to the router channel, the two strings are delivered to the two different
workers. The `roundRobin` policy does not specify which of the target channels
is chosen first, so the output could be the following two lines, in some order:

```
1: one
2: two
```

Or, the following two lines, in some order:

```
1: two
2: one
```

The round-robin routing policy does not have any knowledge about the two target
channels, so it just picks one after another in succession, and then the first one
again when it reaches the end of the target list. Effectively, this policy
constitutes a very simple form of load-balancing.

There are other predefined policies that can be used with the router protocol.
For example, the `Router.random` policy uses a random number generator to route events
to different channels, which is more robust in scenarios when a high-load event
gets sent periodically. Another policy is `Router.hash`, which computes the hash code
of the event, and uses it to find the target channel. If either of these are not
satisfactory, `deficitRoundRobin` strategy tracks the expected cost of each event,
and biases its routing decisions to balance the total cost sent to each target.

The above-mentioned policies are mainly used in load-balancing.
In addition, users can define their own custom policies for their use-cases.
For example, if the router has some information about the load and availability
of the targets, it could take that into account when making routing decisions.


### Router Reactors

In the previous example, we instantiated the router protocol inside an existing
reactor. Sometimes we want to completely dedicate the reactor to routing incoming
events, so it is useful to have a shorthand notation for starting such
*router reactors*.

In the following, we use a router reactor to implement simple sharding for key-value
pairs, which is typically done in distributed hash-tables.
Here, the data contained in a distributed hash table is spread across some number
of computing nodes called *shards*. Each shard will be represented by a separate
reactor, which will have two basic operations - `Put` and `Get`, used to add entries
to the distributed hash table, and to remove them, respectively.

```scala
sealed trait Op[K, V] {
  def key: K
}

case class Put[K, V](key: K, value: V)
extends Op[K, V]

case class Get[K, V](key: K)
extends Op[K, V]
```

The `Put` operation holds information about the key-value pair,
while the `Get` operation contains the desired key.
We implement the distributed hash table functionality in a method
called `distributedHashTable`. This method instantiates the shards,
and then starts the main router reactor.

We will use the following convention - if the key has some hash code `h`,
then its shard is `h % numShards`, where `numShards` is the number of
different shards. Incidentally, this is exactly how the router policy
`Router.hash` behaves.
The distributed hash table implementation is then shown in the following:

```scala
def distributedHashTable[K, V](
  numShards: Int
): Server[Op[K, V], Option[V]] = {
  // Create the shard reactors, used to hold the key-value pairs.
  val shards = for (i <- 0 until numShards) yield {
    system.spawnLocal[Server.Req[Op[K, V], Option[V]]] { self =>
      val data = mutable.Map[K, V]()
      self.main.events onMatch {
        case (Put(key, value), reply) =>
          reply ! data.get(key)
          data(key) = value
        case (Get(key), reply) =>
          reply ! data.get(key)
      }
    }
  }

  // Return the router reactor, which forwards requests
  // to the appropriate shard reactor.
  system.router(Router.hash(shards, _._1.key.##))
}
```

Note that, in our implementation, the distributed hash table's channel type
is a server. Clients must always pass a reply channel on which they
receive the `Option[V]` object - the value that was previously in the hash table.
This will mean that we will be able to use the `?` operator
on the distributed hash table, which was explained in the previous
section on the *server* protocol.

Next, we instantiate the distributed hash table for string key-value pairs,
and specify `32` shards:

```scala
val dht = distributedHashTable[String, String](32)
```

We can now create a client of our distributed hash table,
which inserts one entry into the hash table, and then asks to get it back.

```scala
system.spawnLocal[Option[String]] { self =>
  (dht ? Put("key", "value")) onEvent { prevOpt =>
    assert(prevOpt == None)
    (dht ? Get("key")) onMatch {
      case Some(prev) => println(prev)
      case None => println("unexpected - empty!")
    }
  }
}
```

After the reactor above starts, we should soon see `"value"` printed on the
standard output.

