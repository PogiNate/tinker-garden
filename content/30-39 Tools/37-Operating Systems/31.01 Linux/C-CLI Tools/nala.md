---
tags:
- linux
- package-manager
permalink: tinkering/30-39-tools/37-operating-systems/31.01-linux/c-cli-tools/nala
---

`nala` is a TUI frontend for `apt`. It adds some useful things like concurrent downloads and (more) intelligent mirror selection. 

## Setup
once you have nala installed via apt you can do a few things to make it work better. The first is fetching the fast mirrors:

```bash 
sudo nala fetch
```

This will see how long it takes to fetch from various mirrors and let you choose which ones to use. 

## Config

Nala uses a config file at `/etc/nala/nala.conf`
Some useful things to modify in that file:
```ini
[Nala]
# Set to true to make full-upgrade the default
full_upgrade = true

# Set to true and Nala will always assume yes...
assume_yes = true
```

And then you should be able to just run `sudo nala upgrade` and all is well!