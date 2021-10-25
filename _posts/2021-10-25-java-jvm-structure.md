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