---
layout: default
title: settings
parent: zsh
nav_order: 6
---

# settings
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Ignore case with tab completion
```bash
set completion-ignore-case on
```


## If it matches more than one, show them
```bash
set show-all-if-ambiguous on
```


## Turn off the annoying bell
```bash
set bell-style visible
unsetopt beep
``` 

## Magic space
Turn history references into text after spacing after\
*Prevents you from running unaticipated cammands when accessing older history* \
`!!` Will become the last line \
`!$` Will become the last parameter \
`!-3` Will become the 3rd last line \
`!-5:0` Will before the command you ran 5 lines ago \
`!-4:$` Will become the last parameter from 4 lines ago \
```bash
Space: magic-space
```


## Completes word and not part of
```bash
TAB: menu-complete
```


## Search history by start of line
```bash
"\e[A": history-search-backward
```

```bash
"\e[B": history-search-forward
```

## Lowercase insenitive, Upercase sensitive
```bash
zstyle ':completion:*' matcher-list 'm:{a-z}={A-Za-z}'
```

## Enable Multiline editing
```bash
autoload -U edit-command-line
zle -N edit-command-line
```

## Set the directory by just the name
```bash
setopt autocd
```

## Let's be notified if a job ends
```bash
setopt notify
```

## 
```bash
setopt extendedglob
```

## Set some history handling options.
```bash
setopt inc_append_history appendhistory share_history
```

