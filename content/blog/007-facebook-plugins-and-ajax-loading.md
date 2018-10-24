+++
title = "Facebook plugins and AJAX loading"
date = 2013-02-23T17:53:00Z
slug = "facebook-plugins-and-ajax-loading"
+++

I ran into a small issue earlier today while developing a client website. I was
trying to load an external page fragment using `jQuery.load()`, which worked
just fine until the code broke the Facebook Like button plugin.

The problem is spawned when using the Facebook SDK and plugins on a page. As the
page loads the Facebook SDK is used to parse the created document in order to
insert all sorts of plugin functionalities (such as the Like button) where
needed. After the document has been initially parsed, no “realtime” parsing will
take place. This means any content inserted afterwards will need to be parsed
yet again. You can use the `FB.XFBML.parse()` function to do this:

    $( "#id" ).load( "/test.html #fragment", function()
    {
        FB.XFBML.parse();
    });

Now the jQuery’s `load()` function’s callback executes the `parse()` function
and the newly inserted content will be parsed through for Facebook
functionalities also.

I used jQuery here, but the idea is the same everywhere: if you dynamically load
something after the initial page load and it contains Facebook code, remember to
reparse the document using `FB.XFBML.parse()`.