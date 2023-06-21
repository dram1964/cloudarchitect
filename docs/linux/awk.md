# Awk

Taken from [Tutorials Point](https://www.tutorialspoint.com/awk/index.htm)
See also [Geeks for Geeks](https://www.geeksforgeeks.org/awk-command-unixlinux-examples/)

'awk' is a interpreted programming language designed for text processing. An awk program consists
of commands that are executed sequentially on lines of text. Optional BEGIN and END blocks can
be used for setup and tear-down. Use single-quotes to mark the start and end of the awk script.

In its simplest form awk can be used to print the contents of a file:

    awk '{print}' <filename>

The '-v' option can be used to assign variables before the awk commands are executed:

    awk -v name="World!" 'BEGIN{printf "Hello, %s\n", name}'

'awk' treats input lines and output lines as space-separated fields. Columns from each line can be printed using
their indexes (starting from 1):

    awk '{print $3,$4}' <filename

Or you can insert a tab separator between the output fields:

    awk '{print $3 "\t" $4}' <filename>

Filters can be added to select which lines to print:

    awk '/<pattern>/ {print $3 "\t" $4}' <filename>

In the absence of a body block, awks default action is to print the whole 
record. $0 can be used to refer to the whole input record. These two 
programs therefore produce the same output:

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
