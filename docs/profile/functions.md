---
layout: default
title: functions
parent: zsh
nav_order: 4
has_toc: true
---

# Functions
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## **Miscellaneous:**

### title()
Set window title\
`title Python` Will set the title of the window/tab to "Python"\
`title` Will clear the title   
```bash
function title() {
    printf "\033]0;$1\007"
}

_title () {
    local state 
    _arguments '1: :Title of the window'
}
```
### math()
Quick math at the command line\
`math 4 x 5` Will output 20\
*Note: Use quotes for complex operations including brackets*
```bash
function math() {
    local MATHFUNC="${*}"
    MATHFUNC="$(echo "$MATHFUNC" | sed 's:x:*:g')"
    echo "$MATHFUNC" | /usr/bin/bc -l
}
```

### hr()
Draw a horizontal rule\
`hr +` Will draw + across the screen
```bash
function hr() {
	local CHAR="${1-=}"
	printf "%$(tput cols)s\n" | tr " " "="
}
```

### help()
Adds back the missing help for builtins
```bash
function help(){
    bash -c "help $@"
}
```

## **Man pages:**

### apropos() [Override]
A replacement for apropos that would search only specific sections\
`apropos print 1` would display matches of print only section 1\
`apropos print 128`  would display matches of print only sections 1, 2 and 8
```bash
function apropos() {
    local SEARCH="${1:-.}"
    local SECTIONS="${2:-124678}"

    # Turn off line wrapping:
    printf '\033[?7l'
    /usr/bin/man -k $SEARCH 2> /dev/null | grep -E "\([${SECTIONS}]\)" | grep -v builtin
    # Turn on  line wrapping:
    printf '\033[?7h'
}
```


### cman()
Load the man page with colour\
`cman printf` Will have load the printf man page with coloured syntax
```bash
cman() {
    env \
    LESS_TERMCAP_mb=$(printf "\e[1;31m") \
    LESS_TERMCAP_md=$(printf "\e[1;31m") \
    LESS_TERMCAP_me=$(printf "\e[0m") \
    LESS_TERMCAP_se=$(printf "\e[0m") \
    LESS_TERMCAP_so=$(printf "\e[1;44;33m") \
    LESS_TERMCAP_ue=$(printf "\e[0m") \
    LESS_TERMCAP_us=$(printf "\e[1;32m") \
    /usr/bin/man "$@"
}
```

### gman()
Load man page into Google Chrome (as html)\
`gman printf` Will have load the printf man page in Google Chrome.\
*Note: Can change browser, or set to default*
```bash
gman() {
	groff -t -e -mandoc -Thtml $(/usr/bin/man -w $1) > /tmp/man_$1.html && /usr/bin/open -a /Applications/Google\ Chrome.app/ /tmp/man_$1.html
}
```

### pman()
Load man page into Preview (as pdf)\
`pman printf` Will have load the printf man page in Preview\
*Note: Useful for archiving man pages from previous OSes when having to write backwards compabile code*
```bash
pman() {
    groff -t -e -mandoc -Tps $(/usr/bin/man -w $1) > /tmp/man_$1.ps && /usr/bin/open -a /Applications/Preview.app/ /tmp/man_$1.ps
}
```

## **GIT:**

### git() [Override]
If not in a git directory, change over to the default directory\
`git status` Will take you to your git and run git status
*Note: Usefull for adding over precursors like 2FA* 
```bash
git () {
	local GITHOME="/path/to/default/git/"
    title 'GIT'

    local RESULT=$(/usr/bin/git rev-parse --is-inside-work-tree 2>&1)
    if [[ "$RESULT" =~ "not a git repository" ]]; then
        echo "Changing git directory"
        # If you have the git-home alias set:
        #git-home
        cd "$GITHOME"
    fi

    if [ ! -z "$1" ]; then
        # Keep quotes preserved
        local GITTCMD=''
        for ARG in "$@" ; do
            GITTCMD=${GITTCMD}${GITTCMD:+ }
            if [[ "$ARG" =~ " " ]]; then
                GITTCMD=${GITTCMD}'"'"$ARG"'"'
            else
                GITTCMD=${GITTCMD}$ARG
            fi
        done

        /bin/bash -c "/usr/bin/git ${GITTCMD}"
    else
        echo "Logged in"
    fi
}
```

