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

