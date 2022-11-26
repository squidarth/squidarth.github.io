---
layout: post
title:  "The power of language models"
date:   2022-11-25 00:00:38 -0400
authors: Sid Shanker
categories: machine-learning
---

There's been a lot of buzz and chatter about large language models (LLMs)
in the tech world lately. Lots of claims on Twitter about how models, like OpenAI's GPT,
are going to completely upend how software works.

I finally had my revelatory moment -- where it clicked for me that these models are
in fact, super super powerful.

An intuitive way to start playing around with models like this is to start by
using them to retrieve knowledge. Asking questions like: "Could Barack Obama and George Washington
have shaken hands", that require a couple layers of understanding. A lot of
examples also use prompts like this show off what these models are capable of.

While these are impressive, I think they don't really capture the magic of LLMs.
Knowledge retrieval is certainly a powerful use-case, and will continue to improve
over time, but at the moment, it's fairly often that you'll a model say something
that makes you raise an eyebrow.

On the other hand, the areas where these models really excel is when performing
_tasks involving text_. To solve NLP problems historically, you would typically
need to train a model to solve that particular problem. LLMs have the ability
to _generically solve language problems_.

Let's say you want to build a sentiment analyzer -- tell whether say, Yelp
reviews are positive or negative. While historically you might have trained
a model with examples, with an LLM, you can write a prompt!

Here's an example, using the Flan-T5 text model from Google:

```python
from baseten.models import FlanT5
flan_t5_model = FlanT5()

text_to_analyze = "I hate this restaurant"
flan_t5_model(f"What is the sentiment of the {text_to_analyze}")
# Returns "negative"
```

Bundle that into a function, and now in a single line of code, you have a
sentiment analyzer function! Of course, there's a lot of work left here to use
this in practice (ie: edge cases to handle, dealing with malicious inputs "prompt injection"),
but this is a great start.

Other examples out there include using these models to summarize text -- I love
https://www.explainpaper.com/ as an example of this, which allows you to get an
easy explanation of academic papers.

The exciting part is that there are a lot of these use-cases yet to be discovered.
In this paper [Emergent Abilities of Large Language Models](https://arxiv.org/abs/2206.07682),
the authors discuss how as models are released with more parameters, they gain
the abilities to perform new tasks in a step function way. An example is
that at around 1 billion parameters, GPT-3 starts to begin to be able to
perform arithmetic, which it is unable to do accurately at fewer parameters.

We're continuing to see larger and larger models be released, so expect
even more use-cases like this in the future!