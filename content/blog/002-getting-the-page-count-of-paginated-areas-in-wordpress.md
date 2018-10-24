+++
title = "Getting the page count of paginated queries in WordPress"
date = 2010-07-22T06:07:00Z
slug = "getting-the-page-count-of-paginated-areas-in-wordpress"

[extra]
format = "post"
+++

I was playing around with a WordPress theme and I wanted to display a simple
number indicating the amount of pages a certain paginated part of the site had
(front page’s recent entries in this case). I looked and looked but didn’t find
any straight-forward variable or method which could output the value. Seemingly
the only choice was to make the thing myself.

I have a home page, which displays recent blog entries in a paginated manner. I
can get the current page number with the PHP variable `$paged`. For spice, I
decided not to display the page counts when on the home page itself, with a
conditional: `if ( is_paged() ) {}`.

But how can we get the full amount of pages available? Quite simply just math:
you have a certain amount of posts with a parameter of displaying a certain
amount of them per page. To get the amount of posts, we use the handy
`wp_count_posts()` function provided by WordPress:
`$count_posts = wp_count_posts();`. Next we need to determine that only
published (publicly visible) posts are considered:
`$published_posts = $count_posts -> publish;`. As you might guess, the
`$posts_per_page` variable is used too.

Now we can count the amount of pages using these variables: `$published_posts`
and `$posts_per_page`. Simple math of dividing the post count with the per page
parameter: `$page_amount = $published_posts / $posts_per_page;`. This will give
odd fractions, so it needs to be rounded up for a “full” page:
`$page_amount = ceil( $published_posts / $posts_per_page );`.

To sum it up, here is the whole code:

    <?php
        $count_posts = wp_count_posts();
        $published_posts = $count_posts -> publish;
        $page_amount = ceil( $published_posts / $posts_per_page );

        // Then you may use the variable as such:

        echo 'Recent weblog entries';
        // If we are browsing the recent entries query, and are on page 2 or further:
        if ( is_paged() ) :
            echo ', page ' . $paged . ' of ' . $page_amount;
        endif;
    ?>

There you have it, now the `$page_amount` variable can be used to display the
number of pages in the current paginated query. :)

If you are not displaying all of your posts in your “recent entries” query—or
wherever you might use this code snippet—you need to make sure you properly
count the amount of posts in that query, or the code will bug itself.

**Note**: There may be a simple variable for this, but I did not find it. Sorry
if there is a better way to do this and I’ve taught you pure bullcrap. Thanks.