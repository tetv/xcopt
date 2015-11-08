# **xshopt** library

### **xshopt** definition
* **xshopt** means literally: *eXtreme SH Opt Library*
* **xshopt** is compatible with bash and dash/POSIX scripting

### **xshopt** overview
This command line framework runs multiple *options* and *commands* (respecting the order) as if they were sub-programs. Typically, *options* are used for change state (set behaviour), and *commands* are used to do the real work. Although *options* and *commands* are threated similarly, at least one commands is always expected.

### **xshopt** helps you...
* Creating a sh scripts very quickly (compatible with POSIX);
* Focusing on the functionality (implementing *commands* and *options*);
* Refactoring *commands* and *options* easly just changing the help menu (header).

### **xshopt** takes care for you the need of...
* implementing the *--version* and *--help* commands;
* implementing logging options such as: *--error*, *--warning*, *--info*, *--quiet*, *--log-file=\<file>*
* implementing logging functions such as: *_log_error*, *_log_info*, *_log_exit*, and more... (see below).
* implementing the *--include* and *--exclude* command whitch integrates the library with the main script;
* implementing other helpful methods like *_get_var*, *_get_fn* whitch returns the first valid argument;
* validating if the commands and options specified are correct based on header of the file;
* calling shell functions for each *commands* and *options*

# API

### Built in functions
* **_log_error** \<message...>: Logs an error message to stderr.
* **_log_warn** \<message...>: Logs a warning message to stdout.
* **_log_info** \<message...>: Logs an information message to stdout.
* **_log_debug** [code] \<message...>: Logs a debug message to stdout.
* **_log_done** \<message...>: Logs a debug message to stdout and exits the program (uses exit 0).
* **_log_exit** \<code> \<message...>: Logs an error message to stderr and exits the program (uses exit).
* **_log_abort** \<code> \<message...>: Logs an error message to stderr and terminates the program (uses kill).
* **_log_run** \<cmd+params...>: Logs the debug command to stdout and runs the command.
* **_log_runb** \<cmd+params...>: Logs the debug command to stdout and runs the command on background.
* **_get_var** \<var1> \<var2> ... \<varx>: Returns the first non empty variable.
* **_get_fn** \<fn1> \<fn2> ... \<fnx>: Returns the first existing function/command line.
* **_get_gfn** \<fn1> \<fn2> ... \<fnx>: Returs the first existing gnu command line.
* **_is_fn** \<fn>: Returns exit code 0 if is a valid function/command line.
* **_is_gfn** \<fn>: Returns exit code 0 if is a valid gnu command line.
* **_ensure** \<var> \<code> \<message...>: Ensure that var is not emply else calls _log_about.
* **_matches** \<var> \<regex>: Returns exit code 0 if is the variable is valid based on the regex.
* **_now**: Returns the date based on the format of the variable DATEFRM.

### Functions that you can overriden (original empty)
* **_start** \<args...>: Called before the parsing of the arguments-
* **_cleanup**: Called when there is an interruption (^C or the process was killed)
* **_finish**: Called just before the process finish (after _cleanup of exit 0)

### Hidden functions
* **version**: Command *--version* prints claimer based on script lines started with '#- '.
* **help**: Command *--help* (usage) prints a menu based on script lines started with '## ' and do exit 0.
* **opt_error**: Option *--error* enables logging from only *_log_error*, *_log_exit*, *_log_abort*.
* **opt_warning**: Option *--warning* enables logging from the same as above and *_log_warn*.
* **opt_info**: Option *--info* enables logging from all *_log_...* functions except *_log_debug*.
* **opt_debug**: Option *--debug* enables logging from all *_log_...* functions.
* **opt_log_file** \<file>: Option *--log-file=\<file>* redirects the stdout and stderr logging messages to a file.
* **_run** \<args...>: Main function that does the work of analaysing and call the *commands* and *options* with the correct number of arguments.

# Additional information

