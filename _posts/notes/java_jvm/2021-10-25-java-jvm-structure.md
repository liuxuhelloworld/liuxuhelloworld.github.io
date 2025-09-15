# The class File Format
Compiled code to be executed by the JVM is represented using a hardware- and operating system-independent binary format, typically stored in a file, known as the **class** file format.

# Data Types
Like the Java programming language, the JVM operates on two kinds of types: *primitive types* and *reference types*.

The JVM expects that nearly all type checking is done prior to run time, typically by a compiler, and does not have to be done by the JVM itself.

The instruction set of the JVM distinguishes its operand types using instructions intended to operate on values of specific types. For instance, *iadd*, *ladd*, *fadd*, *dadd* are all JVM instructions that add two numeric values and produces numeric results, but each is specialized for its operand type: **int**, **long**, **float**, and **double**, respectively.

The JVM contains explicit support for objects. An object is either a dynamically allocated class instance or an array. A reference to an object is considered to have JVM type **reference**. Values of type **reference** can be thought of as pointers to objects. More than one reference to an object may exist. Objects are always operated on, passed, and tested via values of type **reference**.

# Primitive Types and Values
The primitive data types supported by the JVM are the *numeric types*, the **boolean** type, and the **returnAddress** type.

## numeric types
The numeric types consist of the *integral types* and the *floating-point types*.

The integral types are:
- **byte**
- **short**
- **int**
- **long**
- **char**, whose values are 16-bit unsigned integers representing Unicode code points in the Basic Multilingual Plane, encoded with UTF-16, and whose default value is the null code point ('\u0000')

The floating-point types are:
- **float**
- **double**

## boolean type
Although the JVM defines a **boolean** type, it only provides very limited support for it. There are no JVM instructions solely dedicated to operations on **boolean** values. Instead, expressions in the Java programming language that operate on **boolean** values are compiled to use values of the JVM **int** data type.

The JVM does directly support **boolean** arrays. Its *newarray* instruction (*§newarray*) enables creation of **boolean** arrays. Arrays of type **boolean** are accessed and modified using the **byte** array instructions *baload* and *bastore* (*§baload*, *§bastore*) .

The JVM encodes **boolean** array components using 1 to represent **true** and 0 to represent **false**. Where Java programming language **boolean** values are mapped by compilers to values of JVM type **int**, the compilers must use the same encoding.

## returnAddress type
The **returenAddress** type is used by the JVM's *jsr*, *ret*, and *jsr_w* instructions (*§jsr*, *§ret*, *§jsr_w*).

The values of the **returnAddress** type are pointers to the opcodes of JVM instructions. Of the primitive types, only the **returnAddress** type is not directly associated with a Java programming language type.

# Reference Types and Values
There are three kinds of **reference** types: class types, array types, and interface types. Their values are references to dynamically created class instances, arrays, or class instances or arrays that implement interfaces, respectively.

The element type of an array type is necessarily either a primitive type, or a class type, or an interface type.

# Run-Time Data Areas
The JVM defines various run-time data areas that are used during execution of a program. Some of these data areas are created on JVM start-up and are destroyed only when the JVM exits. Other data areas are per thread. Per-thread data areas are created when a thread is created and destroyed when the thread exits.

## the pc register
The JVM can support many threads of execution at once. Each JVM thread has its own **pc** (program counter) register.

The JVM's **pc** register is wide enough to hold a **returnAddress** or a native pointer on the specific platform.

## JVM stack
Each JVM thread has a private *JVM stack*, created at the same time as the thread.  

A JVM stack stores frames. A JVM stack is analogous to the stack of a conventional language such as C: it holds local variables and partial results, and plays a part in method invocation and return.

Because the JVM stack is never manipulated directly except to push and pop frames, frames may be heap allocated. The memory for a JVM stack does not need to be contiguous.

This specification permits JVM stacks either to be of a fixed size or to dynamically expand and constract as required by the computation.

## heap
The JVM has a *heap* that is shared among all JVM threads. The heap is the run-time data area from which memory for all class instances and arrays is allocated.

The heap is created on JVM start-up. 

Heap storage for objects is reclaimed by an automatic storage management system (known as a *garbage collector*); objects are never explicitly deallocated. The JVM assumes no particular type of automatic storage management system, and the storage management technique may be chosen according to the implementor's system requirements.

The heap may be of a fixed size or may be expanded as required by the computation and may be contracted if a larger heap becomes unnecessary. The memory for the heap does not need to be contiguous.

## method area
The JVM has a *method area* that is shared among all JVM threads. The method area is analogous to the storage area for compiled code of a conventional language or analogous to the "text" segment in an operating system process. It stores per-class structures such as the run-time constant pool, field and method data, and the code for methods and constructors, including the special methods used in class and interface initialization and in instance initialization.

