
# Zig Wishlist
Just a list of various things that I wish Zig had.

## Anonymous Functions
Zig technically has this, but it's being kept *[deliberately](https://github.com/ziglang/zig/issues/1717#issuecomment-1627790251)* obnoxious. I am not asking for closures, but Zig's standard library is already littered with APIs that expect functions as fields and parameters. Not to mention community-made libraries. It would be nice to be able to just write a function inline, rather than as a sibling function elsewhere in the code, or as the following monstrosity:
```zig
const example: SomeStruct = .{
	.sortFn = (struct {
		pub fn sort(lhs: SomethingElse, rhs: SomethingElse) !void {
			// Whatever
		}
	}).sort,
};
```

## Explicit errdefer invocation
In the ["The Road to Zig 1.0"](https://www.youtube.com/watch?v=Gv2I7qTux7g) video, Andrew framed Zig as "C but with the problems fixed", as opposed to C++ with its overabundance of features. But I would argue that the introduction of error types very much disqualifies Zig as being *just C but better.* This is made most obvious when trying to interface with C (such as passing a C library a callback function) and basically losing most of Zig's language features, notably `errdefer`.

There's currently no ergonomic way to fix this:
```zig
export fn foo(arg: ?*anyopaque) callconv(.c) c_int {
    (tryblock: {
	    // This doesn't work as 'try' will return an error, not 'break' out of 'tryblock'.
	    const example = try something();
	    // And this will never run
	    errdefer example.deinit();
	    
	    try somethingElse();
    } catch {
        // And this obviously cannot access variables within the tryblock block
	    example.deinit();
    });
}
```

And so we're stuck defining duplicate functions, causing a blast radius of litter-functions in our code. And ultimately I feel like this was all so Andrew could add `try` because he didn't want to keep doing:
```c
if (doSomething() == -1) {
    return -1;
}
```

This isn't helped by the Andrew's previously stated hate-boner for anonymous functions:
```zig
export fn foo(arg: ?*anyopaque) callconv(.c) c_int {
    return (fn (z_arg) !c_int {
	    const example = try something();
	    errdefer example.deinit();
	    
	    try somethingElse();
    })(arg) catch -1;
}
```
This is not exactly elegant, but it'd work.

Instead, I think a better solution would be to add an explicit `returnerr` or some special `@error` function:
```zig
export fn foo(arg: ?*anyopaque) callconv(.c) c_int {
    const example = something() catch return -1;
    errdefer example.deinit();
    
    guaranteedToFail() catch |err| return @error(err, -1); // This will invoke the above errdefer
}
```

This will mean that a `-1` is returned from the function, but all relevant `errdefer` blocks are invoked. It does seem a little odd that the *only* way `errdefer` works is to propagate the error upwards.

## Interfaces
Many times I've encountered instances where a function parameter is `anytype` and yet is not checked but used immediately as if it's a very particular type. Take for example [this format function](https://github.com/nektro/zig-time/blob/e946a144423cdb5dac3d46d6856c6e6da73e9305/time.zig#L218) in the [zig-time](https://github.com/nektro/zig-time) library. Why is it like that? Well, because interfaces don't exist in Zig yet, so the language has imported some of JavaScript's "just treat it like it's the correct type anyway" issues. At least it's not `anyopaque`, so *some* level of analysis is possible, but there's no need for the language to *effectively* require type-coyness like this.

There are various ideas on how to fix this, I personally like [this suggestion](https://github.com/ziglang/zig/issues/17198#issuecomment-2533468501), but overall I'd just love a Rust-like trait-impl system.

## Explicit Runtime Execution
Zig has `comptime`, which allows you to force compile-time computation (with limitations, of course) where something would usually be done at runtime. I am asking for the opposite, for a way to force runtime-computation for something that would be done at compile time (with likely similar limitations, of course), similar to Rust's [lazy_static](https://docs.rs/lazy_static/latest/lazy_static/) crate or Dart's [late](https://dart.dev/language/variables#late-variables) keyword. For example:
```zig
// This is a top-level hash-map that has some initial values
// and can be further added to at runtime, like a registry.
const example = lazy createSomeHashMapThatRequiresAllocation(std.heap.page_allocator);
```
This would mean that the "example" variable is known to exist and its type is know, and it's constant, but its value is only set upon first access.

## Better Array and Slice interoperability
*(Glossary: when I use "array" in quotes like that, know that I just mean a collection of elements of a known size. I do not [necessarily] mean Zig [arrays](https://ziglang.org/documentation/0.13.0/#Arrays) or [slices](https://ziglang.org/documentation/0.13.0/#Slices), which have their own specific meanings.)*

This has been the bane of my existence during 2024's Advent of Code: all I want to do is iterate through an "array" of u8's, or what have you. I have a string via `@embedFile("input.txt")` and just want to pass that into function to produce a grid, or what have you. Except it doesn't work because your parameter's `[]const u8` type doesn't match `*[38]const u8`. I know this is probably a skill issue, that there's some way around this... but why does there need to be? And why do they all look so similar? Can I please, for the love of all that is holy, just be able to pass the thing into the other thing and it just work? Is that really so much to ask?
