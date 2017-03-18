---
title: Dependency Injection in Javascript
layout: post
date: 2017-03-17 03:55
tag:
- javascript
- fp
blog: true
---

If you like to develop your Javascript applications in a functional style, often times you have to deal with [Dependency Injection](https://martinfowler.com/articles/injection.html#FormsOfDependencyInjection) when it comes to unit-test functions that depend on other functions. Fortunately, with ES6 [default parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters) DI becomes quite simple. Let’s see an example.

## Example: An API Client

Most client applications need to communicate with a remote server, So let’s build a simple API client module with a few functions. The end goal looks like this:

{% highlight javascript %}
// an usual TODO app...

import * as apiClient from "./lib/api.client";

apiClient
  .addTodo({ text: "wash dishes" })
  .then(todo => console.log("New todo added:", todo))
  .catch(errors => console.log("Something went wrong:", errors));

//=> New todo added: { id: "1", text: "wash dishes", done: false }

// ...

apiClient
  .fetchTodos()
  .then(todos => console.log(todos))
  .catch(errors => console.log("Something went wrong:", errors));

//=> [ { id: "1", text: "wash dishes", done: false } ]
{% endhighlight %}

Our client is just a namespace of functions, each one representing a different HTTP request to a remote server. To send the actual request we intend to use [fetch](https://github.com/matthew-andrews/isomorphic-fetch) which returns a Promise wrapping the response. So the `fetch` function will be our dependency.

## Starting With Tests

Let’s write a few unit test cases to drive the development of our functions. In order  to test them in isolation we need to inject a fake `fetch` function as we don't want to exercise the real HTTP requests.

{% highlight javascript %}
// test/api.client.test.js
import * as client from "../lib/api.client";
import { expect } from "chai";
import { stub } from "sinon";

describe("api client", () => {
  let fakeFetch;

  beforeEach(() => {
    fakeFetch = stub();
  });

  describe("fetchTodos()", () => {
    it("retrieves all todos", () => {
      const fakeResponse = Promise.resolve({
        status: 200,
        json: () => Promise.resolve([{
          id: "1",
          text: "walk the dog",
          done: false
        }])
      });

      fakeFetch.withArgs("/api/todos").returns(fakeResponse); 

      return client.fetchTodos(fakeFetch).then(todos => {
        expect(todos).deep.eq([{
          id: "1",
          text: "walk the dog",
          done: false
        }]);
      });
    });
  });
});
{% endhighlight %}

We stubbed `fetch` to return an arbitrary response and passed as an argument to `fetchTodos`. That can be implemented as a default parameter.

{% highlight javascript %}
// lib/api.client.js
import defaultFetch from "isomorphic-fetch";

export const fetchTodos = (fetch = defaultFetch) =>
  fetch("/api/todos").then(res => res.json());
{% endhighlight %}

We're importing the real `fetch` implementation as the default argument value. This way, we keep the proposed interface unchanged. Let's add a test for `addTodo` as well.

{% highlight javascript %}
// test/api.client.test.js
import * as client from "../lib/api.client";
import { expect } from "chai";
import { stub, match } from "sinon";

describe("api client", () => {
  let fakeFetch;

  beforeEach(() => {
    fakeFetch = stub();
  });

  // ...

  describe("addTodo()", () => {
    it("creates a new todo successfully", () => {
      const requestData = match
        .has("method", "POST")
        .and(match.has("headers"))
        .and(match.has("body", JSON.stringify({ text: "wash dishes" })));

      const fakeResponse = Promise.resolve({
        status: 202,
        json: () => Promise.resolve({
          id: "2",
          text: "wash dishes",
          done: false
        })
      });

      fakeFetch.withArgs("/api/todos", requestData).returns(fakeResponse); 

      return client.addTodo({ text: "wash dishes" }, fakeFetch).then(todo => {
        expect(todo).deep.eq({
          id: "2",
          text: "wash dishes",
          done: false
        });
      });
    });
});
{% endhighlight %}

Here again, we pass the `fakeFetch` as an argument to `addTodo`. The implementation will be similar as the one for `fetchTodos`.

{% highlight javascript %}
// lib/api.client.js
import defaultFetch from "isomorphic-fetch";

export const fetchTodos = (fetch = defaultFetch) =>
  fetch("/api/todos").then(res => res.json());

export const addTodo =  (todo, fetch = defaultFetch) =>
  fetch("/api/todos", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ text: todo.text })
  }).then(res => res.json());
{% endhighlight %}

At this point, we can remove the code duplication by moving the default parameter to the namespace level. For that, we'll define a `createApiClient` function. Let's change the tests accordingly.

{% highlight javascript %}
// lib/api.client.test.js
import createApiClient from "../lib/api.client.";
import { expect } from "chai";
import { stub, match } from "sinon";

describe("api client", () => {
  let fakeFetch;
  let client;

  beforeEach(() => {
    fakeFetch = stub();
    client = createApiClient(fakeFetch)
  });

  describe("fetchTodos()", () => {
    it("retrieves all todos", () => {
      const fakeResponse = Promise.resolve({
        status: 200,
        json: () => Promise.resolve([{
          id: "1",
          text: "walk the dog",
          done: false
        }])
      });

      fakeFetch.withArgs("/api/todos").returns(fakeResponse); 

      return client.fetchTodos().then(todos => {
        expect(todos).deep.eq([{
          id: "1",
          text: "walk the dog",
          done: false
        }]);
      });
    });
  });

  describe("addTodo()", () => {
    it("creates a new todo successfully", () => {
      const requestData = match
        .has("method", "POST")
        .and(match.has("headers"))
        .and(match.has("body", JSON.stringify({ text: "wash dishes" })));

      const fakeResponse = Promise.resolve({
        status: 202,
        json: () => Promise.resolve({
          id: "2",
          text: "wash dishes",
          done: false
        })
      });

      fakeFetch.withArgs("/api/todos", requestData).returns(fakeResponse); 

      return client.addTodo({ text: "wash dishes" }).then(todo => {
        expect(todo).deep.eq({
          id: "2",
          text: "wash dishes",
          done: false
        });
      });
    });
  });
});
{% endhighlight %}

As you can see in the new version of the tests, we're now injecting `fakeFetch` to `createApiClient`. That new function acts like a contructor: it builds the namespace and make the injected `fetch` available to the entire namespace scope. Let's make that change to the application code.

{% highlight javascript %}
// lib/api.client.js
import defaultFetch from "isomorphic-fetch";

const createApiClient = (fetch = defaultFetch) => ({
  fetchTodos: () =>
    fetch("/api/todos").then(res => res.json()),

  addTodo: todo =>
    fetch("/api/todos", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ text: todo.text })
    }).then(res => res.json()),
});

