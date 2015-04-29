---
layout: post
title:  "Promises - Gotchas and Anti Patterns"
date:   2015-04-10 11:06:38
comments: true
---

### tl;dr
- Flatten the promise chain whenever possible
- Straighten multiple nested promises with q.all() and q.spread()
- Don't break the promise chain. Make sure that the last promise is returned to the caller.
- Don't overuse deferreds. Use them to promisify callback functions. In case a function returns a promise, it's just better to form a chain with that same promise.

### The long version

For the past one year, I have been working on [1self](http://www.1self.co), a platform that unites all your personal data to provide you interesting insights. It's been an interesting product journey, and I'll write a separate post about that soon. We have made heavy use of Node.js to build the backend APIs for correlation and aggregation. In this post, I'd like to cover my journey of moving from [callback hell](http://callbackhell.com/) to the promise(d) land.

I am assuming you know/have heard about promises before. If not, see the [General Promise Resources on q.js wiki](https://github.com/kriskowal/q/wiki/General-Promise-Resources) for beginner level resources where people smarter than me have explained the concept really well. For this article, I'll focus on the problems I faced and the mistakes I made while using promises so that you don't make the same mistakes. Let's get right to it.

### 1. Promise hell?
We all know what a callback hell looks like. When starting with promises, one common mistake is to write promises in callback style.

{% gist chinmay185/869c9c8990ab56aba641 %}

In the above example, we're trying to get the user, fetch all tweets for that user and then update the timeline to display those tweets. The nesting of the ```getTweets()``` and ```updateTimeline()``` is unnecessary here. Now lets look at the example below:

{% gist chinmay185/45d356665dad0390a5c9 %}

Although this looks like a contrieved example (and in a way it is) it's possible to make such mistakes, especially if you're used to thinking in terms of callbacks.

#### Conclusion
Flatten the promise chain wherever possible.

### 2. Nested promises
Suppose we want to associate friendship between two users. For this, we would need to fetch both the users and then connect them as friends.

{% gist chinmay185/be28ef3450f9fad0eed6 %}

Ideally, we want to fetch both users in parallel and when both the calls return, we want to connect them as friends.

[q.all()](https://github.com/kriskowal/q#combination) to the rescue!

{% gist chinmay185/ef51e4a0ea820e3c1f98 %}

Here, ```q.all()``` takes an array of promises and turns it into a single promise for the whole array. In the v2 version, we have spread all those promises so that we can individually access the fulfilled values.

#### Conclusion
Straighten multiple nested promises with q.all() and q.spread()

### 3. Missing return
What does the following program print? In case you don't know, ```q.delay(500)``` causes a delay of 500ms before executing the ```then``` block.

{% gist chinmay185/00b76d4aebadacd4e273 %}

Well, you might think that the output here would be ```tweets: ['tweet1', 'tweet2']``` but if you were to run the code, you would see ```tweets: undefined``` as the output.

The reason is: we are missing a ```return``` before ```getTweets(user)``` statement.

This could be very difficult to spot, especially in a long promise chain. Hence, as a first debugging step for long promise chains, it's best to verify that the promise chain is not broken due to missing return.

#### Conclusion
Don't break the promise chain. Make sure that the last promise is returned to the caller.

### 4. Unnecessary use of deferred
If you have worked with callbacks a lot, chances are that you must be using [```q.deferred()```](https://github.com/kriskowal/q#using-deferreds) to promisify a function accepting a callback. However, we sometimes overuse deferreds and that results in the following code:

{% gist chinmay185/f9e629470d2e6e983684 %}

Here, ```userRepository.find(username)``` returns a promise, so we can fix (simplify) the code as follows:

{% gist chinmay185/0818a9c9a650d7ee88d8 %}

#### Conclusion
Don't overuse deferreds. Use them to promisify callback functions. In case a function returns a promise, it's just better to form a chain with the same promise.

### What next?
I am very fascinated by [ClojureScript](https://github.com/clojure/clojurescript) and [core.async](https://github.com/clojure/core.async) and plan to test ride those on a project. If you are a veteran promises user, I highly recommend checking out [David Nolen's](http://swannodette.github.io/) blog post on [CSP](http://swannodette.github.io/2013/07/12/communicating-sequential-processes/) where he talks about powerful ways of handling events.

In the [next post](http://chinmaynaik.in/posts/async-control-flow/), I will talk about using promises to build [asynchronous control flow patterns](http://book.mixu.net/node/ch7.html). I will also implement common async patterns found in [async.js](https://github.com/caolan/async) by using promises and JavaScript's native collection manipulation functions (map, reduce, forEach etc). Stay tuned!
