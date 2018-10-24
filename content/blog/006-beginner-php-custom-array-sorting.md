+++
title = "Beginner PHP: Custom array sorting functions explained"
date = 2012-10-01T16:22:00Z
slug = "beginner-php-custom-array-sorting"
+++

## Custom sorting?

Many of you most probably are familiar with PHP’s built-in array sorting
functions, such as `sort`, `ksort` and similar. The built-in sorting functions
are quite straight-forward and easy to use. But what about situations where you
need to use a custom sorting scheme to order your arrays properly? Say hello to
`usort`, `uasort` and `uksort` (`u` standing for “user-defined”), a set of
functions used to map a custom sorting function to handle the more complex
ordering problems.

`usort` sorts the array values and destroys the array’s keys in the process.
`uasort` sorts the array values while preserving the array keys. `uksort` sorts
the array keys instead.

Custom sorting functions essentially take in two values (parameters) at a time
(while looping through the parameter array of `usort`), and inside the sorting
function the “bigger” value is determined by returning one of three possible
values: `1`, `0` or `-1`. These three return values are applied in sorting to
determine the larger value between `$a` and `$b`. `1` means `$a` is
larger/bigger/etc., `-1` means the `$b` value is larger. If the function returns
`0` the values are determined as being equally large (e.g.
`if ( 2 == 2 ) return 0;`).

## An example of “custom” sorting

I’m going to demonstrate this system by rebuilding the regular `sort` function
using a custom sorting function. Yeah yeah I hear you, not very useful, but at
least we’ll nail the primary concept without a hassle.

Let’s create an array filled with random numerical values for us to sort.

    $numbers = array(
        34,
        3.56
        34.0,
        256,
        0.45,
        1,
        7,
        90.90
    );

Normally we would just call `sort( $numbers );` to do the dirty work for us
(which you always should do by the way), but instead we will call `usort` and
determine a callback function to do customized sorting.

    usort( $numbers, "custom_sorting_function" );

`usort` takes in the array to sort and the name of the function we are going to
use to do the sorting. We don’t need to assing the result of `usort` to a
variable, as the function only returns true on success and false on failure.
These can be useful in real-life situations though. If the custom sorting
function is a class method (usually called as `$class -> method();`), you can
use it in `usort` by passing the function parameter as an array containing the
class and the method name: `usort( $numbers, array( $class, "method" ) );`.

## The custom sorting function

The custom sorting function is the main machine doing the sorting work. Care
must be taken that it returns one of three values: `1`, `0` or `-1`. If not the
value could be cast into another value returning something we don’t/can’t use
and could possibly break the whole program. Let’s start with the base:

    function custom_sorting_function( $a, $b )
    {
        // Determine which is larger, $a or $b.
    }

Inside we will use simple `if-else` conditionals to check which value is larger.

    function custom_sorting_function( $a, $b )
    {
        // $a is larger.
        if ( $a > $b ) {}

        // $b is larger.
        elseif ( $a < b ) {}

        // They are equal.
        elseif ( $a == $b ) {}

        // Values are invalid.
        else {}
    }

Notice the last `else` statement. If the values are somehow incomparable, we can
return an error notice or similar to safely break the execution of the program.
In our case we are just comparing numbers, so we can possibly leave it out or
return the same value as `$a == $b` does.

Now that we have taken all comparisons into account, we can start returning
valid values for each comparison. Remember, `1` means `$a` is bigger, `-1` means
`$b` is bigger, and `0` means they are equal. Let's insert our return values:

    function custom_sorting_function( $a, $b )
    {
        // $a is larger.
        if ( $a > $b ) { return 1; }

        // $b is larger.
        elseif ( $a < b ) { return -1; }

        // They are equal.
        elseif ( $a == $b ) { return 0; }

        // Values are invalid. In this example we can safely assume
        // all values are proper numbers and return a 0 here also.
        else { return 0; }
    }

What happens in the background? The `usort` function in itself is a complex
looping system, that takes each and every relation within the input array and
uses the custom comparison function on each one. Value 1 will be tested against
every other value and sorted accordingly, then `usort` resorts the array and
selects the next value and uses the same function to test against every other
value. This loop continues until all values have been tested and the array
properly sorted. If something goes wrong, `usort` returns `false` and execution
halts.

In the end you should have an array with sorted values as such:

    Array
    [
        0 => 0.45
        1 => 1
        2 => 3.56
        3 => 7
        4 => 34
        5 => 34.0
        6 => 90.90
        7 => 256
    ]

## A real life example

I had to create a quite complex sorting function at work a while back. I won’t
paste it here, but tell you how it essentially worked so you can get your hands
on the idea.

A client had a database filled with employees and their data. On their website
they wanted to list them all, sorted by their job titles. As job titles can
rarely be sorted alphabetically, I had to build a custom function to sort
employees from CEO to trainees.

To make my work easier, I mapped all different titles into an array with
numerical weighs designated for each one. Some could have the same weight and
some wouldn’t have any weight at all. Using a custom sorting function I would
get the designated numerical values of each title and compare the weight number
instead of the title string itself.

In case a job title appeared that wasn’t weighed in the array mapping, I would
default to non-standard values automatically weighing less than ones already
mapped in the array.

The beauty of this system was the array. It was as simple as a single line
addition and I could insert new job titles to the weighing system. If taken all
the way, this weighing system could have been used in conjunction with a
database, allowing the client herself to modify the weighs for each title in the
website’s administration panel.

## Feedback? Questions?

Hopefully this was of some use to you, I was quite confused when I first tripped
onto a custom sorting function, but with some practice and common sense you can
make these functions do a load of dirty work for you.

If this article seems to leave questions unanswered in the workings of custom
sorting, please drop a line and I’ll look into filling the holes.