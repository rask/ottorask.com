+++
title = "Remember to backgroundify your elements"
date = 2009-06-11T07:25:00Z
slug = "remember-to-backgroundify-your-body-elements"

[extra]
format = "post"
+++

Today I was in trouble. My girlfriend had turned the user setting of the page
background to black on her Opera. This she did to “save energy”, which sounds a
bit trivial to me, as she has a small LCD.

Anyway, I tried to open a page, but I didn’t see anything but blue links and a
few images. I wondered what was going on, but realized that it must be the black
background doing its tricks here. I didn’t know where or how it could be set to
white again, so I took a deep breath and opened the page in Internet Explorer.
It loaded fine and everything was viewable again.

So this is a word of advice to all CSS coders: remember to give a background
color and a base text color to your `body` elements:

    body {
        background: #FFF;
        color: #000;
    }

Now user settings will be overwritten and texts are readable with some basic
CSS and you can override these styles later on in your CSS files.

I also told my girlfriend to change the black background to a dark gray. It’s
still dark but I don’t need to open IE if a `body` accident has happened.