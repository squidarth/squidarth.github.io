---
layout: post
title:  "Unmute Yourself From Anywhere"
date:   2022-08-29 00:00:38 -0400
authors: Sid Shanker
categories: productivity
---

We've all been there. You're on a video call, and someone says "Sid, what do you think?" Of course,
at the moment I am reading the slides they sent (and absolutely not on Twitter), and it takes
me a few flustered seconds to find my way back to Google Hangouts.

As is customary, the answer to problems like this is to **write code**!

In this post, I'll cover how you can make a universal unmute button that works, regardless
of what window you have focused at the time. Note that I use a Mac -- and have Zoom and Google Meet
versions.

The high-level approach is to write an [AppleScript](https://gist.github.com/squidarth/6d86589a0baa8bf6e688f68b100347e0), which will auto-focus Zoom or an
active Google Meet window, and trigger a mute/unmute. Then, use the Keyboard Shortcuts
functionality on your Mac to hook up a key of your choosing to that AppleScript.

# Write an AppleScript

1. Open the "Automator" app on your Mac
2. Create a new "Quick Action"
3. On the left, search for "Run Applescript"
4. Create a quick action that takes "no input"
5. Copy in one of the AppleScripts below right below this section (Zoom version or Google Meet version)
6. Save the action

![quick_action_view_automator]({{ "/assets/quick_action_view_automator.png"  }})

## Zoom Version

```
on run
	tell application "zoom.us"
		activate
		tell application "System Events" to tell process "zoom.us" to keystroke "a" using {shift down, command down}
	end tell
end run
```


## Google Meet Version
```
on run
	tell application "Google Chrome"
		activate
		set i to 0
		repeat with w in (windows) -- loop over each window
			set j to 1 -- tabs are not zeroeth
			repeat with t in (tabs of w) -- loop over each tab
				if title of t starts with "Meet" then
					set (active tab index of w) to j -- set Meet tab to active
					set index of w to 1 -- set window with Meet tab to active
					tell application "System Events" to tell process "Google Chrome" to keystroke "d" using command down -- issue keyboard command
					return
				end if
				set j to j + 1
			end repeat
			set i to i + 1
		end repeat
	end tell
end run
```

# Set up Keyboard Shortcut

After you've saved your Automator action, you can then go to the Keyboard preferences for your Mac.

* Click on "Shortcuts", which allows you to set up global keyboard shortcuts
* Click on "Services" on the left, where you should be able to find your Automator shortcut
* Set up a keyboard shortcut (I like using function keys for this)

![keyboard]({{ "/assets/Keyboard.png"  }})

And there you go! No more fishing around for your Hangouts window, or having to hit multiple buttons to
focus and then unmute yourself on Zoom!