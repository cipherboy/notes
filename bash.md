# bash

To use numbers in an if statement:

    if (( $var >= 2 )); then
        echo "$var >= 2"
    fi

To get the number of arugments (excluding the executable path):

    function do_arg_count() {
        echo $#
    }

To remove the first `<n>` arguments from `$@`:

    function do_shift() {
        shift
    }

To move through the history:

    <up arrow> or <ctrl> + p -- backwards
    <down arrow> or <ctrl> + n -- forward
    <ctrl> + r -- reverse search

To echo an escape code:

    echo -e "\e[<code>m"

To read into an array:

    mapfile -t VAR_NAME < <(cmd)

The length of an array:

    ${#array[@]}

The length of a string:

    ${#string}
