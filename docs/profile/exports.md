---
layout: default
title: exports
parent: zsh
nav_order: 3
has_toc: true
---

# Exports
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## $EDITOR
Set your default editor in the commandline and options
```bash
export EDITOR="/usr/bin/vim +startinsert"
```
*Note: Useful for CTRL+X+E for a very quick multiline commands*

## $PROMPT_COMMAND
Keep history from all tabs
```bash
export PROMPT_COMMAND='history -a'
```

## $PS1, $CLICOLOR
Colour and style of your command prompt \
[Explanation of PS1](http://osxdaily.com/2012/02/21/add-color-to-the-terminal-in-mac-os-x/) \
Online Editors
* [http://bashrcgenerator.com/](http://bashrcgenerator.com/)
* [http://linux-sxs.org/housekeeping/lscolors.html](http://linux-sxs.org/housekeeping/lscolors.html)
* [http://ezprompt.net/](http://ezprompt.net/)

My Fvourite configuration:
```bash
export PS1="%{$bg[white]%}%{$fg[black]%}[%?] %D{ %a %b %f %H:%M:%S}%{$reset_color%}
%B%{$bg[yellow]%}%{$fg[black]%}%n@%m %{$reset_color%}%b%{$bg[green]%} %~ %{$reset_color%} %# "
```
Required for colours
```bash
export CLICOLOR=1
```

## $PYTHONPATH
Keep my personal python classes together
```bash
export PYTHONPATH=${PYTHONPATH}:$HOME/python/modules
```