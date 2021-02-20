# Warmup

### Java in Android

#### What version of Java is supported in Android?

That is unclear. Java is a complex set of language features, APIs, bytecode etc. and Android never contained the full set of these.

Android Studio allows you to specify language level like so:

```groovy
android {
  ...
  compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
  }
}
```

This however doesn't bring you all APIs of that Java version, just the glitter of syntactic sugar in specified version's language features. Underneath it compiles down to either Java Virtual Machine version 6 or version 7 \(Android API 26\) bytecode.

Program written in Java is compiled to bytecode compatible with JVM, which is translated to Dalvik bytecode. On versions prior to 5 Lollipop this code is ran by Dalvik VM. Android 4.4 Kitkat introduces experimental support for Android Runtime \(ART\).

ART features Ahead of Time \(AOT\) compilation, which translates Dalvik bytecode to machine's native code during the time of app's installation.

**Want to know more?**

* [https://jakewharton.com/androids-java-8-support/](https://jakewharton.com/androids-java-8-support/)

#### Hungarian Notation

Private fields are prefixed with _m_. Static fields are prefixed with _s_. Constants - static final fields - are ALL\_CAPS. Other variables are in camelCase.

```java
public class Something {
   private Object mObject;
   private static String sString;
   private static final String CONSTANT = "constant";
   public Object otherObject;

   public void setObject(Object object) {
      mObject = object;
   }

   public Object getObject() {
      return mObject;
   }
}
```

### Kotlin in Android

[Kotlin](https://kotlinlang.org) is a modern language developed by JetBrains. It compiles down to Java bytecode \(among others\), so it can be run on existing platforms. In 2017, Kotlin was introduced as alternative language to Java on Android. In 2019, development on Android became oriented as Kotlin first.

#### Kotlin Basics

Kotlin's syntax is similar to Java. Since it shares the same foundations, you can use classes from Java SDK, so the transition from Java to Kotlin shouldn't be difficult. But it doesn't end there, the general interoperability between Java and Kotlin is possible, so you can mix both in one project or import Java libraries into Kotlin code and vice versa.

**Objects Everywhere**

Object is a basic data type. Kotlin doesn't use primitive data types like Java. It introduces own data types for Java's equivalent primitive types: `Boolean`, `Char`, `Byte`, `Short`, `Int`, `Long`, `Float`, and `Double`.

**Mutable and Immutable Variable Types**

Variable declaration can be final - `val`, or its value can be reassigned multiple times - `var`.

```kotlin
var count: Int = 10
count = 15
```

```kotlin
val count: Int = 10
// fails to compile
count = 15
```

**Type Inference**

The Kotlin compiler can infer variable's type base on its value. Explicit type declaration can be omitted.

```kotlin
val count = 10
```

**Nullability**

Since most of runtime crashes in Java are caused by `java.lang.NullPointerException`, furthermore, constant null checks worsen code readability and increase complexity, declarations in Kotlin are implicitly not nullable. You can, however declare a variable as nullable by using `?` symbol, mainly for compatibility reasons.

```kotlin
val count: Int? = null
```

If you use nullable types \(for whatever reason\), you still need to check for null values. Kotlin provides special operators to deal with null checks.

_Safe Call Operator_ `?.`

```kotlin
nullableVariable?.someMethodCall()
```

`!!` _Operator_

If you are sure the variable isn't null you can access it directly with the `!!` operator. As you can see it's represented by 2 exclamation marks, so this tells you, that you are doing something potentially dangerous, because if this variable you are accessing isn't null, you'll see the good old `NullPointerException`.

_Elvis Operator_ `?:`

You may want to use a nullable value in assignment like so:

```kotlin
val b: Int = if (a != null) a.length else -1
```

> Side note: Kotlin doesn't have a ternary operator, it can leverage direct assignment from a conditional expression instead.

The expression above is too verbose, so it can be shortened using the Elvis operator:

```kotlin
val b = a?.length ?: -1
```

**Class Constructors**

A simple class definition can be as short as:

```kotlin
class Person(val firstName: String, val lastName: String) {
    // class body
}
```

The above code depicts so called primary constructor, which is the most common way of defining a constructor.

If you need a place to handle class' properties in the process of object creation, you can do that in an init block:

```kotlin
class Person(val firstName: String, val lastName: String) {
    init {
        println("Person's name is $firstName $lastName")
    }
}
```

_Secondary Constructor_

If you need to define multiple constructors, you can do it like so:

```kotlin
class Person(val firstName: String, val lastName: String) {
    
    // secondary constructor
    constructor(val firstName: String, val lastName: String, val age: Int) {
        // code
    }
}
```

**Class Properties**

One of the key concepts used in Java is _encapsulation_ - the principle when the internal behavior and properties are hidden from outside of the object. The usual scenario was to have private properties accessed by getters and setters.

Kotlin's properties have provided getters and setters by default, without the need of defining them manually. They can be modified if needed.

```kotlin
public var count: Int = 10 // property is visible everywhere
    private set // setter is visible only from within its class
```

Default ****visibility modifier of a property is `public`, thus can be omitted.

```kotlin
var count = 10 // property is visible everywhere
    private set // setter is visible only from within its class
```

**Functions Are Fun**

Functions in Kotlin are declared using the `fun` keyword:

```kotlin
fun double(x: Int): Int {
    return 2 * x
}

fun printHello() {
    println("Hello World!")
}
```

Functions can have default argument values assigned to them:

```kotlin
fun double(x: Int = 10): Int {
    return 2 * x
}
```

**Want to know more?**

* [https://www.youtube.com/hashtag/kotlinvocabulary](https://www.youtube.com/hashtag/kotlinvocabulary)

