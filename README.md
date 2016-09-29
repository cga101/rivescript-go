# RiveScript-Go

[![Build Status](https://travis-ci.org/aichaos/rivescript-go.svg?branch=master)](https://travis-ci.org/aichaos/rivescript-go)

## Introduction

This is a RiveScript interpreter library written for the Go programming
language. RiveScript is a scripting language for chatterbots, making it easy
to write trigger/response pairs for building up a bot's intelligence.

**This project is currently in Beta status.**

## Documentation

* RiveScript Library: <http://godoc.org/github.com/aichaos/rivescript-go>
* RiveScript Stand-alone Interpreter: <http://godoc.org/github.com/aichaos/rivescript-go/cmd/rivescript>
* JavaScript Object Macros: <http://godoc.org/github.com/aichaos/rivescript-go/lang/javascript>

## Installation

`go get github.com/aichaos/rivescript-go`

For the stand-alone binary for testing a RiveScript bot:

`go install github.com/aichaos/rivescript-go/cmd/rivescript`

## Usage

The distribution of RiveScript includes an interactive shell for testing your
RiveScript bot. Run it with the path to a folder on disk that contains your
RiveScript documents. Example:

```bash
# (Linux)
$ rivescript eg/brain

# (Windows)
> rivescript.exe eg/brain
```

See `rivescript --help` for options it accepts, including debug mode and UTF-8
mode.

When used as a library for writing your own chatbot, the synopsis is as follows:

```go
package main

import (
    "fmt"
    rivescript "github.com/aichaos/rivescript-go"
)

func main() {
    bot := rivescript.New()

    // Load a directory full of RiveScript documents (.rive files)
    err := bot.LoadDirectory("eg/brain")
    if err != nil {
      fmt.Printf("Error loading from directory: %s", err)
    }

    // Load an individual file.
    err = bot.LoadFile("brain/testsuite.rive")
    if err != nil {
      fmt.Printf("Error loading from file: %s", err)
    }

    // Sort the replies after loading them!
    bot.SortReplies()

    // Get a reply.
    reply := bot.Reply("local-user", "Hello, bot!")
    fmt.Printf("The bot says: %s", reply)
}
```

## UTF-8 Support

UTF-8 support in RiveScript is considered an experimental feature. It is
disabled by default. Enable it by setting `RiveScript.UTF8 = true`.

By default (without UTF-8 mode on), triggers may only contain basic ASCII
characters (no foreign characters), and the user's message is stripped of all
characters except letters, numbers and spaces. This means that, for example,
you can't capture a user's e-mail address in a RiveScript reply, because of
the @ and . characters.

When UTF-8 mode is enabled, these restrictions are lifted. Triggers are only
limited to not contain certain metacharacters like the backslash, and the
user's message is only stripped of backslashes and HTML angled brackets
(to protect from obvious XSS if you use RiveScript in a web application).
Additionally, common punctuation characters are stripped out, with the default
set being `/[.,!?;:]/g`. This can be overridden by providing a new `Regexp`
object as the `RiveScript.UnicodePunctuation` attribute. Example:

```go
// Make a new bot with UTF-8 mode enabled.
bot := rivescript.New()
bot.UTF8 = true

// Override the punctuation characters that get stripped from the
// user's message.
bot.UnicodePunctuation = regexp.MustCompile(`[.,!?;:]`)
```

The `<star>` tags in RiveScript will capture the user's "raw" input, so you can
write replies to get the user's e-mail address or store foreign characters in
their name.

## Building

I use a GNU Makefile to make building and running this module easier. The
relevant commands are:

* `make setup` - run this after freshly cloning this repo. It runs the
  `git submodule` commands to pull down vendored dependencies.
* `make build` - this will build the front-end command from `cmd/rivescript`
  and place its binary into the `bin/` directory. It builds a binary relevant
  to your current system, so on Linux this will create a Linux binary.
  It's also recommended to run this one at least once, because it will cache
  dependency packages and speed up subsequent builds and runs.
* `make build.win32` - cross compile for Windows, but see below.
* `make run` - this simply runs the front-end command and points it to the
  `eg/brain` folder as its RiveScript source.
* `make dist` - creates a binary distribution (see [Distributiong](#distributing)
  below).
* `make dist.win32` - cross compiles a binary distribution for Windows,
  equivalent to `build.win32`
* `make fmt` - runs `gofmt` on all the source code.
* `make test` - runs `go test`
* `make clean` - cleans up the `.gopath`, `bin` and `dist` directories.

### Cross-compile for Windows

If you're using Go v1.5 or higher, and aren't running Windows, you can build
the `rivescript.exe` file using the cross compilation feature.

If you're using Go as provided by your distribution's package maintainers, you
need to mess with some path permissions for the Win32 build to work. This is
because Go has to build the entire standard library for the foreign system
(for more information, see [this blog post](http://dave.cheney.net/2015/08/22/cross-compilation-with-go-1-5)
on the topic).

Run `mkdir /usr/lib/golang/pkg/windows_386` and `chown` it as your user account.

Then run `make build.win32` to cross compile the Win32 binary and output it to
`bin/rivescript.exe`

You should sanity test that the binary actually runs from a Windows environment.
It might not run properly via Wine.

## Distributing

The GNU Makefile can build distributable binary forms of the RiveScript command
for the host OS (usually Linux) and cross-compile for Windows. Building for Mac
OS X, and building from within a Windows dev environment have not yet been
tested.

* `make dist` - build a distributable for your current system (usually Linux,
  but Mac would probably work too).
* `make dist.win32` - build a distributable for Windows using cross compilation.

The `dist` commands will create a directory named `dist/rivescript` which you
can inspect afterwards, and creates a zip file (Windows) or a `tar.gz` (Linux
and Mac) with the contents of that folder, with names like
`rivescript-0.0.2-win32.zip` and `rivescript-0.0.2-Linux.tar.gz` in the current
directory (the root of the Git repo).

The distributable directory contains only the following types of files:

* The executable binary (`rivescript.exe` for Windows or `rivescript` otherwise)
* Metadata files: `README.md`, `LICENSE`, etc.
* Example brain at `eg/brain`
* (Windows only) a super simple batch file for launching `rivescript.exe` and
  pointing it to the example brain: `example.bat`

## See Also

* [rivescript-go/parser](./parser) - A standalone package for parsing RiveScript
  code and returning an "abstract syntax tree."
* [rivescript-go/macro](./macro) - Contains an interface for creating your own
  object macro handlers for foreign programming languages.

## License

```
The MIT License (MIT)

Copyright (c) 2016 Noah Petherbridge

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## See Also

The official RiveScript website, http://www.rivescript.com/
