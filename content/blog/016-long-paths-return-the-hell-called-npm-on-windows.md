+++
title = "Long paths return: the hell called npm on Windows"
date = 2015-04-21T15:36:00Z
slug = "long-paths-return-the-hell-called-npm-on-windows"
+++

Previously I wrote about too long paths regarding Compass and Sass when using
`.sass-cache`.

If your project is nested deep into a deep subdirectory and you’re running
Windows, npm will most probably break depending on what modules you use. For me
`gulp-imagemin` created a super-deep directory tree of nested `node_modules`
folders, whose filesystem paths were too long in string format for Windows to
handle (way over ~250 characters).

Windows has old Win32 APIs that do not allow file paths to exceed the ~250
character limit. Because of this, running npm might/will cause problems if the
module dependency tree is too deep. E.g. `ENOENT` and “`file not found`” errors.
You can’t event delete the files normally using the basic Windows methods.

I successfully “fixed” the issue by using a few `peerDependencies` directives,
manual dependencies values for certain npm packages, then used `npm dedupe` to
flatten the hierarchies a bit more, and lastly I `npm shrinkwrap`‘d the modules
into a static state. And I would have to do all this with multiple test runs if
I ever want to update the packages to new versions.

I’d love to use Gulp more for instance, but having to go through all what I’ve
mentioned on Windows will be annoying for my team’s Windows developers.

I run on Linux so this isn’t an issue, but two coworkers run Windows so we can’t
have the system breaking on their setups.

Some say this is an issue with NodeJS, some say npm and others blame Windows. I
think Windows mostly, but the developers who keep using the old Win32 APIs are
to blame even more.

Npm devs say they are working on a Windows-friendlier way of flatter package
dependency handling for npm v3+, which is really good news.