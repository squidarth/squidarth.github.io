---
layout: post
title:  "Let's build a Scala mock library"
date:   2020-10-04 12:00:38 -0400
authors: Sid Shanker
categories: scala programming
---

I've been writing tests in Scala for a couple years now, and
something that's always been a mystery to me has been Mock libraries. To make things a little less mysterious, I decided to take a stab at building
one myself! This post will go over what I did, and what I learned. This was a great excuse to learn about fancy advanced Scala features like macros and reflection.

If you've ever been curious about these features, or just generally
are interested in understanding Scala better, this post is for you!

If you want to skip ahead see the code I've written, check out my [Github repo](https://github.com/squidarth/scala-mocking-library).

## What's a Mock Library?

First, the basics: what's a mock library for? Let's say we have a class `Bar`
that depends on some class `Foo`, which has a method with some non-deterministic
behavior, like getting a random number.

```scala
class Foo {
    def doesFooStuff = getRandomNumber()
}

class Bar(foo: Foo) {
    def doesBarStuff =  foo.doesFooStuff + 1
} 
```

In my tests, I want to make sure that `Bar` is behaving correctly.
However, this hard to do, because if we use a *real* `Foo` object,
it will return a random number every time!

Ideally, we could have some object that has the *same interface* as `Foo`, that
instead of getting a random number, returns some dummy value. This would allow us
to actually test `Bar` properly. We could certainly write such a class!

```scala
class FakeFoo extends Foo { 
    override def doesFooStuff = 5
}

/* In a test */
val bar = new Bar(new FakeFoo)
// ... test Bar's behavior, knowing that `doesFooStuff` will always return 3.
```

This achieves what we want. However, this means that for any class that we'd want to
have a "mock" for, we'd have to set up a manual class for it, similar to what we've done here. 

