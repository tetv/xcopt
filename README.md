# **xshopt** library

### **xshopt** definition
**xshopt** means literally: *eXtreme SH Opt Library*
**xshopt** is compatible with bash and dash/POSIX scripting.

### **xshopt** overview
This command line framework runs multiple *options* and *commands* (respecting the order) as if they were sub-programs. Typically, *options* are used for change state (set behaviour), and *commands* are used to do the real work. Although *options* and *commands* are threated similarly, at least one commands is expected.

### **xshopt** helps you
* To create a sh scripts very quickly (compatible with POSIX);
* Focus on the functionality (implementing *commands* and *options*);
* Refactoring *commands* and *options* easly just changing the help menu (header).

### **xshopt** takes care (for you) the need of...
* implementing the *--version* and *--help* commands;
* implementing logging options such as: *--error*, *--warning*, *--info*, *--quiet*, *--log-file=\<file\>*
* implementing logging functions such as: *_log_abort*, *_log_error*, *_log_info*, ... (see below).
* implementing the *--compress* and *--decompress* command whitch integrates the library with the main script;
* implementing other helpful methods like *_get_var*, *_get_fn* whitch returns the first valid argument;
* validating if the commands and options specified are correct based on header of the file;
* calling shell functions for each 'commands' and 'options', E.g.:
  * *--my-command* commands calls a shell function named: *my_command*;
  * *--my-option* option calls a shell function named: *opt_my_option*;

# **xshopt** API

### Built in functions
* **_log_abort** \<code\> \<message...>: Logs an error message to stderr and terminates the program (uses kill).
* **_log_error** <message...>: Logs an error message to stderr.
* **_log_warn** <message...>: Logs a warning message to stdout.
* **_log_info** <message...>: Logs an information message to stdout.
* **_log_debug** <message...>: Logs a debug message to stdout.
* **_log_done** <message...>: Logs a debug message to stdout and exits the program (uses exit 0).
* **_log_run** <cmd+params...>: Logs the debug command to stdout and runs the command.
* **_log_runb** <cmd+params...>: Logs the debug command to stdout and runs the command on background.
* **_get_var** <var1> <var2> ... <varx>: Returns the first non empty variable.
* **_get_fn** <fn1> <fn2> ... <fnx>: Returns the first existing function/command line.
* **_get_gfn** <fn1> <fn2> ... <fnx>: Returs the first existing gnu command line.
* **_is_fn** <fn>: Returns exit code 0 if is a valid function/command line.
* **_is_gfn** <fn>: Returns exit code 0 if is a valid gnu command line.
* **_ensure** <var> <code> <message...>: Ensure that var is not emply else calls _log_about.
* **_matches** <var> <regex>: Returns exit code 0 if is the variable is valid based on the regex.
* **_now**: Returns the date based on the format of the variable DATEFRM.

### Functions that you can overriden (original empty function)
* **_start** <args>: Called before the parsing of the arguments-
* **_cleanup**: Called when there is an interruption (^C or the process was killed)
* **_fisish**: Called just before the process finishs (after _cleanup of exit 0)

# **xshopt** example

### The program dss.sh
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

[ -n "$BASH_VERSION" ] && ARG0="${BASH_SOURCE[0]}" || ARG0="$0"
DIR="$(cd "$(dirname "$ARG0")" && pwd)"
. "$DIR/xshopt.sh"

### BEGIN (DON'T CHANGE THIS LINE) ###

# OPTIONS
METHOD=curl

opt_use() { [ "$1" = wget ] || [ "$1" = wget ] || _log_abort "Unsupported method '$METHOD'."; METHOD="$1"; }
opt_use_curl(){ use curl; }
opt_use_wget(){ use wget; }

# COMMANDS
DEFAULT_ADDR=me@demo.com

download(){
  case "$METHOD" in
    curl) _log_run curl -OL "$1";;
    wget) _log_run wget "$1";;
  asec
}

ssh_start(){
  local ADDR=$(_get_var "$1" $DEFAULT_ADDR)
  local PID="$(ps -ef | fgrep -v grep | fgrep ssh "$ADDR" | awk '{print $2}' | xargs echo)"
  [ -n "$PID" ] && _log_abort "SSH session already running with PID: $PID"
  _log_runb ssh "$ADDR"
  _log_run sleep 2
  PID="$(ps -ef | fgrep -v grep | fgrep ssh "$ADDR" | awk '{print $2}' | xargs echo)"
  [ -n "$PID" ] && _log_info "SSH session running with PID $PID" || _log_abort "SSH session didn't started"
}

ssh_kill{}{
  local ADDR=$(_get_var "$1" $DEFAULT_ADDR)
  local PID="$(ps -ef | fgrep -v grep | fgrep ssh "$ADDR" | awk '{print $2}' | xargs echo)"
  [ -z "$PID" ] && _log_warn "No SSH sessions are running"
  _log_run kill -s TERM $PID
}
```

### Some calling examples
```./myprog.sh (by default without parameters call the help)```
```sh
./myprog.sh --version
```
```sh
./myprog.sh --help
```
```sh
./myprog.sh --debug -s -d http://secure.com/file1 --use-wget -d http://demo.com/file2 -k \
                    -s me@demo.com --use=curl -d http://secure.com/file3 --use wget \
                    -d http://secure.com/file4 -k me@demo.com
```

### The shell funtions called by the framwork (for the last below command line):
```sh
opt_debug
ssh_start
download http://secure.com/file1
opt_use_wget
download http://secure.com/file2
ssh_kill
ssh_start me@demo.com
opt_use curl
download http://secure.com/file3
opt_use wget
download http://secure.com/file4
ssh_kill me@demo.com
```
