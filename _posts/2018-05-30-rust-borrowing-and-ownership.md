---
layout: post
title:  "Fear not the Rust Borrow Checker"
date:   2018-05-31 12:00:38 -0400
categories: rc rust
---

I spent pretty much the whole day banging my head against the wall trying to figure out how [ownership](https://doc.rust-lang.org/book/second-edition/ch04-01-what-is-ownership.html)
and [borrowing](https://doc.rust-lang.org/book/second-edition/ch04-02-references-and-borrowing.html) work in Rust, and finally have a grasp on what's going on.

In this post I'm going to demonstrate how these concepts work through some examples of code
that break Rust's rules, and explain why they're problematic. I assume very little knowledge
of the Rust programming language. I've also added comments to all of the code blocks that indicate
whether the code is valid Rust or not.

# Example 1: Appending values to a vector of strings

In this example, we take two vectors. One of them, `myvec`, is populated with some values,
and is immutable. The other one is instantiated as mutable, and contains no values. We then take
one of the values from `myvec`, and attempt to add it to `othervec` with `Vec::push()`.

```rust
// invalid

fn main() {
  let myvec: Vec<String> = vec![String::from("hello"), String::from("world")];
  let mut othervec: Vec<String> = Vec::new();

  //  `myvec.get(1)` doesn't return a `String`, rather, it returns an `Option`,
  //  and therefore `.unwrap()` needs to be used to get the value out of it
  othervec.push(myvec.get(1).unwrap());
}
```

This seems like a totally reasonable
thing to do--in many other programming langauges, you can totally
get a value from an array or list, and stick it in another data structure.

But what happens in Rust?

```bash
$ cargo run
error[E0308]: mismatched types
..
expected struct `std::string::String`, found reference
...
```

Scary type mismatch error!

The upshot of what is happening here is that `myvec.get(1)` didn't actually return a value
of type `String`, rather it returned a _reference_ to a `String`, or `&String`. References to strings
can be used in readonly contexts, for instance, you can do:

```
println!("{}", myvec.get(1).unwrap())
```

However, since `othervec` is a `Vec` of type `Vec<String>`, you cannot push a value of
type `&String` on it. That's fine, we can use the dereference character `*`, right?

```
// invalid
...
let myvec: Vec<String> = vec![String::from("hello"), String::from("world")];
let mut othervec: Vec<String> = Vec::new();
othervec.push(*myvec.get(1).unwrap());
...
```

Nope. The error we get this time is "cannot move out of borrowed content".

What the compiler is telling us here is that since the return value of `myvec.get(1)`
is a "borrowed" value, it cannot be moved to another variable.

## Borrowing

To expound on this in more detail, values in Rust can only be assigned to one name
at a time. Assigning a value to a different name causes it to no longer be accessible
under the previous name. As an example:

```rust
// invalid
let x = String::from("hello");
let y = x;
println!("{}", x);
```

This errors out with the error "Use of moved value 'x'". This behavior is really important
in Rust, because in Rust, when variables go out of scope, the memory that they refer to
is deallocated--there is no garbage collection. In order to maintain this behavior,
and to prevent invalid memory access, there can only be one "owner" of data at a time.
In this example, `x` is the original owner, and then `y` becomes the owner.

So that being said, if data needs to be read in other contexts, for instance, inside
other functions, references to values can be assigned to other names. For instance,
the following compiles and works:

```rust
// valid
let x = String::from("hello");
let y = &x;
println!("{}", x);
println!("{}", y);
```

The expression `&x` is an example of "borrowing". The name `y` as assigned a borrowed
reference to the value of `x`. References by default are not mutable (mutable references
are a thing), and have other
constraints to them that I'll explain in further detail later.

## Back to the example

Returning to our original example, `myvec.get(1).unwrap()` returns a reference to
the value at index `1` in `myvec`, and does not perform a move. However, in assigning
this value to a variable, that would be performing a move. The current owner of that
value is `myvec`. If we were to do the following:

```rust
// invalid
let myvec: Vec<String> = vec![String::from("hello"), String::from("world")];
let mut othervec: Vec<String> = Vec::new();
let val = *myvec.get(1).unwrap();
```

this would be switching the owner of that data to be the name `val`. That's
not allowed, because you cannot move a value that has been borrowed!

Think through the implications of being able to do this for a minute. If it were
possible to transfer ownership of `myvec.get(1)` to `val`, that would mean that
as soon as `val` went out of scope, that position `1` in `myvec` would now refer
to an invalid block of memory.

Also, we'd be able to do scary stuff like:

```
// invalid
fn main() {
  let myvec: Vec<String> = vec![String::from("hello"), String::from("world")];
  let mut othervec: Vec<String> = Vec::new();
  othervec.push(myvec.get(1).unwrap());
  myvec.clear();
}
```

Since `.clear()` deallocates all the memory, reads on `othervec` would result
in invalid memory reads after it is called.

To fix this, we could assign the reference to `val`, like so:

```rust
// invalid
let myvec: Vec<String> = vec![String::from("hello"), String::from("world")];
let mut othervec: Vec<String> = Vec::new();
let val = myvec.get(1).unwrap();
```

but we would not be able to append this value to a `Vec` of type `Vec<String>`,
because `val` would be of type `&String`.

Alright, so how do we solve this to accomplish what we want? That would have
to depend on what exactly we want to accomplish. We cannot move ownership
of `myvec.get(1)` to `othervec`, so if we wanted `othervec` to refer to the
actual position in the vector, we would have to change `othervec` to instead
be a vector of `&String`:

```rust
// valid
fn main() {
  let myvec: Vec<String> = vec![String::from("hello"), String::from("world")];
  let mut othervec: Vec<&String> = Vec::new();
  othervec.push(myvec.get(1).unwrap());
}
```

This, however, would not allow us to make any modifications to the values in
`othervec`, and while I'll discuss this in more detail in the next section,
we won't be able to take `myvec` out of scope until `othervec` is out of scope.

On the other hand, if we just care about the _value_ at position `1`, then
we can copy the string:

```rust
// valid
fn main() {
  let myvec: Vec<String> = vec![String::from("hello"), String::from("world")];
  let mut othervec: Vec<String> = Vec::new();
  othervec.push(myvec.get(1).unwrap().to_string());
}
```

The `to_string()` method here on `&String` allocates a new `String`, and
gives ownership to `othervec`.

## Borrowing Summary

This constraint that Rust has that requires that value cannot be assigned
to more than one owner solves a big problem that happens in other programming
languages, like C, in which invalid memory can continue to be read. The
implications of solving that problem are pretty crazy too: another way
of looking at this is that when we pull a value out of `myvec` with: `*myvec.get(1).unwrap()`,
the resulting value _knows_ where it came from, and knows that because it has an
owner, it cannot be assigned to another variable. In other programming languages,
if you pull a variable out of a list-like object, like in the following Python:

```python
x = ["a", "b", "c"]
y = x[1]
```

It is irrelevant where the value now stored in `y` came from, it can be treated like
any other string. That's not the case in Rust!

For more information on borrowing and ownership, check out these sections in the
Rust book:

* [Ownership](https://doc.rust-lang.org/book/second-edition/ch04-01-what-is-ownership.html)
* [References and Borrowing](https://doc.rust-lang.org/book/second-edition/ch04-02-references-and-borrowing.html)

# Example 2: With a separate function

Alright, so in one of the examples, I mentioned above, I mentioned that we could
construct `othervec` as a `Vec<&String>`, a vector of string references.
In this section, I'm going to flesh out handling references in more detail.

To demonstrate that, here is another section of invalid code:

```rust
// invalid
fn copy_to_new_vec(myvec: &Vec<String>, othervec: &mut Vec<String> ) -> &mut Vec<String> {
  othervec.push(myvec.get(1).unwrap().to_string());
  return othervec;
}

fn main() {
  let myvec = vec![String::from("hello"), String::from("world")];
  let mut othervec: Vec<String> = Vec::new();
  let newvec: &Vec<String>  = copy_to_new_vec(&myvec, &mut othervec);
}
```

which compiles with the following error:

```bash
$ cargo run
...
expected lifetime parameter
...
```

This is very similar to example 1. The main difference is that in this example,
we extract the logic to copy the value from position `1` in `myvec` in a function
called `copy_to_new_vec`. Instead of passing ownership of `myvec` and `othervec` to
`copy_to_new_vec`, we pass a reference to `myvec`, and mutable reference to `othervec`.
`othervec` needs to be passed as a mutable reference, since we write values to it
in `copy_to_new_vec`. `copy_to_new_vec` in turn returns back a reference to
`othervec`. `newvec`, in turn, is a reference to `othervec`.

Alright, this seems reasonable, why is the compiler complaining to us?

The catch with functions that return references to other objects is that
if the objects that are being referenced go out of scope or are deallocated,
then reading from our reference will result in invalid memory reads.

## Introducing "lifetime parameters"

To solve this problem, Rust has a notion of "lifetime parameters". These are pieces of information
that you can give to the Rust compiler that tell it what exact object your references are referencing.
With that information, if the object that is being referenced goes out of scope while the reference
is still in scope, the Rust compiler will complain.

Lifetime parameters are normally inferred by the compiler, but in this example, because there are two
parameters that the return value could potentially be referencing, the lifetime parameters
must be explicitly provided. They look similar to generics in Rust:

```rust
// valid
fn copy_to_new_vec<'a>(vec: &Vec<String>, othervec: &'a mut Vec<String> ) -> &'a mut Vec<String> {
  othervec.push(vec.get(1).unwrap().to_string());
  return othervec;
}

fn main() {
  let vec = vec![String::from("hello"), String::from("world")];
  let mut othervec: Vec<String> = Vec::new();
  copy_to_new_vec(&vec, &mut othervec);
}
```

Adding a string with a single quote between angle braces parameterizes a function with lifetime
parameters. These can then be attached to parameters and return values, shown above, to indicate a relationship.
With the lifetime parameters added here, we've indicated to the Rust compiler that the return
value of `copy_to__new_vec` is dependent on the second argument. Now that that's the case, if we
try to do something like:

```rust
// invalid
fn copy_to_new_vec<'a>(vec: &Vec<String>, othervec: &'a mut Vec<String> ) -> &'a mut Vec<String> {
  othervec.push(vec.get(1).unwrap().to_string());
  return othervec;
}

fn main() {
  let vec = vec![String::from("hello"), String::from("world")];
  let newvec = copy_to_new_vec(&vec, &mut Vec::new());
  newvec.get(1);
}
```

The compiler complains with the error:

```
$ cargo run
...
Borrowed value does not live long enough
...
```

What this error is saying is that since the return value of
`copy_to_new_vec` is in scope longer than `othervec` in
`copy_to_new_vec`, this is problematic.

If this was allowed, the line `newvec.get(1)` would fail, since
the value that `newvec` is referencing would be have been deallocated.
This is  because immediately after the line
`let newvec = copy_to_new_vec(&vec, &mut Vec::new());`, the second
argument to `copy_to_new_vec` would be out of scope.

Check out more information about lifetimes in Rust [here](https://doc.rust-lang.org/book/second-edition/ch10-03-lifetime-syntax.html)

# Conclusion

Hopefully these examples help with your understanding of the borrow system
in Rust. It definitely takes a lot of getting used to, and can be a struggle at
first. But once you've gotten your head around it, the borrow system becomes
an incredible safety net that prevents a lot of issues that are common
in other systems programming languages, like C.

Happy Rusting!