The method area is created on JVM start-up. The method area may be of a fixed size or may be expanded as required by the computation and may be contracted if a larger method area becomes unnecessary. The memory for the method area does not need to be contiguous.

A *run-time constant pool* is a per-class or per-interface run-time representation of the **constant_pool** table in a **class** file. The run-time constant pool serves a function similar to that of a symbol table for a conventional programming language, although it contains a wider range of data than a typical symbol table.

Each run-time constant pool is allocated from the JVM's method area. The run-time constant pool for a class or interface is constructed when the class or interface is created by the JVM.

## native method stacks
Native methods are methods written in a language other than the Java programming language.

An implementation of the JVM may use conventional stacks, colloquially called "C stacks", to support native methods.

JVM implementations that cannot load native methods and that do not themselves rely on conventional stacks need not supply native method stacks. If supplied, native method stacks are typically allocated per thread when each thread is created.

This specification permits native method stacks either to be a fixed size or to dynamically expand and contract as required by the computation.

# Frames
A frame is used to store data and partial results, as well as to perform dynamic linking, return values for methods, and dispatch exceptions.

A new frame is created each time a method is invoked. A frame is destroyed when its method invocation completes, whether that completion is normal or abrupt (it throws an uncaught exception).

Frames are allocated from the JVM stack of the thread creating the frame. Each frame has its own array of local variables, its own operand stack, and a reference to the run-time constant pool of the class of the current method.

Only one frame, the frame for the executing method, is active at any point in a given thread of control. This frame is referred to as the *current frame*, and its method is known as the *current method*. The class in which the current method is defined is the *current class*.

Note that a frame created by a thread is local to that thread and cannot be referenced by any other thread.

## local variables
Each frame contains an array of variables known as its *local variables*. The length of the local variable array of a frame is determined at compile-time and supplied in the binary representation of a class or interface along with the code for the method associated with the frame.

A single local variable can hold a value of type **boolean**, **byte**, **char**, **short**, **int**, **float**, **reference**, or **returnAddress**. A pair of local variables can hold a value of type **long** or **double**.

Local variables are addressed by indexing. A value of type **long** or type **double** occupies two consecutive local variables. Such a value may only be addressed using the lesser index.

The JVM uses local variables to pass parameters on method invocation. On class method invocation, any parameters are passed in consecutive local variables starting from local variable 0. On instance method invocation, local variable 0 is always used to pass a reference to the object on which the instance method is being invoked (**this** in the Java programming language). Any parameters are subsequently passed in consecutive local variables starting from local variable 1.

## operand stacks
Each frame contains a last-in-first-out (LIFO) stack known as its *operand stack*. 

At any point in time, an operand stack has an associated depth, where a value of type **long** or **double** contributes two units to the depth and a value of any other type contributes one unit. The maximum depth of the operand stack of a frame is determined at compile-time and is supplied along with the code for the method associated with the frame.

The operand stack is empty when the frame that contains it is created. The JVM supplies instructions to load constants or values from local variables or fields onto the operand stack. Other JVM instructions take operands from the operand stack, operate on them, and push the result back onto the operand stack. The operand stack is also used to prepare parameters to be passed to methods and to receive method results.

Each entry on the operand stack can hold a value of any JVM type, including a value of type **long** or type **double**.

Values from the operand stack must be operated upon in ways appropriate to their types. The restrictions on operand stack manipulation are enforced through **class** file verification.

## dynamic linking
Each frame contains a reference to the run-time constant pool for the type of the current method to support *dynamic linking* of the method code.

The **class** file code for a method refers to methods to be invoked and variables to be accessed via symbolic references. Dynamic linking translates these symbolic method references into concrete method references, loading classes as necessary to resolve as-yet-undefined symbols, and translates variable accesses into appropriate offsets in storage structures associated with the run-time location of these variables.

# Representation of Objects
The JVM does not mandate any particular internal structure for objects.

# Special Methods
## instance initialization methods
A class has zero or more *instance initialization methods*, each typically corresponding to a constructor written in the Java programming language.

A method is an instance initialization method if all of the following are true:
- it is defined in a class (not an interface)
- it has the special name \<init\>
- it is **void**

The declaration and use of an instance initialization method is constrained by the JVM.

## class initialization methods
A class or interface has at most one *class or interface initialization method* and is initialized by the JVM invoking that method.

A method is a class or interface initialization method if all of the following are true:
- it has the special name \<clinit\>
- it is **void**
- in a **class** file whose version number is 51.0 or above, the method has its **ACC_STATIC** flag set and takes no arguments

## signature polymorphic methods


