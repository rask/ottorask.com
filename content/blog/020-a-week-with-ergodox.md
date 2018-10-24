+++
title = "A week with ErgoDox"
date = 2017-10-15T15:37:00Z
slug = "a-week-with-ergodox"

[extra]
hero_image = "/blog/md-ergodox-1280x720.jpg"
hero_image_alt = "Photo of ErgoDox Infinity ergonomic mechanical keyboard, with blank black DSA keycaps and acrylic case"
+++

_ErgoDox_ is a mechanical split keyboard built with ergonomics in mind. In the
past, I’ve owned good old staggered QWERTY keyboards so this was my first foray
into split and ortholinear keyboards.

A few weeks back I received my Infinity ErgoDox kit and I built it around a week
ago. The Infinity version by Input Club builds on the original ErgoDox designed
by keyboard enthusiasts Dominic Beauchamp, Fredrik Atmer, and litster. The
original ErgoDox software and hardware are GPL licensed.

_Article image borrowed from Input Club website._

## Split, ortholinear?

As you know, most keyboards on the market these days are mass-produced QWERTY
boards with the same layout that has been around for decades (with minor changes
every now and then).

ErgoDox is a split keyboard meaning it is physically built from two pieces, one
per hand. It is also ortholinear, which means the keys are laid out in aligned
columns instead of aligned rows. The ED has some row alignment adjustments to
make the board more ergonomic.

This means you’re forced to use both hands somewhat equally. You will understand
the fuss about touch-typing and your hands will be in a more natural position
when typing.

It also has neat backlit LCD displays on each half.

## The firmware

ErgoDox supports 100% customizable layouts on the firmware level, meaning you
can flash a layout (and other goodies) once and they work on every computer you
plug your keyboard into.

{{ link(label="I created my firmware with QMK", url="https://github.com/qmk/qmk_firmware/tree/master/keyboards/ergodox_infinity/keymaps/rask", external=true) }},
which is a fork of TMK. TMK and QMK have nice features to make a keyboard truly
your own. The QMK repository has clear instructions and a default example layout
available so getting firmware up and running was easy.

## The build

{{ caption(url="/blog/raskergodoxbuild-1024x1024.jpg", h=1024, w=1024, alt="Unassembled ErgoDox Infinity parts lying on table", caption="The plates and PCBs for each half of the Infinity ErgoDox before assembly.") }}

The Infinity version of ErgoDox is available only as a DIY kit, meaning you need
to know how to solder and assemble a keyboard yourself. I’ve soldered a few
keyboards this past year so that was no issue. I had more trouble with the
acrylic case and the full-hand addons.

I chose MX Clear switches for my build. These are the heaviest tactile switches
in the Cherry MX family. I decided to leave all kinds of LEDs off the build for
now, they’re quite painless to add afterward if needed.

## Some mishaps

The kit I received had some issues: the stabilizer holes in the aluminum plate
were misaligned, which meant I could not install stabilizers for my 2u keys.
This is not a huge issue as I use the 2u keys with my thumbs and my thumbs lay
pretty much in the middle of the key anyways.

The other full-hand addon had some alignment issues as well. The addon would not
tighten properly so I had to run and buy some M4 washers to keep the acrylics in
place properly.

## Initial impressions

First of all, the keyboard is way bigger than any photo lets you understand. The
full-hand addon does play a factor in this though.

The acrylic case is not my cup of tea, but gladly the hardware is open source so
I can create my own case if required. There are premade after-market cases
available too.

My kit came with blank black DSA profile keycaps. This is fine otherwise but
there were no homing keycaps in the set so touch-typing is quite painful on a
new physical layout. DSA is a low uniform profile.

## Relearning keyboarding

With a regular ANSI/ISO QWERTY keyboard, I can pound in English at a solid 50-60
words per minute.

My first test drive with ErgoDox landed me at a whopping 5 words per minute. I
never learned touch typing so the ED layout was giving me a bad time (still is).
Blank uniform profile keycaps are not helping here.

I set out to learn proper touch-typing. Right now I’m around 15-20 words per
minute with loads of typos if I get my fingers properly warmed up. Getting
there!

## Programming with ErgoDox

Programming is not easier, but adjusting to the new layout and symbol positions
is relatively easy. I made my custom layout with programming in mind so I
borrowed most positions from a US ANSI layout I use on my “regular” keyboards.

## Gaming?

Hell yeah! The WASD layout feels more natural in my opinion and you can even
disconnect the right half and put it away while gaming. This allows you to put
your mouse and keys closer if you want to. I put MX Clears in my ED which is
probably not the optimal solution for some games.

## Personal pros

-   I’m finally learning to touch-type
-   Really liking DSA so far (apart from not having homing keys)
-   Too soon to say anything definite, but hopefully, this helps with carpal
    tunnel syndrome which I’m suffering from time to time
-   Great for gaming, though my choice of heavy MX Clear switches makes
    fast-paced games a tad more difficult
-   Cool factor.

## Personal cons

-   The Q and P columns could be a tad lower to make reaching Q and P with
    pinkies less of a chore
-   The thumb cluster is a bit strangely positioned, reaching for the 1u keys
    feels more like reaching for a separate keyboard
-   The acrylic case is somewhat displeasing (subjective opinion)
-   You need to shell out heavy bucks for custom keycap kits that support the
    ErgoDox layout (I paid around $80 for a kit in GMK Yuri)
-   No proper guides for LCD display customization with QMK from what I’ve seen.

## Things I’m going to do

I already ordered some homing keycaps to make typing less irritating. I will
also adjust the keymap a bit. Currently, my Ctrl, Gui, and Alt modifiers are a
bit misplaced for my likings (I already adjusted the QMK keymap regarding this).

I will also continue working on getting my WPM back to around 50. I left my UK78
at the office so I would keep my pace at work at least.

All in all, I’d say the ErgoDox is well worth the investment if you sit on a
computer for the bigger part of your days. I’m also really tempted to order a
new separate left half with switches that are more optimal for FPS gaming.