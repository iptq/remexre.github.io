+++
date = 2018-03-12
tags = ["OftLisp"]
title = "oftb refocus, and a module system redesign"
+++

I'm rewriting the current oftb in the [oftlisp-rs](https://github.com/oftlisp/oftlisp-rs) repo, to focus on a smaller, simpler interpreter.
The original goal of oftlisp-rs was to create a simple interpreter, solely for bootstrapping.
And then I got distracted.
Currently, oftlisp-rs supports multiple backends, arbitrary context payloads, and a host of other features that don't really make sense in the context of a bootstrap interpreter.
Furthermore, I feel like I've got a much tighter grip on the "right way" to write a fast interpreter.

Additionally, now would be a good time to overhaul how modules work, since I'm making deep changes anyway.
Previously, the module system worked similarly to Go's -- there's a root and a home directory, which imports are relative to.
If an import is found in the root directory, it is used immediately.
If not, the home directory is checked.
Both the root and home directories are structured as a tree of Git checkouts.

---

The new module system borrows more from Rust's Cargo package manager than Go.
In the new model, package paths are not Git repository URLs, but rather are "flat" per-build.
These checkouts are stored in a `build/deps` directory, although they may later become symlinks instead.
Import paths are then `foo/bar`, rather than `github.com/remexre/foo/bar`.

Additionally, the information about a package's dependencies is stored in a `package.oftd` file in the root of the package.

The structure of a package is then:

```
foo/
├── .git/
│   └── ...
├── .gitignore
├── build/
│   ├── deps/
│   │   ├── combinargs/
│   │   │   ├── .git/
│   │   │   │   └── ...
│   │   │   ├── .gitignore
│   │   │   ├── package.oftd
│   │   │   └── src/
│   │   │       └── lib.oft
│   │   └── grid/
│   │       ├── .git/
│   │       │   └── ...
│   │       ├── .gitignore
│   │       ├── package.oftd
│   │       └── src/
│   │           └── ...
│   ├── foo*
│   └── objs/
│       ├── combinargs.oftbc
│       ├── foo.oftbc
│       └── grid.oftbc
├── package.oftd
└── src/
    ├── baz/
    │   └── quux.oft
    ├── baz.oft
    ├── lib.oft
    ├── main.oft
    └── xyzzy.oft
```

The `package.oftd` file looks like:

```oftd
(library
  (name foo)
  (version "0.1.0"))

(binary
  (name foo)
  (path "src/main.oft"))

(dependencies
  (combinargs
    (version "0.2.1"))
  (grid
    (git "https://github.com/remexre/oftlisp-grid.git")
    (version "0.1.0")))
```

---

Switching topics, I'm also going to outline the design of the new oftb interpreter.
For speed (and to explore the design space that the eventual compiler will take), it will perform a quick compile to an internal bytecode, and interpret that.
The bytecode will be similar to oftlisp-rs's ANFIR bytecode, which is itself based on Matt Might's article, [Writing an interpreter, CESK-style](http://matt.might.net/articles/cesk-machines/).

The overall pipeline is:

```
+------+    +------+    +---+    +---+    +--------+
|Source|--->|Values|--->|AST|--->|ANF|--->|Flat ANF|
+------+ ^  +------+ ^  +---+ ^  +---+ ^  +--------+
         |           |        |        |
 parser--+           +-----+  |        +--flatanf::Program::flatten
                           |  |
 ast::Module::from_values--+  +---anf::Module::convert
```

Flat ANF's structure will be laid out in a later post, but it's still much higher-level than LLVM or Cretonne's IR; rather than having basic blocks and instructions, it retains a tree structure.
It is low-level enough to eliminate modules (the "flattening" in the name), and uses [de Bruijn indices](https://en.wikipedia.org/wiki/De_Bruijn_index) to improve the efficiency of environment lookups.

---

In conclusion, these design changes should make it easier to get a usable version of oftb out the door (*cough*), while the module system changes should make development easier.
