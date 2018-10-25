+++
title = "Goodbye WordPress, hello Gutenberg!"
date = 2018-10-25T23:08:00Z
slug = "goodbye-wordpress-hello-gutenberg"
+++

My website (this place you're currently visiting) is not running on WordPress
anymore, but is instead built with
{{ link(label="Gutenberg", url="https://www.getgutenberg.io/", external=true) }}
({{ link(label="soon to be called Zola, I think", url="https://github.com/Keats/gutenberg/issues/377", external="true")}}),
a static site generator written in Rust.

## But why?

Why ditch WordPress, "the greatest open-source blogging platform on Earth", for
a static site generator that does not even come with an admin GUI for writing
and managing content?

### I rarely manage content

Take a peek over at the archive. You can see content which was created
around 2009. You also see that the amount of posts is under 25. This would come
to around 2-3 posts per year. Some folks write that amount in a day.

### I don't need a fancy GUI

I feel right at home writing prose, code, and other stuff inside a text editor
or a terminal. The WordPress admin GUI is arguably one of the best available,
but I don't need it.

### Markdown for the win

In my honest opinion the source representation of all text content should be
written in Markdown first, LaTeX/comparable second, HTML third, then lastly
using any WYSIWYG or custom frankenformats.

WordPress does not support Markdown. WordPress plugins do support Markdown, but
my pledge for a plugin-free personal site would've been broken at that point.

Gutenberg is powered by Markdown content, and it also offers a neat shortcode
system for it as well. I can emulate the WordPress capabilities really well with
it.

### Less dependencies

Let's face it: running a beastly PHP CMS for a personal website that sees new
content 2-3 times a year, around 500-1000 unique visitors each month, and
contains no interactibility whatsoever, is a huge waste of resources and means

1.  I need to keep things up to date,
2.  Things might break from time to time for no good reason.

With a static site, I can ditch PHP and MariaDB, and I'm also planning on
switching from Nginx to Caddy for a more lightweight experience.

### Performance and stability

Instead of possibly running zillions of lines of (spaghetti and meatballs) PHP
code on each HTTP request, I can just dump a HTML page with assets onto the
requester, assuming the wanted HTML page exists.

Sure, there is a build-step involved and I need to do a deployment after every
small change, but at least I can see that it works and looks exactly as it
should before every deployment, and I can be sure it looks and works
the same way in production.

### Backed up by default

When using a static site generator, I can dump _everything_ into a Git
repository and stop worrying about backing up databases, ZIPing assets, and so
on.

I can reinstate older versions of the site with a few Git commands if needed.
I can create branches for testing new content ideas without publishing them.

## WordPress and PHP are okay

But they are the wrong tools for my personal website.

I **love** PHP. I like WordPress (I also hate WordPress). PHP is the workhorse
language of web services and complex websites. WordPress is a great CMS for
non-technical end-users. They have paid my rent for years.

## How did the migration go?

It was really quite simple. I was lucky enough to create a simple WordPress
theme that used little-ticonsno WordPress-specific logic, no JavaScript, and the
CSS was built from SCSS.icons

Converting the theme templates to Tera (Gutenberg's templating language, similar
to Twig, Jinja2, etc.) was a breeze, mostly copy-pasting and converting some PHP
logic to Tera, while getting rid of WordPress specifics.

Redoing SCSS was essentially _not redoing it_. Gutenberg comes with a built-in
Sass/SCSS compiler, meaning I could just dump the WP theme's SCSS directory
contents into the Gutenberg `sass` directory and forget about it.

Implementing things like article hero images, captioned content images, and
other little things like server-side externally linked hyperlinks was a little
confusing at first, but I managed to get those working with Gutenberg macros and
shortcodes.

The content migration was tedious (I manually converted raw text into Markdown),
but as mentioned I only have under 25 posts to migrate. This took around an
evening.

As I migrated content I might've missed a few spots so there might be some
formatting errors on the old posts. You can let me know if you see any and I can
fix those.

## Next steps

At the time of writing, the new website source code is living only on my local
hard drive. After I proof read this post and decide it is ready, I will be
creating a GitHub repository to host this wesite source in.

After that I will shutdown the WordPress powered version, and replace it with a
fresh build from the sources of this new site. Then at some point I will be
automating builds and deployments with Travis.