+++
type = "post"
date = 2018-02-13T13:11:15-06:00
draft = false
author = "Brainicorn"
title = "GO Buffering Lexer"
description = "A library for easily building lexers in Go by utilizing an internal capture buffer so consumers can emit the tokens they care about and forget about the tokens they don't."
keywords = ["go", "golang", "library", "parser", "lexer"]
topics = ["golang"]
tags = ["go", "lexer"]
github = "https://github.com/brainicorn/goblex"
[menu]
  [menu.main]
     name = "Blog"
     weight = 2
     identifier = "post"
     url = "/post/"

+++

## Goblex (Go Buffering Lexer)

[Goblex](https://github.com/brainicorn/goblex) is a library for easily building lexers in Go by utilizing an internal capture buffer so consumers can emit the tokens they care about and forget about the tokens they don't.

This library was built out of necessity. It all began when we decided to write a build-time library that could generate json-schema from go structs...

To accomplish this, we quickly realized we needed to build some sort of "java-like" annotation system (see [our other post about that](/post/ganno)) which in-turn required us to use a lexer for finding complex annotations inside of go comments.

After trying a gazillion different go lexer libraries, we found we'd always hit some use-case that the library couldn't handle and so we decided to write our own.

### Yet Another Go Lexer ##
There are a lot of Go libraries available for building lexers and like this one a lot of them borrow
ideas from [Rob Pike's talk on building lexers.](https://www.youtube.com/watch?v=HxaD_trXwRE)

The difference with this library is that it uses an internal capture buffer to capture/emit tokens
instead of relying on marked positions on the input and/or using regular expressions.
