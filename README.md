# rask/ottorask.com

My personal website. Contains a blog and some content pages. Built with Zola.

## Building

Requires [Zola](https://getzola.org) (formerly _Gutenberg_). Clone this repository and inside it run

    $ zola build

To build a production version use

    $ zola --config ./config.production.toml build

The static site will be built into `public/` from which it can be copy-pasted into a server document
root.

For development you can serve a hot-reloaded version locally using

    $ zola serve

## Custom tooling

### Shortcodes

#### Captioned images

To insert a captioned image, use

    {{ caption(
        url="/path/to/image.jpg",
        h=123,
        w=123,
        alt="Alt text",
        caption="Caption text which is shown next to image"
    ) }}

#### Special hyperlinks

To create special hyperlinks which can denote whether the link is an external link or not, use

    {{ link(
        label="Link text",
        url="https://example.com",
        external=true
    ) }}

#### Notice boxes

For a simple box with a neat `i` icon in the corner, use

    {% notice(type="info") %}
    My **awesome** notice content here!
    {% end %}

### Macros

#### SVG icons

To use an SVG icon inside a template file, pick an SVG icon from the SVG symbol source
(`static/img/icomoon-ref.nocmpr.svg`) and insert it using

    {% macro::svg(icon="...") %}

#### Formatted long date

To output a date such as

    3rd June, 2017 at 11:22

Use the following macro:

    {% macro::nicedate(daynum="...", date=...) %}

`daynum` should be a number of day of month (`1` to `31`). `date` should be a full page date object.

## License and copyright

-   Site content, implementation, and design: All rights reserved, copyright Otto Rask
-   Tooling and other assets: Copyrighted and licensed to their providers.