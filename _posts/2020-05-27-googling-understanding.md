---
layout: post
title:  "'Combinatorial' Google Searches"
date:   2020-05-27 18:04:38 -0600
authors: Sid Shanker
categories: learning
---

Let's say you're building a Rails app, for say, a social
network for dogs. You want users
to be able to upload profile pictures, and you want them to be able
to use the browser to take a picture immediately. You Google around
for a cool way to take pictures in the browser, and stumble across
[Uppy](https://uppy.io/), a cool Javascript library that supports both pictures from the browser,
and the user's computer!

Perfect, just what you need! But, wait, how do you integrate this into your
Rails app? You take to Google and [search "Using uppy with rails"](https://www.google.com/search?q=using+uppy+with+rails&oq=using+uppy+with+rails).
   
You thankfully find a lot of super helpful resources, including a tutorial that gets
you just what you need! Great! 

However, let's say instead of Javascript, you were using Typescript, or you have
some less commonly used web service that you want to use to store the image files that users
upload. You might have a little less luck with *that* Google search.

I've been calling this approach to finding information on Google 'Combinatorial' Searches.

## 'Combinatorial' Google Searches

I've been working on a couple different side programming
projects over the last few weeks, in technologies that are either
totally brand new to me, or that I haven't worked in in a while.
It's been fun to work with new stuff,
and I've gotten a chance to reflect a little on my own learning process.

I've noticed that when getting started with new tools, the biggest problems
I run into (besides installation, which is often a pain), are when *combining*
different technologies together. I find that Googling for the technologies that
I'm trying to use in conjunction with each other is often my first impulse when
I run into an issue like this. 

Something I've observed over the years from doing this, however, is that usually
the fact that I'm constructing searches like this demonstrates some gap in my
knowledge about one or more of the tools. And that gap means that there's an
*opportunity to learn something!*

## An opportunity to learn

Let's go back to the original example. I'm trying to use the Uppy Javascript
library with my Rails app. When I search for queries like "Use Uppy with Rails", what
I'm usually hoping for is some tutorial where somebody has using exactly
the set of technologies I'm hoping to use, that I can follow along with,
or at least borrow some code from.

However, something I try to do now is notice if the reason I'm searching for
a tutorial on how to use these things in conjunction is because of a
gap in my knowledge in one of those two tools. For instance, maybe if I had
a better understanding of [Active Storage](https://edgeguides.rubyonrails.org/active_storage_overview.html),
Rails' answer to file storage in a web app, it wouldn't be as much of a mystery
as to how one might use Uppy with Rails. In fact, with a better understanding of
how Active Storage works, it might be clearer how I would integrate any kind
of file uploader with my Rails backend.
 
This isn't to say to *not* look at tutorial that explains how to use the
combination of technologies that you are using -- in fact, that tutorial
might be a great starting point for understanding your tools better! However,
I think that situations like this might be a good opportunity to continue reading
and understand your tools even better, preparing you better for similar situations
in the future.

# Some final thoughts

In summary, if you come across a situation where you're trouble using
the combination of two or more different tools, consider using it as
an opportunity to understand your tools better, rather than searching
for the perfect tutorial for your situation!
