---
layout: post
title:  "Ergodicity, Animated"
date:   2019-04-13 09:00:38 -0400
authors: Sid Shanker
categories: math
---
<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>

<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.4.0/jquery.min.js"></script>


Last year, I wrote a blog post explaining the
concept of [Ergodicity](/math/2018/11/28/ergodicity.html). I
thought it would be helpful in understanding this to make some animations!

**To recap:** we are playing a game where on each turn, we flip a fair coin,
and if the coin lands heads, we win 55% of our bet, and if it lands tails,
we lose 45% of our bet.

The fascinating takeaway here is that if a large number of people play this
game a fixed number of times, say 20, the *average outcome* will be positive,
which makes sense, since the expected value of playing this game is positive.
However, if any individual plays this game long enough, they will lose almost
all of their money.

**A note on terminology:** When I refer to the "current value", I refer to the
amount of money each player holds. For instance, at the start, the value is **100**,
and after one iteration, if the coin lands head, the value will be **155**.

<h1>With a group</h1>
<p>
  In this example, we have 40 players play the game 20 times. Notice
  that each time you run this, the average value for all the players at the end
  will usually be above 100. Notice, however, that the way things tend to shake out, there
  will be one or two players who make a huge amount of money and that most lose
  almost everything.
</p>

<div>
  <span>
    <button id="ensemble-averaging-play-reset">Play</button>
    <span>
      Iterations: <b><span id="ensemble-averaging-iteration-count">0</span></b>
      Average Value: <b><span id="ensemble-averaging-current-value">100</span></b> 
    </span>
  </span>
  <div id="ensemble-averaging"></div>
</div>

<h1>The Individual Case</h1>

<p>
  In this example, when you click "Play", you will begin playing the game. Notice
  that while at any given point in time, you might have made a lot of money, after
  a long time, the "current value" you hold will eventually converge to 0.
</p>

<div>
  <span>
    <button id="time-averaging-play-reset">Play</button>
    <span>
      Iterations: <b><span id="time-averaging-iteration-count">0</span></b>
      Current Value: <b><span id="time-averaging-current-value">100</span></b> 
    </span>
  </span>
  <div id="time-averaging"></div>
</div>

<h2>Conclusion</h2>

It's pretty counterintuitive that playing a game like this, in which each *turn* has a positive
expected value, would in the long run eventually result in ruin. Hopefully these animations
help show visually that very different outcomes occur in this game when a many people play
the game a small number of times (*ensemble-averaging*) and when a single individual plays the game many times
(*time-averaging*). Expected value is a nuanced concept!

If you're curious to learn more, check out my
[my original post](/math/2018/11/27/ergodicity.html) on the subject,
where I try to explain it more rigorously.


<h2>PS</h2>

I made this using [plotly.js](https://plot.ly/javascript/), an awesome & simple charting library, and some very hacky Javascript.
If you're interested in being able to configure more about these animations (say the number of iterations, or players), let me know!

<script>


function ensembleAveragingLoop() {
  var NUM_TURNS = 20;
  var NUM_PLAYERS = 40;
  var layout = {
    yaxis: {
      rangemode: 'tozero',
      title: {
        text: "Current Value"
      }
    },
    xaxis: {
      rangemode: "nonnegative",
      title: {
        text: "Iterations"
      }
    }
  };

  function reset() {
    iterationCount = 0;

    currentValues = [];
    traces = []
    for (var i = 0;i < NUM_PLAYERS;i++) {
      currentValues.push(100);
      traces.push([100]);
    }
    playing = false;
    redrawEnsemble(traces);
    $("#ensemble-averaging-iteration-count").text(0);
    $("#ensemble-averaging-current-value").text(100);
  }

  function redrawEnsemble(data) {
    var traces = $.map(data, function(singlePlayer, idx) {
      return {
        y: singlePlayer,
        name: "Player " + idx,
        type: 'scatter'
      }
    });

    Plotly.newPlot('ensemble-averaging', traces, layout);
  }


  function getIteration() {
    var iterationCount = 0;

    var iteration = function () {
      if (iterationCount < NUM_TURNS) {
        for (var i = 0; i< NUM_PLAYERS; i++) {
          var roll = Math.random();
          if (roll > 0.5) {
            currentValues[i] = currentValues[i] * 1.55
          } else {
            currentValues[i] = currentValues[i] * 0.55
          }

          traces[i].push(currentValues[i]);
        }

        iterationCount += 1;

        $("#ensemble-averaging-iteration-count").text(iterationCount);
        $("#ensemble-averaging-current-value").text(average(currentValues).toFixed(2));

        redrawEnsemble(traces);
      }
    }

    return iteration
  }
  function average(values) {
    var sum = 0;
    for (var i = 0;i < values.length;i++) {
      sum += values[i]
    }
    return sum/values.length;
  }
  var interval;

  $("#ensemble-averaging-play-reset").on('click', function() {
    if (playing) {
      clearInterval(interval);
      $("#ensemble-averaging-play-reset").text("Play");
      reset();
    } else {
      $("#ensemble-averaging-play-reset").text("Reset");
      interval = setInterval(getIteration(), 250);

      playing = true;
    }
  });


  reset();
}

ensembleAveragingLoop();
// Time Averaging Stuff
function timeAveragingLoop() {
  var layout = {
    yaxis: {
      rangemode: 'tozero',
      title: {
        text: "Current Value"
      }
    },
    xaxis: {
      rangemode: "nonnegative",
      title: {
        text: "Iterations"
      }
    }
  };
  var iterationCount;
  var currentValue;
  var chartValues;
  var playing;

  function reset() {
    iterationCount = 0;
    currentValue = 100;
    chartValues = [100];
    playing = false;
    redrawSingle(chartValues);
    $("#time-averaging-iteration-count").text(0);
    $("#time-averaging-current-value").text(100);
  }

  function redrawSingle(values) {
    var trace1 = {
      y: values,
      type: 'scatter'
    };

    var data = [trace1];

    Plotly.newPlot('time-averaging', data, layout);
  }

  function iteration() {
    var roll = Math.random();

    if (roll > 0.5) {
      currentValue = currentValue * 1.55
    } else {
      currentValue = currentValue * 0.55
    }
    iterationCount += 1;

    chartValues.push(currentValue);
    $("#time-averaging-iteration-count").text(iterationCount);
    $("#time-averaging-current-value").text(currentValue.toFixed(2));

    redrawSingle(chartValues);
  }

  var interval = null;

  $("#time-averaging-play-reset").on('click', function(e) {
    if (playing) {
      clearInterval(interval);
      $("#time-averaging-play-reset").text("Play");
      reset();
    } else {
      $("#time-averaging-play-reset").text("Reset");
      interval = setInterval(iteration, 250);
      playing = true;
    }
  });

  reset();
}

timeAveragingLoop();
</script>

