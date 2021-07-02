# What is a JVM? (here we go again)
JVM (stands for Java Virtual Machine) is an abstract machine which executes a bytecode produced a frontend compiler.
All it does is interpretation of .class files.

# General difference between VM and emulator

# Just a tiny bit of history (Nobody cares)


# General structure of JVM
Briefly describe each part

# More about a class loader
## What is a class loader
## Types of class loaders

# More about memory, garbage collection, and bytecode (actually, the hottest part of the talk)

# Various implementations of jvm
Dalvik is not a JVM, actually (lol). It's being replaced by ART.
DVM (Dalvik virtual machine) is register-based, as opposed to JVM's stack-based nature



Thoughts section is kind of set aside, here i scribble some interesting things i happened to learn during digging into JVM internals
# Thoughts
JVM is a stack-based machine. The only fact that it has a stack does not make it a stack-based (every language (i hope) has a stack), but it uses some
internal data structure (which is a stack) to hold some temporary values, evaluate expressions, etc... As an counter-example, if you look into intel(or whatever) assembly
you'll happen to notice that it uses registers for holding those temporary values, all math is done using registers.

JVM interpreter

A very simple JVM interpreter would interpret code in that fashion

```
  void execute(byte[] instructions){
    // initialize an evaluation stack
    EvalStack evalStack = new EvalStack();
    int pc = 0;
    while (true){
      byte currIns = instructions[pc++]
      Opcode op = table[b & 0xFF]
      switch(op){
        case IADD:
          // Take two values from stack, perform an add operation, and push result back onto the stack
          evalStack.doAdd()
      }
    }
  }
```     

Hotspot does a drastically different thing

Template interpreter, dynamic dispatch table at vm start, using ASM native code, calls back to vm for complex operations(constant pool lookup)

# Compiler theory a little bit

## Various optimizations

Escape analysis aids in proper placement of objects. It solves a conundrum of whether
to put an object in heap or onto stack.

Closure analysis is responsible to analyzing internal functions (functions in functions). Not present in Java, btw

Type checking to avoid things like assigning string to float, or even int to float, or vice versa. The latter is important for languages with a very strict typing system, i.e Golang, Rust (which is even stricter).

Static single assignment - minimizes a number of assignment by labeling variables.
For example:
```
      x = y + 1
      z = y + 5
      x = z - y
```
becomes   
```
      x_1 = y_1 + 1
      z_1 = y_1 + 5
      x_2 = z_1 - y_1
```

Common subexpression elimination - computes something once and stores result somewhere for further usage

```java
  public class CSE {
    public static void example(){
      int y = x + 5;
      int z = x + 5;
    }
  }
```

becomes

```java
public class CSE {
  public static void example(){
    int y = x + 5;
    int z = y;
  }
}
```

Dead code elimination - self-explanatory (should elaborate on it anyway)

Dead store elimination - prenons un exemple

```go
  var value int // Assigned to zero (dead store)

  value = 5 // rewriting a value

  // The goal of optimization is to avoid unnecessary stores  
```


# References

https://www.youtube.com/watch?v=7ECbwgkHdAE - Cool talk on implementation of a simple JVM in Rust

https://www.youtube.com/watch?v=uTMvKVma5ms - Talk on SSA (for golang dudes, though)