## **Proxies:**
Easily turn proxies on and off and see the settings
```bash
proxy () {
    PROXYADDR="http://proxies.examples.com:3128"
    OPTION=$(echo "$1" | tr [:upper:] [:lower:])
    if [ "$OPTION" == "0" ] || [ "$OPTION" == "off" ] || [ "$OPTION" == "no" ]; then
        unset http_proxy
        unset https_proxy
        unset HTTP_PROXY
        unset HTTPS_PROXY
        unset no_proxy
        printf "Proxy off\n"
    elif [ "$OPTION" == "1" ] || [ "$OPTION" == "on" ] || [ "$OPTION" == "yes" ]; then
        echo $OPTION
        export http_proxy="$PROXYADDR"
        export https_proxy="$PROXYADDR"
        export HTTP_PROXY="$PROXYADDR"
        export HTTPS_PROXY="$PROXYADDR"
        export no_proxy="localhost,127.0.0.1"

        printf "Proxy configured to $PROXYADDR\n"
    elif [ "$OPTION" == "show" ] || [ "$OPTION" == "" ]; then
        printf "http_proxy=$http_proxy\n"
        printf "https_proxy=$https_proxy\n"
        printf "HTTP_PROXY=$HTTP_PROXY\n"
        printf "HTTPS_PROXY=$HTTPS_PROXY\n"
        printf "Please use 0/1, on/off, yes/no, show or specify proxy\n"
    else
        echo $OPTION
        export http_proxy="$PROXYADDR"
        export https_proxy="$PROXYADDR"
        export HTTP_PROXY="$PROXYADDR"
        export HTTPS_PROXY="$PROXYADDR"
        export no_proxy="localhost,127.0.0.1"
        
        printf "Proxy configured to $PROXYADDR\n"
    fi
}

proxies () {
    proxy $1;
}

proxys () {
    proxy $1;
}
```

## **Permissions Copy:**

### cpown()
Copy ownership from path to path 

