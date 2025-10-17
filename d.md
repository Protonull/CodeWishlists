# D Wishlist
Just a list of various things that I wish D had.

## Unified toolchain

Why on Earth do I need to install approximately 300 separate tools just to *maybe* create a functioning development environment? As while dmd (the default compiler) and dub (the package manager) can be installed through a single `.deb`, you may also need dscanner, dcd, dfmt, and gbd.

I thought I could just do `brew install dmd`, except then my ["D Language"](https://plugins.jetbrains.com/plugin/8115-d-language) plugin freaked out because it couldn't find Phobos, D's standard library. *sigh*

## Mature tooling

D's tooling leaves something to be desired, for example: [static imports](https://dlang.org/spec/module.html#static_imports) require you to use each imported symbol's fully-qualified name, which you may want for [reasons](https://www.youtube.com/watch?v=4NYC-VU-svE&t=134), but the tooling is not smart enough to understand where that fully-qualified symbol comes from, eg:

```d
static import std.stdio;

public void main() {
    std.stdio.writeln("Hello, World!"); // Possibly undefined symbol
}
```

Similarly, if you start using an API from a module you've yet to import, like `Clock.currTime()`, the tooling will not suggest or auto-add `import std.datetime.systime` for you.

Speaking of, when you Ctrl+B (Go to Declaration) for `Clock.currTime()`, it takes you to the static method, which invokes `SysTime`'s constructor. Ctrl+B that and it takes you to the `SysTime` *type*, not the constructor being invoked. Luckily, it's *right there*, but it invokes another constructor, which Ctrl+B doesn't take you to, which also invokes another constructor, etc.

And *speaking of*, my tooling insists that `SysTime` has a field named `dateTime`, but I cannot Ctrl+B it, nor will it compile if I try to use it. It seems like the tooling is just collecting all the constructor-parameter names and assuming they'll all be fields on the struct. *Pst, if you want to get a `DateTime` from a `SysTime`, you need to cast it, which is hilarious.*

## Null-Restricted Types

How on Earth does D not have this yet? D has almost every feature under the Sun *except* for null-restricted types. *HOW?!* I'm not being hyperbolic either, I was able to create a package that approximates null-restricted types by wielding D's other features ([DullSafety](https://github.com/Protonull/DullSafety/)) which is simply absurd.
