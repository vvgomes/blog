---
title: "The Future Is Functional"
layout: post
date: 2016-09-15 01:48
tag:
- fp
- oo
blog: true
---

This week my friend [@ffafara](https://twitter.com/ffafara) and I joined a meetup organized by the [Philadelphia Java Users Group](https://www.meetup.com/PhillyJUG/events/231548149). There was a presentation about some interesting details on Java SE 8 and also some highlights on what's coming up in version 9. The discussion was quite nice, specially regarding the Functional Programming features like lambdas, functional interfaces, streams, and optionals.

One thing called our attention though: the presenter's opinion about the relevance of FP concepts in the future of mainstream programming. Although he highlighted the potential benefits of those concepts in the context of Java, he does **not** believe FP could be a replacement for the Object Oriented mindset when it comes to general application development.

This point of view, though, is not at all a rare one. It's quite frequent to observe people being averse to FP (usually, influenced by old-fashioned [software architects](http://www.martinfowler.com/articles/designDead.html#DoYouWannaBeAnArchitectWhenYouGrowUp)), not only due to the intellectual pain of the paradigm shift, but mainly because they still believe OO should be the primary tool for building systems.

Well, it seems they are missing an important point here.

What they probably don't yet realize is that OO itself is the fundamental reason behind most of the pain points in their daily programming routine:

- Null pointers every now and then;
- Unpredictable side effects;
- Artificial data abstractions obscuring the domain concepts;
- Layers, and more layers of mutable state;
- Flaky unreliable tests;
- You name the next one...

In a world every day more distributed and asynchronous those inconveniences are gradually becoming more apparent and frequent. The reason is simple: OO's basic assumption is **mutable state**. And mutable state doesn't deal well with the increasingly simultaneity of modern computer systems. Let me call [Uncle Bob](https://twitter.com/unclebobmartin) to explain that to us:

> You are almost certainly reading this on the screen of your computer. How many cores does it have? I'm typing this article on a MacBook Pro with 4 real cores. My previous laptop had two cores. And the one before that had just one. My next laptop will truly have eight cores; and the one after that will likely have 16. How are you going to take advantage of every computer cycle available to you when your computer has 4096 cores in it?

[FP Basics E1: What's Functional Programming all about?](https://8thlight.com/blog/uncle-bob/2012/12/22/FPBE1-Whats-it-all-about.html) by Uncle Bob.

> Honestly, we programmers can barely get two Java threads to cooperate. For over half a century programmers have made the observation that the processes running in a computer are concurrent, not simultaneous. Well, boys and girls, welcome to the wonderful world of simultaneity! How are you going to deal with it?

[Pure functions](https://en.wikipedia.org/wiki/Pure_function) impose us two clear constraints: **explicit return values** and **no side effects**. These constraints are important enablers for thread safety and predictable executions. When there are no true *variables*, there is nothing harmfully unexpected one thread can cause to another. Therefore, parallelizing computation becomes trivial. Besides that, when data is immutable and behavior is idempotent, your code becomes easier to test and debug, since people are no longer allowed to sneak in their nasty side effect based workarounds everywhere.

> Clearly, if the value of a memory location, once initialized, does not change during the course of a program execution, then there's nothing for the 131072 processors to compete over. So that's the big deal about functional languages; and it is one big fricking deal.

Of course OO is still a good choice for most single threaded applications. However, in a world every day more and more [event-driven](https://vimeo.com/131636650), asynchronous, and distributed, the insistence on the OO supremacy has been leading to fragile systems which are hard to test, debug, and extend. FP, on the other hand, fits perfectly the requirements of that scenario. That's why we've been observing an increasing focus on FP languages, libraries, and tools by big tech companies like Facebook for instance. Although FP might be challenging to learn in the beginning, the passionate programmer most definitely finds it a fascinating and beautiful concept, which is, above everything else, **powerful**.

I'll leave you with Uncle Bob again:

<iframe src="https://player.vimeo.com/video/97514630" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/97514630">Robert C. Martin - Functional Programming: What? Why? When?</a> from <a href="https://vimeo.com/ndcconferences">NDC Conferences</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

