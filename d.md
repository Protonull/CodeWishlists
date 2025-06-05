# D Wishlist
Just a list of various things that I wish D had.

## Unified toolchain

Why on Earth do I need to install approximately 300 separate tools just to *maybe* create a functioning development environment? As while dmd (the default compiler) and dub (the package manager) can be installed through a single `.deb`, you also need dscanner and dcd. You may also want dfmt and gbd.

I thought I could just do `brew install dmd`, except then my "D Language" IntelliJ plugin freaked out because it couldn't find phobos, D's standard library. Needless to say I got the impression that none of D's toolchain is really designed to work with the other parts of itself.

## Null-Restricted Types

How on Earth does D not have this yet? D has every feature under the Sun *except* for null-restricted types. *HOW?!*

It is possible to approximate null-restricted types through constrained templates, but this is more of a developer signal since it's not checked at compile time.
