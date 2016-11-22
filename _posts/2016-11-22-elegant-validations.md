---
title: Elegant Validations in Javascript
layout: post
date: 2016-11-22 09:15
tag:
- javascript
- fp
blog: true
---

How do you model business rules validations? Although there seems to be a variety of approaches out there, some common abstractions are often present. Given an arbitrary data structure representing an entity in your domain, a validation algorithm will usually be composed by:

- A group of **predicates** to check business rules against the entity.
- A validation result which will either sign **success** or **failure**.
- A collection of validation **errors** in case of failure.

It’s a quite simple protocol. But many people still keep on re-writting validation utilities using slightly variant approaches, specially in the Javascript world.

![Searching validation on NPM](/assets/images/validation-at-npm.jpg)

## Throwing Runtime Errors

When it comes to validation errors, one of the most popular ways to handle them is by throwing **runtime errors** and/or **exceptions**. Particularly in Javascript, you’ll find people using native `Error` objects sometimes combined with `.catch()` calls in promises or just surrounded by `try/catch` blocks.

This approach however has two fundamental issues:

1. Validation errors are not necessarily exceptional. They are rather expected business scenarios. They are not in the same category as network/integration failures or invalid state caused by defects, for instance.

2. When you throw a runtime error, you skip the current thread execution, which makes it virtually impossible to accumulate all the validation errors together. See the example below.

{% highlight javascript %}
  
const isPasswordLongEnough = password => {
  if (!password.length > 6)
    throw new Error("Password must have more than 6 characters");
}

const isPasswordStrongEnough = password => {
  if (!(/[\W]/.test(a)))
    throw new Error("Password must contain special characters");
}

const validate = password => {
  isPasswordLongEnough(password);
  isPasswordStrongEnough(password);
};

validate("foo");
// => Error: Password must have more than 6 characters.

validate("rosesarered");
// => Error: Password must contain special characters.
  
{% endhighlight %}

Notice that the first `validate()` call above should have resulted in two errors instead of just one.

## Monadic Error Handling

