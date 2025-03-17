### [Introduction](#introduction)

_Java Singleton Pattern_ is one of the [_Gangs of Four Design patterns_](https://www.digitalocean.com/community/tutorials/gangs-of-four-gof-design-patterns) and comes in the _Creational Design Pattern_ category. From the definition, it seems to be a straightforward design pattern, but when it comes to implementation, it comes with a lot of concerns.

In this article, we will learn about singleton design pattern principles, explore different ways to implement the singleton design pattern, and some of the best practices for its usage.

## [Singleton Pattern Principles](#singleton-pattern-principles)

- Singleton pattern restricts the instantiation of a class and ensures that only one instance of the class exists in the Java Virtual Machine.
- The singleton class must provide a global access point to get the instance of the class.
- Singleton pattern is used for [logging](https://www.digitalocean.com/community/tutorials/logger-in-java-logging-example), drivers objects, caching, and [thread pool](https://www.digitalocean.com/community/tutorials/threadpoolexecutor-java-thread-pool-example-executorservice).
- Singleton design pattern is also used in other design patterns like [Abstract Factory](https://www.digitalocean.com/community/tutorials/abstract-factory-design-pattern-in-java), [Builder](https://www.digitalocean.com/community/tutorials/builder-design-pattern-in-java), [Prototype](https://www.digitalocean.com/community/tutorials/prototype-design-pattern-in-java), [Facade](https://www.digitalocean.com/community/tutorials/facade-design-pattern-in-java), etc.
- Singleton design pattern is used in core Java classes also (for example, `java.lang.Runtime`, `java.awt.Desktop`).

## [Java Singleton Pattern Implementation](#java-singleton-pattern-implementation)

To implement a singleton pattern, we have different approaches, but all of them have the following common concepts.

- Private constructor to restrict instantiation of the class from other classes.
- Private static variable of the same class that is the only instance of the class.
- Public static method that returns the instance of the class, this is the global access point for the outer world to get the instance of the singleton class.

In further sections, we will learn different approaches to singleton pattern implementation and design concerns with the implementation.

## [1. Eager initialization](#1-eager-initialization)

In eager initialization, the instance of the singleton class is created at the time of class loading. The drawback to eager initialization is that the method is created even though the client application might not be using it. Here is the implementation of the static initialization singleton class:

If your singleton class is not using a lot of resources, this is the approach to use. But in most of the scenarios, singleton classes are created for resources such as File System, Database connections, etc. We should avoid the instantiation unless the client calls the `getInstance` method. Also, this method doesn’t provide any options for exception handling.

## [2. Static block initialization](#2-static-block-initialization)

[Static block](https://www.digitalocean.com/community/tutorials/static-keyword-in-java) initialization implementation is similar to eager initialization, except that instance of the class is created in the static block that provides the option for [exception handling](https://www.digitalocean.com/community/tutorials/exception-handling-in-java "Java Exception Handling Tutorial with Examples and Best Practices").

Both eager initialization and static block initialization create the instance even before it’s being used and that is not the best practice to use.

## [3. Lazy Initialization](#3-lazy-initialization)

Lazy initialization method to implement the singleton pattern creates the instance in the global access method. Here is the sample code for creating the singleton class with this approach:

The preceding implementation works fine in the case of the single-threaded environment, but when it comes to multi-threaded systems, it can cause issues if multiple threads are inside the `if` condition at the same time. It will destroy the singleton pattern and both threads will get different instances of the singleton class. In the next section, we will see different ways to create a [thread-safe](https://www.digitalocean.com/community/tutorials/thread-safety-in-java) singleton class.

## [4. Thread Safe Singleton](#4-thread-safe-singleton)

A simple way to create a thread-safe singleton class is to make the global access method [synchronized](https://www.digitalocean.com/community/tutorials/thread-safety-in-java "Java Synchronization and Thread Safety Tutorial with Examples") so that only one thread can execute this method at a time. Here is a general implementation of this approach:

The preceding implementation works fine and provides thread-safety, but it reduces the performance because of the cost associated with the synchronized method, although we need it only for the first few threads that might create separate instances. To avoid this extra overhead every time, _double-checked locking_ principle is used. In this approach, the synchronized block is used inside the `if` condition with an additional check to ensure that only one instance of a singleton class is created. The following code snippet provides the double-checked locking implementation:

Continue your learning with [Thread Safe Singleton Class](https://www.digitalocean.com/community/tutorials/thread-safety-in-java-singleton-classes).

## [5. Bill Pugh Singleton Implementation](#5-bill-pugh-singleton-implementation)

Prior to Java 5, the Java memory model had a lot of issues, and the previous approaches used to fail in certain scenarios where too many threads tried to get the instance of the singleton class simultaneously. So [Bill Pugh](https://en.wikipedia.org/wiki/William_Pugh_\(computer_scientist\)) came up with a different approach to create the singleton class using an [inner static helper class](https://www.digitalocean.com/community/tutorials/java-inner-class). Here is an example of the Bill Pugh Singleton implementation:

Notice the _private inner static class_ that contains the instance of the singleton class. When the singleton class is loaded, `SingletonHelper` class is not loaded into memory and only when someone calls the `getInstance()` method, this class gets loaded and creates the singleton class instance. This is the most widely used approach for the singleton class as it doesn’t require synchronization.

## [6. Using Reflection to destroy Singleton Pattern](#6-using-reflection-to-destroy-singleton-pattern)

Reflection can be used to destroy all the previous singleton implementation approaches. Here is an example class:

When you run the preceding test class, you will notice that `hashCode` of both instances is not the same which destroys the singleton pattern. Reflection is very powerful and used in a lot of frameworks like Spring and Hibernate. Continue your learning with [Java Reflection Tutorial](https://www.digitalocean.com/community/tutorials/java-reflection-example-tutorial "Java Reflection Tutorial for Classes, Methods, Fields, Constructors, Annotations and much more").

## [7. Enum Singleton](#7-enum-singleton)

To overcome this situation with Reflection, [Joshua Bloch](https://en.wikipedia.org/wiki/Joshua_Bloch) suggests the use of `enum` to implement the singleton design pattern as Java ensures that any `enum` value is instantiated only once in a Java program. Since [Java Enum](https://www.digitalocean.com/community/tutorials/java-enum "Java Enum Examples with Benefits and Class usage") values are globally accessible, so is the singleton. The drawback is that the `enum` type is somewhat inflexible (for example, it does not allow lazy initialization).

## [8. Serialization and Singleton](#8-serialization-and-singleton)

Sometimes in distributed systems, we need to implement `Serializable` interface in the singleton class so that we can store its state in the file system and retrieve it at a later point in time. Here is a small singleton class that implements `Serializable` interface also:

The problem with serialized singleton class is that whenever we deserialize it, it will create a new instance of the class. Here is an example:

That code produces this output:

```
OutputinstanceOne hashCode=2011117821
instanceTwo hashCode=109647522
```

So it destroys the singleton pattern. To overcome this scenario, all we need to do is provide the implementation of `readResolve()` method.

After this, you will notice that `hashCode` of both instances is the same in the test program.

Read about [Java Serialization](https://www.digitalocean.com/community/tutorials/objectoutputstream-java-write-object-file) and [Java Deserialization](https://www.digitalocean.com/community/tutorials/serialization-in-java).

## [Conclusion](#conclusion)

This article covered the singleton design pattern.

Continue your learning with more [Java tutorials](https://www.digitalocean.com/community/tags/java).