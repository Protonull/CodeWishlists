# Java Wishlist
Just a list of various things that I wish Java had.

## Java-based Build System
Java has a number of available build tools, but these tools never use *Java* for its configuration: Gradle uses [Kotlin](https://kotlinlang.org/)/[Groovy](https://groovy-lang.org/), Ant and Maven use [XML](https://www.w3.org/XML/), and Bazel uses [Starlark](https://github.com/bazelbuild/starlark). And going back to javac more than likely means configuring your build in bash or make. It's like these tools are actively avoiding Java. This is in stark contrast to Zig where you configure your build *in Zig.* Hopefully [JEP 330](https://openjdk.org/jeps/330) and [JEP 458](https://openjdk.org/jeps/458) help move the needle here.

## Ergonomic Null Safety
While Java is pursing nullability as part of the type system itself with JEPs [8316779](https://openjdk.org/jeps/8316779) and [8303099](https://openjdk.org/jeps/8303099), Java nonetheless sorely lacks ergonomic ways of handling nulls, such as [optional chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining), [nullish coalescence](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing), and [nullish reassignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing_assignment). What is `Objects.toString(something, "default")` other than a clutter of bytecode because we are unable to simply do `something?.toString() ?? "default"`? The latter is not only more readable, in my opinion, but also more efficient since `"default"` isn't allocated until `something` is found to be null.

## Ergonomic Catching
Java's exception handling is bulky and annoying. For example, `MessageDigest.getInstance("SHA-1")` will never fail as it's a required algorithm for all JVM implementations, and yet you must surround it with a try-catch for the `NoSuchAlgorithmException` exception. This, again, just means more bytecode clutter. Imagine if Java added Zig-style catching, letting you do:
```java
final MessageDigest md = MessageDigest.getInstance("SHA-1") catch unreachable;
```
or
```java
new Thread(() -> {
	while (Thread.interrupted()) {
		Thread.sleep(1000) catch break;
	}
});
```

## Extracting Primitives from Byte Arrays
You have a `byte[]` and want to extract an `int` at a particular offset (or set an `int` at a particular offset), this should be simple, right? Unfortunately not, as while there's a [internal static API](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/jdk/internal/util/ByteArray.java) for this, there's no public static API, so end users are required to use wrapper classes like `ByteBuffer` and call instance methods. This is wasteful. You already have the array and the offset of the integer, why does an object and all its internal-context objects need to be allocated for this one task? Genuinely, why?

## Yielding Values from Blocks
Java has [relevant to this discussion] static blocks, initialisation blocks, and anonymous blocks, but you are unable to yield values from these blocks. That said, Java *does* let you do something somewhat equivalent, such as:
```java
final String hash; {
	final byte[] data; {
		final var stream = new ByteArrayOutputStream(); {
			final var out = new DataOutputStream(stream);
			// put whatever you want into `out`
		}
		data = stream.toByteArray();
	}
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
This is taking advantage of Java's ability to separate a variable's definition and initialisation, which is great. But there are situations where you cannot do this, like passing values into the super-constructor. Java 22 has allowed statements before super ([JEP 447](https://openjdk.org/jeps/447)), but nonetheless it would be nice to allow blocks to yield values when used as an expression. 

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
