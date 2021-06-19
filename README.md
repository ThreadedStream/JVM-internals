# JVM internals

Well, this is a starting point of a continuous journey into the splendid world of JVM. That is nothing but an attempt to unite many resources, including articles,
books, and videos, into some cohesive whole. I think that every software engineer, and java developer in particular, 
should know how JVM works under the hood, since the knowledge of the latter greatly aids in understanding the causes of errors. 
It might reduce the time spent on resolving bugs, increase a productivity level by couple orders 
of magnitude, and finally fill one with an extreme pleasure of being mindful of such complicated things. 


## A bit of history
This section concerns a history of an Oracle's Hotstop JVM. The first release of Hotstop dates back to April 27, 1999. Back then it had the name of "Java HotSpot Performance Engine", which, as wikipedia states, was built on technologies from an implementation of the programming language Smalltalk named Strongtalk.

## A general structure
When talking about JVM, it makes sense to present its general structure. The digram below
perfectly depicts the latter.

<img src="assets/JVM-Model.jpg" width="900" height="700">

The java code comes a long way before being executed on a host machine. The whole 
pipeline consists of many stages, involving various transformations, optimizations, and some 
heuristics making a final program run faster. JVM maintains a stack(or rather, stacks)
to which immediate values and references are pushed and popped from. Each thread has its own
stack and pc(program counter, or instruction pointer). Heap is 
shared among all threads, so it is quite important to provide a good thread safety facility.
JVM also has a native stack, which is kind of similar to usual java stack, but it is 
used to execute native (non-Java) methods. Next section will walk you through
the details of each part in JVM execution pipeline.

## Dissecting each part
Before being fed to the loading stage java code gets translated to 
.class file. For instance, let's take the following Kotlin program

TODO: Include example images

### Loading
Before diving into structure of a class loader, it makes sense to 
define what class loader does. Basically, the name readily produces a valid
definition, that is a class loader is responsible for loading classes at runtime.
Thanks to class loader, JVM does not have to bother about underlying complexity 
of a file system on the host machine. The process of finding classes is based upon a delegation concept.
It means that a child class will delegate searching to a parent class prior
 to doing it on its own. <br>

[NOTE] Not really sure why JVM does it. Is there a higher probability
that needed class will be loaded by a parent class?. I definitely should look that up. 

The principle of delegation is not the only one the JVM class loader conforms to.
The principles of visibility and uniqueness also encompass a set of 
class loading laws. Visibility principle states that child class loader is allowed to see 
all classes loaded by a parent class loader, but not vice versa, whereas the uniqueness
principle states that class is loaded only once. If class isn't found, NoClassDefFoundError or ClassNotFoundException is 
thrown. <br>

A loading facility has 3 divisions, namely Bootstrap Class Loader, Extension Class Loader, and 
System Class Loader (some sources define also User-defined class loaders). 

#### System Class loader
A System Class Loader is responsible for loading all classes residing in classpath.
The command
```bash
    echo $CLASSPATH
```
will reveal what those classes are. 

#### Extension Class Loader
An Extension Class Loader is obliged to load classes located under 
/jre/lib/ext directory. 

#### Bootstrap(Primordial) Class Loader
This is a primary class loader, which is responsible for loading 
JDK internal classes (rt.jar), and other core libraries located under
$JAVA_HOME/jre/lib directory. For instance, it's responsible for loading 
ClassLoader class which is being inherited by custom class loaders, and it actually
makes sense, since ClassLoader is also a class, hence it needs to be loaded. 

### Linking
Similar to loading, the linking stage's also comprised of 3 parts: Verifying, Preparing, and Resolving.
The process of verifying consists of checking a class or interface against
a presence of semantic issues.

## Bytecode, Garbage Collection, and Memory Management

## Optimizations, benchmarks

## Implementations

## Conclusions

## References

Articles
- https://en.wikipedia.org/wiki/Java_virtual_machine
- https://blog.jamesdbloom.com/JVMInternals.html
- https://dzone.com/articles/understanding-jvm-internals
- https://www.developer.com/design/understand-jvm-loading-jvm-linking-and-jvm-initialization/
- https://dzone.com/articles/demystify-java-class-loading

Videos

Coming soon...

I'll be filling up these as i learn something new. So, that's it for now :)
