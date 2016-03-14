---
title: Clojure Flip
layout: post
date: 2016-02-08 22:48
tag:
- functional-programming
- clojure
blog: true
---

Recently, I found myself in need of a [flip](http://hackage.haskell.org/package/base-4.8.2.0/docs/Prelude.html#v:flip) function in [Clojure](http://clojure.org/). Since I could not find anything in the official documentation, Stackoverflow, or Freenode, I came up with this:

{% highlight clojure %}
(defn flip [f]
  (comp (partial apply f) reverse list))
{% endhighlight %}

Its behavior can be described like this (assuming [Midje](https://github.com/marick/Midje)):

{% highlight clojure %}
(facts "about flip"
  ((flip -) 1 2) => 1
  ((flip /) 1 2) => 2
  ((flip >) 1 2) => true
  ((flip str) 'foo 'bar 'baz) => "bazbarfoo")
{% endhighlight %}

Please, let me know in case you have a simpler solution ;)
