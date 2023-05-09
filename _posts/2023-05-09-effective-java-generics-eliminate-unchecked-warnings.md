Eliminate every unchecked warnings that you can. If you can't eliminate a warning, but you can prove that the code that provoked the warning is typesafe, then (and only then) suppress the warning with an **@SuppressWarnings("unchecked")** annotation.

The **SuppressWarnings** annotation can be used on any declaration, from an individual local variable declaration to an entire class. Always use the **SuppressWarnings** annotation on the smallest scope possible.

Every time you use a **@SuppressWarnings("unchecked")** annotation, add a comment saying why it is safe to do so.