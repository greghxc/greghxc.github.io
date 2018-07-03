---
layout: post
title:  "Sequences and Futures in Scala"
author: "Greg"
category: cats-and-code
comments: true
---
![Partyface the Cat](https://farm1.staticflickr.com/892/40630371334_21d9e2dbc5_h.jpg "Partyface the Cat")
The other day I found myself working with a pair in a series of operations in Scala that left us with a `Future[Sequence[Future[Unit]]]` that we needed to evaluate down to a `Future[Unit]`.
<!--more-->
To set the stage, we had a DAO that returned a `Future` of a `Seq` of `MyObject` that needed to be published to a queue. The function that published the messages to the queue had a signature of `publish(object: MyObject): Future[Unit]`. I set up a trivial worksheet to refactor the problem outside of the code base:

{% highlight scala %}
def intSeq(): Future[Seq[Int]] = {
  Future { Seq(1,2,3,4,5) }
}

def publish(input: Int): Future[Unit] = {
  Future.successful(Unit)
}

val eventualInts = intSeq()
{% endhighlight %}

Using maps, I can publish the objects, but I am left with a `Future[Seq[Future[Unit]]]`.

{% highlight scala %}
// Future[Seq[Future[Unit]]]
val eventualResults = eventualInts.map { ints =>
  ints.map { int => publish(int) }
}
{% endhighlight %}

If I can get all the `Future`s on the outside, I feel like I'll have a better shot at evaluating this thing. I remember there's a way to go from a `Seq[Future]` => `Future[Seq]`. I was thinking it was `traverse` but searching docs leads me to `sequence` which seems to have the right effect.

{% highlight scala %}
// Future[Future[Seq[Int]]]
val eventualResults = eventualInts.map{ ints =>
  Future.sequence(ints.map { int => doubleIt(int) })
}
{% endhighlight %}

Now that I'm dealing with a `Future[Future[?]]`, `flatMap` should help clean up the nesting.

{% highlight scala %}
// Future[Seq[Int]]]
val eventualResults = eventualInts.flatMap{ ints =>
  Future.sequence(ints.map { int => doubleIt(int) })
}
{% endhighlight %}

Ah, now that's something I know I can evaluate to a `Future[Unit]`!
{% highlight scala %}
val result = for {
  _ <- eventualResults
} yield ()
{% endhighlight %}

Not doing any parallel evaluations here, can't think of a reason not to put `eventualResults` right in the `for` comprehension.

{% highlight scala %}
val result = for {
  _ <- eventualInts.flatMap{ ints =>
    Future.sequence(ints.map { int => doubleIt(int) })
  }
} yield ()
{% endhighlight %}

And no need for a flatMap in a `for` comprehension, either.

{% highlight scala %}
val result = for {
  ints <- eventualInts
  _ <- Future.sequence(ints.map { int => doubleIt(int) })
} yield ()
{% endhighlight %}

This had the desired result. With my head around the problem a bit better, I want to go back and look up `traverse`, because I still recall us using it to solve a similar problem in the past, and I'm now feeling a little more comfortable in the problem space.

Going to the Scala docs I find that `traverse`:

> Asynchronously and non-blockingly transforms a TraversableOnce[A] into a Future[TraversableOnce[B]] using the provided function A => Future[B]. This is useful for performing a parallel map. For example, to apply a function to all items of a list in parallel.

That *does* sound like what I need to get a Future[Seq[?]] intstead of a Seq[Future[?]] that needs to be transformed. I try it out.

{% highlight scala %}
val result = for {
  ints <- eventualInts
  _ <- Future.traverse(ints){ int => doubleIt(int) }
} yield ()
{% endhighlight %}

Tests still pass. This looks like the best solution yet.
