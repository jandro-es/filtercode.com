Title: Generic classes in Scala
Date: 2014-09-01 10:20
Category: scala
Tags: scala, generics
Slug: generics-scala
Authors: jandro_es
Summary: As many other modern languages Scala includes support for **generic** classes using *type parameters*.

Let's suppose we need a *Stack* in our application to store a list of items, with the usual **Stack** operations. We can define it like this using the useful methods of the **List[T]** type:

~~~~{.language-scala}
	class CustomStack[T] {
	  var elems: List[T] = Nil
	  def push(x: T) { elements = x :: elements }
	  def top: T = elems.head
	  def pop() { elems = elems.tail }
	}
~~~~

and we can create an object like this:

~~~~{.language-scala}
	object CustomStackTest extends Application {
	  val stack = new Stack[String]
	  stack.push("Lorem ipsum")
	  stack.push(" dolore et sumun")
	  println(stack.top) /* it'll print 'Lorem ipsum' */
	  stack.pop()
	  println(stack.top) /* it'll print ' dolore et sumun' */
	}
~~~~

but now suppose that we extend our Stack with a new generic method to sum all the items of the stack:

~~~~{.language-scala}
	class CustomStack[T] {
	  var elems: List[T] = Nil
	  def push(x: T) { elements = x :: elements }
	  def top: T = elems.head
	  def pop() { elems = elems.tail }
	  def sum(): Double = {
		  @tailrec
		  def inner(xs: List[T], accum: Double): Double = {
		    xs match {
		      case x :: tail => inner(tail, accum + x)
		      case Nil => accum
		    }
		  }
		  inner(elems, 0)
		}
	}
~~~~

If we use this Stack with non numeric types we can have unpredicted results, and certainly not the ones expected, we need to limit the admitted types for [T] but we can have several numeric types (Int, Double, etc..). To accomplish this limitation, but keeping the class the most generic possible we can set bounds for it's type parameters.

In this case we need to accept only **numeric** values. We can set up these bounds both at class level (it will affect all the methods using [T]) or at method level (it'll affect only that method). In this case we'll set it up at class level, our Stack will look like this:

~~~~{.language-scala}
	import Numeric._

	class CustomStack[T <: Numeric[T]] {
	  var elems: List[T] = Nil
	  def push(x: T) { elements = x :: elements }
	  def top: T = elems.head
	  def pop() { elems = elems.tail }
	  def sum(): Double = {
		  @tailrec
		  def inner(xs: List[T], accum: Double): Double = {
		    xs match {
		      case x :: tail => inner(tail, accum + x)
		      case Nil => accum
		    }
		  }
		  inner(elems, 0)
		}
	}
~~~~

Just like that, our generic Stack will only accept types that conforms to the trait **Numeric[T]**.

