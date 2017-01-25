+++
draft = false
date = "2017-01-11T11:13:15+01:00"
title = "zsh"
categories = ["Linux","Tooling"]

+++

## Install and switch to ZSH
```
sudo pacman -S zsh zsh-completions
```

Run it once to configure the default settings.
```
zsh
```


## Install Antigen for advanced plugin management
```
yaourt -S antigen-git
```
Add the following source to your `~/.zshrc` file:  
`source /usr/share/zsh/scripts/antigen/antigen.zsh`

Add some awesome plugins like Git support and Syntax hightlighting
```
# Load the oh-my-zsh's library.
antigen use oh-my-zsh

# Bundles from the default repo (robbyrussell's oh-my-zsh).
antigen bundle git

# Syntax highlighting bundle.
antigen bundle zsh-users/zsh-syntax-highlighting

# Load the theme.
antigen theme af-magic

# Tell antigen that you're done.
antigen apply

```


Play around with it and if you like it set as default shell for your user.
```
chsh -s $(which zsh)
```


## Aliasses
Make ZSH look for a .aliases file.

```
echo '' >> .zshrc
echo 'source $HOME/.aliases' >> .zshrc
```
