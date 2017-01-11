+++
draft = false
date = "2017-01-11T11:13:15+01:00"
title = "zsh"
categories = ["Linux","Tooling"]

+++

# ZSH n00b guide
**A n00b guide for ZSH**

## Install and switch to ZSH
```
sudo pacman -S zsh zsh-completions
```

Run it once to configure the default settings.
```
zsh
```

Play around with it and if you like it set as default shell for your user.
```
chsh -s
```


## Aliasses
Make ZSH look for a .aliases file.

```
echo '' >> .zshrc
echo 'source $HOME/.aliases' >> .zshrc
```

Add this awesomely useful alias.
```
echo 'alias ll="ls -al"' >> .aliases
```

## Nice colors and vanity prompt style
```
echo '' >> ~/.zshrc
echo 'autoload -U colors && colors' >> ~/.zshrc
echo 'PS1="%{$fg[green]%}%n%{$reset_color%}@%{$fg[blue]%}%m %{$reset_color%}"in" %{$fg[yellow]%}%~ %{$reset_color%}%% "' >> ~/.zshrc
```
