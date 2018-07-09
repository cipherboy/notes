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
