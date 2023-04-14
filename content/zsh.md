---
title: "zsh"
date: 2023-04-06T22:54:01+02:00
---

## Installation

```bash
apt install zsh # on Debian
emerge -av app-shells/zsh # on Gentoo
pacman -S zsh # on Arch
```

## Configuration

To use the grml-zsh-config, see https://grml.org/zsh/

```bash
wget -O ~/.zshrc https://git.grml.org/f/grml-etc-core/etc/zsh/zshrc
```

To use oh-my-zsh, see https://github.com/robbyrussell/oh-my-zsh

```bash
git clone https://github.com/robbyrussell/oh-my-zsh.git
mv oh-my-zsh ~/.oh-my-zsh
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
```

and edit the "ZSH_THEME" variable, see https://github.com/robbyrussell/oh-my-zsh/wiki/Themes.

## Set zsh as default shell

To change the default shell to zsh

```bash
chsh -s /path/zsh # "whereis zsh"
```

or edit /etc/passwd as root.

## fzf

see https://github.com/junegunn/fzf

### On Arch, Gentoo

Install fzf via package manager
```bash
emerge -av app-shells/fzf # on Gentoo
pacman -S fzf # on Arch
```

and source the config files in ~/.zshrc, see https://wiki.archlinux.org/index.php/Fzf#zsh

```
source /usr/share/fzf/key-bindings.zsh
source /usr/share/fzf/completion.zsh
```

### oh my zsh

Oh My Zsh contains a fzf plugin, see $ZSH/plugins/. To activate the plugin add fzf to the record in ~/.zshrc, e.g.

```
plugins=(fzf)
```

Here's some example configuration:

```
# config fzf
export FZF_DEFAULT_OPTS=$FZF_DEFAULT_OPTS'
  --multi
'

# set colors for fzf
# see https://github.com/junegunn/fzf/wiki/Color-schemes#one-dark
export FZF_DEFAULT_OPTS=$FZF_DEFAULT_OPTS'
  --color=dark
  --color=fg:-1,bg:-1,hl:#c678dd,fg+:#ffffff,bg+:#4b5263,hl+:#d858fe
  --color=info:#98c379,prompt:#61afef,pointer:#be5046,marker:#e5c07b,spinner:#61afef,header:#61afef
'
```

### Manuell

```bash
git clone https://github.com/junegunn/fzf.git
./install (in the gitrepo run)
```