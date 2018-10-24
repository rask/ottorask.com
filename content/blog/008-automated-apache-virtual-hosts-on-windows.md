+++
title = "Automated Apache virtual hosts on Windows"
date = 2013-07-10T13:16:00Z
slug = "automated-apache-virtual-hosts-on-windows"
+++

Generally all web development should take place on a local development
environment. Often usually this includes an AMP-stack. One great feature of
Apache is the ability to set up virtual hosts using configuration files. After
setting the virtual host configuration, changing the system’s `hosts` file to
point a certain domain name to `127.0.0.1` will allow
`http://localhost/projectname` (which could be pointing to
`/path/to/www/projectname/` on the system) to become `projectname.local` for
example.

This might get a bit frustrating though, especially if you constantly need to
create new local development projects and each should be able to be accessed
via `*.local` or similar development development domain names.

On Windows there is a way to achieve automation for this process. All you need
is an Apache installation with
{{ link(label="mod_vhost_alias", url="http://httpd.apache.org/docs/current/mod/mod_vhost_alias.html", external=true) }}
module installed,
{{ link(label="Acrylic DNS Proxy", url="http://mayakron.altervista.org/wikibase/show.php?id=AcrylicHome", external=true) }}
and a few IP addresses from your internet service provider. There are guides for
achieving the same results on Mac and Linux systems, though Acrylic should be
replaced with an alternative such as
{{ link(label="dnsmasq", url="http://en.wikipedia.org/wiki/Dnsmasq", external=true) }}

Please note: I use Windows 7 and XAMPP as my development environment. XAMPP is a
prepackaged installation that handles each Apache, MySQL and PHP at the same
time. You can install them separately in any way you want. MySQL and PHP are not
even needed for this to work.

## Setting up Apache virtual hosts

As mentioned, you need Apache and the `mod_vhost_alias` installed on your
development system.

Often just a simple change to `httpd.conf` enables `mod_vhost_alias` for Apache.
Just browse to the extensions list inside the configuration file and uncomment
the correct line (`LoadModule vhost_alias_module modules/mod_vhost_alias.so` or
similar), unless already uncommented.

If the module needs to be installed by hand, you will need to download
`mod_vhost_alias.so` and place it inside the apache/modules directory. After
that create a new line (or uncomment if already present) to the `httpd.conf`
file extension listing which reads
`LoadModule vhost_alias_module modules/mod_vhost_alias.so`.

After this we need to decide where to keep our virtual hosts (in our case the
directories that will be converted to virtual hosts). In this example we’ll just
go with `C:\localhost\[vhosts here]`.

We need to make an addition inside Apache’s `extra-vhosts.conf` (or where ever
your installation holds virtual host declarations). There you should insert the
following configuration block:

    # Feel free to remove the empty lines to
    # save space, I inserted them here to
    # make the example look cleaner.
    <Virtualhost *:80>

        VirtualDocumentRoot "C:/localhost/%-2+"

        ServerName %-2+.local
        ServerAlias %-2+.local

        UseCanonicalName Off

        <Directory "C:/localhost/%-2+">

            Options Indexes FollowSymLinks MultiViews
            AllowOverride All
            Order allow,deny
            Allow from all

        </Directory>

    </Virtualhost>

I am not entirely certain of the low-level details on these configuration rules
(I’m a web developer, not a server engineer), but I’ll try to explain. Correct
me in the comments in case I say something outright wrong.

First we declare a new `VirtualHost` block, in which we type in our new rules.
After opening the block, we use `VirtualDocumentRoot` to determine the
directories which we want to be served using virtual hosts. In this case we want
to serve all directories under `C:\localhost` as virtual hosts. The `%-2+`
determines which directory is mapped to each domain part. In this situation,
requesting `example.local` would look for `C:\localhost\example`.

After setting the `VirtualDocumentRoot`, we set our server name and server alias
rules. Set `%-2+.local` for both. You can replace `.local` with `.dev` or
similar if you want to (but keep in mind this is what should be inserted to
Acrylic’s hosts file later on).

Remember to turn off `UseCanonicalName`, as we are not setting named virtual
hosts. Named virtual hosts use `UseCanonicalName` to set certain domain names
properly. I’m not exactly sure how it does this, but it relates to request
`Host` headers it seems. Set the value as `Off`.

Lastly we set `Directory` rules for all the virtual host directories. Remember
to save the file before closing it.

Now we need to restart Apache to take the new configuration to use. Refer to the
Apache manual or the AMP-stack provider’s documentation on how to do it. XAMPP
has a little control panel which makes this simple.

That’s it, now we need to tell Windows to map the required types of domain names
to reroute back to localhost for use with Apache.

## Setup Acrylic DNS Proxy

{{ link(label="Acrylic DNS Proxy", url="http://mayakron.altervista.org/wikibase/show.php?id=AcrylicHome", external=true) }}
takes care of mapping our `*.local` requests to our Apache installation (which
in turn checks the requests and maps each to a dynamic virtual host if
possible).

Download and install Acrylic as guided, I installed it under `C:\Applications`, as
I usually do for my software. After installation the Acrylic service should
start by itself.

Next we modify the custom `hosts` file that Acrylic offers. You can find it in
the start menu under Acrylic DNS Proxy → Config → Edit Custom Hosts File. Insert
the following line to the end of the file:

    127.0.0.1 *.local

You can add more on new lines, in case you want to use `.dev` or others. Be sure
not to use any common TLDs such as `.com` and `.net`, as Acrylic may block all
connectivity to those domains if you do so.

Save and close the custom `hosts` file. After the edit(s) we need to restart the
Acrylic service. Inside start menu go to the Acrylic Config section and restart
the service by using Restart Acrylic Service.

## Using the Acrylic DNS Proxy server on Windows

When the Acrylic settings have been setup, our system needs to be told to look
for domain name server data from the system itself. This happens by adding our
own IP (`127.0.0.1`) to our DNS address list for our network interface adapter.

I set the Acrylic server IP (`127.0.0.1`) as the primary DNS address, with the
two next DNS addresses as the DNS IP addresses provided by my internet service
provider.

I’m not going to go through setting custom DNS addresses for Windows, as there a
quite many guides for that on the internet already. Here’s two to get you
started:

-   {{ link(label="How can I change my computers DNS address? at Computer Hope", url="http://www.computerhope.com/issues/ch001161.htm", external=true) }}
-   {{ link(label="How To Manually Set DNS Server(s) On A windows 7 at MixedUpEric.com", url="http://mixeduperic.com/windows/how-to-manually-set-dns-servers-on-a-windows-7.html", external=true) }}

Now the local Acrylic server should be in use when resolving domain names. If
you now try (and have content inside the `VirtualDocumentRoot`), your own
dynamic domains and automated virtual hosts should be working. And on top of
that, the regular `http://localhost/site/` configuration should be working
alongside dynamic virtual hosts just fine.

Please note: some developers have noticed that after setting this automation in
place, their PHP global variable `DOCUMENT_ROOT` is not working properly, and
instead points to the original Apache document root configuration variable
instead. There are some guides and discussions on how to fix this. One option is
to develop websites and applications without relying on the `DOCUMENT_ROOT`
global.

Remember, I’m a web developer and not a server manager. This means I may have
some mistakes here and there, but at least for me this setup seems to be working
just fine. Please do correct me in the comments if you feel like it. I’ll gladly
fix errors and maybe-maybe-nots in this tutorial.