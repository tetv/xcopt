# xshopt - eXtreme SH Opt Library (Compatible with bash and dash/POSIX)

**xshopt** is a command line framework where it can run multiple options and commands (respecting the order) as if they were sub-programs. Typically, *options* are used for change state (set behaviour), and *commands* are used to do the real work. Although *options* and *commands* are threated similarly, at least one commands is expected.

**xshopt** helps you:
* To create a sh scripts very quickly (compatible with POSIX)
* Focus on the functionality (commands or options)

**xshopt** helps you:
* To create a sh scripts very quickly (compatible with POSIX)
* Focus on the functionality (commands or options)

**xshopt** takes care (for you) the need of...
* implementing the *--version* command;
* implementing the *--help* command;
* implementing logging options such as: *--error*, *--warning*, *--info*, *--quiet*, *--log-file=<file>*
* implementing logging functions such as: *log_abort*, *log_error*, *log_warning*, *log_info*, *log_debug*
* implementing the *--compress* and *--decompress* command that integrates the library in the original script;
* implementing other helpful methods like *get_var* and *get_fn* whitch returns the first valid argument;
* validating if the commands and options specified are correct based on header of the file;
* calling shell functions for each 'commands' and 'options', E.g.:
  * *--my-command* commands calls a shell function named: *my_command*;
  * *--my-option* option calls a shell function named: *opt_my_option*;

**xshopt** program example:
```sh
#!/bin/sh
#- Dowload some stuff v1.0 ($PROG) - Copyright (C) 2015 Auther<author@domain.com> with MIT Licence
#- Powered by xshopt v1.0 (eXtream SH Opt Library - Compatible with bash and dash/POSIX)
#- This is free software: you are free to change and redistribute it.
#- There is NO WARRANTY, to the extent permitted by law.
# 
## Dowload some stuff v1.0 ($PROG) - Copyright (C) 2015 Me<me@demo.com> with MIT Licence
## 
## Usage: $PROG [OPTIONS] [COMMANDS]
## Options:
##       --info                   Set log level to info (default)
##       --warning                Set log level to warning
##       --debug                  Set log level to debug
##   -q, --quiet                  Set log level to error (quiet mode)
##   -l, --log-file <file>        Set log file (stdout and stderr)
##       --use-curl               Set the download method curl (default)
##       --use-wget               Set the download method wget
##       --use=<method>           Set the download method (options: curl, wget) - default curl
## Commands:
##   -d, --download=<url>         Dowload a specific <url>
##   -s, --ssh-start [addr]       Start ssh to grant network access
##   -k, --ssh-kill [addr]        Kill ssh to remoke network access
##       --help                   Displays this help and exists
##       --version                Displays output version and exists

if [ -n "$BASH_VERSION" ]; then ARG0="${BASH_SOURCE[0]}"; else ARG0="$0"; fi
DIR="$(cd "$(dirname "$ARG0")" && pwd)"
. "$DIR/xshopt.sh"

### BEGIN (DON'T CHANGE THIS LINE) ###

# OPTIONS

METHOD=curl
opt_use() { [ "$1" = wget ] || [ "$1" = wget ] || log_abort "Unsupported method '$METHOD'."; METHOD="$1"; }
opt_use_curl(){ use curl; }
opt_use_wget(){ use wget; }

# COMMANDS
DEFAULT_ADDR=me@demo.com

download(){
  case "$METHOD" in
    curl) log_debug "Executing: curl -OL $1"; curl -OL "$1";;
    wget) log_debug "Executing: wget $1"; wget "$1";;
  asec
}

ssh_start(){
  local ADDR=$(get_var "$1" $DEFAULT_ADDR)
  local PID="$(ps -ef | fgrep -v grep | fgrep ssh "$ADDR" | awk '{print $2}' | grep . | xargs echo)"
  [ -n "$PID" ] && log_abort "SSH session already running with PID: $PID"
  log_debug "Executing: nohup ssh $ADDR &"
  nohup ssh "$ADDR" > /dev/null 2>&1 &
  sleep 2
}

ssh_kill{}{
  local ADDR=$(get_var "$1" $DEFAULT_ADDR)
  local PID="$(ps -ef | fgrep -v grep | fgrep ssh "$ADDR" | awk '{print $2}' | grep . | xargs echo)"
  [ -z "$PID" ] && log_abort "No SSH sessions are running"
  log_debug "Executing: kill -s TERM $PID"
  kill -s TERM $PID
}
```

**xshopt** program calling examples:
```sh
./myprog.sh (by default without parameters call the help)
./myprog.sh --version
./myprog.sh --help
./myprog.sh --debug -s -d http://secure.com/file1 --use-wget -d http://demo.com/file2 -k -s me@demo.com --use=curl -d http://secure.com/file3 --use wget -d http://secure.com/file4 -k me@demo.com
```

**xshopt** shell funtions called for the below command line:
```sh
debug
ssh_start
download http://secure.com/file1
use_wget
download http://secure.com/file2
ssh_kill
ssh_start me@demo.com
use curl
download http://secure.com/file3
use wget
download http://secure.com/file4
ssh_kill me@demo.com
```
