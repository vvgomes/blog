---
title: The Plain Old Function Call
layout: post
date: 2012-08-30 22:48
tag:
- ruby
- programming
blog: true
---

During a design storm session one of these days, [we](https://github.com/victorykit/victorykit) got into a discussion on alternative approaches for an api interface. The main point in debate was basically the level of simplicity of each solution.

{% highlight ruby %}
# first idea
choose('color', ['red', 'black', 'green'], 'ie7')

#second idea
choose('color').from('red', 'black', 'green').ignore('ie7').any

{% endhighlight %}

To set the context, the intention behind the hypothetical example above is to randomly select one of the options (`’red'` or `’black'` or `’green'`) for a given aspect of the application (`’color'`) in order to perform some sort of [A/B testing](https://en.wikipedia.org/wiki/A/B_testing). Besides that, we also want to prevent this thing from running in some specific browsers (`’ie7'` in this example).

Conceptually, the main difference between the two ideas is that the first one is based on a **plain old function call** with explicit ordered arguments, while the second one uses **objects and method chaining** (AKA [Fluent Interface](http://www.martinfowler.com/bliki/FluentInterface.html)).

For some teammates, the first idea is the simpler one. They believe that approach describes more explicitly the api usage and is less complex to implement, since less data structures and language resources are required. *It’s just a plain old function call, there is no magic in there.* they say.

On the other hand, there was another group which was supporting the second idea. Their central argument was about readability: *It is simpler because it just reads better*. Additionally, they believe it eliminates any ambiguos interpretation on the last parameter role as well as in the statement side effect.

Well, even though both sides have their valid points, they are not really talking about the same kind of simplicity. Of course the first approach is simpler to **implement** because its grammar is restricted to limited set of very simple rules. And everybody pretty much knows how a function works in programming.

But turns out the second approach can be simpler on **maintenance**, since it gives more information about what’s going on without exposing implementation details. Also it requires less mental mapping between the computational representation and the real world thing (it’s closer to the natural language).

# Conclusion

Overall, if our goal is to get something working as fast as we can using a very simple approach to **create** stuff, investing in old-fashioned function calls works fine. However, if the intention is to build something that would be easier and faster to understand and, consequently, **maintain**, the fluent approach sounds better.

Lastly, there is an important implication in that decision we should consider: the **monetary cost**. The Software Engineer history has shown us that maintenance is the most expensive phase of the software life-cycle. And we all know that a given piece of code is usually read several times more than it is written. So, if the maintenance is the real bottleneck, it sounds smart to adopt an attitude towards simpler maintenance.
