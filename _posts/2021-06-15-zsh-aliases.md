---
title: ZSH aliases
published_at: 2021-06-15 12:13:14
updated_at: 2021-06-15 12:13:14
---

<pre>
# ZSH configurations and aliases

# Change Prompt
PROMPT='%F{blue}%n%f:~$'

export EDITOR=/usr/bin/nano

alias cp='cp -iv'                           # Preferred 'cp' implementation
alias mv='mv -iv'                           # Preferred 'mv' implementation
alias mkdir='mkdir -pv'                     # Preferred 'mkdir' implementation
alias ll='ls -FGlAhp'                       # Preferred 'ls' implementation
cd() { builtin cd "$@"; ll; }               # Always list directory contents upon 'cd'

alias cd..='cd ../'                         # Go back 1 directory level (for fast typers)
alias ..='cd ../'                           # Go back 1 directory level
alias ...='cd ../../'                       # Go back 2 directory levels
alias ....='cd ../../../'                   # Go back 3 directory levels
alias c='clear'                             # c:            Clear terminal display
alias which='type -all'                     # which:        Find executables

trash () { command mv "$@" ~/.Trash ; }     # trash:        Moves a file to the MacOS trash
ql () { qlmanage -p "$*" >& /dev/null; }    # ql:           Opens any file in MacOS Quicklook Preview
zipf () { zip -r "$1".zip "$1" ; }          # zipf:         To create a ZIP archive of a folder

alias numFiles='echo $(ls -1 | wc -l)'      # numFiles:     Count of non-hidden files in current dir
alias make1mb='mkfile 1m ./1MB.dat'         # make1mb:      Creates a file of 1mb size (all zeros)
alias make5mb='mkfile 5m ./5MB.dat'         # make5mb:      Creates a file of 5mb size (all zeros)
alias make10mb='mkfile 10m ./10MB.dat'      # make10mb:     Creates a file of 10mb size (all zeros)

alias sbp="source ~/.bash_profile"
alias showfiles="defaults write com.apple.finder AppleShowAllFiles YES; killall Finder /System/Library/CoreServices/Finder.app"
alias hidefiles="defaults write com.apple.finder AppleShowAllFiles NO; killall Finder /System/Library/CoreServices/Finder.app"

alias qfind="find . -name "                 # qfind:    Quickly search for file
ff () { /usr/bin/find . -name "$@" ; }      # ff:       Find file under the current directory
ffs () { /usr/bin/find . -name "$@"'*' ; }  # ffs:      Find file whose name starts with a given string
ffe () { /usr/bin/find . -name '*'"$@" ; }  # ffe:      Find file whose name ends with a given string

alias myip='curl ip.appspot.com'                    # myip:         Public facing IP Address
alias netCons='lsof -i'                             # netCons:      Show all open TCP/IP sockets
alias flushDNS='dscacheutil -flushcache'            # flushDNS:     Flush out the DNS Cache


# GIT shortcuts
alias gp="git pull"
alias gs="git status"
alias gu="git add -u"
alias ga="git add -A"
alias gd="git diff"
alias gc="git commit -m"
alias gco="git checkout"

# Rails
alias rc="bundle exec rails c"
alias rs="bundle exec rails s"

# Google
google () { open "https://google.com/search?q=$*" }

# Increase the limit of file descriptors
# ulimit -Sn 4096      # Increase open files.
# ulimit -Sl unlimited # Increase max locked memory.

</pre>
