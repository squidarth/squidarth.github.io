---
layout: post
title:  "Type Erasure in Scala"
date:   2019-01-11 00:00:38 -0400
authors: Sid Shanker
categories: scala types
---

I've been learning more about Scala's type system, and last week, I hit
some of its limitations for the first time. In this post, I'll
be covering an issue called type erasure. If you're interested in understanding how generic types in Scala work better, or about how to use reflection, keep reading!


And if you're seeing a warning along the lines of:

```
non-variable type argument Int in type pattern Seq[Int] (the underlying
of Seq[Int]) is unchecked since it is eliminated by erasure
```

definitely keep reading!

Note that this post assumes some knowledge of Scala.

# What is Type Erasure?

Let's say you have the following code:

```scala
case class Thing[T](value: T)

def processThing(thing: Thing[_]) = {
  thing match {
    case Thing(value: Int) => "Thing of int"
    case Thing(value: String) => "Thing of string"
    case _ => "Thing of something else"
  }
}

println(processThing(Thing(1)))
println(processThing(Thing("hello")))
```

Here, we have a generic class called `Thing` that contains a single value
of some generic type `T`. We also have a function called `processThing`
that takes a `Thing`. The `_` represents an "existential type", meaning
that you can pass in a `Thing` of any type into the function. `processThing`
pattern matches on the inner type of `Thing`, and returns a `String`
depending on the type.

This seems reasonable--it's not crazy that Scala, when executing this
pattern match, could run through each of the cases and for each of the
cases check the type of the instance of `value`, which will definitely
be known at runtime.

And as you'd expect, this code prints out:

```
$ sbt run
...
Thing of int
Thing of string
```

## Let's try something else

We're going to add another case to our pattern match now:

```scala
def processThing(thing: Thing[_]) = {
  thing match {
    case Thing(value: Int) => "Thing of int"
    case Thing(value: String) => "Thing of string"
    case Thing(value: Seq[Int]) => "Thing of Seq[int]"
    case _ => "Thing of something else"
  }
}

```

Again, this seems pretty reasonable. If we can check the type of the
instance to match it against an `Int` or a `String`, we should be able
to match it against a `Seq[Int]`.

When we compile this, however, we see the following warning:

```
non-variable type argument Int in type pattern Seq[Int] (the underlying
of Seq[Int]) is unchecked since it is eliminated by erasure
```

We've hit an instance of what's called "type erasure". What this means
is that when Scala is compiled, if there are generic types in the program,
information about the specific is checked during compiled time, but
*not available for the runtime to use*.

## What does it mean for types to be erased?

To be more specific about what's happening here--there is a distinction
between how generic types are treated at compile-time in a Scala program,
and how they are treated at runtime. As an example, say you have the
following:

```scala
val seq : Seq[String] = Seq(1,2,3)
```

The Scala compiler will check to make sure that value that you are assigning
to `seq`, which we are asserting is a `Seq[String]`. And we will get a
type mismatch error in compilation.

After these checks happen, the compiler then removes the specific type
information from generics, and this code will become the following, as
far as the Java bytecode that is produced is concerned:

```scala
val seq : Seq = Seq(1,2,3)
```

Because of the compile-time checks, we get nice, type-safe code. However,
this problem with the "underlying" types getting erased, is that you
can no longer do runtime checks on those types.

For instance, if you were to try:

```scala
val seq : Seq[Int] = Seq(1,2,3)
```

Then:

```scala
seq.isInstanceOf[Seq[Int]]
```

is obviously true.

```scala
seq.isInstanceOf[Int]
```

is obviously false.

```scala
seq.isInstanceOf[Seq[String]]
```

This strangely also returns `true`! Again, this is because this code
compiles to seq.isInstanceOf[Seq], since the "underlying" types are
erased. To elaborate on this a little further, typing 
`seq.isInstanceOf[Seq[String]]` does not actually violate any type checks,
so it's valid Scala code. Scala will produce warnings for this, however.

## How serious of an issue is this?

Alright, so the next question is how serious an issue it is if you start
seeing warnings like this. The problem now is that even if the code
compiles, your program's behavior may not be what you expect. Let's
revisit our `processThing` example:

```scala
def processThing(thing: Thing[_]) = {
  thing match {
    case Thing(value: Int) => "Thing of int"
    case Thing(value: String) => "Thing of string"
    case Thing(value: Seq[Int]) => "Thing of Seq[Int]"
    case _ => "Thing of something else"
  }
}
```

If we run this on the following line:

```scala
processThing(Thing(Seq(1,2,3)))
```

This yields the string "Thing of Seq[Int]". If we run this line:

```scala
processThing(Thing(Seq("hello", "yo")))
```

It *also* yields the string "Thing of Seq[Int]".

Annoying, but again, we now know why it happens! The line:

```scala
case Thing(value: Seq[Int]) => "Thing of Seq[Int]"
```

compiles down to:

```scala
// not valid code

case Thing(value: Seq) => "Thing of Seq[Int]"
```

This behavior is surprising for the developer, but things can be even
worse when in the pattern match, you make assumption about what kind
of sequence you have. For instance, if `processThing` is implemented as:

```scala
def processThing(thing: Thing[_]) = {
  thing match {
    case Thing(value: Int) => "Thing of int"
    case Thing(value: String) => "Thing of string"
    case Thing(value: Seq[Int]) => "ints sum to " + value.sum
    case _ => "Thing of something else"
  }
}
```

