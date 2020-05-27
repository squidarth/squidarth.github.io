---
layout: post
title:  "Copy and Paste for tmux & vim on Linux"
date:   2018-12-13 23:04:38 -0600
authors: Sid Shanker
categories: programming 
---

In this [medium post](https://medium.com/@squidarth/a-better-copy-paste-flow-for-tmux-on-macos-5284f82571a2) from a while ago I talk about the flow
I use for copying and pasting for `tmux` on macOS.

Now that I'm using a Linux desktop, time for an update! As a sidebar, I'm using Ubuntu, so this *might* not work
on other distros.

# Tmux

The trick with copy/pasting is that you need way of communicating
from [Tmux](https://github.com/tmux/tmux/wiki) to the X-window system,
which manages your system clipboard on most Linux systems.

The solution to this is [`xclip`](https://github.com/astrand/xclip).
You can install it on Debian-based systems with aptitude:

```bash
$ sudo apt-get install xclip
```

After that, simply add this to your `~/.tmux.conf`:

```
# For binding 'y' to copy and exiting selection mode
bind-key -T copy-mode-vi y send-keys -X copy-pipe-and-cancel 'xclip -sel clip -i'

# For binding 'Enter' to copy and not leave selection mode
bind-key -T copy-mode-vi Enter send-keys -X copy-pipe 'xclip -sel clip -i' '\;'  send -X clear-selection
```

![tmux_copy_pasta]({{ "/assets/tmux-copy-pasta.gif"  }})

# Vim

Getting copy & paste from `vim` to work on Linux is also a little tricky, and doesn't work
out of the box. There are two main steps you need to take here:

1. Add `clipboard` support to vim.

    You can check if your vim has clipboard support by
    typing the vim command:

    ```
    :echo has("clipboard")
    ```

    ![has_clipboard]({{ "/assets/has_clipboard.gif"  }})

    If it says "1" (like mine does), it means you have it, otherwise
    you need to install a vim package that has it, like `vim-gnome`,
    `vim-athena`, and `vim-gtx`.


2. set clipboard=unnamedplus

    In macOS, adding `clipboard=unnamed` is enough to get copy/paste working.
    This command binds yanks in vim to the "\*" register in vim, which in macOS
    corresponds to the system clipboard. In Linux, the "\*" register is different--
    it typically corresponds to the "mouse selection", which is different to the
    normal system keyboard. The normal system clipboard in Linux corresponds to
    the "+" register in vim. In order to get vim to bind yanks to this register,
    use the command:

    ```
    set clipboard=unnamedplus
    ```

    For most setups, this should do the trick. If you are having trouble, the links
    below should be good starting points. Happy copypasta!

# Reference

* (For older versions of tmux) [http://www.rushiagr.com/blog/2016/06/16/everything-you-need-to-know-about-tmux-copy-pasting-ubuntu/](http://www.rushiagr.com/blog/2016/06/16/everything-you-need-to-know-about-tmux-copy-pasting-ubuntu/)
* [https://unix.stackexchange.com/questions/276168/what-is-x11-exactly](https://unix.stackexchange.com/questions/276168/what-is-x11-exactly)
* [https://stackoverflow.com/questions/30691466/what-is-difference-between-vims-clipboard-unnamed-and-unnamedplus-settings](https://stackoverflow.com/questions/30691466/what-is-difference-between-vims-clipboard-unnamed-and-unnamedplus-settings)
* [http://vimcasts.org/blog/2013/11/getting-vim-with-clipboard-support](http://vimcasts.org/blog/2013/11/getting-vim-with-clipboard-support)
* [http://vim.wikia.com/wiki/Accessing_the_system_clipboard](http://vim.wikia.com/wiki/Accessing_the_system_clipboard)
