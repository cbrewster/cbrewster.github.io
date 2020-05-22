@Meta
title = "Building Brewblog: A Static Site Generator Written In Rust"
slug = "building-brewblog"
author = "Connor"
date = "2020-05-21"

@Content

### What is Brewblog?

Brewblog is a static site generator that I wrote to generate this website.
I wrote the bulk of it in an evening and it has just enough features to generate
the pages you see here.

The code for Brewblog is hosted 
[on GitHub](https://github.com/cbrewster/brewblog).

### Why build a static site generator?
 
There are many great static site generators out there: Jekyll, Hugo, Zola, 
etc... so why bother to build another one?

I decided to build my own static site generator as a fun exercise. I don't plan
to use many of the features available in preexisting site generators, so the 
scope of Brewblog is rather small and I only plan to add features that I need 
for this blog. This means I wouldn't recommend using Brewblog for your 
own blog, but I won't stop you. Pull requests are welcome.

I wanted a static site generator that could do the following:
 * Allow creation of pages and indexes
 * Convert markdown into HTML
 * Allow custom templates
 * Watch the content directory and update the output locally when files change

For this project, I chose Rust because it's the language I most enjoy writing
in. Also Rust has excellent crates for markdown parsing, creating CLIs, 
templating, and syntax parsing.

### Building a CLI (Command Line Interface) in Rust

Brewblog provides a simple CLI for building and watching the static site.

```
$ brewblog --help   
brewblog 0.1.4
Connor Brewster

USAGE:
    brewblog <SUBCOMMAND>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

SUBCOMMANDS:
    build    Builds the static site
    help     Prints this message or the help of the given subcommand(s)
    new      Create a new static site
    serve    Builds the static site and serves the site using a local server
```

Brewblog was the first Rust CLI project I have worked on. I chose a crate called
[`clap`](https://github.com/clap-rs/clap) to help build the CLI and it was 
extremely simple to build out a simple CLI that felt very polished. I didn't 
want to spend much time building an amazing interface since I'm likely to be the
only user, but `clap` made building a user-friendly CLI a _snap_.

Clap 3 provides some `derive` macros that make building a CLI a breeze!
The derive macro lets you define the options you would like to allow via a
struct and through annotations you can provide clap with extra information to
output in the help command. Best of all, using the `derive` approach gives you
strong guarantees about type safety.

```rs
#[derive(Clap, Debug)]
#[clap(version = crate_version!(), author = "Connor Brewster")]
struct Opts {
    #[clap(subcommand)]
    command: Command,
}

#[derive(Clap, Debug)]
enum Command {
    New(NewCommand),

    #[clap(about = "Builds the static site")]
    Build,

    #[clap(about = "Builds the static site and serves the site using a local server")]
    Serve,
}

#[derive(Clap, Debug)]
#[clap(about = "Create a new static site")]
struct NewCommand {
    #[clap(about = "The name of the site to be created")]
    name: String,

    #[clap(about = "The directory to create the site in (If blank, the site name will be used)")]
    directory: Option<String>,
}
```

The `help` and `version` outputs are fully generated for you and all you need to
do is use `::parse()` to parse CLI input:

```rs
let opts = Opts::parse();

match opts.command {
    Command::New(new_opts) => new_site(new_opts)?,
    Command::Build => build()?,
    Command::Serve => serve().await?,
}
```

For such a small amount of code, you get an extremely friendly interface. Clap
even provides helpful messages if a user accidentally misspells a command:

```
$ brewblog server  
error: The subcommand 'server' wasn't recognized

        Did you mean 'serve'?

If you believe you received this message in error, try re-running with 'brewblog -- server'

USAGE:
    brewblog <SUBCOMMAND>

For more information try --help
```

The `new` command only creates a new directory at the moment, but I plan to add
a basic template that can be used to spin up a new site.

Overall, using `clap` was a pleasant experience and I would highly recommend it
for building Rust CLIs.

### Tera Templating

To generate pages, Brewblog looks for the following templates in the templates
directory:

 * `page.html.tera` - Default template for any `.md` file.
 * `index.html.tera` - Default template for generating an index of pages.

To render templates I used the [`tera`](https://github.com/Keats/tera) crate.
Tera has a similar syntax to Jinja2. It allows you to pass in a context map
that contains data that can be accessed in the templates. Additionally it
comes with nice built-in functions. For example, the publication date on this
post is formatted using the `date()` built-in.

There's nothing too special about the templating, but it gets the job done!

### Markdown Parsing

For markdown parsing I used
[pulldown-cmark](https://github.com/raphlinus/pulldown-cmark) which is a fast
pull parser for markdown written in Rust.

One important feature of a programming blog is to have built-in syntax
highlighting. While pulldown-cmark does not have this built-in, it is designed
to be very extensible. I searched the project's issues and 
[found](https://github.com/raphlinus/pulldown-cmark/issues/167) some
approaches to syntax highlighting. In the end, I settled with an 
[implementation](https://github.com/cbrewster/brewblog/blob/master/src/markdown.rs)
that intercepts code blocks tagged with a language and inserts the highlighted
code as HTML when the code block is closed.

The ease of extensibility is a real testament to the nice architecture of
pulldown-cmark.

#### Page format

Each markdown page is split into two sections: `@Meta` and `@Content`. I know 
this isn't very standard, but who cares, this is for fun. The content after 
`@Meta` is written in `toml` and it lets you provide some basic metadata about 
the page like the title, slug, author, and publication date. Everything 
following the `@Content` section is the markdown content for the page. The 
markdown does allow HTML as well if you need a bit more control over the output
of the page.

For example, this page starts with:
```rs
@Meta
title = "Building Brewblog: A Static Site Generator Written In Rust"
slug = "building-brewblog"
author = "Connor"
date = "2020-05-21"

@Content

### What is Brewblog?
...
```

### Blog Theme

Brewblog does not have a concept of a theme, it just renders templates
from the templates directory. Since I only plan to use this myself, it didn't
seem worth trying to build in swappable theme setup. This may be an area to
extend in the future, but a templates directory works for now.

I built the theme for this site by hand. I aimed to create a simple layout
with a nice dark color scheme. For this site, in particular, I made a very 
simple theme using [TailwindCSS](https://tailwindcss.com/). Tailwind provides a 
plethora of CSS utility classes which allow you to design websites without 
needing to touch much CSS. It also comes with a great default color palette, 
which I use here.

Since Tailwind uses utility classes, markdown output will be unstyled by
default. To make markdown rendered pages look nice, I wrote some custom
markdown CSS styles using `@apply` and Tailwind classes. The markdown is always
inserted in a `div` with a `#markdown` id. This ensures that the markdown styles
don't leak out of the markdown content.

Here is a short snippet of how I style the HTML generated by markdown:
```css
#markdown h1 {
    @apply text-4xl my-4 text-red-400;
}

#markdown h2 {
    @apply text-3xl my-4 text-red-400;
}

#markdown h3 {
    @apply text-2xl my-2 text-red-400;
}

#markdown h4 {
    @apply text-lg my-2 text-red-400;
}

#markdown a:link {
    @apply underline;
}

...
```

### Result

Overall, I am pretty pleased with Brewblog, creating new posts is very easy
and I have automated deploys using 
[brewblog-deploy-action](https://github.com/cbrewster/brewblog-deploy-action)
with GitHub Actions. 

There are a few remaining features that I would like to add:

 * Static site scaffolding through `brewblog new`
 * Parsing dates from file names

I didn't cover everything that was used to create Brewblog, but the codebase
is pretty small, so feel free to check it out 
[on GitHub](https://github.com/cbrewster/brewblog).

This blog itself is also open source and you can check it out 
[on GitHub](https://github.com/cbrewster/cbrewster.dev) as well.

Pull requests are always welcome!