Functional programmers familiar with [Either](https://hackage.haskell.org/package/base-4.9.0.0/docs/Data-Either.html) will usually think about validations and error handling in terms of functions and data containers. In Javascript, one of the most elegant implementations can be found in the [data.validation](https://github.com/folktale/data.validation) module from [Folktale](http://folktalejs.org/). ([Here](http://robotlolita.me/2013/12/08/a-monad-in-practicality-first-class-failures.html) is an excellent article by [@robotlolita](https://github.com/robotlolita) on the topic.)

That module will give you a `Validation` container that is implemented by two subtypes: `Success` or `Failure`. They act like `Right` and `Left` in `Either` but customized to be explicit and handle error accumulation. See the example below (from the docs).

{% highlight javascript %}
  
import Validation from "data.validation";
const Success = Validation.Success;
const Failure = Validation.Failure;

const isPasswordLongEnough = password =>
  password.length > 6 ?
    Success(password) :
    Failure(["Password must have more than 6 characters"]);

const isPasswordStrongEnough = password =>
  /[\W]/.test(password) ?
    Success(password) :
    Failure(["Password must contain special characters"]);

const validate = password =>
  Success(x => y => password)
    .ap(isPasswordLongEnough(password))
    .ap(isPasswordStrongEnough(password));

validate("foo");
// => Validation.Failure([
//      "Password must have more than 6 characters.",
//      "Password must contain special characters."
//    ])

validate("rosesarered");
// => Validation.Failure([
//      "Password must contain special characters."
//    ])

validate("rosesarered$andstuff");
// => Validation.Success("rosesarered$andstuff")
  
{% endhighlight %}

`data.validation` gives you (a) abstractions to represent success and failure, (b) the ability to accumulate errors, and (c) provides you with an [interface](http://docs.folktalejs.org/en/latest/api/data/validation/Validation.html) to manipulate the results in a functional way.

The one inconvenience though is the fact that error accumulation requires a workaround. Take a look at the `validate()` function above: we need to call `Success` with **two** levels of function nesting because we are running **two** validation predicates. The explanation from the docs:

> To aggregate the failures, we start with a Success case containing a curried function of arity N (where N is the number of validations), and we just use an `.ap`-ply chain to get either the value our Success function ultimately returns, or the aggregated failures.

That limits our ability to define a dynamic validation algorithm that would work with an arbitrary number of predicates. There is though a [solution](https://github.com/robotlolita/monads-as-first-class-failures/blob/master/validation.js) proposed in the article mentioned earlier. But I believe we can bridge that better by using one of my favorite JS libraries: [Ramda](http://ramdajs.com/).

## Ramda For The Rescue

The first thing we would like to do now is to be able to dynamically define an auto-curried version of a N-ary function (N being the number of predicates) and then pass it to `Success`. We can accomplish that with Ramda's `curryN()`.

{% highlight javascript %}
  
// ...

import { curryN } from "ramda";

const validate = password =>
  Success(curryN(2, (x, y) => password))
    .ap(isPasswordLongEnough(password))
    .ap(isPasswordStrongEnough(password));

validate("foo");
// => Validation.Failure([
//      "Password must have more than 6 characters.",
//      "Password must contain special characters."
//    ])
  
{% endhighlight %}

That's the first step. It doesn't give us much though. `curryN()` is defining an auto-curried version of the `(x, y) => password` function, so we don't need nesting anymore. But we are still not dynamic, since the number of predicates is hard-coded. In order to fix that, we need an intermediate step: pass the validations being applied (in the`.ap()` calls) as an argument.

{% highlight javascript %}
  
// ...

import { curryN, reduce } from "ramda";

const validate = (validations, password) =>
  reduce(
    (result, validation) => result.ap(validation(password)), // reducer fn
    Success(curryN(2, (x, y) => password), // initial value
    validations); // input

validate([isPasswordLongEnough, isPasswordStrongEnough], "foo");
// => Validation.Failure([
//      "Password must have more than 6 characters.",
//      "Password must contain special characters."
//    ])
  
{% endhighlight %}

Now, we are at least generic about the validations being applied to the initial `Success` state. For that we used Ramda's `reduce`. The next step is to eliminate the hardcoded values in the `curryN()` call.

{% highlight javascript %}
  
// ...

import { curryN, reduce, length, always } from "ramda";

const validate = (validations, target) =>
  reduce(
    (result, validation) => result.ap(validation(target)), // reducer fn
    Success(curryN(length(validations), always(target))), // initial value
    validations); // input

validate([isPasswordLongEnough, isPasswordStrongEnough], "foo");
// => Validation.Failure([
//      "Password must have more than 6 characters.",
//      "Password must contain special characters."
//    ])
  
{% endhighlight %}

The trick this time was to use the length of the `validations` array in order to determine how many curry levels we need. We also used Ramda's `always` to create a function that regardless of the arguments passed will always return `target`.

Finally, the `validate()` function is completely agnostic about the arbitrary validations. It became a generic validator which will work for any number of validation functions returning a `Validation` subtype, so we could even rename the `password` argument to just `target`.

There is still room for improvement though. Let's revisit our predicate based validations and try to remove the duplication in there. 

{% highlight javascript %}
  
// ...

const isPasswordLongEnough = password =>
  password.length > 6 ?
    Success(password) :
    Failure(["Password must have more than 6 characters"]);

const isPasswordStrongEnough = password =>
  /[\W]/.test(password) ?
    Success(password) :
    Failure(["Password must contain special characters"]);
  
{% endhighlight %}

It's clear that those two functions have the exact same logic except by (a) the **predicate** being executed and (b) the associated **error** message. Let's extract this duplicated logic and try to push it to the `validate()` function.

{% highlight javascript %}
  
// ...

const accumulateValidationResults = target => (acc, validation) =>
  acc.ap(validation.predicate(target) ? Success(target) : Failure([validation.error]));

const validate = (validations, target) =>
  reduce(
    accumulateValidationResults(target),
    Success(curryN(length(validations), always(target))),
    validations);

const passwordValidations = [
  {
    error: "Password must have more than 6 characters",
    predicate: password => password.length > 6
  },
  {
    error: "Password must contain special characters",
    predicate: password => /[\W]/.test(password)
  }
];

validate(passwordValidations, "foo");
// => Validation.Failure([
//      "Password must have more than 6 characters.",
//      "Password must contain special characters."
//    ])
  
{% endhighlight %}

And that was the last step of our journey. The `validate()` function is now able to determine validation results based only on a data structure representing arbitrary validations to be performed against a target object.

## Success / Failure

Why is it useful to return a container as a result?

Well, for at least two reasons:

1. `Success` and `Failure` have explicit semantics about what happened and can be used to drive subsequent actions without leaving the current thread.

2. Those containers provide a useful [interface](https://github.com/folktale/data.validation) to safely manipulate the results in a monadic way. You can, for instance, define callbacks for success and failure and fold `validate()` return value accordingly. 

{% highlight javascript %}
  
// ...

const passwordValidations = [
  {
    error: "Password must have more than 6 characters",
    predicate: password => password.length > 6
  },
  {
    error: "Password must contain special characters",
    predicate: password => /[\W]/.test(password)
  }
];

const onSuccess = password => "This is an excellent password!";
const onFailure = errors => "This is invalid because:\n" + errors.map(e => "* " + e).join("\n");
 
const message = validate(passwordValidations, "foo").fold(onFailure, onSuccess);
 
console.log(message);
// => This is invalid because: 
// => * Password must have more than 6 characters. 
// => * Password must contain special characters. 
  
{% endhighlight %}

## Predicado

The above exercise on defining a Folktale based `validate()` utility resulted in a tiny but brave library called [Predicado](https://www.npmjs.com/package/predicado). Check it out and feel free to send feedback.
