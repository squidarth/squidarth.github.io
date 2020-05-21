---
layout: post
title:  "Kingfig Solves the Worst Parts of Managing Settings Pages"
date:   2020-05-20 18:04:38 -0600
authors: Sid Shanker
categories: project devtool
---

A couple weeks ago, I started working on a tool called [Kingfig](https://github.com/squidarth/kingfig).
It's a command-line tool to manage your settings in different applications via config files,
rather than using the GUI that the application provides. The motivation here is that when teams
collaborate on projects, you're often sharing a lot of applications. A lot of these applications,
like Twilio and Pagerduty, have quite a few settings, and don't necessarily have features built-in
to help keep track of *when* settings where changed, *why* they were changed, or *who* changed them.

So what if instead of using the GUIs for those apps, you had an easy way to do so in config files,
that can be checked into version control? That's exactly what Kingfig solves -- it's an easy way
to manage a web application's settings in configuration files.

The project is basically just a proof-of-concept now, and can only be used to manage Github Repositories,
but I'm interested in figuring out what other services people would be interested in managing
via config files, and adding support for those.

Check out a live demo!

<iframe width="800" height="515" src="https://www.youtube.com/embed/wcyVPvf_GbM" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# Motivation

The motivation for this project is that when working with complex developer tools like Pagerduty and Twilio,
it can often feel "scary" to change settings, because it's not always easy to undo those changes,
and there isn't an easy way to communicate your intent to change that setting to the rest of your team.
Where with code in a software project, if I want to make a change, I can put my change up for
code review and have a team member look at it, and then revert it later if I want, I have no ability
to do anything like this when changing my Pagerduty settings.

I'm interested in seeing if there are good reasons why teams aren't using solutions that are out on the
market already, and if something like this would be lightweight and easy for them to use.

# Development Strategy

My plan for developing this is to try to discover a service that people would be excited to have
managed via config files. Once I have that integration done, there's a lot I'd love to do.
I'd love to have a plugin system that allows people to write new integrations easily. It would
be especially cool if that plugin system also created automatic documentation for how to use
that integration.

# Making Writing Plugins Easy

At the moment, the logic that deals with "Github Repositories" is pretty intertwined with
the rest of the code for `kingfig`. I'd like to get to the point where all it takes to write
a plugin for `kingfig` is to supply:

* A struct defining a resource
* A method that fetches a resource from the server
* A method that updates a resource from a server

And that's it.

Then, with comments around the `struct` that the plugin author writes, we can
generate automatic documentation.

# What's next?

I'm excited to see where this goes! Drop me a line if you think this is
interesting or if it would be valuable, and definitely let me know if you
have a web service you'd love to use this for.

Also, let me know if you have a better idea for a name!
