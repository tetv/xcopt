# **xcopt** library

### **xcopt** definition
* **xcopt** means literally: *eXtreme Command OPTions library*
* **xcopt** is compatible with bash and dash/POSIX scripting

### **xcopt** overview
This command line framework runs multiple *options* and *commands* (respecting the order) as if they were sub-programs. Typically, *options* are used for change state (set behaviour), and *commands* are used to do the real work. Although *options* and *commands* are threated similarly, at least one commands is always expected.

### **xcopt** helps you...
* Creating a sh scripts very quickly (compatible with POSIX);
* Focusing on the functionality (implementing *commands* and *options*);
* Refactoring *commands* and *options* easly just changing the help menu (header).

### **xcopt** takes care for you the need of...
* implementing the *--version* and *--help* commands;
* implementing logging options such as: *--error*, *--warning*, *--info*, *--quiet*, *--log-file=\<file>*
* implementing logging functions such as: *log_error*, *log_info*, *log_exit*, and more... (see below).
* implementing the *--include* and *--exclude* command whitch integrates the library with the main script;
* implementing other helpful methods like *get_var*, *get_fn* whitch returns the first valid argument;
* validating if the commands and options specified are correct based on header of the file;
* calling shell functions for each *commands* and *options*

# API

### Built in functions
* **log_error** \<message...>: Logs an error message to stderr.
* **log_warn** \<message...>: Logs a warning message to stdout.
* **log_info** \<message...>: Logs an information message to stdout.
* **log_debug** [code] \<message...>: Logs a debug message to stdout.
* **log_done** \<message...>: Logs a debug message to stdout and exits the program (uses exit 0).
* **log_exit** \<code> \<message...>: Logs an error message to stderr and exits the program (uses exit).
* **log_abort** \<code> \<message...>: Logs an error message to stderr and terminates the program (uses kill).
* **log_run** \<cmd+params...>: Logs the debug command to stdout and runs the command.
* **log_runb** \<cmd+params...>: Logs the debug command to stdout and runs the command on background.
* **log_exit_if_empty** \<var> \<code> \<message...>: Log the the first argument is not emply else calls log_exit.
* **get_var** \<var1> \<var2> ... \<varx>: Returns the first non empty variable.
* **get_fn** \<fn1> \<fn2> ... \<fnx>: Returns the first existing function/command line.
* **get_gfn** \<fn1> \<fn2> ... \<fnx>: Returs the first existing gnu command line.
* **is_fn** \<fn>: Returns exit code 0 if is a valid function/command line.
* **is_gfn** \<fn>: Returns exit code 0 if is a valid gnu command line.
* **is_regex** \<var> \<regex>: Returns exit code 0 if is the variable is valid based on the regex.
* **_now**: Returns the date based on the format of the variable DATEFRM.

### Functions that you can overriden (original empty)
* **_start** \<args...>: Called before the parsing of the arguments-
* **_cleanup**: Called when there is an interruption (^C or the process was killed)
* **_finish**: Called just before the process finish (after _cleanup of exit 0)

### Hidden functions
* **version**: Command *--version* prints claimer based on script lines started with '#- '.
* **help**: Command *--help* (usage) prints a menu based on script lines started with '## ' and do exit 0.
* **opt_error**: Option *--error* enables logging from only *log_error*, *log_exit*, *log_abort*.
* **opt_warning**: Option *--warning* enables logging from the same as above and *log_warn*.
* **opt_info**: Option *--info* enables logging from all *log_...* functions except *log_debug*.
* **opt_debug**: Option *--debug* enables logging from all *log_...* functions.
* **opt_log_file** \<file>: Option *--log-file=\<file>* redirects the stdout and stderr logging messages to a file.
* **_run** \<args...>: Main function that does the work of analaysing and call the *commands* and *options* with the correct number of arguments.

# Additional information

### Parameters:
* **\<gender>** or **GENDER**:      Required and can contain any value combination (E.g. *male*, *female*, ...)
* **male** or **(male)**:           Required and must contain the value specified (E.g. *male*)
* **(male|female)**:                Required and can caontain any value specified (E.g. *male* or *male*)
* **[\<gender>]** or **[GENDER]**:  Optional and can contain any value combination (E.g. *male*, *female*, ...)
* **[male]** or **[(male)]**:       Optional and must contain the value specified (E.g. *male*)
* **[male|female]**:                Optional and can contain any value specified (E.g. *male* or *male*)

