+++
title = "Pinebook Pro OSDev: Forth"
draft = true

[taxonomies]
categories = ["post"]
tags = ["forth", "osdev", "pbp"]

[extra]
# comment_issue = 5
+++

*This post assumes you've read the [previous one in the series](@/pbp-osdev/hello-world.md).*

Now that I'm able to run code on the Pinebook, I want a REPL there. [Forth](https://en.wikipedia.org/wiki/Forth_(programming_language)) is the best language I've found for bare-metal development. A Forth interpreter is only a couple thousand x86 opcodes, and it lets one peek and poke at memory directly, without worrying about C's arcana. It also has far stronger metaprogramming capabilities than most languages.
