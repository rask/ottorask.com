+++
title = "Returning multiple values from PHP functions"
date =  2011-02-19T20:15:00Z
slug = "returning-multiple-values-from-php-functions"
+++

We are learning some basic PHP programming at school this semester. An exercise
required to use functions and display data with possible error reporting on that
data. I had a validation function to check my form, but I needed to return more
than `true` or `false` to make the error reporting work properly. I needed to
get the amount of errors and the description of each error.

PHP as is doesn’t support returning multiple values in functions, but you can
“fake” it with arrays and a function called `list`.

Lets make a simple function which should return multiple values using an array:

    <?php

    function multiple_returns()
    {
        return array(1, 2, "three");
    }

The greatest effect from this is that we can return anything in the array, even
other arrays. So we can make a highly complicated function to return any kind of
stuff one should need. But how can we map them to proper variables so we can use
them without any hassle? We use the `list` function to map the array values to
single variables the simple way:

    <?php

    list($one, $two, $three) = multiple_returns();

`list` takes in an array and puts the array values to each variable in the list.
`list`s require arrays with numerical keys (0, 1, 2, and so on) to work
properly. If you need to skip an array item just insert an empty parameter to
the function as like `list( $one, , $three )`.

Now we have some variables so we can just echo them to see how it works.

    <?php

    echo $one . ', ' . $two . ', ' . $three . '.';

This should result in `1, 2, three` or similar being written on the page.

This method might not be needed that often, but it is quite useful. Of course
you don’t have to use the `list` function to play with the function return
values. You could do a bit more complicated `foreach` loop to work with the
values too.