---
tags:
- cli
- linux
- open-source
type: tool
permalink: tinkering/30-39-tools/37-operating-systems/31.01-linux/c-cli-tools/tmux
---

# tmux

`tmux` [^1] is a great way to accomplish a few related tasks. 
1. You can use one terminal to view and interact with several different command line utilities at once.
2. You can leave a `tmux` session running on a remote machine when you disconnect, and when you reconnect to that machine your programs will all be there waiting for you. And even better, they will remain active while you are disconnected. 
## Configuration
you can set up tmux defaults in a file named `.tmux.conf` in your home directory. Some common configuration changes are things like remapping the "trigger" keystroke that lets you issue commands to tmux directly and setting up easy ways to change the size of panes and stuff. 

```.conf
# prefix settings
set -g prefix C-a
bind C-a send-prefix
unbind C-b
unbind x

# Color Handling
set -g default-terminal "screen-256color"

# Base index and escape time
set -g base-index 1
setw -g pane-base-index 1
set -s escape-time 1

# Windows, Panes, and Sessions
bind | split-window -h
bind - split-window -v
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R
bind r source-file ~/.tmux.conf \; display "Reloaded."
bind x kill-pane
bind -r H resize-pane -L 5
bind -r J resize-pane -D 5
bind -r K resize-pane -U 5
bind -r L resize-pane -R 5
bind X kill-session
```

[^1]: Short for "terminal multiplexer" or "Muxer" depending on who you ask. Look. I didn't make up those words.