[![Build Status](https://travis-ci.org/skx/kilua.png)](https://travis-ci.org/skx/kilua)
[![license](https://img.shields.io/github/license/skx/kilua.svg)]()


# Kilua

Kilua is an extensible, small, portable, and Lua-powered text editor.

The project is built upon the minimal [kilo editor](https://github.com/antirez/kilo) originally written by @antirez, and [introduced here on his blog](http://antirez.com/news/108).  This derived work was put together by [Steve Kemp](https://steve.kemp.fi/) and features many updates and additions compared to the original project:

* The ability to open/edit/view multiple files
   * This is done [via buffers](#buffers).
* The addition of undo-support.
   * Documented later [in this file](#undo-support).
* The addition of an embedded Lua instance.
   * You can define functions in your [init-files](#lua-support), and invoke them via `M-x function()`.
* The addition of regular expression support for searching.
* The addition of regular-expressions for [syntax-highlighting](#syntax-highlighting).
* The addition of [copy and paste](#copy--paste).
* The notion of [named marks](#marks).
* Several bugfixes.

Launching `kilua` works as you would expect:

    $ kilua [options] [file1] [file2] ... [fileN]

Once launched the arrow keys will move you around, and the main keybindings
to learn are:

    CTRL-l: Read lua from the user, and evaluate it.
    CTRL-o: Open a new file.
    CTRL-s: Save the current file.
    CTRL-q: Quit
    CTRL-f: Find string in file (ESC to exit search, arrows to navigate)



## Command Line Options

The following command-line options are recognized and understood:

* `--config file`
    * Load the named (lua) configuration file, in addition to the defaults.
* `--eval`
    * Evaluate the given lua, post-load.
* `--version`
    * Report the version and exit.


## Lua Support

* On startup our initialization files are read:
    * `~/.kilua.lua`.
    * `./kilua.lua`.
    * If zero startup files are loaded this is a fatal error.

It is assumes you'll edit the [supplied startup](kilua.lua) file, to
change the bindings to suit your needs, and add functionality via
the [supplied lua primitives](PRIMITIVES.md).

Pull-requests implementing useful functionality will be recieved with things,
even if just to add syntax-highlighting for additional languages.


## Callbacks

In the future more callbacks might be implemented, which are functions the
C-core calls at various points.

Right now the following callbacks exist and are invoked via the C-core:

* `on_idle()`
    * Called roughly once a second, can be used to run background things.
* `on_key(key)`
    * Called to process a single key input.
* `on_loaded(filename)`
    * Called when a file is loaded.
    * This sets up syntax highlighting in our default implementation for C and Lua files.
* `on_saved(filename)`
    * Called __after__ a file is saved.
    * Can be used to make files executable, etc.



## Buffers

`kilua` allows multiple files to be opened, via the use of buffers.  If `kilua` is launched without any filename parameters there will be two buffers:

* `*Messages*`
    * This receives copies of the status-message.
* An unnamed buffer for working with.
    * Enter your text here, then use `M-x save("name")` to save it.

Otherwise there will be one buffer for each file named upon the command-line,
as well as the `*Messages*` buffer.  (You can kill the `*Messages*` buffer
if you wish, but it's a handy thing to have around.)

The default key-bindings for working with buffers are:

Action                             | Binding
---------------------------------- | --------------
Create a new buffer.               | `Ctrl-x c`
kill the current buffer.           | `Ctrl-x k`
kill the current buffer, forcibly. | `Ctrl-x K`
Select the next buffer.            | `Ctrl-x n` or `M-right`
Select the previous buffer.        | `Ctrl-x p` or `M-left`
Choose a buffer, via menu.         | `Ctrl-x b` or `Ctrl-x B`

It's worth noting that you can easily create buffers dynamically, via lua, for
example the following function can be called by `M-x uptime()`, and does
what you expect:

* Select the buffer with the name `*uptime*`.
     * If that buffer doesn't exist then create it.
* Move to the end of the buffer.
     * Insert the output of runing `/usr/bin/uptime` into the buffer.

      -- Run `uptime`, and show the result in a dedicated buffer.
      function uptime()
          local result = select_buffer( "*uptime*" )
          if ( result == 0 ) then create_buffer( "*uptime*" )   end
          end_of_file()
          insert(cmd_output( "uptime" ) )
      end


## Copy & Paste

We've added a notion of a `mark`.  A mark is set by pressing `Ctrl+space`,
and at any time you can cut the region between the cursor and the mark by
pressing `Ctrl-w`.

You can also cut the current line via `Ctrl-y`.

In both cases you can yank the selection back with `Ctrl-u`.


## Marks

You can record your position (i.e. "mark") in a named key, and
later jump to it, just like in `vi`.

To record the current position use `M-SPACE`, and press the key
you wish to use.  To return to it use `M-m XX` where XX was the
key you chose.


## Syntax Highlighting

Syntax highlighting is defined in lua, and configured by calling:

    -- Setup "keywords"
    set_syntax_keywords({ "void|", "int|", "while", "true" } )

    -- Setup "comments"
    set_syntax_comments( "//", "/*", "*/" )

    -- Enable highlighting of numbers/strings
    syntax_highlight_numbers( 0 | 1 )
    syntax_highlight_strings( 0 | 1 )

The `on_loaded()` function has code currently for highlighting C, C++,
and Lua files, as well as `Makefiles` and some simple highlighting for
both Markdown and plain-text files.

If you wish to change the syntax-highlighting once a file is loaded,
you can do that by entering `M-x` to open the Lua-prompt then
typing (for example):

    set_syntax( "pl" )

Currently we include syntax for Perl, Lua, C, C++, and shell.


## Undo Support

The basic actions of inserting characters, deleting characters, and
moving the cursor can be undo via the lua `undo` function.  (Which is
bound to `Ctrl-z` by default).

**NOTE**: Due to the implementation some actions might corrupt the
undo-stack.  Specifically difficult functions are:

* `kill`.
* `cut_selection`.


## Discussion on Hacker News

https://news.ycombinator.com/item?id=12137698

## The Future

There are no obvious future plans, but [bug reports](https://github.com/skx/kilua/issues) may be made if you have a feature to suggest (or bug to report)!

One thing that might be useful is a split-display, to view two files
side by side, or one above the other.  This is not yet planned, but
I think it could be done reasonably cleanly.

Steve  
\--  
https://steve.kemp.fi/
