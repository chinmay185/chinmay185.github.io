---
layout: post
title:  "Asynchronous Control Flow Patterns"
date:   2015-04-20 11:06:38
comments: true
---

In the [previous post](http://chinmaynaik.in/posts/promises-gotchas-antipatterns), I covered how to avoid common pitfalls when using promises. In this post, we'll look at how promises and JavaScript's collection manipulation functions(read ```map``` and ```reduce```) can be combined together to implement a few async control flow patterns. 

Suppose you have the following requirement:
 
### I want to run an async operation for each value in a collection and when all async operations succeed, I want to do something else.

An example would be:

Given a list of product ids, you want to fetch those products from db, and find total cost of those products.

{% gist chinmay185/30d6b7d70868f99d610b %}

The important line in the above sample is ```productIds.map(getProductFromDb);```

This converts an array of values (```productIds```) to an array of promises (```productPromises```). When all the promises resolve, we get an array of products which can be used to calculate the total price.

Here, the database queries (```getProductFromDb()```) are executed simultaneously. However, what do we do when we have limited connections to the database? In that case, only a few queries will execute simultaneously. Other queries will be blocked until the previous ones release the db connection. The next scenario poses a similar problem.

### I want to run an async operation for each value in collection where each async operation is executed only when the previous one is completed.

A practical example for this might be an api with a rate limit. 

Suppose we want to access an api that has a rate limit of one request per client. So we'll be able to make only one api request at a time. Now, if we have a list of api urls, we won't be able to ```map``` over the list as it will fire all api requests simultaneously. We want to fire the second request only after we receive a response for the first request, and so on.

So here's what we can do.

{% gist chinmay185/035a07cc0234a75ece03 %}

The important part in the sample above is the call to ```reduce```. 

We inject a resolved promise as a starting value to ```reduce```. This starts the promise chain. We then fire off a request to the rate limited api and once it is completed, we save the response. The promises from the further iterations are accumulated to form a single promise chain which ```reduce``` returns. When this promise chain is completed, we print the ```"all done."``` message to the console. 

But what if we had a custom requirement like the one below.

### I want to iterate over a collection performing an async task for each item in the collection, but consume the results in the same order.

Here, we're saying that it's okay to run the async tasks simultaneously but we want to process the result in the given order. If you think about it, this is a combination of the two samples we saw earlier.

A practical example for this might be:

Suppose a large file is divided into multiple parts and those parts are stored on the server. We have an array of urls to those parts as given below:

{% highlight js %}
var fileParts = ["http://myfiles.com/file1/part1", 
		"http://myfiles.com/file1/part2", 
		"http://myfiles.com/file1/part3"];
{% endhighlight %}

We want to fetch all the parts in parallel, but in order to create a consistent file, we need to traverse the parts in the given order.

Here's how we can do this by combining ```map``` and ```reduce``` 

{% gist chinmay185/ecbdb8ed141511499323 %}


### Summary

In this post, we saw how ```map``` and ```reduce``` can be combined with promises to construct simple async control flows. For more advanced requirements, I'd highly recommend checking out [bluebird](https://github.com/petkaantonov/bluebird) and its [API](https://github.com/petkaantonov/bluebird/blob/master/API.md). With bluebird, you can implement more advanced async control flows like initiating a race between multiple promises, managing promise timeouts, setting concurrency limits when working with multiple promises and much more.

On a side note, I recently found this awesome link about [designing promises from scratch](https://github.com/kriskowal/q/blob/v1/design/README.js). I believe the best way to learn about promises in depth is to actually try and build them from scratch! This link provides great insights about the design and constraints of building your own promise library.