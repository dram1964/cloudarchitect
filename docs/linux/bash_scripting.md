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
        echo "$1 $2"
    }

    f1 "Hello," "World!"