export default createApiClient;
{% endhighlight %}

With the refactoring above, our initial code snippet will have to change a little bit. Now we have to call first `createApiClient` instead of just importing all functions.

{% highlight javascript %}
// an usual TODO app...

import createApiClient from "./lib/api.client";
const apiClient = createApiClient();

apiClient
  .addTodo({ text: "wash dishes" })
  .then(todo => console.log("New todo added:", todo))
  .catch(errors => console.log("Something went wrong:", errors));

//=> New todo added: { id: "1", text: "wash dishes", done: false }

// ...

apiClient
  .fetchTodos()
  .then(todos => console.log(todos))
  .catch(errors => console.log("Something went wrong:", errors));

//=> [ { id: "1", text: "wash dishes", done: false } ]
{% endhighlight %}

The code above represent a sample usage of our module. Notice that the calling code doesn't have to pass `fetch` to `createApiClient` since we used a default parameter. Besides that, we are now able to add more functions to the module (like `toggleTodo`, for instance) and have `fetch` available to them.

## Conclusion

ES6 default parameters enable functional DI in simple and elegant way. The approach demonstrated in this post suggests a new point of view on function arguments:

- **The regular arguments** - the arguments which matter to the application code.
- **Dependencies** - collaborator functions defaulting to the real implementation and stubbed in unit tests.

Check out the complete example code at [github.com/vvgomes/es6-di-example](https://github.com/vvgomes/es6-di-example).

