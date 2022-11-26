---
layout: post
title:  "Teaching Stable Diffusion about Myself"
date:   2022-11-26 00:00:38 -0400
authors: Sid Shanker
categories: productivity
---

[scroll down if you just want to get to the images]

Stable Diffusion is an amazing new open source machine learning model that can be used
to convert text to images. Given a text prompt like, "Painting of a chicken with colorful feathers",
you might get something like this:

![chicken-colorful-feathers]({{ "/assets/chicken-colorful-feathers.png"  }}){: width="400" }

You can give it a shot yourself on this [page](https://app.baseten.co/apps/VBlnMVP/operator_views/nBrd8zP) 
(disclosure, I work for [Baseten](http://baseten.co/), which hosts this demo).          

As-is, Stable Diffusion can produce some pretty amazing results and is pretty fun to play with.
However, the images that Stable Diffusion can produce are limited to the dataset is trained on.
If you want it to produce images of your dog, for instance, it unfortunately has no way of doing
that out of the box.

# Fine-tuning

While Stable Diffusion can't do this out of the box, techniques have been developed to easily
"fine-tune" the model on new images. [Dreambooth](https://dreambooth.github.io/) out of Google
gives Stable Diffusion users the ability to provide a small number of images (~20), and teach
Stable Diffusion a new concept. The main things that you can do with Dreambooth are:
* Teach Stable Diffusion a new style, to allow it to create images in a particular artist style (like [Studio Ghibli](https://huggingface.co/nitrosocke/Ghibli-Diffusion))
* Teach Stable Diffusion a new concept, like your dog, or yourself.

I decided to try out the second approach, and teach SD about myself! I used [this collab notebook](https://colab.research.google.com/drive/1PsOTCIMONQe2wUWsWSmWXlcvTN_MXHtQ#scrollTo=qEsNHTtVlbkV), that I borrowed from
[TheLastBen's Stable Diffusion Repo](https://github.com/TheLastBen/fast-stable-diffusion).

The notebook was very easy to clone and try out. I used around 20 images to fine-tune the model. Here are
some of the results:

![sid-painting]({{ "/assets/sid-favorite painting.png"  }}){: width="400" }

_Painting of Sid Shanker_

![sid-rembrandt]({{ "/assets/sid-rembrandt-1.png"  }}){: width="400" }

_Painting of Sid Shanker in the style of Rembrandt_

![sid-pixar]({{ "/assets/sid-pixar.png"  }}){: width="400" }

_Sid Shanker in the style of pixar_

![sid-pixar]({{ "/assets/sid-family-guy.png"  }}){: width="400" }

_Sid Shanker in the style of Family Guy_

# Some takeaways

## Fine-tuning requires a lot of experimentation

This didn't work immediately! It took a couple tries to fiddle around
the input prompts and the images I uploaded to start getting good
results. One of the early mistakes was using too generic a
name for myself ("Sid Shanker"), which resulted in me seeing results like this:

![sid-disney-villain-2]({{ "/assets/sid-disney-villain-2.png"  }}){: width="400" }

Here, it doesn't look trained to me at all. After changing the prompt to being something
more unusual ("squidarth", my Github handle), I started getting the results I wanted.

The other big lever is using the right mix of images (> 20, different angles, high resolution).

## You have to know a little bit about what's going on

The other interesting thing here I observed is that fine-tuning right now
isn't a totally clean abstraction -- you do have to know a little bit
about what's going on, and what problems a tool like Dreambooth sets out to solve
to get the results that you want. 

A major example here is that it's really easy to overfit with Dreambooth training.
When I tried running "Sid with other person", I got this monstrosity: 

![sid-pixar]({{ "/assets/sid-with-person.png"  }}){: width="400" }

Huggingface has a [really good guide](https://huggingface.co/blog/dreambooth#tldr-recommended-settings) on how to configure the settings for your project,
but interpreting these understands some amount of knowledge about Stable Diffusion (what's the text encoder? what's the unet?)

It's hard to get away from requiring that level of configurability, since you likely want different
settings for training a style vs. training on a face.

## Pre-trained models are really awesome

It's really awesome that someone like me, with a fairly limited amount of ML experience,
was able to fine-tune a model like this to get the results I wanted. The power that
pre-trained models bring to developers is very exciting.