Notes for "Learning the bash shell" by O'Relly
===============================

## Useful commands
### I/O
- `cut`: extract specific column
- `sed`: perform editing on input
- `tr`: translate or delete specific character
- `tee`: Read from standard input, print to a file and standard output 
- `printf "%q" string` prints string in the form of shell input
- `read var1 var2 ...`: read a line from standard input and assign space-separated strings to variables. All remaining strings are assigned to the last variable, if number of variables is smaller than number of words in the line. Useful when processing line-based text. Exit status is 1 when reaching EOF. 

### Mis
- `whereis`: searches standard locations for any possibly useful binaries, executables, packages, etc
- `which`: searches user-defined paths for executables only
- `type`: prints the full path of a given command/alias, etc. This is how shell will interpret the queried command. `type -a` shows all possible interpretations, and `type -p` shows scripts and executables only. 
- `cd -`: goes to the previous visited directory

--------------------------------

## Pattern
- Set: e.g. `[a-g]`, `[^1-5]`, `[!A-Z]`
- Bracket: e.g. `{a,b,e}`, `{2..5}`
    - **Note** that bracket expansion does not have to match a pathname.   

Matching a list of patterns (if `shopt extglob` is on)
`*`,`+`,`?`,`@`,`!`, followed by a list of patterns (i.e. `(pattern|pattern|...)`) , matches different number of patterns (respectively: none or any, at least one, 1 or 0, exactly 1, exactly none)

---------------------------------

## Environment
- alias: bash only search for alias for the first word of command. If other words are meant to be searched (e.g. directory shortcut), the command must be an alias ending with a blank (e.g. `alias cd='cd '`)
- options: `set -o optionname` turns an option on, and `set +o optionname` turns it off. Also, more advanced `shopt -s optionname` and `shopt -u optionname` can be used. 
- `.bash_profile` is executed by the login shell, while `.bashrc` is executed at the start of a subshell. Only commands with specific output (e.g. login logs) should be put in `.bash_profile` (usually followed by a `source .bashrc`), while options and environment variables should be put in the `.bashrc` file.  

### Environment Variables
- `CDPATH`: path of `cd` to look for if the directory entered does not exist. Useful when making frequent visits to a group of directories. 
- `export`: sets a variable as environment variable (and thus makes it visible for all subprocesses). For temporary variable, use `varname=varvalue command`.

---------------------------------

## Function
#### Definition
```bash 
function funcname 
        {

        }
```
or 
```bash     
funcname ()
        {

        }
```

#### Positional Arguments
*Applies to both functions and scripts*
- `$*` is all *arguments* concatenated with the first character in $IFS (normally space), then surrounded with double quotes (i.e. considered as a single string)
- `$@` is all *arguments* surrounded with double quotes, separated by spaces (i.e. considered as separate strings, which can be used as arguments of other functions)
- `$i` is the i-th argument. `$0` is usually the function/script name
- `$#` is the number of arguments

**Note** `$i` is abbreviation of `${i}`. `$i` can be safely used if no letter, number or underscore follows it.

**Note** `$@` and `$*` don't include `$0` (script name) 

#### Variable Scope
Only positional arguments `$1,$2,...` are local to functions. Other variables, if not declared `local` explicitly, are shared outside the function (restricted to the subshell, if the script is run with `bash`)

**Note** positional argument `$0` is shared and read-only within the scope of  script, no matter in function or not. 

### Order of precedence for shell commands
1. aliases
2. keywords, including functions and flow control words (e.g. `if`)
3. functions
4. built-ins, e.g. `cd`
5. scripts and executables 

#### Mis
- `declare -f` shows all function definitions, and `declare -F` shows all functions (name only)

