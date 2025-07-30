# Multicall Binaries

There have been several approachaes over the years to create binaries (on Unix like systems) that can
include multiple commands, such as [BusyBox](https://busybox.net/), commonly called _multicall_
binaries. Alternatively, this idea is called _software multiplexing_[^2], "_BusyBox-like_", or a
[_crunched binary_](https://man.freebsd.org/cgi/man.cgi?query=crunchgen&sektion=1).

## Why?

To keep this brief, we look at one core feature: Eventual code size. If you know BusyBox, then you
probably also know that it is popular for embedded systems where size is a concern. For example, it is
integrated in [Buildroot](https://buildroot.org/downloads/manual/manual.html#init-system) and
[OpenWrt](https://openwrt.org/docs/guide-developer/write-shell-script).

There are essentially 3 approaches when it comes to executables:

1. shared libraries; common code is moved to files used by multiple programs/commands
2. static binaries; every program stands for itself, potentially duplicating a lot of code
3. multicall; one binary contains all programs, and symbolic links determine the command

NOTE: The multicall approach is also found in shells that have built-in commands.
See e.g. how [U-Boot commands](https://docs.u-boot.org/en/latest/develop/commands.html) work.

## Goals

When it comes to building multicall binaries, there are concerns in terms of software engineering,
licenses, and possibly others. We focus on the software engineering side, but we want to enumerate
a few projects that have been created at least in part for license reasons:

- [toybox](https://landley.net/toybox/about.html) (BSD license)
- [BeastieBox](https://beastiebox.sourceforge.net/) (BSD license)

In general, the programming language is a core issue. Not all languages work well together, so it may
be necessary to focus on a specific set of languages or even a single one.

Another aspect is special treatment. The project setup for a multicall binary may require writing
framework specific code - be it extra code to pull in an external tool or tightly integrating all
code to implement a certain command.

## Implementations

The following are a handful of examples highlighting the aforementioned issues.

[GoBox](https://github.com/surma/gobox/), untouched for 8 years as of July 2025, required wrapping
every command as an _applet_ based on a template. They would be combined in `cmd/gobox/applets.go`.
Using Go's module architecture, it could be possible nowadays to pull in external modules.

[Go Busybox](https://github.com/u-root/gobusybox) from the [u-root](https://u-root.org) project can
take any existing Go command besides its own ones by design. It leverages the Go compiler itself,
working on the AST to rewrite the given code. I.e., commands used with Go Busybox are no different
from standalone commands one may use directly in that the code needs no manual changes.

The [uutils](https://uutils.github.io/) project, written in Rust, has a coreutils collection that
[provides a macro](https://github.com/uutils/coreutils/blob/e48c4a7b96a437e9be90f65e26770f9a6de58b08/src/uucore/src/lib/lib.rs#L175)
to call into a common main function. While this also leverages language specific features, it does
require special code changes in every command. To select which commands to build into the resulting
binary, the native Cargo `--features` flag for its build command is used, so it is familiar to Rust
developers.

There have been a paper[^2] and another proposal[^1] to build multicall binaries based on LLVM.
One possibility is to leverage the intermediate representation (IR) during the compile process.
Using LLVM has the benefit that it serves as a backend to multiple language frontends. That would
possibly allow for having both Rust and C code packed into one binary.

[^1]: <https://groups.google.com/g/llvm-dev/c/HxXbT3qKYgg>
[^2]: <https://dl.acm.org/doi/pdf/10.1145/3276524>
