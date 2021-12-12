---
layout: post
title:  "Web3 Development With Clojurescript Pt. 1"
date:   2021-12-11 12:09:26 -0500
categories: clojurescript blockchain 
---

In the [first post of this series]({% post_url 2021-12-10-Web3-Development-With-Clojurescript-0  %}), we went over setting up a basic Clojurescript project. We wrote a basic component, `hello-component`, which renders a greeting when we pull up our page. Now we'll make it greet people by their wallet address. Let's get started.

## Interacting With The Blockchain

We'll be relying on the user to have a browser with a wallet plugin already installed, for example Metamask in Chrome. We will leverage metamask for all of our Web3 interactions, from querying user details to querying blockchain state. To do so, we will use the Javascript library [ethers.js](https://docs.ethers.io/v5/).

### Installing ethers.js

Like I mentioned in the previous post, shadow-cljs makes it really easy to use npm packages from Clojurescript. We can install ethers.js with the following command:

``` bash
npm install --save ethers
```

### Handling Promises

Most of the methods we use to query blockchain info will return Javascript Promise objects. There are [a few](https://clojurescript.org/guides/promise-interop) different ways of working with Promises from Clojurescript. We're going to leverage a really powerful library called [core.async](https://github.com/clojure/core.async) which makes it really easy to handle asynchronous code.

We'll add `core.async` to our dependencies in `shadow-cljs.edn`

```clojure
...
 :dependencies
 [[reagent "1.1.0"]
  [org.clojure/core.async "1.5.644"]]
...

```

## A Quick Note About State

We won't dive into the weeds of state management. There's a lot you can do better than what I'll show once your application grows in complexity. 

In this phase of the tutorial I'm simply trying to illustrate how to talk to the blockchain from Clojurescript and then show this information on the page. We'll worry about proper state management later.

We're only going to maintain one piece of state, which is the name by which we greet our user. The initial state is "Web3", but we will use Ethers.js to replace Web3 with the users' address, and Reagent to render this address on the page. We'll be using an `atom` to hold a string which contains the name we're using. Reagent provides an `atom` implementation of their own which adds some features around re-rendering components.

## Introducing Mutable State

Until now, our state was fully static. Now we're going to have a display name that changes when a user connects their wallet.

### Rendering Initial State

Let's create an `atom` to store this state, and we'll call it `addr`. We'll add the `reagent.core` import to `app.cljs`and declare a name for our atom. We'll pass this atom to our `hello-component` instead of the string we're passing now.

```clojure
(ns helloweb3.frontend.app
  (:require
   [helloweb3.frontend.views :as views]
   [reagent.core :as r]
   [reagent.dom :as rdom]))

(def addr (r/atom "Web3"))

(defn init []
  (rdom/render [views/hello-component addr]
               (-> js/document
                   (.getElementById "root"))))
```

Instead of passing a string to our `hello-component`, we're passing it an atom containing a string. The component will "dereference" the atom and pull out its contents, updating any time it changes.

In `views.cljs`, we will make a *one character* change to do this. We will change `name` to `@name`. The `@` character means "derefernce this atom and give me its value".

```clojure
(defn hello-component
  [name]
  [:div
   [:p "Hello " @name]])
```

