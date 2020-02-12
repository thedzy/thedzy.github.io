---
layout: default
title: aliases
parent: zsh
nav_order: 1
has_toc: true
---

# Aliases
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Directory listing
Some of my aliases to format output
```bash
alias goop="ls -aGoOp"
alias ll="ls -l"
alias sortfiles="du -sm .[!.]* *| sort -n"
```

## git
Pull before push 
```bash
alias git-push="git pull --rebase && git push"
```
Jump to a default Git direcotry 
```bash
alias git-home='cd /path/to/default/git/
```
Jump to an alternate Git direcotry
```bash
alias git-alt='cd /path/to/alternate/git/'
```

## powershell
Start a powershell session\
*Note: [How to install PowerShell](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-macos)*\
```bash
alias cmd="powershell"
alias cmd.exe="powershell"
```

## sudo, politely
```bash
alias please='sudo'
```


## Webservers
Start a webserver at current directory\
**HTTP server:**
```bash
alias webhere="/usr/bin/Python -m SimpleHTTPServer"
```
**PHP server:**
```bash
alias phphere="/usr/bin/php -S localhost:8000"
```

## Who doesn't love to stop and enjoy a good game
```bash
alias dunnet='/usr/bin/emacs -batch -l dunnet'
```