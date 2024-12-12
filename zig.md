
# Zig Wishlist
Just a list of various things that I wish Zig had.

## Yeild Blocks
While Zig does have yieldable blocks, they're unnecessarily verbose:
```zig
const example: i32 = brk: {
	const raw = "364";
	break :brk std.fmt.parseInt(i32, raw, 10);
};
```
whereas I advocate for something like the following:
```zig
const example: i32 = produce {
	const raw = "364";
	yield std.fmt.parseInt(i32, raw, 10);
};
```
The `produce` keyword is optional. I figure this is a nice compromise, similar to how `try` is shorthand for `catch |err| return err;`

## Try Blocks
While you are able to do a scuffed version of try-blocks, they are also unnecessarily verbose, nor do they work with `try`:
```zig
(tryblock: {
	// This doesn't work as 'try' will return an error, not 'break' out of 'tryblock'.
	try something();
} catch {
	other.deinit();
});
```
This is most needed when dealing with C code, or wasm code, or any other code that isn't compatible with Zig's errors. And while it's obviously possible to delegate that code into its own Zig function, I find it somewhat ridiculous that the solution to this is to double-up every function that touches C code, or wasm code, or what have you.

## Anonymous Functions
This is another thing where Zig technically has it, but it's *[deliberately](https://github.com/ziglang/zig/issues/1717#issuecomment-1627790251)* obnoxious. I am not asking for closures, but Zig is already littered with structs that expect functions as fields. It would be nice to be able to just write a function inline, rather than as a sibling function, or as the following monstrosity:
```zig
const example: SomeStruct = .{
	.sortFn = (struct {
		pub fn sort(lhs: SomethingElse, rhs: SomethingElse) !void {
			// Whatever
		}
	}).sort,
};
```

## Interfaces
Many times I've encountered instances where a function parameter is `anytype` and yet is not checked but used immediately as if it's a very particular type. Take for example [this format function](https://github.com/nektro/zig-time/blob/e946a144423cdb5dac3d46d6856c6e6da73e9305/time.zig#L218) in the [zig-time](https://github.com/nektro/zig-time) library. Why is it like that? Well, because interfaces don't exist in Zig yet, so the language has imported some of JavaScript's "just treat it like it's the correct type anyway" issues. At least it's not anyopaque, so *some* level of analysis is possible, but there's no need for the language to *effectively* require type-coyness like this.

There are various ideas on how to fix this, I personally like [this suggestion](https://github.com/ziglang/zig/issues/17198#issuecomment-2533468501), but overall I'd just love a Rust-like trait-impl system.

## Explicit Runtime Execution
Zig has `comptime`, which allows you to force compile-time computation (with limitations, of course) where something would usually be done at runtime. I am asking for the opposite, for a way to force runtime-computation for something that would be done at compile time (with likely similar limitations, of course). For example:
```zig
// This is a top-level hash-map that has some initial values
// and can be further added to at runtime, like a registry.
const example = lazy createSomeHashMapThatRequiresAllocation();
```
This would mean that the "example" variable is known to exist and its type is know, and it's constant, but its value is only set upon first access.

## Better Array and Slice interoperability
*(Glossary: when I use "array" in quotes like that, know that I just mean a collection of elements of a known size. I do not [necessarily] mean Zig [arrays](https://ziglang.org/documentation/0.13.0/#Arrays) or [slices](https://ziglang.org/documentation/0.13.0/#Slices), which have their own specific meanings.)*

This has been the bane of my existence during 2024's Advent of Code: all I want to do is iterate through an "array" of u8's, or what have you. I have a string via `@embedFile("input.txt")` and just want to pass that into function to produce a grid, or what have you. Except it doesn't work because your parameter's `[]const u8` type doesn't match `*[38]const u8`. I know this is probably a skill issue, that there's some way around this... but why does there need to be? And why do they all look so similar? Can I please, for the love of all that is holy, just be able to pass the thing into the other thing and it just work? Is that really so much to ask?