**Note** `source` a script runs it in the current shell, and `bash` a script runs it in the subshell (where no functions/variables affect the current shell). *Only* environment variables are passed to subshell (and they can't be modified by the subshell, even with `export`). 

**Note** Function, or any statement block (flow control block or statements wrapped in `{}`), has its own standard file descriptors. 

**Note** Functions belong its parent shell (or the subshell running the script). Therefore any `trap` in the parent shell also works in the function, and any `trap` in the function alwo works in the parent shell, after the function is run once. 

------------------------------

## String Manipulation
#### String Substitution Operators
- `${varname:-word}`: return a default value `word` is `varname` is undefined (doesn't define `varname`)
- `${varname:=word}`: set `varname` to its default value `word` if it's undefined, then return `word`. Since positional arguments can't be defined manually, this can't be applied to positional arguments. 
- `${varname:?message}`: abort the command and print the error `message` if `varname` doesn't exist. 
- `${varname:+word}`: return `word` if `varname` exists and isn't null. otherwise return null. 
- `${varname:offset:length}`: return substring of `varname` starting at `offset`, with length `length`. Negative offset is supported. 

**Note**: only `${varname:=word}` changes value of `varname`. 

#### Pattern-matching Operators
- `${variable#pattern}`: if matching the *beginning* of the variable's value, delete the *shortest* part that matches and return the rest. 
- `${variable##pattern}`: if matching the *beginning* of the variable's value, delete the *longest* part that matches and return the rest. 
- `${variable%pattern}`: if matching the *end* of the variable's value, delete the *shortest* part that matches and return the rest. 
- `${variable%%pattern}`: if matching the *end* of the variable's value, delete the *longest* part that matches and return the rest. 
- `${variable/pattern/string}`: The longest match to pattern in variable is replaced by string. Only the *first* match is replaced. If pattern begins with `#` or `%`, it must match with the beginning or end of the variable. 
- `${variable//pattern/string}`: same as `${variable/pattern/string}`, except now *all* matches are replaced. 

**Note** none of these operators modifies variable. 

**Note** if the variable in replace operation (only the one with one slash) is `@` or `*`, then each argument is considered separately, then concatenate together. If the replace operation is meant for the entire argument string, then `@` or `*` should be assigned to a variable first, then perform the operation. 

#### Length Operator
- `${#varname}`: returns length of string

#### Command Substitution
- `$(command)`: refers to the *standard output* of a UNIX command
- `$(< filename)`: the contents fo a file with all trailing newlines removed

---------------------------------

## Flow Control
#### Condition and Exit Status

The condition is the exit status of a given command. 0 is true, other values are false. 

- `$?`: gives the exit status of the last command 
- `return N`: returns N as exit status
    - **Note** N is an integer in [0,255] and should not be used as return value. For return value, use command substitution. 
- `return`: return the exit status of the last command as the exit status
    - **Note** if no `return` statement is given, the exit status of the last command is automatically returned. 
-`exit N`: exit the entire script with exit status N, no matter how deep the statement is nested in functions

**Condition operators**
- between test statements: `&&` for and, `||` for or, `!` for not
- inside test statements: `-a` for and, `-o` for or, `!` for not. statements must be surrounded by `\(` and `\)`

Short-circuit evaluation (the next statement is evaluated only when the value of the entire conditional is still unclear after evaluating the current one) is followed between test statements, but not inside one (i.e. work for `&&`, but not for `-a`). 

#### Condition and Statement

A test statement in square bracket `[ ... ]` or `[[ ... ]]` is considered as condition and evaluate to an exit status. 

**String comparison**
- `=` for equal (only *ONE* equal sign)
- `!=` for not equal
- `<` for smaller than (no smaller or equal sign exists)
- `>` for greater than 
- `-n str` str is not null
- `-z str` str is null 
- **Note** for string comparison, space must surround the comparison sign. 

**File Attributes**
- `-a`, `-e`: file exists
- `-d`: file exists as a directory
- `-f`: file exists as a regular file
- `-r`: user has read permission of the file
- `-w`: user has write permission of the file
- `-x`: user has execute permission of the file (or search permission, if it's a directory)
- `-s`: file exists and is not empty
- `-N`: file modified since it's last read
- `-O`: user owns the file
- `-G`: file's group matches the user's
- `file1 -nt file2`: file1 is newer than file2 (consider modification time)
- `file1 -ot file2`: file1 is older than file2 (consider modification time)

**Number comparison**
`-eq`, `-lt`, `-gt`, `-le`, `-ge`, `-ne`, with obvious effect from corresponding abbreviation. 

#### Syntax
**if/else**

```bash 
if condition
then            # then must be put on the next line, otherwise a semicolon must follow condition
    statement
elif condition
    statement
else 
    statement
fi
```

**for**

```bash
for name in list    # if in list is omitted, $@ is used. 
do 
    statement
done
```

**Arithmetic `for` loop**

```bash
for ((initialization ; continue condition ; update)); do 
    statements
done
```

- The loop is omitted if initialization evaluate to false

**case**

Used for multiple cases of pattern matching

```bash
case expression in 
    pattern )   statements;;
    pattern )   statements;;
    pattern1 | pattern2 ) statements;;
esac
```

**select**

Prompt options for user to choose. If a valid number is entered, corresponding string is assigned to the variable, otherwise it's null. 

*Trick* The string `PS3` is displayed when options are shown. It's probably better to assign a meaningful string to it. 

**Note** Select is an infinite loop unless explicitly `break`

**Note** Select prompt is directed to standard error, not standard output

```bash
select name in list; do 
    statements
done
```

**while/until**

`while` and `until` has the same syntax. The only difference is that `while` loops until the condition is false, while `until` loops until the condition becomes true. *Both* check for the condition *before* running.   

```bash
while condition; do 
    statement
done
```

---------------------------------

## Script Option Handling
#### Straightforward way
- `shift` shifts positional arguments `$i` to `$i-1` for all `i`. Used when dealing with options.
- `shift N` run `shift` `N` times

#### `getopts`

`getopts options varname`: processes `$@`, one option at a time, and save the option (without '-') in varname. A colon following an option indicates that option requires an argument. An error is thrown if invalid option is found, unless `options` begins with a colon, in which case '?' is assigned to `varname`. `getopts` returns 0 as long as a valid option is found, so it's usually used in `while` loop. 

Arguments associated with options are saved in `$OPTARG`. 

The number of the next argument to be processed is saved in `$OPTIND`, so `shift $(($OPTIND - 1))` is needed after processing all options. 

---------------------------------

## Integer

- Use `declare -option varname` to declare variable as a certain type. `-a` for array, `-i` for integer and `-r` for readonly

**Expressions**
- Use `$((expression))` to evaluate an arithmetic expression
- Use `let varname=expression` to evaluate an expression and assign the *string* value to `varname`
- If not declared as integers, variables will be considered as string, but automatically converted to integers when used in arithmetic expressions.  

**Integer Comparison**
- Integer comparison can be performed in expressions (e.g. `$(( 2>1 ))`). The return value is 0 or 1. **Note** that such expressions are considered as values, and thus cannot be used directly as conditions (i.e. need to compare with 1). 
- Integer comparison can also be performed using `test` (i.e. `[...]`). e.g. `[ 3 -gt 2 ]`

## Array

**Declaration**
- `declare -a varname` declare an empty array
- `varname=([1]=value1,[3]=value3)` initialize an array
- `varname=(value1,value3,...)` initialize an array with index starting from 0

**Refering to an entry**
- `${varname[index]}` get the value of the index-th entry
- `${varname[@]}` can be used in for loop to iterate through all values, in increasing order of index
- `${!varname[@]}` gives the indices with meaningful entries
- `${#varname[@]}` gives the number of meaningful entries 

---------------------------------

## Processes

All background processes of a shell is given a job number (local to the shell) and a process id (unique within the system). 

**Referring to a job**

- `%+` and `%%` refers to the most recent job, and `%-` is the next most recent job. 
- `%N` job number N
- `%string` refers to the most recent job whose command begins with the string. 
- `%?string` refers to the most recent job whose command contains the string.  

**Basics**

- `jobs` shows all jobs. 
- `jobs -x command` substitutes all `%job_number`, `%command` to process ids and then execute the command
- `fg` brings a job to foreground and resumes it, if it's suspended. If none supplied, the lastest one is brought to foreground
- `bg` brings a job to background and resumes it, if it's suspended. 
- CTRL-Z suspend a job and puts it in background. The job can be resumed if brought to foreground with `fg`. Especially *useful* when writing source code with vi. 
- `$$` process id of the current shell
- `$!` process id of the last background command

**`kill`**

`kill -s SIG process` send a signal to process. By default TERM is sent. In the increasing order of power: TERM, QUIT, KILL
`kill -l` shows a list of signal number and name (*varies across systems*)

**`ps`**

`ps`, by default, list all processes under the current shell. It can't display job number since it's not a shell command. 

- `ps -a` list all processes under terminals (no matter which one)
- `ps -ax` list all processes, regardless of whether they belong to a terminal

**`trap`**

`trap command sig1 sig2 ...`: executes `command` when any of signals is triggered, then resumes from where it's interrupted. 

If `command` is "", the signal is ignored. If `command` is -, the action is reset to the default action to the signal. 

**Mis**
- `nohup command`: execute command, ignoring hangup signal (network outage, e.g. ssh logout) 
- `disown process`: disown a process by the shell
- `disown -h process`: same as `nohup` 
- `nice`: lower the command priority (by default all commands of the same user enjoys the same priority)

---------------------------------

## Note about Bash Main Syntax
1. Bash is heavily based on string, and everything is considered as a string, separated by spaces. Therefore, spaces are encouraged whenever available. 
2. Since everythine is considered as a string (as opposed to number, variable, etc.), when refering to values, `$` is always needed. (assignment is an exception)
3. `=` serves as the string equality test. Therefore, to distinguish equality test from variable assignment, no spaces are allowed in assignment syntax. 
    - **Note** to check for string equality, use `[ $str1 = $str2 ]`. Omitting spaces around `=` will always gives true (assignment with exit status 0)
4. Only inside `$()` and at position of test conditions are strings considered as commands. 
5. Operators (like string operators) and commands are handled differently. Operators usually have meaningful return values, while commands have meaningful output (like to stdout), with exit status in [0,255].  

**Question** The use of semicolon and how it's processed is still unclear. This syntax is important in flow control.

---------------------------------

## Tricks
- Redirect output to `less` to enable output reading control
- CTRL-S suspends standard output, and CTRL-Q resumes it
- CTRL-U erase the entire line of command (useful for reentering commands), and CTRL-W erase the previous word
- `!!` re-executes the last command, `!string` re-exectutes the last command starting with *string*, `!?string?` re-executes the last command containning *string*
- Use `echo -e -n` to get rid of trailing endline unless explicitly adding \n. 

**Note** The pattern is automatically expanded to all matching filenames when sent to command (unless no match can be found). So the command cannot see the pattern used unless no match exists. Therefore quotation is required if the command expects a pattern that's not expanded by bash automatically (e.g. `find`). 