Here, `value.sum` is only valid if `value` is a `Seq` of numeric types,
like `Int`. If `value` is a `Seq[String]`, this will blow up with the
error:

```
java.lang.ClassCastException: java.lang.String cannot be cast to java.lang.Integer
```

# Can we solve this with reflection?

Scala provides a [reflection](https://docs.scala-lang.org/overviews/reflection/overview.html) API that allows you to inspect the types of your
instances at runtime. Let's explore the possibilities of the reflection
API.

It turns out that we can accomplish *some* of what we want with the
reflection API.

## TypeTag

The Scala reflection API has a mechanism called `TypeTag` that allows
you to inspect the type of instances, including the types of generics,
at runtime. As the [docs](https://docs.scala-lang.org/overviews/reflection/overview.html), state "TypeTags can be thought of as objects which carry along all type information available at compile time, to runtime".

Let's see how we can rewrite our `processThing` function to use `TypeTag`:

```scala
import scala.reflect.runtime.universe._
...

def processThing[T: TypeTag](thing: Thing[T]) = {
  typeOf[T] match {
    case t if t =:= typeOf[Seq[Int]] => "Thing of Seq[Int]"
    case t if t =:= typeOf[Seq[String]] => "Thing of Seq[String]"
    case t if t =:= typeOf[Int] => "Thing of Int"
    case _ => "Thing of other"
  }
}
```

Alright, let's unpack what's happening here:

1. We added a generic type to the function that's now associated with
`TypeTag`. This is an indicator to the compiler that the compiler should
capture the type information about `T` when `processThing` is invoked.

2. We now pattern match, instead of on `Thing` itself, but on the `typeOf[T]`.


This works correctly though, and now we can actually distinguish between
a `Thing` containing a `Seq[Int]`, and one containing a `Seq[String]`.

## What if we want to operate on the value in Thing?

Alright, now remember the example where we actually do something with
the `value` inside of `Thing`? Can we use reflection to do that too?

Well, now that we have the ability to check the types of things, we can
add in a type check to ensure that we're hitting the cases that we care
about.

```scala
def processThing[T : TypeTag](thing: Thing[T]) = {
  thing match {
    case Thing(value: Int) => "Thing of int " + value.toString
    case Thing(value: Seq[Int]) if typeOf[T] =:= typeOf[Seq[Int]] => "Thing of seq of int" + value.sum
    case _ => "Thing of something else"
  }
}
```

Alright, so now, this code, with the additional guard, `typeOf[T] =:= typeOf[Seq[Int]]`, can now distinguish between a `Seq[Int]` and a `Seq[String]`. We finally have achieved the behavior that we want.

However, during compilation, we still see the original warning that we
were getting:

```
non-variable type argument Int in type pattern Seq[Int] (the underlying
of Seq[Int]) is unchecked since it is eliminated by erasure
```

This makes sense--adding in the `if` statement guarantees that in practice
that the value that emerges in that `case` statement will in fact be a
`Seq[Int]`, but because that guard code is executed at runtime, it is
impossible to at compile-time to assert that `value` is in fact a
`Seq[Int]`.

Because of the way Scala type erasure works, this is the best we can do
in this case. You probably want to add an `@unchecked` annotation here as
well to suppress the warning:

```
case Thing(x: Int @unchecked) => ...
```

## Other options

This is somewhat out of scope of the post, but it's worth noting that this
is really only problem because the type `T` of `Thing[T]` in
`processThing` is completely unconstrained. If you knew in advance what
possible types you might stick into a `Thing`, you could develop a different
type, with subtypes that you could match on instead:

```
sealed trait ThingValue
case class SeqIntThingValue(value: Seq[Int]) extends ThingValue
case class SeqStringThingValue(value: Seq[String]) extends ThingValue

def processThing[T <: ThingValue](thing: Thing[T]) = {
  thing match {
    case Thing(SeqIntThingValue(value: Seq[Int])) => "Seq of Int" + value.sum
    case _ => "Other thing"
  }
}
```

# Why does Scala do this?

This seems like an annoying constraint, and given that in *general*, in
Scala, you can pattern match on arbitrary types of values, it is fairly
surprising. The reason type erasure is required in Scala is because
Scala is a JVM-based language and needs to interop with Java. Java in its
early iterations did not actually have generics, and when generics were
added, they were implemented with type erasure so that the bytecode would
be interoperable with bytecode generated from older versions of Java.

# Conclusion

Scala has a powerful type system, but because of quirks of the JVM, there
are still some limits.

Hope that this post helps you solve your problems with erased type!

# Further Reading

**More on Type Erasure**

* https://medium.com/@sinisalouc/overcoming-type-erasure-in-scala-8f2422070d20
* http://www.angelikalanger.com/GenericsFAQ/FAQSections/TechnicalDetails.html#FAQ001

**TypeTag**

* https://stackoverflow.com/questions/12218641/scala-what-is-a-typetag-and-how-do-i-use-it

**The drawbacks of using runtime information in an unconstrained way**

* https://failex.blogspot.com/2013/06/fake-theorems-for-free.html
* https://typelevel.org/blog/2014/11/10/why_is_adt_pattern_matching_allowed.html
