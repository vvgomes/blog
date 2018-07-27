---
title: Event Driven Architectures & Serverless
layout: post
date: 2018-07-23 1:36
tag:
- event-sourcing
- serverless
blog: true
---

After a 2.5 years journey building an event-driven services platform for a large financial organization, I’ve recently joined a new team in the middle of their on-going migration to the public cloud. One of the things we’ve been exploring is the Serverless model in order to release software with close to none operations effort.

Turns out there are significant commonalities between EDAs (Event-Driven Architectures) and Serverless which can lead to interesting advantages from the architectural point of view when applied to the appropriate business context. That was the topic of my recent presentation at the [North America XConf](https://www.thoughtworks.com/xconf-na).

<script async class="speakerdeck-embed" data-id="a35a11a8037a4266a54291d01f6723a1" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>

## Event Driven Architectures

An EDA is a distributed system where the communication between the functional components is asynchronous and implicit - in opposition to the more traditional service-oriented architectures where the communication is often times synchronous and explicit. EDAs rely on a message broker (AKA *Event Bus*) responsible for integrating producers and consumers.

<div style="text-align:center">
  <img class="image" src="/assets/images/eda.jpg" alt="EDA" style="max-width:600px">
  <figcaption>Elements of an EDA</figcaption>
</div>

There are [different flavors](https://martinfowler.com/tags/event%20architectures.html) of Event Driven Architectures, but the one I’d like to focus on is based on [Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html) (ES) and [Command Query Responsibility Segregation](https://martinfowler.com/bliki/CQRS.html) (CQRS).

Essentially, a CQRS application has two separate data models: *Command* (writes) and *Query* (reads). Although both models are part of the same bounded-context they are independent and communicate implicitly through the Event Bus. The Command side holds the event-sourced state of the application while the Query side maintains an aggregated projection of that state.

<div style="text-align:center">
  <img class="image" src="/assets/images/cqrs.jpg" alt="CQRS" style="max-width:600px">
  <figcaption>Internal structure of a ES/CQRS application</figcaption>
</div>

Overall, the benefits you’re likely to get when adopting an ES/CQRS EDA are:

* **Low coupling**. Services don’t communicate explicitly so that there is no direct dependency between them. It doesn’t really matter which service publishes an event as long as the consumer is able to deserialize the message from the Event Bus.
* **Resilience**. Services proactivelly consume events of their interest. As a consequence, they’re always aware of latest valid global state. That helps to avoid cascading failures when a given service goes down.
* **Performance**. All data is aggregated asynchronously so that there is no internal ad-hoc communication when a client request comes in.
* **Immutability**. As far as data goes, the source-of-truth is the *Event Store* - an immutable, append-only database. There is no current mutable state.
* **User intent**. Modeling changes as typed *Commands* often gives us more expressive power than using HTTP methods against resources directly.
* **Chronology**. With the Event Store we record every intermediate state of a user journey, not only the final state. That’s a powerful tool for UX analysis.
* **Audit Log by design**. Keeping an audit log is a quite strong requirement for certain domains. That comes for free when you have an Event Store.
* **Debugging**. Having access to the event history allows developers to better understand and debug the system behavior, specially when it comes to distributed transactions.

Check out my QCon talk from last year to get the details on this approach.

<div style="text-align:center">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/3H3hycR2HIQ" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
</div>

## Serverless

According to Mike Roberts’ [definition](https://www.martinfowler.com/articles/serverless.html), *Serverless* can  be described as an approach to build and deploy applications with low operations cost through the combination of *BaaS* and *FaaS* components.

* **BaaS** or Backend-as-a-Service refers to the infrastructure components managed by a cloud provider. Examples are [AWS S3](https://aws.amazon.com/s3/), [AWS DynamoDB](https://aws.amazon.com/dynamodb/), [Google BigQuery](https://cloud.google.com/bigquery/).
* **FaaS** or Function-as-a-Service refers to execution environments pre-defined and managed by a cloud provider. Examples are [AWS Lambda](https://aws.amazon.com/lambda/), [Google Cloud Functions](https://cloud.google.com/functions/), [Azure Functions](https://azure.microsoft.com/en-us/services/functions/).

The Serverless model aims to minimize configuration management and server provisioning up to the point where your cost of operations is close to zero. In order to accomplish that, you rely heavily on the offerings of the cloud provider of choice. The main benefits of this approach are:
* **Abstraction**. When you work with VMs you abstract the hardware; when you work with containers you abstract the kernel; when you work with cloud functions you abstract the execution environment, so that you can focus 100% on the application logic.
* **Deployment**. Your cloud provider fully manages deployments so you don’t need to worry about downtimes or strategies like Blue/Green or rollbacks.
* **Scalability**. Functions would dynamically scale by default without any custom configurations or infrastructure setup.
* **Monitoring**. Your cloud provider gives you access to logs, metrics, and alerts around your functions without the need of setting up monitoring tools yourself.
* **Cost**. The FaaS billing model is optimized for usage, so that you don’t get billed for idle time.

There is another important aspect which can not exactly be considered a benefit or a disadvantage: the **granularity** of a function. When it comes to *Microservices* we tend to think of a bounded context with correlated functionality running on its own process. A function, on the other hand, tends to be more granular. It often represents a single self-contained functionality running on a independent process. We can think of a service in Serverless as a combination of multiple functions and infra components. Although this granularity might enable more flexibility on deployments - as you can update parts of a service independently - it also adds more complexity when it comes to integration/contract testing. I plan to discuss that further in a future post.

## Event-Driven Serverless

Turns out a Serverless application is fundamentally an Event-Driven application. Functions are triggered by events - either from BaaS components or from other functions through a message broker. Examples of triggers on AWS:

* A file uploaded to an S3 bucket;
* Inserts on a DynamoDB table;
* A message published to an SNS topic;
* A CloudWatch alert.

Moreover, there is no direct communication between functions which makes them loosely coupled and more resilient - the same way as it happens in an EDA. (Even when it comes to HTTP, the communication actually happens through events produced by a *gateway* of some sort, like the [AWS API Gateway](https://aws.amazon.com/api-gateway/).)

Moving further, we can apply the CQRS/ES pattern when modeling services as functions. Here’s an example using AWS abstractions:

<div style="text-align:center">
  <img class="image" src="/assets/images/serverless-cqrs.jpg" alt="Serverless CQRS" style="max-width:600px">
  <figcaption style="margin-top:40px">Internal structure of a Serverless ES/CQRS application</figcaption>
</div>

The red lines on the diagram represent new data being written; the green lines represent existing data being read. Similarly to the non-serverless version, we also have the Command side on the top (*Commands*, *Event Store*, *Event Publisher*) and the Query side on the bottom (*Event Listener 1/n*, *Query DB*, *Queries*) as well as an Event Bus.

As you may have noticed, our service is composed by 5 functions and 4 BaaS components described below.
* The **API Gateway** represents the entry point for both command and query requests.
* Both the **Event Store** and the **Query DB** are implemented with DynamoDB.
* The **Event Bus** is implemented as a set of SNS topics.

As far as the data flow goes, we basically have a *Commands* function handling incoming command requests and inserting new domain events to the store. Those new events would be broadcasted by the *Event Publisher* function to the bus. All consumers are notified and - asynchronously - would update the query projections on the *Query DB*. The last *Queries* function simply responds to incoming queries from the *API Gateway*.

Check out a sample implementation of this idea on my Github: [http://gitbub.com/vvgomes/serverless-restaurant]()

## Concluding Thoughts

Here are a few takeaways I believe are important to keep in mind in case you are considering to adopt a Serverless EDA approach.

**Cost of operations** - If operations is a significant cost to your organization, moving to the Serverless model can be an interesting financial option, as you’d be able to transfer almost all the server provisioning and configuration management burden to the cloud provider.

**Vendor Lock-in** - On the other hand, once you adopt a particular cloud provider, you are pretty much restricted to the offerings of that provider. For instance: there is no [BigQuery](https://cloud.google.com/bigquery/) in AWS, there is no [RedShift](https://aws.amazon.com/redshift/) in GCP, etc.

**Customization** - Another point to consider is the level of customization you’ll need to introduce on infrastructure components in order to support the peculiarities of your problem domain. BaaS solutions are usually not very customization-friendly.

**Predictability** - Although the FaaS billing model is pretty interesting for situations where you don’t really know what kind of traffic to expect, for scenarios where the load is reasonably high and constant it is probably cheaper to just keep a few robust IaaS compute engines running permanently instead.

**Nature of the problem** - Overall, going Serverless - and even more - going Event-Driven is something that might not be applicable to certain domains. There are situations where using those patterns just doesn't make sense. After all, [there is no silver bullet](https://amzn.to/2NP0MNE).
