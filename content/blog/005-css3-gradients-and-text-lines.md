+++
title = "Using CSS3 gradients to create text leading lines"
date = 2011-05-08T12:55:00Z
slug = "css3-gradients-and-text-lines"
+++

The new CSS3 gradients are quite powerful and simple when in need of simple
repeating background patterns. Although these will not work in older browsers
and partly in newer browsers, they are an option to look for to lessen image
usage for things that may work without the image file. This entry will guide
you through to make leading lines for text (as has been for ages in {{ link(label="notebook paper", url="http://www.photos-public-domain.com/2010/12/31/notebook-paper-texture/", external="true") }}
and similar stuff) using some CSS3. This effect can be useful for lines of code
within `<pre>` tags and similar.

Many implementations use JavaScript to split out the wanted element to
block-level line elements (using either `p`aragraphs, `div`s or `span`s) for
which then bottom (or top) borders are styled to create the text line effect.
This is good for lines that need to wrap and “expand” the text line to denote
what each single line holds in it. But some folks may have JavaScript disabled
for various reasons, leaving the text lines unstyled. CSS3 offers a solution
without JavaScript, but is less expandable. The best solution could be to mix
both CSS3 and JavaScript.

This styling can be added to any HTML element that accepts background images
(practically any block level element needed in layout code and text content).
Here is the HTML we are using in this example:

    <div class="example-element">
        <p>Some text before awesome text line styled content.</p>
        <pre>
    Some awesome text with even more awesome CSS3
    styled background text lines. Let's try some code:

    <?php
        $hello = "Hello world!";
        if ( $hello != "" )
        {
            echo $hello;
        }
    ?>
        </pre>
    </div>

Nothing complex. Just a content holder `div` with some `pre`-content inside it.
Here is the CSS for styling the text lines:

    .example-element pre {
        /** We set the up and bottom padding to zero to avoid background movement. */
        padding: 0 5px;
        line-height: 20px;

        /** This will be used even if the browser doesn't know CSS3. */
        background-color: #EEE;

        /** The CSS3 background-image declarations: */
        background-image: background-image: -webkit-gradient( linear, 0 0, 0 100%,
            color-stop( .95, transparent ),
            color-stop( .99, rgba( 0, 0, 0, 0.5 ) ),
            to( rgba( 0, 0, 0, 0.5 ) ) );
        background-image: -o-linear-gradient( transparent 95%,
            rgba( 0, 0, 0, 0.5 ) 99%,
            rgba( 0, 0, 0, 0.5 ) );
        background-image: -moz-linear-gradient( transparent 95%,
            rgba( 0, 0, 0, 0.5 ) 99%,
            rgba( 0, 0, 0, 0.5 ) );
        background-image: linear-gradient( transparent 95%,
            rgba( 0, 0, 0, 0.5 ) 99%,
            rgba( 0, 0, 0, 0.5 ) );

        /** Background image sizing must be scaled to our line height to look correct: */
        -webkit-background-size: 20px 20px;
        -moz-background-size: 20px 20px;
        background-size: 20px 20px;
    }

Alright, a bit more complex than the HTML code we are using. Let me explain if
something isn’t as clear as it should be from the code, mainly the gradient
declarations.

First we set `padding` to zero for the top and bottom values, so we don’t get
odd spacing for the background lines. You could change the `background-position`
value to fix this too. Then we declare a proper `line-height` for the whole
element. The same value is used in the `background-size` declaration later on.

The true work happens inside the `background-image` rules and the gradients.
Each gradient is a whole 100% of size. We declare 95% of this as `transparent`.
This way any element having backgrounds under the text line element will be
shown too. You can use a solid color or a translucent shade here if you want.
The rest 5% is filled with a solid color or a quite opaque color. I used a 50%
shaded black (`rgba( 0, 0, 0, 0.5 )`). This way 95% of our background is
`transparent` and the rest is a wanted color. Note: if your `line-height`
exceeds about a hundred pixels in height, you should increase the percentage
(95%) of the `transparent` area to avoid slight blurs in the background lines.

Without the `background-size` rule the single gradient would be stretched all
over out `pre` element. By setting the `background-size` to the same value as
our `line-height` is, we get a repeating pattern for all of our content lines.

Here is the result (remember, you need a CSS3 compliant browser to see it
properly):

**NOTE at 2018-10-24: my current CMS does not support running custom CSS inside
post contents, so the example below _will not_ look like anything it is supposed
to look like. Sorry for the hassle.**

    Some awesome text with even more awesome CSS3
    styled background text lines. Let's try some code:

    <?php
        $hello = "Hello world!";
        if ( $hello != "" )
        {
            echo $hello;
        }
    ?>

With some more complex implementations you can create varying colors for varying
lines (every second line could be red for instance) and have vertical lines too
(with multiple background image gradients). With JavaScript you can add some
line numbering and other effects to your elements. I’m not a performance master
(which is faster: this or a small image?), but this method leaves room for
easily changing the `line-height` of the background lines with a few numbers,
instead of loading up a complex image creating software (Photoshop or such) to
create the image.

A big thanks to {{ link(label="Lea Verou for the idea and inspiration", url="http://leaverou.me/2010/12/checkered-stripes-other-background-patterns-with-css3-gradients/", external=true) }} to craft this one.

Update (25.05.2011): Lea Verou reminded me in the comments to ensure
compatibility with browsers that use text zooming (instead of full page zooming)
to change the pixel values to ems or other relative units. Using pixels instead
could break the line leading rhytm in text zooming browsers. Just something to
keep in mind!