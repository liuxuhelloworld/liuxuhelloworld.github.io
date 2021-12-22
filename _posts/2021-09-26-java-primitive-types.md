Java有8种primitive types: byte, short, int, long, float, double, char, boolean

# Integer Types
Java中整型数据的字节数和取值范围是固定的。

与C/C++不同，Java没有unsigned整型。

十六进制前缀0x或0X，八进制前缀0，二进制前缀0b或0B.

从Java SE7开始，number literal可以加\_来提高可读性。

# Floating-Point Types
所有浮点运算都遵循IEEE 754 specification.

Floating-point numbers are not suitable for financial calculations in which roundoff errors cannot be tolerated. If you need precise numerical computations without roundoff errors, use the **BigDecimal** class.