# JVM internals

Well, this is a starting point of a continuous journey into the splendid world of JVM. That is nothing but an attempt to
unite many resources, including articles, books, and videos, into some cohesive whole. I think that every software
engineer, and java developer in particular, should know how JVM works under the hood, since the knowledge of the latter
greatly aids in understanding the causes of errors. It might reduce the time spent on resolving bugs, increase a
productivity level by couple orders of magnitude, and finally fill one with an extreme pleasure of being mindful of such
complicated things.

## A bit of history

This section concerns a history of an Oracle's Hotstop JVM. The first release of Hotstop dates back to April 27, 1999.
Back then it had the name of "Java HotSpot Performance Engine", which, as wikipedia states, was built on technologies
from an implementation of the programming language Smalltalk named Strongtalk.

## A general structure

When talking about JVM, it makes sense to present its general structure. The digram below perfectly depicts the latter.

<img src="assets/JVM-Model.jpg" width="650" height="500">

The java code comes a long way before being executed on a host machine. The whole pipeline consists of many stages,
involving various transformations, optimizations, and some heuristics making a final program run faster. JVM maintains a
stack(or rather, stacks)
to which immediate values and references are pushed and popped from. Each thread has its own stack and pc(program
counter, or instruction pointer). Heap is shared among all threads, so it is quite important to provide a good thread
safety facility. JVM also has a native stack, which is kind of similar to usual java stack, but it is used to execute
native (non-Java) methods. Next section will walk you through the details of each part in JVM execution pipeline.

## Dissecting each part

Before being fed to the loading stage java code gets translated to .class file. For instance, let's take the following
Kotlin program

```kotlin
 package src

 fun main() {
     val dummy = 432
     localFunction(dummy)
 }

 fun localFunction(dummy: Int){
     println(dummy)
 }

```
Here's a corresponding .class file

```kotlin
// IntelliJ API Decompiler stub source generated from a class file
// Implementation of methods is not available

package src

public fun localFunction(dummy: kotlin.Int): kotlin.Unit { /* compiled code */ }

public fun main(): kotlin.Unit { /* compiled code */ }

```

As you can see, .class file defines components in a more explicit manner.


### Loading

Before diving into structure of a class loader, it makes sense to define what class loader does. Basically, the name
readily produces a valid definition, that is a class loader is responsible for loading classes at runtime. Thanks to
class loader, JVM does not have to bother about underlying complexity of a file system on the host machine. The process
of finding classes is based upon a delegation concept. It means that a child class will delegate searching to a parent
class prior to doing it on its own. <br>

[THOUGHTS] Not really sure why JVM does it. Is there a higher probability that needed class will be loaded by a parent
class?. I definitely should look that up.

The principle of delegation is not the only one the JVM class loader conforms to. The principles of visibility and
uniqueness also encompass a set of class loading laws. Visibility principle states that child class loader is allowed to
see all classes loaded by a parent class loader, but not vice versa, whereas the uniqueness principle states that class
is loaded only once. If class isn't found, NoClassDefFoundError or ClassNotFoundException is thrown. <br>

A loading facility has 3 divisions, namely Bootstrap Class Loader, Extension Class Loader, and System Class Loader (some
sources define also User-defined class loaders).

#### System Class loader

A System Class Loader is responsible for loading all classes residing in classpath. The command

```bash
    echo $CLASSPATH
```

will reveal what those classes are.

#### Extension Class Loader

An Extension Class Loader is obliged to load classes located under /jre/lib/ext directory.

#### Bootstrap(Primordial) Class Loader

This is a primary class loader, which is responsible for loading JDK internal classes (rt.jar), and other core libraries
located under $JAVA_HOME/jre/lib directory. For instance, it's responsible for loading ClassLoader class which is being
inherited by custom class loaders, and it actually makes sense, since ClassLoader is also a class, hence it needs to be
loaded.

Let's take a real example of a problem during class loading. For demonstration purposes, I've written a very simple
program in Java. Here's a source code

```java
public class Main {
    public static void main(String... args) {
        sayHello();
    }

    public static void sayHello() {
        System.out.println("Hello, Pal");
    }
}
```

It has successfully produced the following Main.class file

```java

// IntelliJ API Decompiler stub source generated from a class file
// Implementation of methods is not available

public class Main {
    public Main() { /* compiled code */ }

    public static void main(java.lang.String... args) { /* compiled code */ }

    public static void sayHello() { /* compiled code */ }
}
```

Here comes an interesting part.

```bash
  java Main
```

Execution of command above yields the following result

```bash
Error: Could not find or load main class Main
Caused by: java.lang.ClassNotFoundException: Main
```

However, once I add an additional parameter, things become better

```bash
glasser@threadedstream:~/toys/presentation$ java -cp out/production/presentation/ Main 
Hello, Pal
glasser@threadedstream:~/toys/presentation$ 
```

The cause of a problem hides in the fact that class loader doesn't know about a location of Main.class file, so one
 has to provide a location. 

[THOUGHTS] I'm pretty sure that class is loaded by System Class Loader. 
Does JVM implicitly assume that class should be loaded only by System Class Loader? 
Does delegation principle still hold?  


### Linking

Similar to loading, the linking stage's also comprised of 3 parts: Verifying, Preparing, and Resolving. The process of
verifying consists of checking a class or interface against a presence of semantic issues.

## Bytecode, Garbage Collection, and Memory Management
If you've ever written something in compiled language (C, C++), you already know that written code should first be translated into its 
lower level equivalent, i.e machine(object) code. The latter is what processor understands and hence is able to execute.
JVM, on the other hand, works a bit differently. It relies on the notion of intermediate
representation called bytecode. The generated bytecode is interpreted by an interpreter and executed
on the host machine. However, sometimes Java code is compiled right into machine code.
Not all code is compiled, though. Instead, JVM will choose the hottest parts, which are executed most often, and 
compile them into a native machine code.  


### Heap structure

JVM's heap consists of young and old generations. Young generation is further divided into 
3 parts:
 - Eden space
 - Survivor space 'From'
 - Survivor space 'To'

Eden space is designed to hold freshly allocated objects (initialized with new), whereas Survivor space 
holds objects not reclaimed during garbage collection in Eden space. 
An old generation holds objects, lifetime of which has hit a certain threshold.
A visual below greatly depicts the structure of JVM heap.
<img src="assets/jvm_heap_java8.png" width="600px" height="300px">
You might as well noticed a space I haven't told about - the Metaspace (actually, it's not a part of the heap).
Formerly known as a Permanent generation space (PermGen), the Metaspace has brought around many decent 
features. 

### Permanent Generation Space
Permanent Generation Space was designed to hold all loaded class's metadata. 


## Optimizations, benchmarks

## Implementations

## Conclusion

## References

Articles

- https://en.wikipedia.org/wiki/Java_virtual_machine
- https://blog.jamesdbloom.com/JVMInternals.html
- https://dzone.com/articles/understanding-jvm-internals
- https://www.developer.com/design/understand-jvm-loading-jvm-linking-and-jvm-initialization/
- https://dzone.com/articles/demystify-java-class-loading

Videos
- https://www.youtube.com/watch?v=UwB0OSmkOtQ (Oldie, but very useful)


I'll be filling up these as i learn something new. So, that's it for now :)
