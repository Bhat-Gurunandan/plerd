# Plerd 

Plerd is meant to be an ultralight blogging platform for Markdown fans that plays well with (but does not require) Dropbox.

It allows you to compose and maintain blog posts as easily as adding and modifying Markdown files in a single folder. Plerd creates an entirely static website based on the content of this one folder, automatically updating the site whenever this content changes.

## Purpose

Plerd allows a blogger to publish new posts to their blog simply by adding Markdown files to a designated blog-source directory. By mixing in Dropbox, this directory can live on their local machine. They can also update posts by updating said files, and unpublish posts by deleting or moving files from that same folder.

The generated website comprises a single directory containing only static files. These include one "permalink" HTML page for every post, a recent-posts front page, a single archive page (in the manner of [Daring Fireball](http://daringfireball.net/archive)), and syndication documents in Atom and [JSON Feed](http://jsonfeed.org) formats. All these are constructed from simple, customizable templates.

That's it! That's all Plerd does.

If you have the time and inclination, you may watch [a 20-minute presentation about my reasons for creating Plerd](http://fogknife.com/2015-06-09-my-yapcna-2015-talk-about-blogging.html).

## Project status

Travis says, "[![Build Status](https://travis-ci.org/jmacdotorg/plerd.svg?branch=master)](https://travis-ci.org/jmacdotorg/plerd)!"

Plerd is released and stable. It still has plenty of room for improvement, and I welcome community feedback and patch proposals, but it will continue to do what it does now, in more or less the same fashion, for the foreseeable future.

## Setup

### Installation

This version of Plerd is intended to run directly from its own source directory, rather than from a formally installed location on your system. You will likely need to install its various library dependencies, however, and you'll need to do a little bit of further configuration and customization.

To install Plerd's dependencies, run the following command from the top level of your Plerd repository (the directory that contains this here README file):

    curl -fsSL https://cpanmin.us | perl - --installdeps .
    
This should crunch though the installation of a bunch of Perl modules that Plerd needs. It'll take a few minutes. When it's all done, Plerd will be ready for configuration.

### Configuration

1. Create a new directory for Plerd's sake. Then, create these subdirectories inside of it:

    * `source`: This will hold your blog's Markdown-based source files.
    * `templates`: Holds your blog's templates.
    * `docroot`: Will hold your blog's actual docroot, ready for serving up by the webserver software of your choice.
    * `db`: Will contain metadata about your blog's posts.
    
    You can freely add other files or directories in this directory if you wish (a `drafts` folder, perhaps?). Plerd will happily ignore them.
    
    *Alternately*, you can simply choose four directories anywhere on your  filesystem to serve these purposes. Just make sure that whatever user runs Plerd's processes has write access to both the source and docroot directories.

1. Copy `conf/plerd_example.conf` to `conf/plerd.conf`, and then update it to best suit your blog. 

    * Set the `path` attribute to the full path of the directory you created in the first step.
    
        (If you took the alternate route of choosing different directories, then set the `source_path`, `publication_path`, `template_path`, and `database_path` directories instead, just like the commented-out lines in `conf/plerd_example.conf` demonstrate.)
    
    * Set the other attributes as should be obvious, based on the provided examples.

1. Copy the contents of this repository's `templates` directory into the new `templates` subdirectory you created in the first step.

    These are sample templates that you can customize as much as you'd like. They are rendered using [Template Toolkit](http://www.template-toolkit.org). You can't change these template files' names, but you can add new sub-template files that the main temlates will invoke via [the [% INCLUDE %] directive](http://www.template-toolkit.org/docs/manual/Directives.html#section_INCLUDE), and so on.

1. Configure the webserver of your choice such that it treats the synced `docroot` subdirectory (which you created as part of the first step) as your new blog's own docroot.

    Plerd does not provide a webserver; it simply generates static HTML & XML files, ready for some other process to serve up.

## Usage

### Running Plerd

Plerd includes two command-line programs, both found in this distribution's `bin` directory:

* __plerdall__ creates a new website in Plerd's docroot directory, based on the contents of its source and templates directories.

    Run this program (with no arguments) to initially populate your blog's docroot, and at any other time you wish to manually regenerate your blog's served files.

* __plerdwatcher__ runs a daemon that monitors the Dropbox-synced source directory for changes, republishing files as necessary.

    ___This is where the magic happens.___ While both _plerdwatcher_ and Dropbox's own daemon process run on your webserver's machine, any changes you make to your blog's source directory will instantly update your blog's published static files as appropriate.
    
    Launch plerdwatcher through this command (assuming your working directory is Plerd's top-level directory):
    
        bin/plerdwatcher start
    
    It also accepts the verbs `stop`, `restart`, and `status`, as well as [all the command-line options listed in the App::Daemon documentation](https://metacpan.org/pod/App::Daemon#Command-Line-Options).

### Composing posts

To start writing a new blog post, just create a new Markdown file somewhere _outside of your blog's source directory_. (Recall that any change to the source directory instantly republishes the blog, something you won't want to do with a half-written entry on top.) You can name this file whatever you like, so long as the filename ends in either `.markdown` or `.md`.

You must also give your post a title, sometime before you're ready to publish it. You define the title simply by having the first line of your entry say `title: [whatever]`, followed by two newlines, followed in turn by the body of your post.

For example, a valid, ready-to-publish source file could be called `today.markdown`, and it could contain this, in full:

    title: My day today

    I had a pretty good day today. 
    
    I hung out at [the coffee shop](http://empireteaandcoffee.com). Then I went home.

    Well, that's all for now. Bye bye.

### Publishing posts

To publish a post, simply move it to Plerd's source directory. (Take care not to overwrite an older post's source file that may have the same name.)

Plerd will, once it notices the new file, give the file a timestamp recording the date and time of its publication. This timestamp will appear in its own line, after the title line.

Normally, Plerd will set the publication time to the moment that you added the file to the source directory. Plerd recognizes two exceptions to this rule:

* If you manually give your post a `time:` timestamp, and it's in W3C date-time format, then Plerd will leave that timestamp alone.
    
* If you leave the timestamp out, _and_ include in your post's filename a date of yesterday or earlier (e.g. `1994-06-10-i-like-ace-of-base.md`), then Plerd will set the post's timestamp to midnight (in the local time zone) of that date. This allows you to batch-backdate many posts at once -- useful, perhaps, for populating a new blog with existing writing.

(Note that Plerd assumes you use a text editor smart enough to see that the source file has both moved and had additional lines added to it from an external process, and to react to this in a graceful fashion.)

Once it has prepared the source file, Plerd will update the blog. It will create a new HTML file for the new entry, and add a link to it from the `archive.html` page. It will also appear in the recent-posts sidebar of every other entry, as well as the Atom and JSON Feed documents (unless you decided to manually backdate the entry by specifying your own date attribute within the file).

### Updating or deleting posts

To update a blog post, just edit its source Markdown file, right in the source directory. Any changes you make will immediately update your published blog as appropriate

To unpublish a blog post, simply move it out of the synced source directory -- or just delete it.

## Using Plerd with Dropbox

Plerd loves Dropbox! (Indeed it had Dropbox affinity in mind from the beginning of its design.)

To have Plerd work with Dropbox, just place its working directory (the one containing the source, docroot, and template subdirectories) somewhere in your synced Dropbox folder, and specify the local path to this folder (from your webserver's point of view) in your Plerd config file.

Now, you can create, update, and delete blog posts just by moving and editing files, _no matter what computer you're using_, so long as it has access to that Dropbox folder.

In this way you could, for example, compose and edit blog posts via Markdown in your favorite text editor while sitting by the fire with your laptop in the back of your favorite coffee shop, publishing updates to your blog by hitting _File &rarr; Save_ in your text editor, and not directly interacting with your webserver (or, indeed, with the Plerd software itself) in any way. [What what.](https://vine.co/v/OB5j0jdn1Pt)

## Advanced use

### Customizing templates

For a brief guide to the template files and how to customize them for your blog, please [see the Plerd wiki on GitHub](https://github.com/jmacdotorg/plerd/wiki/Plerd-template-guide).

### User-defined attributes

You can add any attributes you'd like to your posts, and then refer to them from your templates via a hash named `attributes` attached to every post object. For example, if a post's metadata looks like this:

    title: Example of user-defined attributes
    byline: Sam Handwich

Then you can refer to `post.attributes.byline` to fetch that value from within the `post.tt` template file, even though "byline" is not an attribute that Plerd otherwise recognizes. (If a template refers to an attribute key that a post's source file does not define, it will simply return a blank value.)

### Social-media metatags

By defining some extra attributes in both your blog's configuration file, you
can direct Plerd to add [Open Graph](http://ogp.me) and [Twitter Card](https://dev.twitter.com/cards/overview)-enabling metadata tags to each of your posts. This will allow services like Facebook, Twitter, and Slack to present attractive little summaries of your blogposts when displaying links to them.

These blog configuration options (all optional) are:

* __twitter_id__: If present, then Plerd will try to attach Twitter Card metadata to each post, associated with the given Twitter username. (No leading '@', please. Yes, I know. It confuses the YAML parser. Sorry...)

* __facebook_id__: If present, then Plerd will try to attach Open Graph metadata to each post, associated with the given Facebook app ID.

* __image__: If present, Plerd will use this URL as the location of a default image to use in the metadata for any post that doesn't define its own _image_ attribute.

    If _not_ present, Plerd will _not_ generate any social-media metadata for any post lacking an _image_ attribute.

* __image_alt__: A textual description of the image referenced by the `image` attribute. (Equivalent in usage to the "alt" attribute in an HTML `<img>` tag.) Plerd will just leave this blank, if you don't define it yourself.

To see examples of all the above, please see the file `conf/plerd_example.conf`.

Once you've configured your blog as described above, you can add these attributes to any post:

* __description__: A very brief summary of this post.

    If not defined, then Plerd will try to use the first paragraph of your post's text (after stripping out any markup) as the post's description.

* __image__: The URL of an image to associate with this post within social-media links. (This could refer to image that also appears in your post by way of an HTML `<img>` tag, but it doesn't have to.) 

    If not defined, then Plerd will instead use the blog's _image_ configuration directive. If _that_ is also undefined, then Plerd will not generate any social-media metadata for this post.
    
* __image_alt__: A textual description of the image referenced by the `image` attribute. (Equivalent in usage to the "alt" attribute in an HTML `<img>` tag.) Plerd will just leave this blank, if you don't define it yourself.

### Webmention

Plerd includes experimental support for [Webmention](https://alistapart.com/article/webmentions-enabling-better-communication-on-the-internet), an open technology that allows websites to send simple "Hey, this page of mine contains a link to that page of yours!" messages to other websites. If the linking page employs [Microformats2](http://microformats.io) metadata, then the target page can choose to display salient information about the mention, such as its author, or a summary of its content. It can adjust the format of this display depending upon the mention's apparent type -- a "like", a repost, a comment-style response, and so on.

Thus, with certain options enabled, Plerd can send webmentions to websites that your blog posts link to. It can also receive and display webmentions when pages elsewhere on the internet link to specific posts of yours.

These features are sparsely documented or tested as of August 2018, but I expect that to improve presently. Adding Webmention support has been the focus of my Plerd development since March, and I expect it to receive more complete documentation and test coverage by the end of the year.

For the time being, you can consult the documentation of the `bin/plerdwatcher` and `bin/plerdall` programs for a list of Webmention-related options, and see `templates/post.tt` among the included sample templates for an example of webmention display technique. Note also the commented-out `rel="webmention"` line in `templates/wrapper.tt`, which requires uncommenting for webmention receipt to work.

You can see a variety of real-life displayed webmentions on [this post](http://fogknife.com/2018-04-22-announcing-two-new-indieweb-modules-for-perl.html#webmentions), part of my own Plerd-powered blog. (Note that this includes mentions from other, subsequent posts within the same blog, making for an organic sort of "Related Posts" feature.)

If you are interested in helping me test Webmention support working with your own Plerd setup while it's still in this experimental stage, please do feel free to get in touch via email.

## A note about encoding

Plerd assumes that all your source and template files are encoded as UTF-8.

## Support

To report bugs or file pull requests, visit [Plerd's GitHub repository](https://github.com/jmacdotorg/plerd).

For now, the best way to ask questions about Plerd is to simply [email me, Jason McIntosh](mailto:jmac@jmac.org). I am always interested to hear about other folks making use of Plerd, and I will do whatever I can to help them with it. All such feedback does tend to make the software that much better, after all!

[Plerd has a homepage at its creator's website.](http://jmac.org/plerd)

## See Plerd at work

This software powers Jason McIntosh's blog, [Fogknife](http://fogknife.com), for which it was written.

## Credits

Plerd is by Jason McIntosh (<jmac@jmac.org>). I would love to hear any thoughts about Plerd you would care to share, and welcome any questions or suggestions about it.

Contributors include:

* Christian Sánchez (<http://www.chsanch.info/>)

* David Turner (<novalis@novalis.org>)

* Rebecca Turner (<me@re-becca.org>)


This repository contains the image "Envelope" designed by <a href="http://www.thenounproject.com/jontesta">Jon Testa</a>, and the image "RSS" designed by [useiconic.com](https://thenounproject.com/useiconic.com). Both are shared through a [Creative Commons Attribution 3.0 United States](https://creativecommons.org/licenses/by/3.0/us/) license, and come to this project via [The Noun Project](https://thenounproject.com).