### Tips:
* To remove logging features, just remove it from the script header (don't need to touch **xshlog** library)
* The *command* implementation function names matches the long name. [E.g. --my-command => my_command(){}]
* The *options* implementation function names must have the prefix opt_. [E.g. --my-option => opt_my_option(){}]
* **Paramters**:
 * Long parameters with arguments may use as well equals sign: (E.g. *--cmd=param* or *--cmd param*]
 * Parameter like **\<name>** are required and doesn't have restrictions. [E.g. *--cmd=-1* or *--cmd -1*]
 * Parameter like **name** are required and must contains that value. [E.g.: *--cmd=name*]
 * Parameter like **[name]** are optional with some restrictions. [E.g. *--cmd=-1*, *--cmd 1* or *--cmd -- -1*]
 * Note: The optional values can't start with - unless if used the = version. The workarround is use -- before.

## Include and exclude [Future]
* It's possible to *--include* the **xshopt** library as part of them main script. (Simplifies distribution)
* It's possible to *--exclude* the **xshopt** library from the main script.

### Limitations:
* All *command* and *options* needs to have a long name starting with --. [E.g.: *--help*]
* All *command* and *options* can have a short version starting with -. [E.g.: *-h*]
* For now only one parameter is supported in *commands* and *options*.
* The argument validations are done in real time and aren't done ahead (before starting calling *commands*).

### Future:
* Support multi arguments per *commands* and *options*. [E.g.: *--swap <x> <y>)
* Support format like *(male|female|unknown)* for accept mandatorly one specific value
* Support format like *[male|female|unknow]* for accept optionaly one specific value
* Support format like *[<param>] for accept any value as a optional value.

# Example

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
##   -s, --ssh-start=[addr]       Start ssh to grant network access
##   -k, --ssh-kill=[addr]        Kill ssh to remoke network access
##       --help                   Displays this help and exists
##       --version                Displays output version and exists

[ -n "$BASH_VERSION" ] && ARG0="${BASH_SOURCE[0]}" || ARG0="$0"
DIR="$(cd "$(dirname "$ARG0")" && pwd)"
. "$DIR/xshopt.sh"

### BEGIN (DON'T CHANGE THIS LINE) ###

# OPTIONS
METHOD=curl

opt_use() { [ "$1" = curl ] || [ "$1" = wget ] || _log_exit 11 "Unsupported method '$METHOD'."; METHOD="$1"; }
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
  [ -n "$PID" ] && _log_exit 12 "SSH session already running with PID: $PID"
  _log_runb ssh "$ADDR"
  _log_run sleep 2
  PID="$(ps -ef | fgrep -v grep | fgrep ssh "$ADDR" | awk '{print $2}' | xargs echo)"
  [ -n "$PID" ] && _log_info "SSH session running with PID $PID" || _log_error 13 "SSH session didn't started";
}

ssh_kill{}{
  local ADDR=$(_get_var "$1" $DEFAULT_ADDR)
  local PID="$(ps -ef | fgrep -v grep | fgrep ssh "$ADDR" | awk '{print $2}' | xargs echo)"
  [ -z "$PID" ] && _log_warn "No SSH sessions are running"
  _log_run kill -s TERM $PID
}
```

### Some calling examples
```./myprog.sh``` (by default without parameters call the help)
```./myprog.sh --version```
```./myprog.sh --help```
```
./myprog.sh --debug -s -d http://secure.com/file1 --use-wget -d http://secure.com/file2 -k \
                    -s me@demo.com --use=curl -d http://secure.com/file3 --use wget \
                    -d http://secure.com/file4 -k me@demo.com
```

### Shell funtions called (for the last above calling exmple):
```sh
opt_debug
ssh_start
download 'http://secure.com/file1'
opt_use_wget
download 'http://secure.com/file2'
ssh_kill
ssh_start 'me@demo.com'
opt_use curl
download 'http://secure.com/file3'
opt_use wget
download 'http://secure.com/file4'
ssh_kill 'me@demo.com'
```
