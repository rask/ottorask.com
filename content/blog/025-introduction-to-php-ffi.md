+++
title = "Introduction to PHP FFI"
date = 2020-02-01T00:13:12Z
slug = "introduction-to-php-ffi"
+++

{% notice(type="info") %}
[This post has been originally published at my current company's developer blog](https://dev.to/verkkokauppacom/introduction-to-php-ffi-po3), be sure to go check it out there as well and add a like or comments if you feel like it. Thanks!
{% end %}

In this series of posts, we will be getting acquainted with a new PHP feature: FFI!

In addition to getting to know PHP FFI, we will be learning how to create and use FFI libraries written in Rust.

FFI is a new addition to PHP starting from version 7.4. In a nutshell, it allows you to run external binary code via a wrapper written in PHP. The binary code can be anything which supports the C-ABI.

{% notice(type="info") %}
Disclaimer: I am not a C/Rust/ABI/FFI expert, I am writing this series to learn things as a PHP developer as well. If you see errors or other things that should be changed, please do let me know.
{% end %}

## FFI? ABI?

FFI ([_Foreign Function Interface_](https://en.wikipedia.org/wiki/Foreign_function_interface)) is a way for two separate codebases to integrate via an ABI ([_Application Binary Interface_](https://en.wikipedia.org/wiki/Application_binary_interface)). One of the most common ABIs for FFI usage is the C-ABI, which defines the binary calling conventions and data structures for compiled C programs. The C-ABI convention is supported by various other non-C languages a well, such as Rust.

In essence you can load and call code from a C library inside PHP, or any other language that supports FFI via C-ABI.

## Why would I want to use FFI?

FFI is great if you need to run resource intensive code and your "own" programming language is not up to the task (e.g. extensive parsing, number crunching, complex rendering, etc.). Instead of using two entirely different programs to do a single thing in steps, you can "glue" two codebases together inside a single program to do the single thing more efficiently. Or maybe you have a good C-ABI library readily available, and just want to save time by not rewriting it in PHP for instance.

FFI can provide performance boosts, but sometimes it might actually make your programs slower. We will be looking at some simple benchmarks on what works well and what does not at some point. Tricky business, but hopefully we can find the good spots where FFI will be of help. If not, we will see good and bad use-cases in the future when more PHP folks start using FFI.

Python has become one of the most popular languages for machine learning and number crunching partly because of FFI. It is relatively simple to load and use powerful C-ABI libraries in Python (via FFI), which in itself is a relatively easy language to learn.

## What to expect from this blog series

This series will walk us through the following in some way or another:

-   Introduction FFI in PHP (this post),
-   Creating FFI-libraries for PHP in the Rust programming language,
-   How to use C types in PHP,
-   Benchmarking a few FFI use-cases, to see when FFI is the more performant choice and vice versa,
-   Creating sane and testable FFI abstractions in PHP,
-   How to properly integrate FFI with the PHP preloading feature,
-   Creating distributable PHP Composer packages that make use of FFI,
-   Ensuring safety when writing and using FFI libraries in PHP.

I mainly use Linux Ubuntu when programming, so you may have to alter the steps I describe throughout this series to make things work right in other environments. Occasionally I might show a cross-platform way to do things, but do not rely on those working as I have no other systems to test things out on at the moment.

I am writing this series mainly for two reasons:

1.  It gives me a good excuse to learn more about PHP, FFI, Rust, and C,
2.  It gives me a good excuse to teach somebody something useful, as I like to do when blogging.

So hopefully I will learn new stuff and you will learn at least a thing or two as well in the process.

Let's go!

## Example source code

To support the blog series, I have created a code repository that contains example code. The examples are runnable, meaning you can see what happens when you run them and change things.

The example code is available at [github.com/rask/php-ffi-examples](https://github.com/rask/php-ffi-examples). You can download/clone the repository to your machine to run and modify as you wish. If you see errors there, you can create issues or send pull requests.

To run the PHP examples, you need a FFI-compatible PHP interpreter build, which we'll take a look at next.

## Installing and configuring FFI for PHP

PHP 7.4 ships with FFI built in. It is a core _extension_, meaning it can be disabled. PHP can also be compiled without the FFI extension.

At the time of writing, PHP 7.4 is still under unstable version development. This means we do not have any official stable builds available for PHP FFI coding.

If you do not have a FFI-enabled PHP build available, you may have to compile a compatible binary of PHP yourself. NOTE: Some popular package repositories provide unstable PHP 7.4 builds that have FFI enabled.

There are tons of guides online for compiling PHP from sources in various ways, but if you're somewhat familiar with the process or compiling C programs in general, you can follow the steps outlined below:

```sh
$ sudo apt-get install build-essential autoconf automake libtool bison re2c
$ git clone https://github.com/php/php-src
$ cd php-src
$ git checkout php-7.4.0beta4
$ ./buildconf
$ ./configure --prefix=/home/<user>/php/7.4.0beta4 --with-ffi --with-zlib
$ make
$ make test
$ make install
```

We pass in `--with-zlib` to be able to use some compressed data such as compressed PHARs while working. You may need a dependency for it. With `--prefix` we set where the installation process puts the compiled binaries and other required data.

After these steps (if successful) you will have a PHP CLI binary sitting in `/home/<user>/php/7.4.0beta4/bin/php`. You can check that FFI is available by running

```sh
$ /home/<user>/php/7.4.0beta4/bin/php -m | grep FFI
```

If the FFI module is not loaded, verify your build configuration, build steps, and the generated `php.ini` file (found inside `/home/<user>/php/7.4.0beta4/lib` or a similar location).

I recommend adding the built PHP binary to your `$PATH`, as we will be using it on the command line quite a bit over the course of this series.

## Configuring PHP FFI

By default FFI is enabled only in the following cases:

1.  You're running PHP using the CLI SAPI (i.e. you're running PHP scripts in your terminal or via cron or similar use-cases)
2.  You're using the new PHP 7.4 preloading feature when running a web application

If you want to disable FFI, you need to alter your `php.ini`. Look for a line similar to

```ini
ffi.enable=preload
```

And set the value from `preload` to `false`. If the line is prefixed with a `;` comment marker, remove the marker as well.

If you want to have FFI enabled _at all times_ instead, you can set the value to `true`. This means FFI is available outside preload and CLI. You might really want to reconsider using this setting in production environments, as it will most probably make your application slower and less safe. For web application development environments it might be a good choice, as you do not need to rely on the preload mechanism when working on your code, and implement the preloading mechanism later on.

Now that we have PHP with FFI available, we can start coding.

## FFI Hello World

{% notice(type="info") %}
Example code provided in [`101-hello-world`](https://github.com/rask/php-ffi-examples/tree/master/101-hello-world) directory of the examples repository.
{% end %}

What would a tutorial or series be without a _Hello World_ example? Behold:

```php
<?php declare(strict_types = 1);

$ffi = \FFI::cdef(
    'void printf(char *const str, ...);',
    'libc.so.6'
);

$ffi->printf('Hello %s!', 'world');
```

If you run it, it should print `Hello world!` into your terminal window, as expected from a hello world program.

Now let's go through it piece by piece.

```php
<?php declare(strict_types = 1);

// Here we begin creating a new FFI definition.
//
// `cdef` is used to first input C headers, then provide the location of a C-ABI
// library that provides the implementation for the header.
$ffi = \FFI::cdef(

    // Here we provide the C header. It can be a string, which is loaded
    // from an actual file as well using `file_get_contents()` or similar.
    //
    // In this example we define the `printf` function that accepts a regular
    // string. Note the `...`, which is for C variadics.
    'void printf(char *const str, ...);',

    // Next we point to the C-ABI library file that hosts the
    // implementation itself.
    //
    // In this example we load the global system `libc.so` library, meaning the C
    // standard library. This may or may not exist depending on your setup,
    // meaning you need to validate which library file to load.
    //
    // The "6" at the end is part of the filename, and carries no special meaning
    // in terms of loading a library.
    'libc.so.6'
);

// Lastly, we make use of the generated FFI definition. We have the
// functions defined in the C header available to be used through
// the FFI object.
$ffi->printf('Hello %s!', 'world');
```

So far so good. We now know how to load a C-ABI library and call its functions from PHP. And it is not really that verbose either! If you want a PHP analogy, it looks a little like we're using a factory method to instantiate a class filled with magic methods that have been defined elsewhere.

## Alternate method for loading

{% notice(type="info") %}
Example code provided in [`102-hello-world-with-load`](https://github.com/rask/php-ffi-examples/tree/master/102-hello-world-with-load) directory of the examples repository.
{% end %}

You can use the `cdef` method for loading a library, but you may also use the `load` method, which loads and parses a single C header file and infers the library to load from it.

Given you have a C header file called `header.h` written as such:

```h
#define FFI_LIB "libc.so.6"

void printf(char *const str, ...);
```

The `FFI_LIB` definition is used to set the path to a dynamic library that should be loaded for this header file.

Otherwise it can be a somewhat regular C header file.

You can replace the `cdef` in the hello world example above with:

```php
<?php declare(strict_types = 1);

$ffi = \FFI::load(__DIR__ . '/header.h');

$ffi->printf('Hello %s!', 'world');
```

This makes the loading a little simpler, and you get some separation of concerns when it comes to defining what libraries you intend to use, and what functions and other goodies you want to use from those libraries.

## Scoped method for loading

There is a third method for loading a C library, using a feature called FFI scopes.

Scopes are available only when you are using the PHP 7.4 preloading functionality. The following example is really simple and unrunnable, and you will need to setup your preloading configuration and scripts properly if you intend to use scopes.

Given you have a `header.h` file as such:

```h
#define FFI_SCOPE "helloworld"
#define FFI_LIB "libc.so.6"

void printf(char *const str, ...);
```

You can then use something like the following in your preloading script:

```php
<?php declare(strict_types = 1);

\FFI::load(__DIR__ . '/header.h');

function my_printf(string $text, string ...$subs) : void
{
    static $ffi;

    if (!$ffi instanceof \FFI) {
        // This string argument is the same as the
        // `FFI_SCOPE` definition in the `header.h` file
        $ffi = \FFI::scope('helloworld');
    }

    $ffi->printf($text, ...$subs);
}
```

And then in your "nonpreloading" normal code you could call FFI as such:

```php
<?php declare(strict_types = 1);

my_printf('Hello %s!', 'world');
```

This is of course a very bare example, and you also need the preloading configuration and boilerplate in place to make use of scopes properly. Scoped loading is preferred when preloading, as it provides some performance improvements over the `cdef` and `load` methods.

## FFI errors

The FFI feature throws errors in a few situations:

-   Invalid library header or binary definition for `cdef` and friends,
-   Runtime errors when accessing FFI state or resources.

For example, if your C header is malformed, you will be greeted with an error:

```php
<?php declare(strict_types = 1);

$ffi = \FFI::cdef(
    'hello this is invalid',
    'libc.so.6'
);
```

This will throw an `\FFI\ParserException` which lets you know that your header is not valid.

When accessing resources that do not exist (e.g. are not available in the loaded C-ABI library), you might receive something like follows:

```php
<?php declare(strict_types = 1);

$ffi = \FFI::cdef(
    'void idonotexist();',
    'libc.so.6'
);

// PHP Fatal error:  Uncaught FFI\Exception: Failed resolving C function 'idonotexist'
```

If you attempt to call a C-ABI function that does not exist, you receive the following:

```php
<?php declare(strict_types = 1);

$ffi = \FFI::cdef(
    'void printf(char *const str, ...);',
    'libc.so.6'
);

// Above is fine and works, but now we attempt to call something that is not available:

$ffi->some_function();

// Uncaught FFI\Exception: Attempt to call undefined C function 'some_function'
```

If you try loading a C-ABI library that does not exist, another exception is thrown:

```php
<?php declare(strict_types = 1);

$ffi = \FFI::cdef(
    'void printf(char *const str, ...);',
    'idonotexist.so'
);

// Uncaught FFI\Exception: Failed loading 'idonotexist.so'
```

### Handling errors

{% notice(type="info") %}
Example code provided in [`103-hello-world-with-trycatch`](https://github.com/rask/php-ffi-examples/tree/master/103-hello-world-with-trycatch) directory of the examples repository.
{% end %}

All FFI errors are either instances of `\FFI\Exception`, or descendants of it. So the simplest approach to making your FFI loading a little less fragile is to use a generic try-catch as follows:

```php
<?php declare(strict_types = 1);

try {
    $ffi = \FFI::load(__DIR__ . '/header.h');
} catch (\FFI\Exception $e) {
    // log error, do fail tasks, prevent running, etc.
}

$ffi->printf('Hello %s!', 'world');
```

This is a simple example. We have not taken care of the following situation for instance:

```php
<?php declare(strict_types = 1);

try {
    $ffi = \FFI::load(__DIR__ . '/header.h');
} catch (\FFI\Exception $e) {
    // log error, do fail tasks, prevent running, etc.
}

$ffi->function_which_is_not_defined(); // oops
```

Handling raw FFI exceptions is not the best solution, and you will most probably want to abstract your FFI interactions behind a class or similar, which provides saner error handling itself.

Another option could be to run some up-front tests against the FFI instance during loading or in automated tests to make sure the library works. This might not work well enough in the end though.

In any case these are outside the scope of this introductory post, and we will look at FFI abstractions in a later installment of this series.

## Wrapping up

In this part we took a look at the requirements for FFI installation and configuration, and also learned a few ways to load and use a C library inside PHP using FFI.

In the next part, we're going to learn how to write a C-ABI library using Rust, and hopefully learn a bit about how the C-ABI works and how Rust libraries should be constructed to be a little more FFI-friendly.

