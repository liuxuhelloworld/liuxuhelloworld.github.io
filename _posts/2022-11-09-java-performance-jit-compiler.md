The JIT (just-in-time) compiler is the heart of the JVM, nothing controls the performance of your application more than the JIT compiler.

Computers, more specifically CPUs, can execute only a relatively few, specific instructions, which are called *machine code*. All programs that the CPU executes must therefore be translated into these instructions. Language like C++ and Fortran are called *compiled languages* because their programs are delivered as binary code: the program is written, and then a static compiler produces a binary. The assembly code in that binary is trageted to a particular CPU. Languages like PHP and Perl, on the other hand, are interpreted. The same program source code can be run on any CPU as long as the machine has the correct interpreter. The interpreter translates each line of the program into binary code as that line is executed. For a number of reasons, interpreted code will almost always be measurably slower than compiled code: compilers have enough information about the program to provide optimizations to the binary code that an interpreter simply cannot perform.

Java attempts to find a middle ground here. Java applications are compiled, but instead of being compiled info a specific binary for a specific CPU, they are compiled into an intermediate low-level language. This language (known as *Java bytecode*) is then run the **java** binary (in the same way that an interpreted PHP script is run by the **php** binary). Because it is executing an idealized binary code, the **java** program is able to compile the code into the platform binary as the code executes. This compilation occurs as the program is executed: it happens "just in time".

# JIT in HotSpot JVM

The name (HotSpot) comes from the approach it takes toward compiling the code. In a typical program, only a small subset of code is executed frequently, and the performance of an application depends primarily on how fast those sections of code are executed. These critical sections are known as the hot spots of the application; the more the section of code is executed, the hotter that section is said to be.

When the JVM executes code, it does not begin compiling the code immediately. First, if the code is going to be executed only once, then compiling it is essentially a wasted effort; it will be faster to interpret the Java bytecodes than to compile them and execute (only once) the compiled code. The second reason is one of optimization: the more times that the JVM executes a particular method or loop, the more information it has about the code. This allows the JVM to make numerous optimizations when it compiles the code.

# Client Compiler vs Server Compiler

Once upon a time, the JIT compiler came in two flavors, known as C1 compiler (*client* compiler) and C2 compiler (*server* compiler). The primary difference between the two compilers is their aggressiveness in compiling code. 

> 
> -client
> 
> -server
> 

Since Java 8, those flags don't do anything.

The C1 compiler begins compiling sooner than the C2 compiler does. This means that during the beginning of code execution, the C1 compiler will be faster, because it will have compiled correspondingly more code than the C2 compiler. 

The engineering trade-off here is the knowledge the C2 compiler gains while it waits: the knowledge allows the C2 compiler to make better optimizations in the compiled code. Ultimately, code produced by the C2 compiler will be faster than that produced by the C1 compiler.

Couldn't the JVM start with the C1 compiler and then use the C2 compiler as code gets hotter? That technique is known as *tiered compilation*, and it is the technique all JVMs now use.

# Tuning the Code Cache

When the JVM compiles code, it holds the set of assembly-language instructions in the code cache. The code cache has a fixed size, and once it has filled up, the JVM is not able to compile any additional code. It is easy to see the potential issue here if the code is too small. Some hot methods will get compiled, but others will not: the application will end up running a lot of (very slow) interpreted code.

>
> -XX:InitialCodeCacheSize=*N*
> 
> -XX:ReservedCodeCacheSize=*N*
> 

The initial size of the code cache is 2496 KB, and the default maximum size is 240 MB. There really isn't a good mechanism to figure out how much code cache a particular application needs. A typical option is to simply double or quadruple the default.

# Inspecting the Compilation Process

> 
> -XX:PrintCompilation
> 

This flag gives us visibility into the workings of the compiler. If *PrintCompilation* is enabled, every time a method (or loop) is compiled, the JVM prints out a line with information about what has just been compiled.

If the program was started without the -XX:PrintCompilation flag, you can also get limited visibility into the working of the compiler by using **jstat**.

```bash
jstat -compiler pid
```
The **-compiler** option supplies summary information about the number of methods compiled.

```bash
jstat -printcompilation pid 1000
```
You can use the **-printcompilation** option to get information about the last method that is compiled.

# OSR (on-stack replacement)

JIT compilation is an asynchronous process: when the JVM decides that a certain method should be compiled, that method is placed in a queue. Rather than wait for the compilation, the JVM then continues interpreting the method, and the next time the method is called, the JVM will execute the compiled version of the method.

For a long-running loop, when the code for the loop has finished compiling, the JVM replaces the code (on stack), and the next iteration of the loop will execute the much faster compiled version of the code. This is OSR.