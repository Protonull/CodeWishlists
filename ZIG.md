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
I do believe there's an issue open for this. I'll add a link to it when I can.
