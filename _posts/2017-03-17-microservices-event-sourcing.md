---
title: When Microservices Meet Event Sourcing
layout: post
date: 2017-05-29 3:55
tag:
- event-sourcing
- microservices
blog: true
---

For the last two years my team has been working in the development of a distributed services platform for a major US-based bank. This platform is intended to onboard a few products in the beginning but gradually grow as legacy products are also migrated to it. There are several challenges and cross-functional requirement involved like extensibility, audibility, analytics, etc, which have naturally driven us to move towards an [Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html) and [CQRS](https://www.martinfowler.com/bliki/CQRS.html) design.

After the successful release of the first few products in production running on this architectural style, we felt it was a good time to share our learnings with the world. Recently I had the honor to present our journey twice: in April at the [O'Reilly Software Architecture Conference](https://conferences.oreilly.com/software-architecture/sa-ny/public/schedule/detail/56806) and in May at the [ThoughtWorks Tech Talks NYC](https://www.meetup.com/ThoughtWorks-Tech-Talks-NYC/events/239465465/). The feedback from the audience was above expected, both with skepticism and enthusiasm. Checkout the presentation video:

<div style="text-align:center">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/cISNDnwlSgw" frameborder="0" allowfullscreen></iframe>
</div>

- The slides are published here: [speakerdeck.com/vvgomes/when-microservices-meet-event-sourcing](https://speakerdeck.com/vvgomes/when-microservices-meet-event-sourcing)
- The demo source code is here: [github.com/vvgomes/restaurant-oreillysacon](https://github.com/vvgomes/restaurant-oreillysacon)

## Frequently Asked Questions

Many people came to me and to people from my team with question about this architectural style. Here are the most popular ones:

### How should clients deal with Eventual Consistency?

[Eventual Consistency](https://martinfowler.com/articles/microservice-trade-offs.html#consistency) is one of the accidental complexities of event-driven CQRS applications. The problem basically consists of the fact *command execution* is usually implemented in the *fire-and-forget* style which means there are no guaranties about when the result is available for querying.

A possible answer for that question can be the separation between the **command execution result** and the **data result**. Right after the client sends a command to be processed, the CQRS application must - at least - synchronously respond with a result. This is either success (2XX status code) or a failure (> 4XX status code). Besides that, it is a good practice to return an URI (in the `Location` HTTP header, for instance) that guides the client where to find the data result. As far as querying the data itself, there are a few known options:

1. **Polling** - the client code keeps polling the URI returned by the command execution until the data is ready.
2. **Websockets** - the client passivelly waits for the server to send the data back through a websocket connection.
3. **Optimistic Rendering** - based on a successful command execution, the client can optimistically render the results without actually querying the backend.

In fact, our experience shows that in the vast majority of the cases the data will be available for querying pretty much at the same time as the command request is done, which means that the client wil mostly be able to just query the data right after it is done sending the command. Anyway, in case you know about alternative options to deal with Eventual Consistency from the client side please let me know.

### How do you deal with changes in the event payload structure?

Since events are immutable and the event-store append only, there is no room for database migrations in Event Sourcing. Therefore, when it comes to schema changes we need to define mapping functions which will translate older to newer version of events. This technique is known as [Event Upcasting](https://docs.axonframework.org/v/3.0/part3/repositories-and-event-stores.html#event-upcasting).

On the other hand, we don't want to break consumers after changing the payload structure of an event. That can be accomplished by having *Contract Tests* running in your CI pipeline. In this context, Contract Tests will pretty much verify whether consumers are still able to successfully deserialize the most recent version of a given event.

### What about performance?

Over time, the amount of events in the event-store can grow to a point in which rebuilding the current state of the application becomes expensive. To minimize this problem we can persist intermediate states as [Snapshots](https://martinfowler.com/eaaDev/EventSourcing.html) and then only replay the most recent events on top of it. Archiving very old events in a secondary storage and using snapshots permanently is also an option in extreme cases. Overall, managing an in-memory cache of the current state in a running application is a reasonable solution for most cases.

### How can we integrate with non-event-driven systems?

When it comes to integrating event-driven applications with non-event-driven (internal or external) systems, there are a lot of different aspects to be discussed like real-time vs batch processing, communication protocols, resilience, SLAs, etc. Although that goes beyond my intended scope, you can still find most of the answers on that regard by watching this excellent presentation by my teammate [@javatarz](https://twitter.com/javatarz):

<div style="text-align:center">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/4cJ4GyyOfII" frameborder="0" allowfullscreen></iframe>
</div>
