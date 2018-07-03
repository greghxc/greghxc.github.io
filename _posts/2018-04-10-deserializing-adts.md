---
layout: post
title:  "Algebraic Data Type Serialization"
author: "Greg"
category: cats-and-code
comments: true
---
Working on algebraic data type serialization with spray-json.
<!-- more -->
First the data classes.

{% highlight scala %}
sealed trait Shape
case class Rectangle(height: Int, width: Int) extends Shape
case class Circle(radius: Int) extends Shape
{% endhighlight scala %}

And the data contract I'd like to support.

{% highlight json %}
{
  "type": "circle",
  "parameters": { "radius": 6 }
}
{% endhighlight json %}

{% highlight json %}
{
  "type": "rectangle",
  "parameters": { "height": 4, "width": 5 }
}
{% endhighlight json %}

Actually, I'll go ahead and codify that behavior with some tests.

{% highlight scala %}
import spray.json._

class ShapeTest extends FunSuite with Matchers {
  val circle: Shape = Circle(5)
  val circleString: String = """{"type":"circle","parameters":{"radius":5}}"""

  val rectangle: Shape = Rectangle(4,5)
  val rectangleString: String = """{"type":"rectangle","parameters":{"height":4,"width":5}}"""

  test("serializes circle") {
    assert(circle.toJson.compactPrint == circleString)
  }

  test("deserializes circle") {
    assert(circleString.parseJson.convertTo[Shape] == circle)
  }

  test("serializes rectangle") {
    assert(rectangle.toJson.compactPrint == rectangleString)
  }

  test("deserializes rectangle") {
    assert(rectangleString.parseJson.convertTo[Shape] == rectangle)
  }

  test("invalid shapes") {
    val invalidShapeStrings = Seq(
        """{"type":"rectangle"}""",
        """{"type":"sirkle","parameters":{"radius":5}}""",
        "{}")

    invalidShapeStrings.foreach(string =>
      the [DeserializationException] thrownBy {
        string.parseJson.convertTo[Shape]
      } should have message "Invalid Shape"
    )
  }
}
{% endhighlight scala %}

Great. And of course everything fails without any Json readers or writers.

I know from experience we'll need some `DefaultJsonProtocol`s for the case classes, and I can start sketching out the `RootJsonFormat[Shape]`.

{% highlight scala %}
object RectangleProtocol extends DefaultJsonProtocol {
  implicit val rectangleFormat = jsonFormat2(Rectangle)
}

object CircleProtocol extends DefaultJsonProtocol {
  implicit val circleFormat = jsonFormat1(Circle)
}

object Shape {
  implicit object ShapeFormat extends RootJsonFormat[Shape] {
    import RectangleProtocol.rectangleFormat
    import CircleProtocol.circleFormat
    override def write(obj: Shape): JsValue = ???
    override def read(json: JsValue): Shape = ???
  }
}
{% endhighlight scala %}

For the write, I use `match` and build out the shape I want for the data to match the expected contract.

{% highlight scala %}
override def write(obj: Shape): JsValue = {
  obj match {
    case r: Rectangle =>
      JsObject(Map("type" -> JsString("rectangle"), "parameters" -> r.toJson))
    case c: Circle =>
      JsObject(Map("type" -> JsString("circle"), "parameters" -> c.toJson)))
  }
}
{% endhighlight scala %}

For the read, I pick the `JsObject` apart a bit to determine which case class to `convertTo`.

{% highlight scala %}
override def read(json: JsValue): Shape = {
  val jsonAsJsObject = json.asJsObject
  jsonAsJsObject.fields.get("type") match {
    case Some(JsString("circle")) => jsonAsJsObject.fields("type").convertTo[Circle]
    case Some(JsString("rectangle")) => jsonAsJsObject.fields("type").convertTo[Rectangle]
    case _ => throw DeserializationException("Invalid Shape")
  }
}
{% endhighlight scala %}

Very close, but the exceptions aren't normalized. I decide to match on a `tuple` instead so I can make better decisions.

{% highlight scala %}
override def read(json: JsValue): Shape = {
  val jsonAsJsObject = json.asJsObject
  (jsonAsJsObject.fields.get("type"), jsonAsJsObject.fields.get("parameters")) match {
    case (Some(JsString("circle")), Some(p)) => p.convertTo[Circle]
    case (Some(JsString("rectangle")), Some(p)) => p.convertTo[Rectangle]
    case _ => throw new DeserializationException("Invalid Shape")
  }
}
{% endhighlight scala %}

All tests pass and we're happy for now.

Code is available [here](https://github.com/greghxc/scala-tests).
