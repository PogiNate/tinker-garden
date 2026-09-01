---
title: fish
type: shell
permalink: 30-39-tools/37-operating-systems/31.01-linux/a-shells/fish
icon: 🐠
---

# 🐠 The Fish Shell

A shell that tries to be more user friendly than those that have come before.

the `fish` shell tries to make things more sane by making things work in a consistent manner. A few things to know

## Functions
Pretty much everything you do in `fish` is (or should be) a function. By default functions are saved in `~/.config/fish/functions`. fish has some shortcuts to make common functions. 

### Aliases

Aliasing commands is a common pastime of mine. So having a way to do it _simply_ is a huge win. Fish has an `alias` command that works as follows:

```fish
alias -s rd="rm -rf"
```

Will create a function called `rd.fish` that looks like this:

```fish
function rd --wraps='rm -rf' --description 'alias rd=rm -rf'
    rm -rf $argv
    
end
```

And if you just run `alias` at the command prompt you will get a list of all the alias _functions_ you have created. 

### Creating new Functions

fish has a built in utility to create a new function: `funced` [^1]. When you run this with a function name it will edit or create a function with that name. If you have set `$VISUAL` or `$EDITOR` it will use that editor. If not it will use a built-in one. I have set `nvim` as my `$EDITOR` so it uses that. 

Once you have created the function it's available immediately. If you want to save it run `funcsave` with the name of the function in question. 

#### Example

```fish
funced t-start
```

this will open an `nvim` window. Create the function. Close it and  you can test the function out. Then run `funcsave t-start` and the function is saved in `~/.config/fish/functions` as mentioned previously.

[^1]: short for "function edit". Probably.