### Tips:
* To remove logging features, just remove it from the header definition (don't need to touch **xcopt** library)
* The *command* implementation function can have the prefix cmd_. [E.g. --my-command => [cmd_]my_command(){}]
* The *options* implementation function must have the prefix opt_. [E.g. --my-option => opt_my_option(){}]
* **Paramters**:
  * Long parameters with arguments may use as well equals sign: (E.g. *--cmd=param* or *--cmd param*]
  * Parameter like **\<name>** are required and doesn't have restrictions. [E.g. *--cmd=-1* or *--cmd -1*]
  * Parameter like **name** are required and must contains that value. [E.g.: *--cmd=name*]
  * Parameter like **[name]** are optional with some restrictions. [E.g. *--cmd=-1*, *--cmd 1* or *--cmd -- -1*]
  * Note: The optional values can't start with - unless if used the = version. The workarround is use -- before.

## Include and exclude [Future]
* It's possible to *--include* the **xcopt** library as part of them main script. (Simplifies distribution)
* It's possible to *--remove* the **xcopt** library from the main script (in is integrated on it).

### Limitations:
* All *command* and *options* needs to have a long name starting with --. [E.g.: *--help*]
* All *command* and *options* can have a short version starting with -. [E.g.: *-h*]
* For now only one parameter is supported in *commands* and *options*.
* The argument validations are done in real time and not ahead (before starting calling *commands* or *options*).

### Future:
* Support multi arguments per *commands* and *options*. [E.g.: *-s, --swap <x> <y>*)

# Example

### The program dss.sh
```sh
#!/bin/sh
#- $PROG (Dowload some stuff) 1.0
#- Copyright (C) 2015 Author, <author@domain.com>
#- License MIT <http://opensource.org/licenses/MIT>.
#- * This is free software: you are free to change and redistribute it.
#- * There is NO WARRANTY, to the extent permitted by law.
#
## $PROG (eXtreme Commmand OPTions library) 1.0
## Compatible with bash and dash/POSIX
## 
## Usage: $PROG [OPTION|COMMAND]...
## Options:
##       --info                   Set log level to info (default)
##       --warning                Set log level to warning
##       --debug                  Set log level to debug
##   -q, --quiet                  Set log level to error (quiet mode)
##   -l, --log-file=<file>        Set log file (stdout and stderr)
##       --use-curl               Set the download method as curl (default)
##       --use-wget               Set the download method as wget
##       --use=(curl|wget)        Set the download method
## Commands:
##   -d, --download=<url>         Dowload a specific <url>
##   -s, --ssh-start=[addr]       Start ssh to grant network access
##   -k, --ssh-kill=[addr]        Kill ssh to remoke network access
##       --help                   Displays this help and exists
##       --version                Displays output version and exists

[ -n "$BASH_VERSION" ] && ARG0="${BASH_SOURCE[0]}" || ARG0="$0"
DIR="$(cd "$(dirname "$ARG0")" && pwd)"
. "$DIR/xcopt"

### BEGIN (DON'T CHANGE THIS LINE) ###

# VARIABLES
METHOD=curl
DEFAULT_ADDR=me@demo.com

# OPTIONS
opt_use() { [ METHOD="$1"; }
opt_use_curl(){ use curl; }
opt_use_wget(){ use wget; }

# COMMANDS
download(){
  case "$METHOD" in
    curl) log_run curl -OL "$1";;
    wget) log_run wget "$1";;
  asec
}

ssh_start(){
  local ADDR=$(get_var "$1" $DEFAULT_ADDR)
  local PID="$(ps -ef | fgrep -v grep | fgrep ssh "$ADDR" | awk '{print $2}' | xargs echo)"
  [ -n "$PID" ] && log_exit 12 "SSH session already running with PID: $PID"
  log_run ssh "$ADDR" printf ''
  log_runb ssh -tt "$ADDR"
  log_run sleep 2
  PID="$(ps -ef | fgrep -v grep | fgrep ssh "$ADDR" | awk '{print $2}' | xargs echo)"
  [ -n "$PID" ] && log_info "SSH session running with PID $PID" || log_error 13 "SSH session didn't started";
}

ssh_kill{}{
  local ADDR=$(get_var "$1" $DEFAULT_ADDR)
  local PID="$(ps -ef | fgrep -v grep | fgrep ssh "$ADDR" | awk '{print $2}' | xargs echo)"
  [ -z "$PID" ] && log_warn "No SSH sessions are running"
  log_run kill -s TERM $PID
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
