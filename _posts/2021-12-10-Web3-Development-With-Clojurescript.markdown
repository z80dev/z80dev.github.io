---
layout: post
title:  "Web3 Development With Clojurescript"
date:   2021-12-10 18:09:26 -0500
categories: jekyll update
---
Clojure has been my favorite language for a long time. It is not without its faults, but it is the language in which I can express my ideas in the most efficient way. It has a powerful macro system that lets you mold the language to your needs. Most Clojure code can run on the JVM as well as in a Javascript runtime (as Clojurescript). This lets us leverage two rich package ecosystems with a convenient lisp syntax.

While much of the code will be self-explanatory, a familiarity with Clojure (or other lisps) will be helpful.

## What We Will Cover

This series will walk you through all the steps of creating a Web3 dApp. We'll dive into frontend in Clojurescript and write some smart-contracts in Solidity. This is mainly targetting EVM blockchains, but its possible I'll expand the series to cover integrating Solana support to our site.

In this first post, we'll go over setting up your project's dependencies in order to call Javascript Web3 libraries from Clojurescript. We'll create convenient wrappers around some important functions which will let us write idiomatic Clojurescript which interacts with the blockchain.

In later posts, we'll write some smart contracts of our own in Solidity, and then hook them up to our frontend so we can write code in Clojurescript that lets us interact with the code we've deployed on the Blockchain.

## Getting Started

Writing Clojure code is pretty simple and doesn't require much boilerplate. We will need to install a few dependencies though. 

### Installing Dependencies

#### Clojure & shadow-cljs

Install Clojure by following [this guide](https://clojure.org/guides/getting_started).

shadow-cljs is the easiest (imo) way to compile your clojurescript to execute in the browser. It makes it dead-simple to integrate npm dependencies into your Clojurescript code.

Follow the steps to install [shadow-cljs](https://github.com/thheller/shadow-cljs) as well as the steps laid out under their [Quick Start](https://github.com/thheller/shadow-cljs#quick-start) guide to get us on the same page. This will walk you through the basics of getting a Clojurescript project running. We will modify this code in order to add Web3 capabilities. I used the name `helloweb3` instead of `acme`. This makes no difference other than the name used for some files and namespaces. 

#### Reagent

[Reagent](https://github.com/reagent-project/reagent) is a minimalistic, react-like library for Clojurescript. I hope you come to love it as much as I do.

Add it to your dependencies in `shadow-cljs.edn`:

{% highlight clojure %}
;; shadow-cljs configuration
{:source-paths
 ["src/dev"
  "src/main"
  "src/test"]

 :dependencies
 [[reagent "1.1.0"]]

 :dev-http {8080 "public"}
 :builds
 {:frontend
  {:target :browser
   :modules {:main {:init-fn helloweb3.frontend.app/init}}
   }}}
{% endhighlight %}

and run this in your terminal from within the project directory:

{% highlight bash %}
npm i react react-dom
{% endhighlight %}

and 

{% highlight bash %}
npx shadow-cljs compile frontend // make sure our dependencies are installed
{% endhighlight %}

Now we're ready to get started writing some useful code!

## Saying hello

Let's start by creating a reagent component that will say hello to whoever we're dealing with. Create the file `src/main/helloweb3/frontend/views.cljs` and put the following code in it:


{% highlight clojure %}
(ns helloweb3.frontend.views)

(defn hello-component ;; components are just functions
  ;; accepts one argument, bound to the symbol `name` 
  ;; returns a vector describing the structure of what we want to render
  [name] 
  [:div
   [:p "Hello " name]])
{% endhighlight %}

We can then modify `app.cljs` to use this component instead of the hardcoded text it has now:

{% highlight clojure %}
(ns helloweb3.frontend.app
  (:require
   [helloweb3.frontend.views :as views]
   [reagent.dom :as rdom]))

(defn init []
  (rdom/render [views/hello-component "Web3"]
               (-> js/document
                   (.getElementById "root"))))
{% endhighlight %}

Start shadow-cljs by running

{% highlight bash %}
npx shadow-cljs watch frontend
{% endhighlight %}

and then navigate to [http://localhost:8080](http://localhost:8080) where you should see the following:

![hello web3](/assets/site1.png)
