
# Dart Wishlist
Just a list of various things that I wish Dart had.

## `orelse` 
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

## Explicit access modifiers
Dart unfortunately has a very Go-like mindset to access-modifiers: things are private in Go when the first-letter of its name is lowercase; things are private in Dart when the first-letter of its name is an underscore. And while Dart's is less objectionable because the distinction is more extreme, I still find this incredibly silly.

## Explicit function keyword
It's strange because Dart actually has this but only for *type* definitions, ie, `void Function(int)`, but when actually defining a function, there's no `Function`, `function`, `func`, `fn` (etc) prefix. And while this is just something to get used to, it is weird how similar functions and variables look.

## First-party build system
Dart has a compiler, obviously, and it has dependency management... but it has no build system... someone had to [make one](https://pub.dev/packages/build_runner).

## More integers!
Dart has only one integer type (`int`) which has a platform-dependent size, maxed out at 64bits. This is pretty shocking, but it's clearly an artefact of being so tied to JavaScript. Nor do unsigned integers exist. This is effectively like using JavaScript's `bigint` for everything, except that Dart doesn't even use `bigint` when compiling to JavaScript!

## Expanded standard library
Dart has an extremely, *extremely* slim standard library, to put it politely. Dart is the underlying language for [Flutter](https://flutter.dev/), a UI toolkit, so it's understandable that Dart is more limited, similarly to how Swift was written for iOS so its design is based around that. However, the mere act of hashing a value requires importing a third-party library. That is shocking.