Easily copy ownership from a file/directory to files/dictories  
*Example:*  
*cpown /path/to/file /path/to/directory*  
*cpown /path/to/file /path/to/directory/*  
*cpown /path/to/file /path/to/directory /path/to/file /path/to/otherfile*  
```bash
cpown () {
    SOURCE="$1"
    
    if [ ${#@} -eq 0 ]; then
        printf "Copies ownership from file A to file C, D, E ...\n"
        printf "usage: cpown source [destination ...]\n"
        return 0
    fi
    
    if [ ${#@} -lt 2 ]; then
        printf "Requires 2 or more parameters\n"
        return 1
    fi
    
    if [ ! -e "$SOURCE" ]; then
        printf "Source does not exist\n"
        return 2
    fi
    
    # STAT
    {
        stat "$SOURCE" 
    } || {
        sudo stat "$SOURCE"
    }
    
    # 2nd + parameters
    for DEST in ${@:2}; do
        if [ ! -e "$DEST" ]; then
            printf "%s does not exist\n" "$DEST"
            continue
        fi
    
        {
            chown $(stat -f%u:%g "$SOURCE") "$DEST" &>/dev/null
        } && {
            stat "$DEST"
        } || {
            # Try again with sudo
            printf "Failed, sudoing... \n"
        
            sudo chown $(sudo stat -f%u:%g "$SOURCE") "$DEST"
            sudo stat "$DEST"
        }
    done
}
```   

### cpmod()
Copy file modes from path to path 

Easily copy file modes from a file/directory to files/dictories  
*Example:*  
*cpown /path/to/file /path/to/directory*  
*cpown /path/to/file /path/to/directory/*  
*cpown /path/to/file /path/to/directory /path/to/file /path/to/otherfile*  
```bash
cpmod () {
    SOURCE="$1"
    
    if [ ${#@} -eq 0 ]; then
        printf "Copies file mode from file A to file C, D, E ...\n"
        printf "usage: cpmod source [destination ...]\n"
        return 0
    fi
    
    if [ ${#@} -lt 2 ]; then
        printf "Requires 2 or more parameters\n"
        return 1
    fi
    
    if [ ! -e "$SOURCE" ]; then
        printf "Source does not exist\n"
        return 2
    fi
    
    # STAT
    {
        stat "$SOURCE" 
    } || {
        sudo stat "$SOURCE"
    }
        
    for DEST in ${@:2}; do
        if [ ! -e "$DEST" ]; then
            printf "%s does not exist\n" "$DEST"
            continue
        fi
    
        {
            chmod $(stat -f%Mp%Lp "$SOURCE") "$DEST" &> /dev/null
        } && {
            stat "$DEST"
        } || {
            # Try again with sudo
           printf "Failed, sudoing... \n"
       
           sudo chmod $(sudo stat -f%Mp%Lp "$SOURCE") "$DEST"
           sudo stat "$DEST"
        }
    done
}
```

### cpacl()
Copy file ACLs from path to path  

Easily copy file ACLs from a file/directory to files/dictories  
*Example:*  
*cpown /path/to/file /path/to/directory*  
*cpown /path/to/file /path/to/directory/* 
*cpown /path/to/file /path/to/directory /path/to/file /path/to/otherfile*  
```bash
cpacl() {
    SOURCE="$1"
    
    if [ ${#@} -eq 0 ]; then
        printf "Copies ACLs from file A to file C, D, E ...\n"
        printf "usage: cpacl source [destination ...]\n"
        return 0
    fi
    
    if [ ${#@} -lt 2 ]; then
        printf "Requires 2 or more parameters\n"
        return 1
    fi
    
    if [ ! -e "$SOURCE" ]; then
        printf "Source does not exist\n"
        return 2
    fi
    
    # STAT
    {
        stat "$SOURCE" 
    } || {
        sudo stat "$SOURCE"
    }
    
    for DEST in ${@:2}; do
        if [ ! -e "$DEST" ]; then
            printf "%s does not exist\n" "$DEST"
            continue
        fi
        
        {
            (ls -lde "$SOURCE"  | tail +2 | sed 's/^ [0-9]*: //'; echo) | chmod -E  "$DEST" &> /dev/null
        } && {
            stat "$DEST"
        } || {
            # Try again with sudo
            printf "Failed, sudoing... \n"
        
            (sudo ls -lde "$SOURCE"  | tail +2 | sed 's/^ [0-9]*: //'; echo) | sudo chmod -E  "$DEST"
            sudo stat "$DEST"
        }
    done

}
```

### cpperm()
Copy ownership, file modes and file ACLs from path to path  

Easily copy ownership, file modes and file ACLs from a file/directory to files/dictories  
*Example:*  
*cpown /path/to/file /path/to/directory*  
*cpown /path/to/file /path/to/directory/*  
*cpown /path/to/file /path/to/directory /path/to/file /path/to/otherfile*  
```bash
cpperm() {
    SOURCE="$1"
    
    if [ ${#@} -eq 0 ]; then
        printf "Copies ownership, file modes, and ACLS from file A to file C, D, E ...\n"
        printf "usage: cpperm source [destination ...]\n"
        return 0
    fi
    
    if [ ${#@} -lt 2 ]; then
        printf "Requires 2 or more parameters\n"
        return 1
    fi
    
    if [ ! -e "$SOURCE" ]; then
        printf "Source does not exist\n"
        return 2
    fi
    
    # STAT
    {
        stat "$SOURCE" 
    } || {
        sudo stat "$SOURCE"
    }
        
    for DEST in ${@:2}; do
        if [ ! -e "$DEST" ]; then
            printf "%s does not exist\n" "$DEST"
            continue
        fi
        
        # OWN
        {
            chown $(stat -f%u:%g "$SOURCE") "$DEST" &>/dev/null
        } || {
            # Try again with sudo
            printf "Failed to change owner, sudoing... \n"
        
            sudo chown $(sudo stat -f%u:%g "$SOURCE") "$DEST"
        }
        # MOD
        {
            chmod $(stat -f%Mp%Lp "$SOURCE") "$DEST" &> /dev/null
        } || {
            # Try again with sudo
           printf "Failed to change file modes, sudoing... \n"
       
           sudo chmod $(sudo stat -f%Mp%Lp "$SOURCE") "$DEST"
        }
        # ACL
        {
            (ls -lde "$SOURCE"  | tail +2 | sed 's/^ [0-9]*: //'; echo) | chmod -E  "$DEST" &> /dev/null
        } || {
            # Try again with sudo
            printf "Failed to change ACLs, sudoing... \n"
        
            (sudo ls -lde "$SOURCE"  | tail +2 | sed 's/^ [0-9]*: //'; echo) | sudo chmod -E  "$DEST"
        }
        # STAT
        {
            stat "$DEST"
        } || {
            sudo stat "$DEST"
        }
    done
}
```

## **Automation:**

### batch() [Override]
Batch commands performed on mutliple servers: 

Easily run commands, copy files, and receive files across multiple servers list

*Usage:*  
Verbs: cmd get send addservers removeservers setservers customset defaults

| verb         | action                                                         |
|:-------------|:---------------------------------------------------------------|
|cmd           | Run a command on the batch of servers                          |
|get           | Get files and directories from the batch of servers            |
|send          | Send files and directories to the /tmp on the batch of servers |
|addservers    | Add (a) server(s) to the batch of servers                      |
|removeservers | Remove (a) server(s) to from batch of servers                  |
|clearservers  | Clear all servers to from batch of servers                     |
|setservers    | Set (a) server(s) to be batch of servers                       |
|customset     | Set the servers to a custom batch                              |
|defaults      | Set the servers to a default batch                             |
|list          | List the batch of servers                                      |

*Examples:* \
`batch cmd 'ls /tmp'` List the contents of /tmp on all the servers \
`batch get /path/to/file` Get /path/to/file from all the servers \
`batch cmd 'ls /tmp'` List the contents of /tmp on all the servers \
`batch setservers {win,tor}-windc-01` Set the servers to win-windc-01 and tor-winddc-01 \
`batch list` See servers that commands apply to 


```bash
SERVERS=( )
export SERVERS

batch() {

    [ $1 != "init" ] && ssh
    
    # Verbs
    case $1 in
        ( cmd )
            # Run a command on the servers
            for SERVER in ${SERVERS[@]}; do
                if host $SERVER &>/dev/null; then
                    printf "\n${KGRN}$SERVER: ${KNRM}\n"
                    ssh $USER@$SERVER "${@:2}"
                    hr
                fi
            done
            return
            ;;
        ( get )
            # Get files and directories from the servers
            DOWNFOLDER="$HOME/Downloads/$(date +"%Y.%m.%d-%H.%M.%S")"
            mkdir $DOWNFOLDER

            for SERVER in ${SERVERS[@]}; do
                if host $SERVER &>/dev/null; then
                    printf "\n${KRED}$SERVER: ${KNRM}\n"
                    mkdir "$DOWNFOLDER/$SERVER"
                    for FILE in ${@:2}; do
                        /usr/bin/scp -r "$SERVER:$FILE" "$DOWNFOLDER/$SERVER"
                    done
                    hr
                fi
            done
            return
            ;;
        ( send )
            # Send files and directories to the servers /tmp directory
            for SERVER in ${SERVERS[@]}; do
                if host $SERVER &>/dev/null; then
                    printf "\n${KYEL}$SERVER: ${KNRM}\n"
                    for FILE in ${@:2}; do
                        /usr/bin/scp -r "$FILE" "$SERVER:/tmp"
                    done
                    hr
                fi
            done
            return
            ;;
        ( addservers | addserver )
            # Add specific servers
            for SERVER in ${@:2}; do
                printf "Addding $SERVER ... "
                if [[ ! " ${SERVERS[@]} " =~ " $SERVER " ]]; then
                    SERVERS+=( $SERVER )
                    printf "${KGRN}Added${KNRM}\n"
                else
                    printf "${KRED}Already exists${KNRM}\n"
                fi
            done
            ;;
        ( removeservers | removeserver )
            # Remove specific servers
            for SERVER in ${@:2}; do
                printf "Removing $SERVER ... "
                if [[ " ${SERVERS[@]} " =~ " $SERVER " ]]; then
                    SERVERS=( "${SERVERS[@]/$SERVER}" )
                    printf "${KGRN}Removed${KNRM}\n"
                else
                    printf "${KRED}Nothing to remove${KNRM}\n"
                fi
            done
            ;;
        ( clearservers )
            # Clear all servers from the list
            SERVERS=( )
            printf "${KGRN}Removed all servers\n"
            printf "${KNRM}"
            ;;
        ( setservers )
            SERVERS=( ${@:2} )
            printf "${KGRN}Servers set to: ${KNRM}\n"
            ;;
        ( customset )
            # Create your own unique sets of servers here, examples:
            case $2 in
                ( vandc )
                SERVERS=( )
                for SERVER in van-windc-0{1..5}.example.com; do
                    if host $SERVER &>/dev/null; then
                        SERVERS+=( $SERVER )  
                    fi
                done
                ;;
                ( windc )
                SERVERS=( )
                for SERVER in win-windc-0{1..5}.example.com; do
                    if host $SERVER &>/dev/null; then
                        SERVERS+=( $SERVER )  
                    fi
                done
                ;;
                 ( tordc )
                SERVERS=( )
                for SERVER in win-windc-0{1..5}.example.com; do
                    if host $SERVER &>/dev/null; then
                        SERVERS+=( $SERVER )  
                    fi
                done
                ;;
            esac
            ;;
        ( defaults | default | init | reset )
            # Reset to a default
            SERVERS=( )
            [ ${#@} -lt 2 ] && printf "${KGRN}Servers reset to: \n"
            for SERVER in {van,win,tor}4-win{dc,fs}-0{1..9}.example.com; do
                if host $SERVER &>/dev/null; then
                    SERVERS+=( $SERVER )  
                fi
            done
            printf "${KNRM}"
            [ ${#@} -ge 2 ] && return
            ;;
        ( * )
            # Handle list and print and anything else really
            printf "${KGRN}Current set:\n${KNRM}"
            ;;
    esac
    
    printf "%s\n" ${SERVERS[@]}
}
batch init 0


# If using zsh
_batch () {
    local state
    _arguments '1: :->verb' '2: :->option'


    case $state in
        ( verb )
            _describe 'command' "( \
            'cmd:Run a command on the batch of servers' \
            'get:Get files and directories from the batch of servers' \
            'send:Send files and directories to the /tmp on the batch of servers' \
            'addservers:Add (a) server(s) to the batch of servers' \
            'removeservers:Remove (a) server(s) to from batch of servers' \
            'clearservers:Clear all servers to from batch of servers' \
            'setservers:Set (a) server(s) to be batch of servers' \
            'customset:Set the servers to a custom batch' \
            'defaults:Set the servers to a default batch' \
            'list:List the batch of servers' \
            )"
            ;;
        ( option )
            if [ "$words[2]" = "customset" ]; then
                _describe 'command' "( \
                'vandc:Get Vancouver DCs' \
                'windc:Get Winnipeg DCs \  
                'tordc:Get Toronto DCs \    
                )"
            fi
            ;;
    esac


}
compdef _batch batch



```
