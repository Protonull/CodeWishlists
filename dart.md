# Dart Wishlist
Just a list of various things that I wish Dart had.

## Expressions after `??`
Zig has this wonderful `orelse` keyword that you can use inline. It's very similar to Dart's ["if-null" operator](https://dart.dev/language/operators) (??), however, Zig allows you to put return, break, continue, etc, after. I came across this issue during 2024's Advent of Code in one of the grid-based trails.
```dart
for (final direction in CARDINAL_DIRECTIONS) {
    final neighbourCell = grid.getNeighbouringCell(currentPos, direction.asVector());
    if (neighbourCell == null) {
        continue;
    }
    final (neighbourValue, neighbourPos) = neighbourCell;
}
```
The `getNeighbouringCell` method returns `null` if the cell does not exist (like being out of bounds), or otherwise returns a tuple of the neighbour's value and position. I would absolutely love being able to do:
```dart
for (final direction in CARDINAL_DIRECTIONS) {
    final (neighbourValue, neighbourPos) = grid.getNeighbouringCell(currentPos, direction.asVector()) ?? continue;
}
```
This reduces the clutter of variables in scope and reduces lines of clutter-code.

## Destructuring tuple parameters
Another grid-related frustration, but given how tied Dart is to JavaScript, how tuples are native to Dart, and how JavaScript permits this, that this isn't possible yet:
```dart
typedef Pos = (int, int);

// Why is this necessary?
void example(Pos pos) {
    final (x, y) = pos;
}

// Why can't I do this?!
void example(Pos (x, y)) {

}
```

## Yeild Blocks
While Dart does have technically have this, it's in a very JavaScript-y, over the top way by defining and immediately invoking an anonymous function:
```dart
final example = (() {
	final raw = "364";
	return int.parse(raw);
})();
```
whereas I advocate for something like the following:
```dart
final example = produce {
	final raw = "364";
	yield int.parse(raw);
};
```
The `produce` keyword is optional.

## Explicit access modifiers
Dart unfortunately has a very Go-like mindset to access-modifiers: things are private in Go when the first-letter of its name is lowercase; things are private in Dart when the first-letter of its name is an underscore. And while Dart's is less objectionable because the distinction is more extreme, I still find this incredibly silly.

## Explicit function keyword
It's strange because Dart actually has this but only for *type* definitions, ie, `void Function(int)`, but when actually defining a function, there's no `Function`, `function`, `func`, `fn` (etc) prefix. And while this is just something to get used to, it is weird how similar functions and variables can look.

## Build system as code
Dart comes with a compiler and dependency management but no build system, which is odd, right? What's confusing is that, if you set up your dependency management, you *can* just run `dart run`, and it will find your programme's entrypoint based on its name... yet you cannot manually specify one. Dart does have a [build system](https://pub.dev/packages/build_runner), but it's a dev-dependency, nor is it for the feint of heart. I would actually rather have a [Zig-style build-system-as-code](https://ziglang.org/learn/build-system/#getting-started) than all the yaml build\_runner requires.

## Way more integer types
Dart has only one integer type (`int`) which has a platform-dependent size, maxed out at 64-bits. There was [an issue](https://github.com/dart-lang/sdk/issues/52250) raised about this, but it was closed as unplanned. Nonetheless, it's clearly an artefact of being so tied to JavaScript. This has interesting side-effects, like there being no equivalent to Zig's `@sizeOf(i16)` or even Java's `Short.BYTES`, so you're operating blindly. Can this 64-bit value fit? I guess you just have to try and find out.

## Ergonomic Catching
Dart's exception handling is, for the most part, bulky and annoying. Imagine if you could do the following:
```dart
final int example = int.parse("123") catch 0;
```
You would then not need this anymore:
```dart
final int example = int.tryParse("123") ?? 0;
```

## Expanded standard library
Dart has an extremely, *extremely* slim standard library, to put it politely. Dart is the underlying language for [Flutter](https://flutter.dev/), a UI toolkit, so it's understandable that Dart is more limited, similarly to how Swift was written for iOS so its design is based around that. However, the mere act of hashing a value requires importing a third-party library. That is shocking.
