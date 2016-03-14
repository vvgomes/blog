---
title: "Ramda vs Lodash"
layout: post
date: 2015-09-03 22:48
tag:
- javascript
- functional-programming
blog: true
---

Javascript is almost certainly the most popular functional programming language in the world. However, in case you’re still using ECMA5 it is practically impossible to accomplish well crafted functional code without an utilities library.

Most people I know would adopt [Underscore](http://underscorejs.org/) or [Lodash](https://lodash.com/) in that case. However, after reading the [Mostly Adequate Guide to Functional Programming](https://github.com/MostlyAdequate/mostly-adequate-guide) and watching the “[Hey Underscore, You're Doing It Wrong!](https://www.youtube.com/watch?v=m3svKOdZijA)” talk by Brian Lonsdorf, I decided to give [Ramda](http://ramdajs.com/) a try in my last project.

The main advantages of Ramda, in my opinion, are the following:

- All Ramda functions are auto-curried;
- No side effects are allowed, unless you do it manually;
- The API for lists favors point-free code;
- There are lots of useful utilities, like `flip`, `pipe`, `complement`, `identity`, `lens`, etc.

The example bellow shows these Ramda capabilities in a direct comparison with the Lodash version.

{% gist vvgomes/451ea5ca2c65e87c92e4 %}

## Conclusion

In the context of ECMA5 Javascript, I feel like Ramda leads to more declarative code with less implicit side-effects. The fact you less frequently need to explicit mention your data and you don’t need to declare anonymous functions everywhere definitely reduces the amount of noise in your code and makes it fairly cheaper to maintain.
