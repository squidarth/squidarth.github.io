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
concept of [Ergodicity](http://squidarth.com/math/2018/11/27/ergodicity.html). Here's a little animated demonstration that
explains the difference between time and ensemble averaging.

**To recap:** we are playing a game where on each iteration, we flip a fair coin,
and if the coin lands heads, we win 55% of our bet, and if it lands tails,
we lose 45% of our bet.

The fascinating takeaway here is that if a large number of people play this
game a fixed number of times, say 20, the *average outcome* will be positive,
which makes sense, since the expected value of playing this game is positive.
However, if any individual plays this game long enough, they will lose almost
all of their money.

<h1>Ensemble Averaging</h1>
<p>
  In this example, when you click "Play", you will begin playing the game. Notice
  that while at any given point in time, you might have made a lot of money, after
  a long time, the "current value" you hold will go to 0.
</p>

<div>
  <span>
    <button id="ensemble-averaging-play-reset">Play</button>
    <span>
      Iteration Count: <b><span id="ensemble-averaging-iteration-count">0</span></b>
      Current Value: <b><span id="ensemble-averaging-current-value">100</span></b> 
    </span>
  </span>
  <div id="ensemble-averaging"></div>
</div>

<h1>The Individual Case</h1>

<p>
  In this example, when you click "Play", you will begin playing the game. Notice
  that while at any given point in time, you might have made a lot of money, after
  a long time, the "current value" you hold will go to 0.
</p>

<div>
  <span>
    <button id="time-averaging-play-reset">Play</button>
    <span>
      Iteration Count: <b><span id="time-averaging-iteration-count">0</span></b>
      Current Value: <b><span id="time-averaging-current-value">100</span></b> 
    </span>
  </span>
  <div id="time-averaging"></div>
</div>

<h1>An explanation</h1>


<script>
// Time Averaging Stuff
function timeAveragingLoop() {
  var layout = {
    yaxis: {
      rangemode: 'tozero'
    }
  };

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

