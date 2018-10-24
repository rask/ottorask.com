+++
title = "Sass compiling and too long directory paths"
date =  2014-10-15T16:02:00Z
slug = "sass-compiling-and-too-long-directory-paths"
+++

Recently I ran into a problem with a Compass project: I couldn’t compile any
SCSS assets into CSS assets and Ruby threw a file not found error.

    Errno::ENOENT on line ["123"] of c: No such file or directory

Digging deeper, I found a Ruby `tempfile.rb` method `rmdir` could not find a
file or directory to remove. File access rights and such were all as they should
be. I was running latest stable versions of all gems and executables (Compass,
Sass and Ruby). My SCSS files we’re not faulty (compilation succeeded on a
co-workers environment just fine).

Seemingly Sass, Compass or Ruby at some point had tried to create a temporary
file but failed. Then `tempfile.rb`‘s rmdir tries to remove the inexisting file
from the system, resulting in a `ENOENT` error.

A few google searches suggested reinstalling everything and starting a fresh
Compass project (not an option). Then I found a reference to a bug that
disallows too long path names (more than 255 characters) in Sass procedures.
More specifically the caching system, stuff that is inserted in `.sass-cache`.

I set my `config.rb` to send cache files to `C:\temp\sass\` (using the
`cache_path` configuration variable) instead of the regular project root
directory, and suddenly Sass compilation started working again.

After the cache path was shorther than 255 characters (after character encoding)
compilations worked as they should. If your project resides in a deep directory
structure, you might run into the same problem.

The Compass project I’ve been working on resides under 7 to 9 directories. In
the project directory I have the `.sass-cache` directory, which contains cached
assets. Under the cache directory Sass generated a whole new directory structure
based on where `@import`ed Compass SCSS files resided (which would be
`C:\Ruby193\...\*.scss`. This would result in pretty long pathnames for certain
files:

    C:/localhost/project/wordpress/wp-content/themes/themename/.sass-cache/75fcaf1b4852ceb732871195e41567cc2a7d8997/c%058%092Ruby193%092lib%092ruby%092gems%0921.9.1%092gems%092compass-core-1.0.1%092stylesheets%092compass%092utilities%092general%092_hacks.scssc20141015-8380-b2vix7.lock

I copied `compass compile`‘s `--trace` dump encoded path above. That’s some 281
characters, which is over the 255 character threshold.

I’m uncertain whether this problem is restricted to Windows environments.

## Workaround in config.rb

I solved this problem by adding a temporary configuration trick into
`config.rb`. It checks whether a certain file exists in the project root:

    if File.file?( './.nosasscache' )
        #cache = false # Uncomment to disable cache.
        #cache_path = '/path/to/new/cache/location/' # Uncomment to change location of Sass cache.

        # You can add additional configuration parameters here if you want to.
    end

If a file named `.nosasscache` exists in the project root, cache will be either
turned off or the location of it will be changed.

I tried using Ruby’s require to load an external configuration file, but
seemingly Compass configurations do not work that way. Chime in at
{{ link(label="this `config.rb` related Stack Overflow question", url="http://stackoverflow.com/questions/19034434/ruby-compass-config-file-overrides", external=true) }}
if you have any ideas on how to include external configuration files inside
Compass’ `config.rb`.

## Why not just hard-code the path into config.rb?

Yes, that would work wonders too. But if the `config.rb` file is committed into
version control and is distributed between development team members, there’s no
knowing whether a `cache_path` directive points to a non-existant location or
would overwrite something else in someone’s filesystem. Paths can also be
different across operating systems (is there a “`C`” drive on Linux systems?).

By using this file existance check we can conditionally set a semi-local
configuration directive. You could use varying filenames to alter what kind of
configurations to use.