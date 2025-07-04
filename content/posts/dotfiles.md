---
title: "Dotfiles"
# weight: 1
# aliases: ["/first"]
tags: ["vim"]
showToc: true
TocOpen: false
draft: false
hidemeta: true
comments: false
description: "nix home-manager"
disableHLJS: false
disableShare: true
hideSummary: false
searchHidden: true
ShowReadingTime: false
ShowBreadCrumbs: false
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: false
UseHugoToc: false
---

the average user tends to set and forget their configurations. despite the endless number of dotfile managers ranging from bash scripts to symlinkers, you can leave it to _timmy_ to not even use one. if we are going to pour copious amounts of time into ricing, we can at least make it declarative along the way.

---

## Centralized

your dotfiles should be easily accessible. i store mine in a git repo hosted on github.com. if need be, use a flash drive and carry it.

> "a man without his dots is as good as dead"
> -- nietzsche

### Nix

This **configuration** leverages the use of the [nix](https://nixos.org/) package manager.

nix is a purely functional package manager that treats your system configuration like code. you define how your system will be through code and will be this exact way. most importantly, this will **centralize** our configuration so that no matter where we are, we will have a spot-for-spot system.

```nix
  programs.ghostty = {
    enable = true;
  };

```

above is a snippet of nix code that will install **ghostty** for your local user. you are also capable of configuring ghostty through nix as well. this is done through the use of [attribute sets](https://nix.dev/manual/nix/2.24/language/syntax#attrs-literal), but that is for a later date.

## Tools of the Trade

now that we have a way to configure and recreate our dotfiles on any system, lets talk about mine.

### Essential

- **`shell = zsh`**: i've tried [fish](https://fishshell.com/), it just doesn't have what i want. with nix, i can install [omzsh](https://ohmyz.sh/) and get what i want out of the box like fish...and [POSIX](https://en.wikipedia.org/wiki/POSIX) complaint.
- **`terminal = ghostty`**: at this point, it doesn't seem to matter what terminal you use. alacritty is fast, kitty is fast, ghostty is fast. shaders are cool, but my wallpaper is black so it doesn't do me much good.
- **`editor = neovim`**: [this says all](/posts/neovim)
- **`wm = hyprland`**: it doesn't matter as long as i have a window manager. floating windows is a joke i die laughing at. if you have not used a wm, please try out [i3](https://i3-corps.com/). it is a tried and true x11 windows manager, and if you are on wayland tryout [sway](https://swaywm.org/).
- **`browser = arkenfox`**: stop using chrome, stop using default firefox, you are dumb. it takes three seconds to switch/modify your browser so that you are not being stalked. arkenfox is a hardened version of firefox designed with privacy in mind. its a step closer to [ladybird](https://ladybird.org/).

## Oxidants

i mainly use the terminal. i have a list of rust programs that i've switched to and never going back from. to name a couple i regularly use:

- **`zoxide`**: my personal favorite. i use it everyday. it will cd you into a dir given just part of the dir name. it's my main navigation tool.
- **`ouch`**: this is an archiving program. quick and easy to use.
- **`eza`**: an ls replacer. it will also provide color to option listings, i.e. `eza -la`.

---

my dotfiles are listed on my [github](https://github.cim/Sheepheerd) page. i have spent a lot of time trying to configure what i want. it is not the cleanest, and that is ok. it gets the job done and it gets it done anywhere.
