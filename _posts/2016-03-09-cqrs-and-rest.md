---
title: "CQRS & REST"
layout: post
date: 2016-03-09 22:44
tag:
- programming
- cqrs
- rest
blog: true
---

Assuming:

* [This](http://martinfowler.com/bliki/CQRS.html) CQRS definition;
* [HATEOAS](https://en.wikipedia.org/wiki/HATEOAS);
* Awareness of the existing [discussion](https://groups.google.com/forum/#!msg/dddcqrs/CrDAI2RrLNQ/uy6MDz6SDQAJ);

CQRS will tell you to divide your application in two major architectural components: **commands** & **queries**. A command represents an user intent - all changes in the state of the application should be initiated exclusively by commands. Queries, on the other hand, represent questions an user can ask about the application state.

When it comes to exposing a REST-like API, it is important that a CQRS application makes it explicit the separation between commands and queries, so that clients can dynamically build [task-based user interfaces](http://codebetter.com/gregyoung/2010/02/16/cqrs-task-based-uis-event-sourcing-agh/). Assuming a JSON over HTTP API, that explicit separation could be accomplished by exposing commands as **resources**.

{% highlight bash %}
  GET /orders/de305d54-75b4-431b-adb2-eb6b9e546014/commands
{% endhighlight %}

{% highlight json %}
  {
    "links": [
      {
        "rel": "Place",
        "href": "/orders/de305d54-75b4-431b-adb2-eb6b9e546014/commands/place"
      },
      {
        "rel": "Cancel",
        "href": "/orders/de305d54-75b4-431b-adb2-eb6b9e546014/commands/cancel"
      },
      {
        "rel": "Modify",
        "href": "/orders/de305d54-75b4-431b-adb2-eb6b9e546014/commands/modify"
      }
    ]
  }
{% endhighlight %}

In the example above, all the possible commands that can be sent in the context of a given resource (`/orders/{id}`) are available as a nested resource collection (`/commands`). 

## Verbs

One caveat that might sound a little odd is the fact that the command URI contains the command name - which is usually a **verb** denoting an action to be performed on the parent entity. In some extent, that differs from regular REST where resources are always named after nouns and stateful operations on them are always performed by using the standard HTTP methods (POST, PUT, PATCH, DELETE).

In CQRS, though, the standard HTTP methods are not flexible enough to represent all possible commands associated to an entity. The example above demonstrates that: an existing *order* can be *placed*, *canceled*, or *modified*. In traditional REST style, these operations would end up being performed by sending a PUT (or PATCH) request to the *order* resource, which results in not enough context to identify the user intent.

In the proposed approach however, the point of view is slightly different: instead of modifying the `/orders/{id}` resource, we would "send a new command" using POST with a command URI and a (optinal) request body. Like this:

{% highlight bash %}
  POST /orders/de305d54-75b4-431b-adb2-eb6b9e546014/commands/place
{% endhighlight %}

That gives us the flexibility of defining custom commands as well as making it easy to API clients to discover them, as the command name is part of the URI. Additionally, commands are preferably asynchronous operations, so that the result of the request above would ideally be a 202 (Accepted). Sometimes it is also useful to return a unique id for newly created resources or command status.

## Embedded Commands

In many cases, it'd be useful for API clients to request a resource and get back the command list as part of the response, like this: 

{% highlight bash %}
  GET /orders/de305d54-75b4-431b-adb2-eb6b9e546014
{% endhighlight %}

{% highlight json %}
  {
    "created_at": "2016",
    "status": "opened",
    "links": [ ],
    "commands": [
      {
        "rel": "Place",
        "href": "/orders/de305d54-75b4-431b-adb2-eb6b9e546014/commands/place"
      },
      {
        "rel": "Cancel",
        "href": "/orders/de305d54-75b4-431b-adb2-eb6b9e546014/commands/cancel"
      },
      {
        "rel": "Modify",
        "href": "/orders/de305d54-75b4-431b-adb2-eb6b9e546014/commands/modify"
      }
    ]
  }
{% endhighlight %}

That would make possible for a Javascript client, for instance, to build an UI like this (assuming *pseudo-angularjs*):

{% highlight html %}
  <ul>
    <li ng-repeat="command in order.commands">
      <button ng-click="send(command)">
        {% raw %}{{ command.rel }}{% endraw %}
      </button>
    </li>
  </ul>
{% endhighlight %}

## Conclusion

CQRS provides you with a flexible way to build scalable asyncronous systems, specially when combined with [event sourcing](http://www.martinfowler.com/eaaDev/EventSourcing.html). Addopting REST over HTTP as the delivery mechanism can be tricky in that context. The model described above is one way to do accomplish the combination of the two techniques. Let me know in case you have alternative ideas.


[1]: http://daringfireball.net/projects/markdown/
[2]: http://www.fileformat.info/info/unicode/char/2163/index.htm
[3]: http://www.markitdown.net/
[4]: http://daringfireball.net/projects/markdown/basics
[5]: http://daringfireball.net/projects/markdown/syntax
[6]: http://kune.fr/wp-content/uploads/2013/10/ghost-blog.jpg
