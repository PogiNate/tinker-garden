---
title: sudo Settings
type: note
permalink: tinkering/30-39-tools/37-operating-systems/31.01-linux/b-configurations/sudo-settings
---

`sudo` is so useful! But the settings are ancient. There are a few things that I need to remember each time.

## Bypass password

When I have created a new Linux machine it rightly asks for a password each time I do something using `sudo`. 

To fix this you add a line to the `sudoers` file. If your username is `nate` you would add this line

```sudoers
nate ALL=(ALL) NOPASSWD: ALL
```