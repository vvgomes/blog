---
title: "The Beauty of Recursion"
layout: post
date: 2016-03-31 11:00
tag:
- clojure
blog: true
---

[Recursion](https://en.wikipedia.org/wiki/Recursion) is one of the most fascinating and elegant concepts in Computer Science. The feeling of figuring out recursive algorithms & data structures is a special one for a software developer. In this context, there is probably no better source than [The Little Schemer](http://amazon.com/dp/B00HRFIVOS) book, for both understanding and satisfying your recursive appetite.

<div style="text-align:center">
  <img class="image"
    src="http://ecx.images-amazon.com/images/I/41n1PjcsAKL._SX371_BO1.jpg"
    alt="The Little Schemer book cover">
</div>

Although this might seen like just a book about [Scheme](https://en.wikipedia.org/wiki/Scheme_%28programming_language%29), I'd say this is mostly a book about **thinking recursively**. Based entirely on questions, puzzles, and exercises which guide you gradually from the most basic concepts, like *atoms* and *s-expressions*, to the most mind-blowing ones, like the [Y Combinator](https://en.wikipedia.org/wiki/Fixed-point_combinator#Y_combinator), the teaching method is quite engaging and very unusual when compared to traditional programming books.

> **Does your hat still fit?**
>
> Perhaps not after such a mind stretcher.

The Little Schemer seems to be a popular book among **Clojure** developers like my friend [Marcos](http://www.marcosccm.io/posts/the-little-schemer). Even though there are idiomatic differences to Scheme, it is totally possible to implement all the exercises in the book using Clojure. Here is my test-driven version: [github.com/vvgomes/little-schemer](https://github.com/vvgomes/little-schemer)