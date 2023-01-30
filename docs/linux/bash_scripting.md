# Bash Scripting Notes

Common Bash scripting techniques.

## Variable names

Encapsulating variables in curly brackets to avoid ambiguity:

    FirstThree=ABC
    echo "The first six are ${FirstThree}DEF"

Encapsulating variable in double quotes preserves whitespace:

    FirstThree='A  B  C'
    echo $FirstThree    # prints 'A B C'
    echo "$FirstThree"   # prints 'A  B  C'

Use $() or backticks to assign variables from the output of a command:

    now=$(date +%Y-%m-%d)
    echo $now

Assign an array using brackets around the list values. Use {} for disambiguation, and '@' 
to access the whole array or an index value for a single element:

    contents=($(ls))
    echo ${contents[@]}
    echo ${#contents[@]}  # number of elements in array
    echo ${contents[0]}   # first element
    echo ${contents[-1]}  # last element

You can also assign directly to an array element:

    contents[3]=banana

Arguments are passed to scripts as space-separated values. '$0' holds the filename,
'$1' is the first argument. '$#' holds the number of arguments passed to the script,
and $@ holds a space-separated list of all arguments. 

## Arithmetic and Strings

Bash supports arithmetic operators using an arithmetic expression $(()):

    INCOME=10000
    TAX=$(($INCOME / 100 * 25))
    echo $TAX

Several string operators are available. It is worth noting that the 'index' operator returns the 
numerical position of the search term, rather than the index-value of the search:

    STRING="Nice to see you, to see you, nice"
    echo ${#STRING}                         # length of string

    expr index "$STRING" "s"                # 9: position of first 's' IN $STRING
    echo ${STRING:$((9-1)):3}               # see: 3 characters from index 8
    echo ${STRING:9-1:3}                    # see: 3 characters from index 8
    echo ${STRING:8}                        # string from index 8 to end
    expr index "$STRING" "you"              # 7: position of first 'y','o' or 'u' IN $STRING
    echo ${STRING[@]/you/me}                # replace first 'you' with 'me'
    echo ${STRING[@]//you/me}               # replace all occurrences of 'you' with 'me'
    echo ${STRING[@]// you/}                # delete all occurrences of ' you'
    echo ${STRING[@]/#Nice/Good}            # replace characters at the start of the string
    echo ${STRING[@]/%nice/OK}              # replace characters at the end of the string
    echo ${STRING[@]/%nice/$(pwd)}          # replace characters at the end of the string

## Conditionals

The basic 'if' statement takes this form:

    if [ <condition> ]; then
        # statements to execute
    elif [ <condition ]; then
        # statements to execute
    else
        # statements to execute
    if

Conditionals can be a single test or combine multiple tests using '&&' or '||'. When using 
multiple tests, surround the conditional in double brackets '[[]]'. 

Numeric comparisons are:

| Operator | Meaning |
| --- | --- |
| -lt | less than |
| -gt | greater than |
| -le | less than or equal to |
| -ge | greater than or equal to |
| -eq | equal to |
| -ne | not equal to |

String comparisons are: 

| Operator | Meaning |
| --- | --- |
| = | is the same as |
| == | is the same as |
| != | is different | 
| -z | is empty |

Use double quotes around strings in comparisons to avoid shell expansion.

Case statements take the form: 

    case "$variable" in
        "$value1" )
            command
        ;;
        "$value2" | "$value3")
            command
        ;;
        ...
        * )
            default command
        ;;
    esac

## Loops

- 'for' loops look like this: 

        for arg in [list]
        do
            command $arg
        done

- 'while' loops look like this: 

        while [ condition ]
        do
            command
        done

- 'until' loops look like this: 

        until [ condition ]
        do
            command
        done

Loop constructs support 'break' and 'continue' statements.  

An example 'for' loop:

    NUMBERS=(12 13 16 18 21 25 23 28 33 35)

    for n in ${NUMBERS[@]}
    do
        if [ $n = 33 ]; then 
            break
        elif [ $(($n % 2)) = 1 ]; then
            echo $n
        else
            echo "Not an odd number"
        fi
    done

## Functions

Functions are declared are declared with the key word 'function':

    function f1 {
        # access parameters as $1, $2, etc
    }

Call the function by name, supplying any required arguments: 

    function f1 {
        echo "${1}, ${2}"
    }

    STMT=$(f1 "Hello" "World!")

    echo $STMT

Use command substitution to collect the output from the function as a return value. Use the 'local' 
keyword to scope your variables to the functions scope to avoid changing variables in the
global scope:

    function f1 {
        GREET="Hello"
        local SCOPE="Earth"
        echo "${GREET}, ${SCOPE}!"
    }

    GREET="Goodbye"
    SCOPE="World"

    f1 $GREET $SCOPE           # changes $GREET globally

    echo "${GREET}, ${SCOPE}!" # prints "Hello, World!"

When using command substitution around your function calls, the variables are scoped to the 
statement, so variables in the function will not overwrite globals:


    function f1 {
        GREET="Hello"
        local SCOPE="Earth"
        echo "${GREET}, ${SCOPE}!"
    }

    GREET="Goodbye"
    SCOPE="World"

    echo $(f1 $GREET $SCOPE)   # prints "Hello, Earth!" but does not change global $GREET

    echo "${GREET}, ${SCOPE}!" # prints "Goodbye, World!"

## Special Variables and Signals

A number of special variables are available to scripts and functions:

| Variable Name | Meaning |
| --- | --- |
| $0 | filename of current script | 
| $n | *n*th argument to script or function | 
| $# | number of arguments passed |
| $@ | all arguments passed |
| $\* | all arguments passed |
| $? | exit status of last command |
| $$ | process id of current shell |
| $! | process number of last background job |

You can use trap to intercept OS signals such as Ctrl+C (SIGINT) or Ctrl+D (SIGQUIT):

    trap <code to execute> $SIG1 $SIG2 ...

Signals can be specified by name or number: use `kill -l` for a list of signals. The code 
to execute can be inline or a function name. 

## File Tests

| Name | Test |
| --- | --- |
| -e | file exists |
| -d | directory exists |
| -r | file is readable |


## Awk

'awk' is a interpreted programming language designed for text processing. An awk program consists
of commands that are executed sequentially on lines of text. Optional BEGIN and END blocks can
be used for setup and tear-down. Use single-quotes to mark the start and end of the awk script.

In its simplest form awk can be used to print the contents of a file:

    awk '{print}' <filename>

The '-v' option can be used to assign variables before the awk commands are executed:

    awk -v name="World!" 'BEGIN{printf "Hello, %s\n", name}'

'awk' treats input lines as space-separated fields. Columns from each line can be printed using
their indexes (starting from 1):

    awk '{print $3 "\t" $4}' <filename>

Filters can be added to select which lines to print:

    awk '/<pattern>/ {print $3 "\t" $4}' <filename>

In the absence of a body block, awks default action is to print the whole record. $0 
can be used to refer to the whole input record: These two programs therefore produce the
same output:

    awk '/<pattern>/ {print $0}' <filename>
    awk '/<pattern>/' <filename>

Variables can be used without prior declaration:

    awk '/<pattern>/{++counter} END{print "Lines matched = ", counter}' <filename>

'length' is a built-in function to return the length of its operand. To print all lines longer than
82 characters use:

    awk 'length($0) > 82' <filename>    # using default action of {print}

Awk comes with a number of built-in variables:

| Variable | Meaning |
| --- | --- |
| ARGC | number of arguments provided at the command line | 
| ARGV | an array of the command line arguments | 
| CONVFMT | the conversion format for numbers | 
| ENVIRON | hash of environment variables | 
| FILENAME | the current filename | 
| FS | current field separator (default is space) | 
| RS | current record separator (default is newline) |
| NF | number of fields in current record | 
| NR | number of the current record |  
| FNR | number of current record in current file |
| OFMT | the output format for numbers | 
| OFS | the output field separator (default space) | 
| ORS | the output record separtor (default newline) |
| RLENGTH | the length of the string matched in `match` function | 
| RSTART | the first position in the string matched in `match` function |
| SUBSEP | the separator character for array subscripts |

    awk 'BEGIN {print "Arguments = ", ARGC}' one two three four

    awk 'BEGIN {
        for (i=0 ; i < ARGC -1 ; i++) {
            printf "ARGV[%d] = %s\n", i, ARGV[i]
        }
    }' one two three four

    awk 'BEGIN { print ENVIRON["USER"] }'

    awk 'NF < 10' <filename>                # print lines with less than 10 fields
    awk 'NF < 10 {print NF}' <filename>     # print number of fields in lines with less than 10 fields

    awk 'NR < 11' <filename>                # prints first 10 records 

    awk 'BEGIN { if (match("Hello, World!", "or")) {print RLENGTH}'

Additional variables are available with GNU AWK (gawk):

| Variable | Meaning | 
| --- | --- |
| ARGIND | Index in ARGV of the current file being processed |
| BINMODE | used to set binmode for file I/O |
| ERRNO | error number if getline or close fails |
| FIELDWIDTHS | use a space-separated list of fieldwidths instead of a delimiter |
| IGNORECASE | used to make awk case-insensitive |
| LINT | used to dynamically set lint options | 
| PROCINFO | used to access process information | 
| TEXTDOMAIN | text domain of the AWK program | 
