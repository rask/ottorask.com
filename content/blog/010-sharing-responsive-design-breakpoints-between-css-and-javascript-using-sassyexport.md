+++
title = "Sharing responsive design breakpoints between CSS and JavaScript using SassyExport"
date = 2014-07-04T17:46:00Z
slug = "sharing-responsive-design-breakpoints-between-css-and-javascript-using-sassyexport"
+++

Wouldn’t it be useful if you could define breakpoints in one place and your
whole frontend code would update accordingly? That’s what I wanted a while back
on a project. I came up with a way to automate this procedure using
_SassyExport_, a Compass plugin used to export Sass maps to JSON files.

## The problem

Responsive web design is a major part of website development these days. The
most probable way of achieving responsive designs are media queries in CSS. But
what if you want to use the same media queries in Javascript? You’ll most
probably go and manually code in a set of breakpoints that match the ones in
your CSS code.

What if this procedure was automated, allowing you to define media query
breakpoints once in your CSS and have it working in your Javascript instantly
after?

## SassyExport

{{ link(label="SassyExport", url="https://github.com/ezekg/SassyExport", external=true) }}
is a Compass plugin that takes in a Sass map and then outputs it as a `.json`
file into a directory of your choosing. Nothing much to that is there?

Currently I got SassyExport working only on an alpha release of Compass
(`1.0.0.alpha.20` to be precise at the moment of writing). You can install a
pre-release version of Compass and SassyExport using the following terminal
commands:

    $ gem install compass --pre-release
    ...
    $ gem install SassyExport

Implementing SassyExport in your Compass project is as simple as requiring it in
your `config.rb` file and importing it in your `.scss` files where needed:

    # config.rb
    require 'SassyExport'
    // style.scss
    @import 'SassyExport';

Using SassyExport is simple, define a Sass map then `@include` SassyExport with
a few parameters. The example below defines a few breakpoints as a Sass map and
exports them to `breakpoints.json` right next to `config.rb`:

    # style.scss

    // Define a Sass map containing breakpoints.
    $breakpoints: map(
        phone: 480,
        tablet: 720,
        desktop: 960
    );

    // Use SassyExport to output a .json file. I'll go over the parameters in a bit.
    @include SassyExport( '/breakpoints.json', $breakpoints, true, true );

The SassyExport parameters go as follows:

-   `'/breakpoints.json'`: The first parameter defines the output file. It is
    relative to your project’s `config.rb` file. In this example the output file
    will be in the same directory as the `config.rb` file.
-   `$breakpoints:` The second parameter simply defines what Sass map variable
    to use. In the example we defined $breakpoints and we use it as the
    parameter.
-   `true`: The third parameter defines whether to output pretty JSON or
    minified JSON. Using true allows us to see a more readable JSON file, while
    using false outputs a minified and slightly less readable JSON file.
    \*insert tada.mp3\*
-   `true`: The last parameter determines whether SassyExport should do
    debugging or not. Currently the debug mode just outputs the output file and
    write operation result in your terminal. If you wish to see what’s going on
    use `true`, otherwise you can use `false`\*.

\* Currently I can’t seem to use SassyExport without leaving debugging turned on
(setting the last parameter as `true`). SassyExport’s author is investigating at
the time of writing. Experiment and see whether `false` works for you or not.

## The JSON

Hooray, we successfully defined breakpoints in SCSS and then exported a JSON
file out of them! Here’s the JSON file that would be exported from the example
above:

    {
        "phone": 480,
        "tablet": 720,
        "desktop": 960
    }

Magnificient! Nothing else to see here. Let’s see how we can get and use these
values in Javascript.

## Loading the breakpoints in Javascript

We’ve got some awesome Sass mapping going on and some great JSON exporting with
SassyExport going on. Lastly we need to get the JSON data to Javascript in order
to use the values.

In the following examples I’ll be using jQuery’s AJAX in order to keep the code
to minimum and handle cross-browser inconsistensies.

In order to load the JSON data for use in Javascript we need to load it using
AJAX:

    // Set a breakpoints property for our window, you can set it elsewhere too.
    window.breakpoints = {};

    // GET the breakpoint data.
    $.get( 'http://example.com/breakpoints.json' )
        .done( function( res ) {
            // Parse the JSON.
            var scss_breakpoints = JSON.parse( res );

            // Set our breakpoints data.
            window.breakpoints = scss_breakpoints;
        });

Note: you should handle failure conditions too, I’m leaving them out to keep the
example clean.

Yay! Now we’ve got the breakpoint JSON file loaded to Javascript. Reference
breakpoints by using `window.breakpoints.phone` for instance.

## Extra: making it Git-friendly

I use Git for version control in my projects. As the JSON we’ve been generating
using the examples above is a generated resource, we need to make sure there are
no conflicts (or at least too frustrating conflicts) during merges.

First, we need to define a so called “ditch all other changes” merge driver also
known as “ours” in our Git configuration. In your global `.gitconfig` file, add
the following:

    [merge "ours"]
        name = "Ours merge"
        driver = true

The `driver = true` line determines that only the other merge branch changes
should be included in the merge commit that is generated by Git.

Now that we’ve got the new merge driver running, create a `.gitattributes` file
if you don’t have one already and insert the following configuration:

    ./breakpoints.json -diff
    ./breakpoints.json merge=ours

This way the generated JSON file is treated as a binary file (such as images
are) and the new `ours` merge driver disregards changes from other branches and
leaves no conflicts behind.

Next up we can set up some Git-hooks to automate Sass/JSON generation on merges
and rebases. This way the Sass and JSON generation is automated and included
within the merge and rebase commits Git creates (so we don’t have to manually
generate the changed sources for each and commit). Create the following Git
hooks (which should reside in `.git/hooks`):

    #!/bin/sh
    # filename: post-rewrite

    if [[ $1 = "rebase" ]]; then
        compass compile /path/to/project/root/dir # Set absolute path.
        git add -u
        git commit --amend --no-edit
    fi

    #!/bin/sh
    # filename: prepare-commit-msg

    if [[ $2 == "merge" ]]; then
        compass compile /path/to/project/root/dir
        git add ./breakpoints.json # Relative to Git-project root, can be a directory.
    fi

The hook codes above are in shell script. The `post-rewrite` hook is run when a
commit rewrite happens, usually either an amend commit or a rebase commit. We
run a Compass compilation on each rebase type commit rewrite and add the changed
files right before re-creating the commit. In the `prepare-commit-msg` hook we
determine whether the commit is a merge commit or not and compile and include
the generated contents right before creating a merge commit (thus removing the
need to recompile after a merge).

## All done, get coding!

Now you should have a system where you define responsive breakpoints in SCSS and
don’t care about breakpoints afterwards. Javascript loads the breakpoints from a
generated JSON file that contains the breakpoints. If you use Git, we just
created a system that keeps the breakpoint data up to date without creating
conflicts between developers.