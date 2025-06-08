# D Wishlist
Just a list of various things that I wish D had.

## Unified toolchain

Why on Earth do I need to install approximately 300 separate tools just to *maybe* create a functioning development environment? As while dmd (the default compiler) and dub (the package manager) can be installed through a single `.deb`, you also need dscanner and dcd. You may also want dfmt and gbd.

I thought I could just do `brew install dmd`, except then my "D Language" IntelliJ plugin freaked out because it couldn't find phobos, D's standard library. Needless to say I got the impression that none of D's toolchain is really designed to work with the other parts of itself.

## Null-Restricted Types

How on Earth does D not have this yet? D has every feature under the Sun *except* for null-restricted types. *HOW?!*

I'm not being hyperbolic either, it's possible to approximate null-restricted types through all the other features D provides:
```d
import std.stdio;
import std.traits;
import std.exception;

void main() {
	log_info("Hello, World!".into());
}

void log_info(NonNull!string message) {
    // This will print the internal string, not the template class, because of the "alias _val this"
    writeln(message);
}

// Stealing the "into" convention from syntax from Rust
// Notice also how this is used as a method / extension function above?
NonNull!T into(T)(T val)
    if (isAssignable!(T, typeof(null)))
{
    return NonNull!T(val);
}

struct NonNull(T)
    if (isAssignable!(T, typeof(null)))
{
    T _val;

    alias _val this;

    this(T value) {
        enforce(value !is null, T.stringof ~ " must not be null");
        this._val = value;
    }

    this() @disable;

    this(typeof(null)) @disable;

    // Prevent a variable, field, or parameter from being [re]assigned null.
    void opAssign(typeof(null)) @disable;
}
```
