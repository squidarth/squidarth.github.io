---
layout: post
title:  "Building an NYC Subway Arrivals Board"
date:   2022-08-04 00:00:38 -0400
authors: Sid Shanker
categories: projects
---

For a while, I've wanted to have an easy way to see what trains
are arriving at my subway station, so I know when I need to leave
my house.

I have some free time now before I start a new job, so I decided
to actually build an arrivals board!

Here's what it looks like:

![transit_board]({{ "/assets/transit_board.jpeg"  }})

Check out the code for the webapp I built as a part of this here: [https://github.com/squidarth/nyc-transit-hud](https://github.com/squidarth/nyc-transit-hud).

# Overall Setup

The approach I took for this was pretty simple:

1. Build a webapp that displays the arrival times for trains at a given station. ([https://nyc-transit-hub.onrender.com/)](https://nyc-transit-hub.onrender.com/)).
2. Connect a Raspberry Pi to a monitor and load up the web app in Chrome
3. Mount the monitor on the wall

There was some work to do on the Raspberry Pi to load the page in full screen on startup,
but this all mostly worked without a ton of finagling.

# Building the Web App

It turns out that the MTA has a web API for all of their arrival times! The APIs are actually in a standardized
specification used by other transit agencies called [GTFS](https://developers.google.com/transit/gtfs-realtime/).
This made it really easy to just pick my subway station, and filter for all upcoming arrivals.

I was then easily able to display all of this information with a simple React app.

The way that arrivals are displayed are in the API is that each train currently running is listed,
along with upcoming stops:

```
entity {
  id: "000493"
  trip_update {
    trip {
      trip_id: "103750_7..N"
      start_date: "20220804"
      route_id: "7"
    }
    stop_time_update {
      departure {
        time: 1659647850
      }
      stop_id: "726N"
    }
    stop_time_update {
      arrival {
        time: 1659648040
      }
      departure {
        time: 1659648060
      }
      stop_id: "725N"
    }
    ...
```

Here you can see that this is a "7" train, that will be stopping at the stations "726N" and "725N", with
provided timestamps for those stops.

The MTA provides a simple CSV lines that gives pretty names to the stop ids.

One thing to note here is that the "N" (or "S" for the Southerly trains) at the end of the stop_id indicate the direction
that the train is going -- "North" or "South". Unfortunately, this is the only indication of the direction
of the train, so it's not easy to list out the terminus' or clearer directions (ie: Stillwell Ave or Wakefield-241st St) without hard-coding this for
each train. Luckily, for my station, all trains are either going North or South, but for other lines, like the L or 7, it could be
more confusing.

### Did this need to be built

There are a lot of apps that display information in the way that I wanted here (such as https://transitapp.com/), but
nothing that was displayable in a web app in exactly the way that I wanted. 

# Raspberry Pi Setup

I lazily opted to not actually render the graphics on the Raspberry Pi, and instead just open a web browser and
load up the site that I made. This made the Raspberry Pi setup here really easy -- as all I had to do configure
the Pi to load a web app in full screen on startup.

I was able to follow this guide pretty successfully: https://smarthomepursuits.com/open-website-on-startup-with-raspberry-pi-os/,
which uses the `autostart` capabilities of the Pi to load up the website on startup. I had to tweak the chromium-browser settings
a little bit to remove the "Restore Session" window, which was annoying:

```
# ~/.config/lxsession/LXDE-pi/autostart
@lxpanel --profile LXDE-pi
@chromium-browser --incognito --disable-session-crashed-bubble --start-fullscreen --start-maximized https://nyc-transit-hub.onrender.com/
```

# Future Work

Right now, the site I built is only useful if you live near the Barclays Center -- the
next main step here is adapting the site to work for other subway stations (perhaps by using query parameters).

If that's something that would be useful to you, I'd love to hear about it!