Thankfully, Scala has a facility for automatically generating classes at compile-time, [macros](https://docs.scala-lang.org/overviews/macros/overview.html). This is
the mechanism used in the commonly used [ScalaMock](https://scalamock.org/) library, for instance!


## What should the API be? 

Before we dive into the actual implementation of this mock library, let's first decide
on the interface that we want to implement.

As might have been hinted in the previous section, we are going to be writing a macro
that instantiates a "Mock" object for a given class. For this, we'll use the same interface that [ScalaMock](https://scalamock.org/) uses:

```scala
val fooMock = mock[Foo]
```

The expectation in this code is that calling `mock[Foo]` returns a mock object that has
the same interface as `Foo`.

The next thing that we need is a way of specifying, as the author of a test, what we
we want the return values to be for the methods of `Foo`. For this, we need
to be able to specify for given a particular argument, what value ought to be returned.

For this, we'll use the same interface that the [Mockito](https://github.com/mockito/mockito-scala) uses:

```scala
when(fooMock.doFooStuff).thenReturn(5)
```

There will be more discussion about this API later in this post.

## Creating the Mock Object

Let's start with what we mentioned in the first section, and write the macro
that creates a mock object!

### What's a Macro?

[Macros](https://docs.scala-lang.org/overviews/macros/overview.html) are a language feature in Scala that allow developers to modify the syntax tree
of a a program at compile-time. Macros are defined in a similar manner to normal
functions in Scala, using the `def` keyword, taking arguments, and having a return
type.

Macros then require an implementation be defined, which is a function that takes
as arguments "Expression" types that correspond to the arguments of the macro, and
returns an "Expression" type, that when evaluated corresponds to the return type
of the macro. These are "Expression" types, because macros run at compile-time, and the arguments to the macro *have not been evaluated yet*. It's in this implementation function that you can inspect the arguments
to the macro, and programmatically define new types & classes. I'll make this clear with an example:

```scala
import scala.reflect.macros.blackbox

def exampleMacro(x: Int) : Int  = macro exampleMacroImplementation

def exampleMacroImplementation(c: blackbox.Context)(x: c.Expr[Int]): c.Expr[Int] = {
    /* ... */
}
```

Note a couple things about this -- first, that `exampleMacroImplementation`, it takes
a macro `Context` object. Here, we use a `blackbox` macro -- see [this page](https://docs.scala-lang.org/overviews/macros/blackbox-whitebox.html) for what this means. Also, that the arguments and
return type are of type `Expr`, parameterized by the arguments and return type of
`exampleMacro`.

Another important point is that **macros are executed at compile-time**. This means that if `exampleMacro` is called as `exampleMacro(1 + 1)`, the full `1 + 1` expression
can be read in `exampleMacroImplementation`. What `x` is in `exampleMacroImplementation` is is a an **expression that results in an Integer**. The
same story stands for the return type, what `exampleMacroImplementation` returns is an
unevaluated expression that _returns_ an `Int`.

#### Defining an expression

So, now that we have an interface for our implementation function, what actually goes
into an implementation?

Again, to re-iterate, the macro runs at *compile-time*, so our implementation operates 
on syntax trees. The macro `Context` object has classes that allow developers to 
build syntax trees. For instance, this is how you would construct an expression,
consisting of just the literal `10`:

```scala
  import c.universe._
  val s = Literal(Constant(10))
  c.Expr(s)
}
```

Other classes include `ValDef`, for defining values, `DefDef` for defining functions, `Block` for defining blocks, and so on.

Scala has a convenient way of generating these objects, called [quasiquotes](https://docs.scala-lang.org/overviews/quasiquotes/intro.html), that I'll be using for the rest of the post.

With quasiquotes, you can write code in a string that represents the syntax tree
that you are trying to build, making the code much more reasonable. For instance,
for a function definition, you might write:

```scala
def exampleMacroImplementation(c: blackbox.Context)(x: c.Expr[Int]): c.Expr[Int] = {
  import c.universe._
  val funcDef = q"""
  def blah(x: Int) =  x + 5
  """

  /* And you could then use this function using: */
  val block = q"""
  $funcDef
  blah(5)
  """

  c.Expr(block)
}
```

The variable `funcDef` above resolves to a `DefDef` object, and the `block`
variable resolves to a `Block` object. So when the call to `exampleMacro` is made,
for instance `exampleMacro(6)`, this would expand to:

```
def blah(x: Int) =  x + 5
blah(5)
```

and return `10` when the code is actually run.

See [this page](https://docs.scala-lang.org/overviews/quasiquotes/expression-details.html) for more information about how quasiquotes expand.

#### Back to mocks

Alright, so now that we have a rough idea of how macros work, let's go over what
we actually want to achieve here:

We want a macro called `mock`, that returns an object with the same interface as
whatever type is being mocked, with all methods being overridden. For this example,
let's use something similar. We want to mock a class `Foo`, with a single method
called `fooify`:

```scala
class Foo {
    def fooify(x: Int) = x + 3

}
```

`fooify` takes an `Int`, and returns an `Int`.

Given what we know about macros so far, we can write a macro that returns
a instance of `Foo`, with `fooify` overridden. 

```scala
def mock[T]: T = macro mockImpl

def mockImpl[T](c: blackbox.Context): c.Expr[T] =  {
    val result = q"""
    new Foo {
        override def fooify(x: Int) = 10
    }
    """
    c.Expr(result)
}
```

Now, calling `mock[Foo]` will return a instance of type `Foo`, for which whenever
`fooify` is called, returns the dummy value of `10`.

Of course, since the macro always returns an instance of `Foo`, it does not exactly
solve the problem that we set out to address. We want to be able to mock objects of
any type, and mock any of their methods.

Notice  that in this mock, we use generics, and have a single type parameter of `T`,
which indicates the type that we are mocking. In order to achieve want we actually
want to achieve here, we need to be able to:
1. Read the type of `T`, so that we can return an expression that returns a type of `T`
2. Read the methods of `T`, so that we can override them

In order to do this, we will be using [Reflection](https://docs.scala-lang.org/overviews/reflection/overview.html). 

### Using Reflection

[Reflection](https://docs.scala-lang.org/overviews/reflection/overview.html) is a
feature that allows you to inspect the types of objects -- both at compile-time and runtime. So in the case of a function that takes in an object with generic type `T`, with reflection, you could inspect
the *actual* type of that object. So for instance, if it a particular invocation `T` is `Int`, you could discover that using reflection. In addition to
discovering the *type* of that object, you
can find out other information, like what the `members` (methods and fields) of
an object are.

The way reflection works is that for functions where you want to use reflection,
you add an implicit parameter, called "evidence",
that lets the compiler know you'd like to operate on a type:

```scala
import scala.reflect.runtime.universe._

def f[T](v: T)(implicit ev: TypeTag[T]) = ev.toString
```

If `f` is called with `5`, this will return the string "TypeTag[Int]".

A syntactic short-hand for this is to use a "context-bound" on the type:

```scala
import scala.reflect.runtime.universe._

def f[T: TypeTag](v: T) = typeOf[T].toString
```

With this syntatical approach, you can use the `typeOf` function from the reflection
library to get the type of `T`. This type object contains a lot of information about
`T`, in addition to its name, importantly, you can get access to its members:

```scala
import scala.reflect.runtime.universe._

def f[T: TypeTag](v: T) = typeOf[T].members
```

`members` contains the field and methods of `T`, and for each of these, we can then
obtain the parameter lists and return types. 

### Putting it all together

With the basics of macros and reflection, we finally know enough now to put it all
together and write a `mock[T]` macro that supplies dummy values for each of its
members.

The approach we take here is to first, use reflection to obtain the `members` of the
type `T` that we are mocking, and then using quasiquotes, construct a new object
of type `T`, with each member overridden. We will sort out what each overridden method
ought to do after this (this code is incomplete):

```scala
def mock[T] : T = macro mockImpl[T]
/* Macros require that WeakTypeTag is used -- this is a more general
 * form of TypeTag that can be used to detect abstract & generic type params.
 */
def mockImpl[T: c.WeakTypeTag](c: blackbox.Context): c.Expr[T] = {
  val mockingType = weakTypeOf[T]
  val methodDefs = mockingType.members.map { member => 
    val method = member.asMethod
    val returnType = method.returnType
    /* It's required that param lists are a sequence of 
     * ValDefs
     */
    val paramsString = method.paramLists.map { paramList => 
      paramList.map {  symbol =>
        q"""val ${symbol.name.toTermName}: ${symbol.typeSignature}"""
      }
    }
    
    val name = method.name

    /* We will fill in these definitions, but need a mechanism for 
     * storing the dummy values first.
     */
    q"""
    override def ${name.toTermName}(...${paramsString}) : ${returnType} = {
        ???
    }
    """ 
  }.toList

  /* This extra variable is required in order for quasiquotes to 
   * interpret these as the correct types, see: https://docs.scala-lang.org/overviews/quasiquotes/syntax-summary.html  */
  var classBody = q"""
   ..${methodDefs}
  """
  val result = q"""new ${mockingType.resultType} { ..$classBody}"""
  c.Expr(result)
}
```

A few notes about this:
1. Notice that in terms of organization, we first use quasiquotes to create
method definitions for each of T's members, and then use quasiquotes to stick
this into a `Block`.
2. We still need to figure out how to configure dummy values for each of these
methods.
3. There are a number of methods that are defined on *every* object that we are
overriding here. We'll have to filter these out

## Specifying & Reading Mock values

The next problem to deal with is figuring out exactly what goes into the method
body for the overridden methods in the mock object. Should it be some sort of
new field or method defined on the mocked object itself?

One of the problems here is that once we call:

```scala
val myMock = mock[Foo]
```

`myMock` has the same interface as `Foo`. From the perspective of *this* code, where
we are calling `mock`, `myMock` has the same interface, and no methods can be called
besides the ones that are already on `Foo`. So this rules out an approach where we have
a method that we attach to this instance of `Foo` called `setMockValue` that we can
call like this:

```scala
/* Not possible as an API */
val myMock = mock[Foo]
myMock.setMockValue(methodName = "fooify", argument = 3, returnValue = 5)
```

We know that this kind of API is not possible here. And additionally, because
we are limited in what the API of this mock object can be, it's probable that
we don't want to store information about dummy values *on the mock itself*. We
discussed earlier in the article that a common API used by other mocking libraries
looks like this:

```scala
val myMock = mock[Foo]
when(myMock.fooify(3)).thenReturn(10)
```

Given that we can't mutate the state on the mock itself, how can we achieve
this API?

The approach that libraries like `ScalaMock` take here is to have an object **external**
to the mock object that can be used in tests that keeps track of the state of Mocks!

In ScalaMock in particular, there is a class called `MockContext` that is used to
keep track of Mock state in tests. We will follow a similar approach here.

The rough outline for how we will achieve this is:
1. Require that any Scala codepath that wants to use this mock library must
extend a trait, lets call it `Mocking` that has defined an implicit instance
of `MockContext`.
2. In `MockContext` itself, support adding "handlers" to the MockContext, each corresponding to *what* whould be returned for a *given mock* , when *some particular* argument is passed (this will be a list of tuples)
3. In `MockContext`, the notion of a "currently being mocked" method is supported, the reason for this will be clear soon 
3. We modify the `mock` macro to take an `implicit` `MockContext`, such that this
gets passed into `mock` calls automatically.
4. Then, in the `mock` macro, when we override the methods of the mock class, we change
them to do a lookup in the `MockContext` object, to see if there is a dummy value
configured for that particular call or not, and to return an exception otherwise. It should also then set itself "currently being mocked" method on the `MockContext`.

Once `mock` and `MockContext` have been updated to match this behavior, the next thing to do is implement 
`when`. `when` has a very simple purpose--to execute *some function*, and catch
the exception specified in step 4. It also needs to return some object with a
`thenReturn` method on it, which will then make use of the "currently being mocked"
object in `MockContext` to set up a return value.

As a note, we will limit our example here to functions that take a *single parameter*, but it shouldn't be hard to see how we might extend this to support other parameters.

Let's see the code!

We'll start with the `MockContext` class:

```scala
trait Mock[T]

class MockContext {
  /* Mutable list of tuples, for each of the methods mocked.
   * An entry in this Buffer looks like this:
   * (mock: Mock[_], functionName: String, argument: Any, returnValue: Any)
    */
  val handlers : Buffer[Any] = Buffer[Any]()

  /* This is used to keep track of the current method
   * that we are mocking, and contains the mock, function name,
   * and argument */
  var currentMockMethod: (Mock[_], String, Any) = null 

  def appendHandler[Value](value: Value) = {
    val fullCall = currentMockMethod match {
      case (mock, methodName, arg) => (mock, methodName, arg, value)
    }
     handlers.append(fullCall)
  }

  def setCurrentMockMethod[Arg](mock: Mock[_], funcName: String, arg: Arg) = {
    currentMockMethod = (mock, funcName, arg)
  }

  /* Search through the existing handlers, and find one matching the given
   * mock, function name, and argument
   */
  def findMatchingHandler(mock: Mock[_], funcName: String, arg: Any): Option[Any] = {
    handlers.collect { handler =>
      handler match {
        case (savedMock, savedFunctionName, savedArg, value) if mock == savedMock && funcName == savedFunctionName && arg == savedArg => 
          value
      }
    }.headOption
  }
}
```

As described, the main feature provided by this class is the `handlers` field, which
stores a list of calls that we are mocking. Note that I also added a `Mock[T]` trait,
that we'll be adding to the objects produced by `mock`, to allow these to be
type-checked.

Since the functions we're mocking could have any parameter or return types, the
`handlers` field cannot be constrained any further than being `Any`. I'll elaborate
more on this later.

Next, let's go through the changes that we need to make to the `mock[T]` macro:

```scala
class MockUndefinedException(s:String) extends Exception(s)

def mock[T](implicit mockContext: MockContext) : T with Mock[T] = macro mockImpl[T]
def mockImpl[T: c.WeakTypeTag](c: blackbox.Context)(mockContext: c.Expr[MockContext]): c.Expr[T] = {
  import c.universe._

    ...

    val firstParamName = method.paramLists.headOption.flatMap(_.headOption).map { symbol => symbol.name}.get

    q"""
    override def ${name.toTermName}(...${paramsString}) : ${returnType} = {
      ${mockContext}.setCurrentMockMethod(this, ${name.toString()}, ${firstParamName.toTermName})
      val foundHandler = ${mockContext}.findMatchingHandler(this, ${name.toString()}, ${firstParamName.toTermName})
      foundHandler match {
        case Some(value) => value.asInstanceOf[${returnType}]
        case None => throw new MockUndefinedException("no mock found")
      }
    }
    """
  }.toList
  
  ...
}
```

I skipped the parts of the code that were the same from the pervious example
(see the [Github repo](https://github.com/squidarth/scala-mocking-library) for the full code). Here, we both call the
`setCurrentMockMethod` to set the currentMockMethod on the `MockContext`,
and query the `handlers` on the `MockContext` to get a value. If there's no
value, we throw a `MockUndefinedException`.

Next, we implement `when` and `thenReturn`:

```scala
class Stubbing[T](implicit val mockContext: MockContext) {
  def thenReturn(returnVal: T) = {
    mockContext.appendHandler(returnVal)
  }
}

object MockHelpers {
  def when[T](getReturnVal: => T)(implicit mockContext: MockContext): Stubbing[T] = {
    try {
      getReturnVal
    } catch {
      case e: MockUndefinedException => ()
      case e: Throwable => throw e
    }
    new Stubbing[T]()
  }
}
```

These are pretty straightforward -- we have a `Stubbing` object that allows us
to set return values for the currently being mocked method, and a `when` function
that executes a provided function and then returns a new `Stubbing`. The `T` in this
function refers to the return type of the given function being mocked. So in

```scala
when(mockfoo.fooify(3)).thenReturn(10)
```

`T` is `Int`.

Finally, let's add some additional setup to run this code:


```scala
/* In a file called Mock.scala */
trait Mocking {
  import scala.language.implicitConversions

  implicit val mockContext = new MockContext
}

/* In a file called Main.scala */
import MockHelpers._

object Main extends App with Mocking {
  val fooMock = mock[Foo]
  when(fooMock.fooify(7)).thenReturn(200)
  println(fooMock.fooify(7)) // returns 200
}
```

And there we have it! Our very own mock library! Again, see the [Github Repo](https://github.com/squidarth/scala-mocking-library/blob/master/src/main/scala/Mock.scala) for
how this all fits together.

## Is there a way of getting around using `Any`?

One of the main things I thought about after finishing writing this was that it
seems like a code smell to be using `Any` in the `MockContext` handlers. The main
problem with this is that right now, we could intoduce a bug in the `mock` implementation, causing us to a handler for a method with the incorrect type, without
there being a compile-time error. While this has implications
for *developers* of the mock library, if you are not changing the mock library itself,
there isn't really a risk of the use of `Any` causing a problem.

All that said, I spent some time exploring possibilities here, including different data
structures, and the generic programming
library [Shapeless](https://github.com/milessabin/shapeless). However, because
the `MockContext` class doesn't have any context about what mocks might be created,
and because this is only known at runtime, it looks like it might be tricky to
constraint the types further.

However, I cannot confirm that this cannot be done. 

## Conclusion

Building a mock library was a great exercise that taught me a lot about Scala. It's an experience that forced me to think a lot more about how
Scala's type system works -- and I definitely feel like I understand the language
much better as a result.

My high-level takeaway is that it's valuable to take on projects that require you
to use a language in a way that you don't normally. For this project, a lot of the
insight I gained came from hitting up against constraints in the language.

I hope if you've made it this far that you learned something new about Scala too!
For more detail, check out my [full code](https://github.com/squidarth/scala-mocking-library) on Github, and
as always, feel free to reach out if you have any questions!

## References

* [Github Repo for this project](https://github.com/squidarth/scala-mocking-library)
* [ScalaMock](https://scalamock.org/)
* [Mockito](https://github.com/mockito/mockito-scala)
* [Scala Reflection Docs](https://docs.scala-lang.org/overviews/reflection/overview.html)
* [TypeTags and Manifests](https://docs.scala-lang.org/overviews/reflection/typetags-manifests.html)
* [DEF HELLO = MACRO WORLD](https://scalac.io/def-hello-macro-world/) - a great primer on Scala Macros
* [quasiquotes](https://docs.scala-lang.org/overviews/quasiquotes/intro.html)
* [Fake Type Providers](https://meta.plasm.us/posts/2013/07/11/fake-type-providers-part-2/)
* [Learning Scala Macros](http://imranrashid.com/posts/learning-scala-macros/)