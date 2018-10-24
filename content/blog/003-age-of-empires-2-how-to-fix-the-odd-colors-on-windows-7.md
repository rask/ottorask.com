+++
title = "Age of Empires 2: how to fix the odd colors on Windows 7"
date = 2011-01-20T20:10:00Z
slug = "age-of-empires-2-how-to-fix-the-odd-colors-on-windows-7"
+++

As some of you might have noticed, Age of Empires 2 (and its The Conquerors
expansion) has a color bug on Windows 7 (and most probably on Vista too). Grass
turns red, water becomes purple and every second tree is a christmas tree (or
palm for that matter). Gameplay related problems include player colors
displaying wrong either on the minimap or when outlining units behind obstacles.
Annoying as hell and no normal option is around to fix it.

Fortunately some people have discovered several solutions of which some may work
on some systems and installations while others seem to do nothing. Here I
present three different solutions to fixing the broken colors in Age of Empires
2 on Windows Vista and 7. Remember, if you’re in doubt on what to exactly do,
ask someone who knows more to do this for you.

## 1. The batch file launcher (“explorer.exe killer”)

It seems the Windows Explorer (the file system browser and handler, not Internet
Explorer) has some graphics settings which overrun even full screen applications
(such as games) and the game gets mixed up on what to display. Not certain of
that, but this fix kills the Explorer while you’re playing and worked for me.

This solution is the most time consuming, but most probably works. It involves
creating a new executable batch file which performs some preset actions through
the Windows 7 command line and opens the game for you. This method isn’t harmful
to your system, although you might lose your open folders and directories when
explorer.exe is terminated. Your applications such as IRC-clients, BitTorrent
clients, web browsers and such keep running in the background with no errors.

Browse over to the folder where Age of Empires 2 is installed (and where the
`.exe` file—most probably named either “`EMPIRES2.exe`” or “`age2_x1.exe`“—for
the game is located) and create a new text file (`.txt`). Make sure you’re
showing you file extensions (you should be able to see the “`.txt`” after the
filename)! Rename the text file to a batch file by changing the extension from
`.txt` to `.bat.` Windows will tell you to “be careful” and that “this file
might not work after changing the extension”. Hit OK to dismiss the message.
Name your batch file descriptively (I named mine “`Age2-Colorfixed.bat`“) so you
know what to run when you want to play.

The batch file is nothing more than a text file with a different extension so
you can edit them in Notepad like you normally would. In Windows 7 when
right-clicking on a batch file you should see an “Edit” option. Right-click on
your new batch file and edit it. Notepad should open up with an empty text file.
In this text file you can write all sorts of commands that the command line
recognizes (yes, even a “`format C:`” is possible through it). We will only need
three different commands. Copy and paste the code below and we’ll check out what
it does:

    taskkill /F /IM explorer.exe
    age2_x1.exe
    start explorer.exe

On line one we use `taskkill` to terminate the explorer.exe process.
Additionally we insert the `/F` parameter to end the process without any
confirmations or popup messages (which streamlines our process of getting to
play the game). Line two is for launching the game executable. This batch file
selects The Conquerors expansion’s executable file and launches it (change to
“EMPIRES2.exe” if you’re not playing the expansion). Line three then restarts
the explorer.exe so we get our Windows’ file explorer back after closing the
game.

Save the changes (but remember to keep it as a “`.bat`” file) and close the edit
window. The batch file is ready! Double-clicking on it should now kill
explorer.exe and launch the game (don’t worry if your desktop looks like crap
right before the game opens). After closing the game Windows Explorer should be
normally open again. If not, open task manager, select “New task” from under the
“File” menu and type in “`explorer.exe`“. When hitting OK the process should
start normally.

Now you can make a shortcut of this batch file like any other file. I have mine
on the desktop. Opening the file closes explorer.exe and opens the game
normally. I had some trouble with case-sensitivity in the file at first, but the
version above works properly.

## 2. The screen size window trick

Some people have reported that opening the Windows’ own screen resolution window
before opening the game works to get rid of the weird colors. Here’s how it
works:

1.  Open your desktop view (with all the fancy icons and the desktop mini-apps
    open on it).
2.  Right-click and open the “Screen resolution” window from the dropdown menu.
3.  Leave it open and start Age of Empires 2.
4.  Done!

As I said, this might work to some of you. It didn’t work for me, but could be
the game version or my Windows 7 installation which broke it. Simple and easy if
it works!

## 3. The compatibility mode tickbox

The usual place to go when old games act odd on newer systems is the programs
properties (Right-click on program icon and select “Properties”) window’s
compatibility tab. There’s is a dropdown list of older Windows operating systems
such as Vista, XP and even 95.

Under the dropdown list are several checkboxes: “256 colors”, “640×800 screen
resolution”, “disable visual themes”, “disable desktop composition”, “disable
display scaling on high DPI settings” and “run as administrator”.

Here is what people have instructed to do in order to get the game colors
working:

1.  Tick “Run this program in compatibility mode for:”.
2.  Select “Windows XP (Service pack 2)”.
3.  Tick “Run in 256 colors”.
4.  Tick “Disable visual themes”.
5.  Apply the settings and start your game normally.

This reportedly works for some people. I combined this and the batch file
procedure to make my game work properly. Your desktop might look odd right
before the game opens, but it is part of disabling full color palettes such.

*Update on 06.03.2013:*

## 4. Change your graphics setting to “medium”

*febrix10* (comment gone due to age of this post, sorry) posted a comment in
which s/he suggests to try and change your in-game graphics quality to medium.
I have not verified this, but hopefully it works for some players. Thanks
*febrix10*!

## Any other options?

Have you heard of other options that make Age of Empires 2 work properly on
newer systems? Post some in the comments and I can add them to this entry.

Hopefully this entry was useful. Enjoy your properly colored Age of Empires 2!