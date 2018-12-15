---
layout: post
title:  "Ergodicity"
date:   2018-11-28 00:00:38 -0400
authors: Sid Shanker
categories: math
---

<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

My dad recently introduced me to the concept of [ergodicity](https://en.wikipedia.org/wiki/Ergodicity) in probability. It's a powerful, counter-intuitive concept that is best introduced with the following paradox:


## The Game

We are going to play a game. We toss a coin, and if the coin comes up **heads, you win 50%** of your bet. If the coin comes up **tails, you lose 40%** of your bet. So if you bet $100, if it comes up head, you will win $50, and if the coin comes up tails, you lose $40.

Doesn't sound like a bad deal right?

## Computing your expected value

Alright, so before planning to retire early, let's actually do some math. What is the expected value of a single play of this game? Remember that expected value is the sum for all events of the probability of that event times the "value" of that event:

$$0.5 * 50 + 0.5 * (-40)= 5$$

Alright, so this lines up with our expectations, we *would* expect the expected value to be some positive number given that you win more on heads than you lose on tails. The expected value of this game is +%5.

## Let's play!

Ok, so we've determined that it's advantageous for us to play this game, so let's take that bet! If we play the game enough times, we should ultimately end up making money, right?

Let's play:

We bet $100. The coin comes up heads! Woot! So now we have $150. We bet $150 now. Coin comes up heads again! Now we have $225.  Tails, back down to $135. Tails again, down to $81.

## What's going on here?

Wait, this is kinda weird! We got 2 heads, and 2 tails, but somehow ended up with less money than what we started with. And this is the case for any scenario where we get the same number of heads and tails.

Alright, let's get back to doing some math. If we get the *same* number of heads and tails after tossing the coin *N* times, how much money should we expect to get? Well, since we multiply our amount by 1.5 when we get heads, and .6 when we get tails, it's simply:

$$1.5^{n/2} * 0.6^{n/2} = (0.6*1.5)^{n/2} = 0.9^{n/2}$$

This all of a sudden doesn't look so good, and the implication of this equation is that if you play the game long enough, you will eventually lose all of your money.

## A different analysis

Our initial analysis of the game using expected value clearly didn't yield a useful result, given that even though the expected return of the game is positive. What purpose does "expected value" serve in this context?

Consider a different scenario, in which we get 100 people, and each person is going to play this game 20 times. What would we expect the average amount of money for each person to have at the end be?

The right analysis for this is our expected value computation. If each person was to play the game a *single time*, then we would expect roughly 50 people to get heads and 50 to get tails. The average return across all people would be $5. If they were to toss the coin twice, 25% of people would get HH, 50% HT or TH, and 25% of people would get TT, and we could compute the expected return as:

$$0.25 * 1.5^{2} * 100 + 0.5 * .9^{2} * 100 + 0.25 * .6^{2} * 100 = 105.75$$

We expect on average for someone playing to win $5.75.

This analysis is the same for any number of coin tosses.

Therein lies the paradox—that when many people play the game a fixed number of times, the average return is positive, but when a fixed number of people play the game many times, they should expect to lose most of their money.

## Time Averaging vs. Ensemble Averaging

This game demonstrates the differences between time averaging and ensemble averaging. If the game is thought of as a *random process*, time averaging is taking the average value is the process continues, while ensemble averaging is taking the average value of many processes running for some fixed amount of time.

For many probabilistic processes, there is no difference between the time average and ensemble average.  Processes where there is no difference are called ergodic processes, while processes where there is a difference are called non-ergodic processes.

## Conclusion

The upshot here is that expected value is not necessarily the right analysis to use to understand the behavior of any random process. This is not some mathematical curiosity—there are many real world processes that are non-ergodic, particularly in finance, and it's important to not be be fooled by financial processes advertising their ensemble average, when what you really care about is the time average.

If you're curious and want to learn more, check out [Ole Peters' blog](https://ergodicityeconomics.com), he's thought a lot about this stuff.

