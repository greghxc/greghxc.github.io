---
layout: post
title:  "Futures and Futures and ListenableFutures in Scala"
author: "Greg"
category: cats-and-code
comments: true
---
![Ichabod the Cat](https://farm5.staticflickr.com/4373/36812923541_2c311bd93f_h.jpg "Ichabod the Cat")

In a [previous post]({% post_url 2018-04-08-sequences-and-futures %}), I mentioned a client that publishes individual items to a queue. This client is mostly a wrapper for a Java library that returns a `java.util.concurrent.Future`.

https://google.github.io/guava/releases/23.0/api/docs/com/google/common/util/concurrent/JdkFutureAdapters.html
https://stackoverflow.com/questions/18026601/listenablefuture-to-scala-future

<!--more-->
