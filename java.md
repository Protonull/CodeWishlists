# Java Wishlist
Just a list of various things that I wish Java had.

## Java-based Build System
Java has a number of available build tools, but these tools never use *Java* for its configuration: Gradle uses [Kotlin](https://kotlinlang.org/)/[Groovy](https://groovy-lang.org/), Ant and Maven use [XML](https://www.w3.org/XML/), and Bazel uses [Starlark](https://github.com/bazelbuild/starlark). And going back to javac more than likely means configuring your build in bash or make. It's like these tools are actively avoiding Java. This is in stark contrast to Zig where you configure your build *in Zig.* Hopefully [JEP 330](https://openjdk.org/jeps/330) and [JEP 458](https://openjdk.org/jeps/458) help move the needle here.

## Ergonomic Null Safety
While Java is pursing nullability as part of the type system itself with JEPs [8316779](https://openjdk.org/jeps/8316779) and [8303099](https://openjdk.org/jeps/8303099), Java nonetheless sorely lacks ergonomic ways of handling nulls, such as [optional chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining), [nullish coalescence](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing), and [nullish reassignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing_assignment). What is `Objects.toString(something, "default")` other than a clutter of bytecode because we are unable to simply do `something?.toString() ?? "default"`? The latter is not only more readable, in my opinion, but also more efficient since `"default"` isn't allocated until `something` is found to be null.

*Note: You actually can do a really cursed form of optional chaining using instanceof ternaries: https://www.youtube.com/watch?v=zg8xM0xxFa8&t=1837s.*

## Unreachable keyword
This could also be done as a premade exception class. Either way, this would serve as a way to handle the various times where the compiler demands exhaustiveness, but the exhaustive route(s) are known to be unreachable. For example:

```java
return switch (index % 2) {
    case 0 -> (screen.width / 2) - X_PADDING - widget.getWidth();
    case 1 -> (screen.width / 2) + X_PADDING;
    default -> unreachable;
};
```

## Switch-branch capturing
While Java has this to some extent with instanceof cases ([JEP 441](https://openjdk.org/jeps/441)), it isn't true for const cases or default cases, so obtaining the value being switched upon requires a predefined variable:
```java
final int side = index % 2;
return switch (side) {
    case 0 -> (screen.width / 2) - X_PADDING - widget.getWidth();
    case 1 -> (screen.width / 2) + X_PADDING;
    default -> throw new IllegalStateException("Invalid side: " + side);
};
```

Whereas, languages like Zig, like you capture values in context, which in Java would let you do:
```java
return switch (index % 2) {
    case 0 -> (screen.width / 2) - X_PADDING - widget.getWidth();
    case 1 -> (screen.width / 2) + X_PADDING;
    default |side| -> throw new IllegalStateException("Invalid side: " + side);
};
```

It does feel like Java is attempting to solve this with [JEP 488](https://openjdk.org/jeps/488), which would let you do:
```java
return switch (index % 2) {
    case 0 -> (screen.width / 2) - X_PADDING - widget.getWidth();
    case 1 -> (screen.width / 2) + X_PADDING;
    case final int side -> throw new IllegalStateException("Invalid side: " + side);
};
```
However, this has a weird "satisfies" side effect where the switch'd value could be a float that's also a valid int. I think it'd be better to just allow capturing, even for const cases like those `case 0` and `case 1`.

## Ergonomic Catching
Java's exception handling is very bulky and often unnecessary. For example:
```java
final MessageDigest md;
try {
    md = MessageDigest.getInstance("SHA-1");
}
catch (final NoSuchAlgorithmException impossible) {
    throw new IllegalStateException("This should never happen!", impossible);
}
```
The above will NEVER throw, or at least, should never throw on any JVM runtime that correctly implements the specification. And yet, there's no quick and easy way to dismiss catch-requirements that will never happen. As such, it would be fantastic if we could do:
```java
final MessageDigest md = MessageDigest.getInstance("SHA-1") catch unreachable;
```

## Extracting Primitives from Byte Arrays
You have a `byte[]` and want to extract an `int` at a particular offset (or set an `int` at a particular offset), this should be simple, right? Unfortunately not, as while there's an [internal static API](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/jdk/internal/util/ByteArray.java) for this, there's no public static API, so end users are required to use wrapper classes like `ByteBuffer` and call instance methods. This is wasteful. You already have the array and the offset of the integer, why does an object and all its internal-context objects need to be allocated for this one task? Genuinely, why?

## Yielding Values from Blocks
Java does have something similar to this with its ability to initialise variables separately from their definition:
```java
final String hash; {
    final MessageDigest md;
    try {
        md = MessageDigest.getInstance("SHA-1");
    }
    catch (final NoSuchAlgorithmException impossible) {
        throw new IllegalStateException("This should never happen!", impossible);
    }
    md.update(data);
    hash = HexFormat.of().formatHex(md.digest());
}
```

Additionally, JDK 22+ allows statements before super ([JEP 447](https://openjdk.org/jeps/447)). Both of these make the absence of yieldable blocks less painful.

Well, okay, so there *is* a way to do yieldable blocks, but it's *extremely* cursed:
```java
final String hash = switch (0) { default -> {
    final MessageDigest md;
    try {
        md = MessageDigest.getInstance("SHA-1");
    }
    catch (final NoSuchAlgorithmException impossible) {
        throw new IllegalStateException("This should never happen!", impossible);
    }
    md.update(data);
    yield HexFormat.of().formatHex(md.digest());
}};
```

It would be nice if there were a proper way to do this.

## Unsigned Integers
Duh. But I'd be remiss if this went unmentioned.

## Namespace Classes
These would be classes that only permit static methods and fields, akin to Lombok's [`@UtilityClass`](https://projectlombok.org/features/experimental/UtilityClass), eg:
```java
public namespace Shortcuts {
    public MessageDigest sha1() {
        try {
            return MessageDigest.getInstance("SHA-1");
        }
        catch (final NoSuchAlgorithmException impossible) {
            throw new IllegalStateException("This should never happen!", impossible);
        }
    }
}
```

## Cascade operator
Dart has this fantastic [cascade operator](https://dart.dev/language/operators#cascade-notation) which allows you to call methods or set fields of an object, and the return value will be the object regardless. This would allow you to do:
```java
final String hash = HexFormat.of().formatHex(
    Shortcuts.sha1()
        ..update(data)
        .digest()
);
```
This works since although `.update()` has a return value of void, this is ignored in favour of returning the MessageDigest instance, which then let's us call `.digest()`. The cascade operator can be used as many times as you want, which makes it excellent for builders.

## Immutable Arrays
Java is actually pursuing this with [JEP 8261007](https://openjdk.org/jeps/8261007).

## Tuples
While this can be achieved with records and other such classes, it would be nice if it were an ergonomic part of the language, where you could simply do:
```java
public [int, int] getPosition() {
    return [getX(), getY()];
} 
```

## Destructuring
Java is pursuing destructuring ([JEP 440](https://openjdk.org/jeps/440)), but it's currently limited to record instanceof patterns, as opposed to turning this:
```java
for (final var entry : map.entrySet()) {
    final var key = entry.getKey();
    final var value = entry.getValue();
}
```
into this:
```java
for (final [key, value] : map.entrySet()) {

}
```
