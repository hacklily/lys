# lys - Run lilypond as a server

This package provides a way to keep Lilypond running in the background as a server. It then extends the `lyp compile` command to compile user files by sending requests to the running Lilypond instance. The server can also be connected to using `telnet` or from any programming language by opening a TCP port.  

## Why?

Because Lilypond is just too damn slow...

## Installation

Install using [lyp](https://github.com/noteflakes/lyp):

```bash
$ lyp install lys
```

# Compiling using the server

To compile files using the server command, use the `lyp compile` command and add the `-s` or `--server` option:

```bash
$ lyp compile -s myfile.ly
```

You can of course add any other Lilypond option you wish for, e.g.:

```bash
$ lyp compile -s --png myfile.ly
```

The server is started automatically. You can kill it at any time by the usual means.

## Mode of operation

The server listens on port 1225 (by default). Clients can connect and pass arbitrary scheme expressions. For example, to typeset a lilypond file, a client can send `(lys:compile dir opts...)` where pwd is the current working directory and opts is a list of command line options:

```scheme
(lys:compile "/home/sharon" "--png" "myfile")
```

## How fast is it?

Small files compile in 0.3-0.4s (on a modern laptop). Generally one can expect to shave 0.5-0.6s off compilation time.

## Manually starting a Server

To start a server, run the following lilypond script:

```lilypond
\require "lys"

% listen on port 1225 ("ly" in numbers)
#(lys:start-server)

% to specifiy the listening port:
#(lys:start-server 12321)
```

Or alternatively, run from the shell:

```bash
$ lilypond -r lys -e "#(lys:start-server)"
```

## Connecting from the shell

Commands can be sent to the server by piping to `nc`:

```bash
$ echo "(lys:compile-file \"~\" \"myfile.ly\")" | nc localhost 1225
```

## API

See also the included [example client](test/lyc.sh).

### lys:close

Usage: `lyp:close`

Normally, a connection to the server stays open until the client disconnects. A client connecting through `nc` can ask the server to shutdown the connection by sending `(lyp:close)`.

### lys:compile-file

Usage: `lys:compile-file pwd opt ...`

Compile a lilypond file, where `pwd` is the client's working directory, and opt is one or more [lilypond command line options](http://lilypond.org/doc/v2.18/Documentation/usage/command_002dline-usage.en.html). `lys:compile-file` currently handles all of lilypond's command line options except `--loglevel` and `--include`.

Example: `(lys:compile-file "/Users/dudu" "--png" "myfile.ly")`

### lys:stdin-eval-loop

Usage: `lys:stdin-eval-loop`

Start an evaluate loop on stdin. This can be used to start a long-running slave lilypond process that evaluates arbitrary scheme expressions received on stdin.

### lys:spawn

Usage: `lys:spawn expr ...`

Fork and evaluate a sequence of expressions in the child process. Returns the child pid.

Example: `(lys:spawn (lys:compile-file "/Users/dudu" "--png" "myfile.ly"))`

### lys:typeset

Usage: `lys:typeset-slice music filename`

Compile the given music variable into to the given output filename.

Example: `(lys:typeset myMusic "my-music")`

### lys:typeset-slice

Usage: `lys:typeset-slice music start-moment end-moment filename`

Compile a range of the given music variable between two moments and output to the given filename. The moments are specified as lists that are converted into ly:moment.

Example: `(lys:typeset-slice myMusic '(3 1) '(6 1) "music3-6")`

