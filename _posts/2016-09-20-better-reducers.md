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

This is literally the reducer suggested in the Redux official docs. I wonder if people are really following this approach (I hope not). As you can see, the code above is very busy to read and not at all modular. Hopefully, we can do better than that. Let's see.

## Introducing Maybe

The algorithm of the reducer we just saw can be described as:

- It chooses a behavior by pattern-matching the `action.type`;
- Then, it executes the matching behavior returning a new state;
- In case there is no match, it returns the unchanged state.

Turns out that logic can be accuratelly implemented using a [Maybe](https://wiki.haskell.org/Maybe) container, as the Haskell programmers would probably suggest here. Java 8 developers can think of it as an [Optional](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html). In Javascript, we can get Maybe from the amazing [Folktale](http://folktalegithubio.readthedocs.io/en/latest/api/data/maybe/Maybe.html?highlight=maybe) lib. Let's see the result.

{% highlight javascript %}
import Maybe from "data.maybe";

const reducer = (state, action) => 
  Maybe
    .fromNullable(actionHandlers[action.type])
    .map(handler => handler(state, action))
    .getOrElse(state);
{% endhighlight %}

The Maybe version of our reducer relies on an addional step: we extracted all the action-specific behaviors to separate functions, which are accessible in a map we called `actionHandlers`.

{% highlight javascript %}
const ADD_TODO = (state, action) =>
  state.concat({ text: action.text, completed: false });

const TOGGLE_TODO = (state, action) =>
  state.map((todo, index) =>
    index === action.index ?
      Object.assign({}, todo, { completed: !todo.completed }): todo);

const actionHandlers = { ADD_TODO, TOGGLE_TODO };
{% endhighlight %}

Now each action handler is an independent function with a single and succint responsibility. The same applies to the reducer which uses the Maybe container to drive the logic around the pattern matching. And just that.

Those who are not familiar with Maybe can think of it as a container for a value that might or might not be present. In our example, an action type might or might not be found in the map. The `.fromNullable` constructor does that for us: it gets an actual function or it gets nothing in case there is no match. Next, we call `.map` which executes a present behavior. The interesting thing about `.map` is that is does not run in case the Maybe is holding an empty value. The last call `.getOrElse` returns the value from inside the container if the value is present. Otherwise, it uses the default argument, which in our case is the unchanged `state`.

## Conclusion

Redux is great not only because it is so simple and concise but also because it is based on strong Functional Programming concepts. Additionally, Maybes, Eithers, Validations and other containers from [Folktale](http://folktalejs.org/) are very handy in a diversity of situations, but specially useful when it comes to [functional error handling](http://robotlolita.me/2013/12/08/a-monad-in-practicality-first-class-failures.html). The reducer example is just a small demonstration on how well the Redux idea fits with other FP based libs. For a complete and more detailed example take a look at my Redux + Folktale + Ramda [Todo App](https://github.com/vvgomes/redux-todo/).

### Update

The idea behind this post inspired the development of the [Redux Definer](https://www.npmjs.com/package/redux-definer) NPM package. Pretty handy to avoid reducer boilerplate code. In case you also want to avoid action creators boilerplate, you can go straight to [Redux Actions](https://www.npmjs.com/package/redux-actions).

