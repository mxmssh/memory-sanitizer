Q. Why does passing an uninitialized value as an argument of a function call not trigger a warning?

Q. Why does reading an uninitialized value from memory not trigger a warning?

A. MSan only reports errors when the uninitialized value is used in a way which could affect externally visible program behavior, like when it is used in a conditional branch, to index another memory operation, or passed to a libc function. Copying uninitialized memory as well as doing simple arithmetic operations on it is allowed, and, in fact, happens quite often in practice.