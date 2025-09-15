A class or interface whose declaration has one or more *type parameters* is a *generic* class or interface. Generic classes and interfaces are collectively known as *generic types*. For example, **List\<E\>**

Each generic type defines a set of *parameterized types*, which consists of the class or interface name followed by an angle-bracketed list of *actual type parameters* corresponding to the generic type's formal type parameters. For example, **List\<String\>**

In addition, each generic type defines a *raw type*, which is the name of the generic type used without any accompanying type parameters. For example, **List**. Raw types behave as if all of the generic type information were erased from the type declaration. They exist primarily for compatibility with pre-generics code.

Don't use raw types, if you use raw types, you lose all the safety and expressiveness benefits of generics.

While you shouldn't use raw types such as **List**, it is fine to use types that are parameterized to allow insertion of arbitrary objects, such as **List\<Object\>**.

While you can pass a **List\<String\>** to a parameter of type **List**, you can't pass it to a parameter of type **List\<Object\>**. There are sub-typing rules for generics, and **List\<String\>** is a subtype of the raw type **List**, but not of the parameterized type **List\<Object\>**.

You might be tempted to use a raw type for a collection whose element type is unknown and doesn't matter. The safe alternative is to use *unbounded wildcard types*. If you want to use a generic type but you don't know or care what the actual type parameter is, you can use a question mark instead. For example, **Set\<?\>**.

You must use raw types in class literals. The specification does not permit the use of parameterized types (though it does permit array types and primitive types). In other words, **List.class**, **String\[\].class** and **int.class** are all legal, but **List\<String\>.class** and **List\<?\>.class** are not.