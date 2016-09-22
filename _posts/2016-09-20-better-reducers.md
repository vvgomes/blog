---
title: Maybe Better Reducers
layout: post
date: 2016-09-20 22:48
tag:
- javascript
- fp
- redux
blog: true
---

[Redux](http://redux.js.org/) is one of them most elegant ideas in the Javascript ecosystem in the last few years. Inspired by Facebook's [Flux](https://facebook.github.io/flux/) architecture, this small but brave library helps you to build applications based primarily on immutable data structures and pure functions. Folks with bias on Functional Programming (like me) will feel at home with Redux.

> In summary, a Redux app is a container for a immutable data structure representing the **current state**. That container accepts **actions** which are just plain objects describing user intent. Once an action is accepted, a **reducer** function runs and creates the **next state** based on the action and the current state. The reducer function then is the core of a Redux app.

## The Reducer

Redux doesn't impose any restrictions on the way you define your reducer, as long as it comforms with the following signature:

`reducer :: (State, Action) -> State`

Most examples all over the internet - including the one in the official [documentation](http://redux.js.org/docs/basics/Reducers.html) - will suggest you to define the reducer function using a `switch` statement on the action type. 

{% highlight javascript %}

function reducer (state = [], action) {
  switch (action.type) {
    case "ADD_TODO":
      return state.concat({
        text: action.text,
        completed: false
      });
    case "TOGGLE_TODO":
      return state.map((todo, index) =>
        index === action.index ?
          Object.assign({}, todo, { completed: !todo.completed }): todo
      );
    default:
      return state;
  }
}
{% endhighlight %}

The example above is literally the suggested reducer in the official Redux docs. I wonder if people are really following this approach (I hope not). As you can see, the code above is very busy to read and not very modular. Hopefully, we can do better than that. Let's see.

## Introducing Maybe

The algorithm of the reducer above can be described as:

- Chooses a behavior by pattern-matching the `action.type`;
- Executes the matching behavior returning a new state;
- Returns the unchanged state if no match.

Turns out that logic can be accuratelly implemented using a [Maybe](https://wiki.haskell.org/Maybe) container, as the Haskell programmers would probably suggest here. In Javascript, we can use the Maybe object from the amazing [Folktalke](http://folktalegithubio.readthedocs.io/en/latest/api/data/maybe/Maybe.html?highlight=maybe) lib. Let's see the result.

{% highlight javascript %}
import Maybe from "data.maybe";

const reducer = (state, action) => 
  Maybe
    .fromNullable(actionHandlers[action.type])
    .map(handler => handler(state, action))
    .getOrElse(state);
{% endhighlight %}

The Maybe version of our reducer relies on an addional step: we extracted all the action-specific behaviors to a map. We called it `actionHandlers`.

{% highlight javascript %}
const ADD_TODO = (state, action) =>
  state.concat({ text: action.text, completed: false });

const TOGGLE_TODO = (state, action) =>
  state.map((todo, index) =>
    index === action.index ?
      Object.assign({}, todo, { completed: !todo.completed }): todo);

const actionHandlers = { ADD_TODO, TOGGLE_TODO };
{% endhighlight %}

Now each action handler is an independent function with a single and succint responsibility. The same applies for the reducer which uses the Maybe container to drive the logic around the pattern matching. And just that.

## Conclusion

Redux is great not only because it is so simple and concise but also because it is based on strong Functional Programming concepts. Additionally, Maybes, Eithers, Validations and other containers from [Folktale](http://folktalejs.org/) are very handy in a diversity of situations, but specially useful when it comes to [functional error handling](http://robotlolita.me/2013/12/08/a-monad-in-practicality-first-class-failures.html). The reducer example is just a small demonstration on how well the Redux idea fits well with other FP based libs. For a complete and more detailed example take a look at my Redux + Folktale + Ramda [Todo App](https://github.com/vvgomes/redux-todo/).

