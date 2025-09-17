# C# Operators

C# operators are special symbols that perform operations on variables and  
values. This tutorial covers the various types of operators and expressions  
in the C# language. Expressions are constructed from operands and operators,  
where operators indicate which operations to apply to the operands.

## C# Operator List

C# provides a rich set of operators organized into several categories:

| Category | Operators |
|----------|-----------|
| Sign operators | `+` `-` |
| Arithmetic | `+` `-` `*` `/` `%` |
| Logical (boolean and bitwise) | `&` `|` `^` `!` `~` `&&` `||` `true` `false` |
| String concatenation | `+` |
| Increment, decrement | `++` `--` |
| Shift | `<<` `>>` |
| Relational | `==` `!=` `<` `>` `<=` `>=` |
| Assignment | `=` `+=` `-=` `*=` `/=` `%=` `&=` `|=` `^=` `??=` `<<=` `>>=` |
| Member access | `.` `?.` |
| Indexing | `[]` `?[]` |
| Cast | `()` |
| Ternary | `?:` |
| Delegate concatenation and removal | `+` `-` |
| Object creation | `new` |
| Type information | `as` `is` `sizeof` `typeof` |
| Overflow exception control | `checked` `unchecked` |
| Indirection and address | `*` `->` `[]` `&` |
| Lambda | `=>` |
| Pattern matching | `is` (with patterns) |
| Range and index | `..` `^` |
| Null-coalescing | `??` `??=` |

An operator typically works with one or two operands. Operators that work  
with only one operand are called *unary operators*. Those that work with  
two operands are called *binary operators*. There is also one ternary  
operator `?:`, which works with three operands.

Certain operators may be used in different contexts. For example, the `+`  
operator adds numbers, concatenates strings or delegates, and indicates  
the sign of a number. We say that the operator is *overloaded*.

## C# Unary Operators

C# unary operators include: `+`, `-`, `++`, `--`, cast operator `()`, and  
negation `!`.

### C# Sign Operators

There are two sign operators: `+` and `-`. They are used to indicate or  
change the sign of a value.

```csharp
Console.WriteLine(2);
Console.WriteLine(+2);
Console.WriteLine(-2);
```

The `+` and `-` signs indicate the sign of a value. The plus sign can be  
used to indicate that we have a positive number. It can be omitted and  
usually is.

```csharp
int a = 1;
Console.WriteLine(-a);
Console.WriteLine(-(-a));
```

The minus sign changes the sign of a value.

```
$ dotnet run
-1
1
```

### C# Increment and Decrement Operators

Incrementing or decrementing a value by one is a common task in programming.  
C# has two convenient operators for this: `++` and `--`.

```csharp
int x = 6;

x++;
Console.WriteLine(x);

x--;
Console.WriteLine(x);
```

We use the increment and decrement operators on a variable.

```csharp
x++;
```

We use the increment operator. Now the variable equals 7.

```csharp
x--;
```

We use the decrement operator. Now the variable equals 7.

```
$ dotnet run
7
6
```

### C# Explicit Cast Operator

The explicit cast operator `()` can be used to cast one type to another.  
Note that this operator works only on certain compatible types.

```csharp
float val = 3.2f;
int num = (int) val;

Console.WriteLine(num);
```

In the example, we explicitly cast a `float` type to `int`.

### Negation Operator

The negation operator `!` reverses the meaning of its operand.

```csharp
bool val = true;
Console.WriteLine(!val);
```

In this example, we apply the negation operator to a boolean value.

```
$ dotnet run
False
```

## C# Assignment Operator

The assignment operator `=` assigns a value to a variable. A variable is  
a placeholder for a value.

```csharp
int x = 1;
Console.WriteLine(x);
```

We assign number 1 to the `x` variable and print it to the console.

It is possible to assign a value to multiple variables:

```csharp
int a, b, c;
a = b = c = 5;

Console.WriteLine($"{a} {b} {c}");
```

```
$ dotnet run
5 5 5
```

## C# String Concatenation

The `+` operator is also used to concatenate strings.

```csharp
string name = "Jane";
string message = "Hello " + name;
Console.WriteLine(message);
```

We concatenate two strings with the `+` operator.

```
$ dotnet run
Hello Jane
```

## C# Arithmetic Operators

The following is a table of arithmetic operators in C#:

| Symbol | Name |
|--------|------|
| `+` | Addition |
| `-` | Subtraction |
| `*` | Multiplication |
| `/` | Division |
| `%` | Remainder |

The following example shows arithmetic operations:

```csharp
int a = 10;
int b = 11;
int c = 12;

int add = a + b + c;
int sb = c - a;
int mult = a * b;
int div = c / 3;
int rem = c % a;

Console.WriteLine($"Addition: {add}");
Console.WriteLine($"Subtraction: {sb}");
Console.WriteLine($"Multiplication: {mult}");
Console.WriteLine($"Division: {div}");
Console.WriteLine($"Remainder: {rem}");
```

In the preceding example, we use addition, subtraction, multiplication,  
division, and remainder operations.

```
$ dotnet run
Addition: 33
Subtraction: 2
Multiplication: 110
Division: 4
Remainder: 2
```

### C# Division of Integers

Pay special attention to the division of integers. When we divide two  
integers, the result is an integer. This is integer division.

```csharp
int c = 5 / 2;
Console.WriteLine(c);
```

In this example, we divide two integers. The result is an integer. The  
result is 2, not 2.5 as we might expect.

If one of the operands is a double or float, we perform a floating point  
division:

```csharp
double c = 5 / 2.0;
Console.WriteLine(c);
```

Now the result is 2.5 as expected.

```
$ dotnet run
2.5
```

## C# Boolean Operators

In C#, we have the following logical operators. Boolean operators work  
with boolean values.

| Symbol | Name |
|--------|------|
| `&&` | logical and |
| `||` | logical or |
| `!` | logical not |

The logical and `&&` operator evaluates to `true` only if both operands  
are `true`.

```csharp
bool a = true;
bool b = true;
bool c = false;
bool d = false;

Console.WriteLine(a && b);
Console.WriteLine(a && c);
Console.WriteLine(c && d);
```

Only the first expression evaluates to `true`.

```
$ dotnet run
True
False
False
```

The logical or `||` operator evaluates to `true` if either of the operands  
is `true`.

```csharp
bool a = true;
bool b = true;
bool c = false;
bool d = false;

Console.WriteLine(a || b);
Console.WriteLine(a || c);
Console.WriteLine(c || d);
```

The first two expressions evaluate to `true`.

```
$ dotnet run
True
True
False
```

The negation operator `!` makes `true` `false` and `false` `true`.

```csharp
bool a = true;
bool b = false;

Console.WriteLine(!a);
Console.WriteLine(!b);
```

```
$ dotnet run
False
True
```

The `&&` and `||` operators are short-circuited. Short-circuited means  
that the second operand is not evaluated if the first operand suffices  
to determine the expression. This behavior can improve performance and  
prevent potential errors.

```csharp
Console.WriteLine(true || GetValue());
Console.WriteLine(false && GetValue());

bool GetValue()
{
    Console.WriteLine("GetValue() method called");
    return true;
}
```

In this example, the `GetValue()` method is never called.

```
$ dotnet run
True
False
```

## C# Relational Operators

Relational operators are used to compare values. These operators always  
result in boolean values.

| Symbol | Name |
|--------|------|
| `<` | strictly less than |
| `<=` | less than or equal to |
| `>` | strictly greater than |
| `>=` | greater than or equal to |
| `==` | equal to |
| `!=` | not equal to |

```csharp
Console.WriteLine(3 < 4);
Console.WriteLine(3 == 4);
Console.WriteLine(4 >= 3);
```

```
$ dotnet run
True
False
True
```

## C# Bitwise Operators

Decimal numbers are natural to humans. Binary numbers are native to  
computers. Binary, octal, decimal, or hexadecimal symbols are only  
notations of the same number. Bitwise operators work with bits of a  
binary number. Bitwise operators are seldom used in higher level  
languages like C#.

| Symbol | Name |
|--------|------|
| `~` | bitwise not |
| `^` | bitwise exclusive or |
| `&` | bitwise and |
| `|` | bitwise or |
| `<<` | left shift |
| `>>` | right shift |

The bitwise and operator `&` performs bit-by-bit comparison. The bit  
result is 1 only if both bits are 1.

```csharp
Console.WriteLine(6 & 3);
Console.WriteLine(3 & 6);
```

The first number is 6, which is 110 in binary. The second is 3, which  
is 011 in binary. The result is 010, which is 2 in decimal.

```
110
011
---
010
```

The bitwise or operator `|` performs bit-by-bit comparison. The bit  
result is 1 if either of the bits is 1.

```csharp
Console.WriteLine(6 | 3);
Console.WriteLine(3 | 6);
```

The result is 111, which is 7 in decimal.

```
110
011
---
111
```

The bitwise exclusive or operator `^` performs bit-by-bit comparison.  
The bit result is 1 if exactly one of the bits is 1.

```csharp
Console.WriteLine(6 ^ 3);
Console.WriteLine(3 ^ 6);
```

The result is 101, which is 5 in decimal.

```
110
011
---
101
```

The bitwise not operator `~` reverses all bits of the number.

```csharp
Console.WriteLine(~7);
Console.WriteLine(~-8);
```

```
$ dotnet run
-8
7
```

The left shift operator `<<` shifts bits to the left. The right operand  
specifies the number of positions to shift the left operand.

```csharp
Console.WriteLine(9 << 1);
Console.WriteLine(9 << 2);
```

9 is 1001 in binary. 9 << 1 returns 10010, which is 18 in decimal.  
9 << 2 returns 100100, which is 36 in decimal.

```
$ dotnet run
18
36
```

The right shift operator `>>` shifts bits to the right. The left operand  
is shifted the number of places specified by the right operand.

```csharp
Console.WriteLine(9 >> 1);
Console.WriteLine(9 >> 2);
```

9 >> 1 returns 100, which is 4 in decimal. 9 >> 2 returns 10, which is  
2 in decimal.

```
$ dotnet run
4
2
```

## C# Compound Assignment Operators

The compound assignment operators consist of two operators. They are  
shorthand operators.

```csharp
int a = 1;
a = a + 1;

Console.WriteLine(a);

a += 5;
Console.WriteLine(a);

a *= 3;
Console.WriteLine(a);
```

In the example, we use two compound operators.

```csharp
int a = 1;
a = a + 1;
```

The `a` variable is initiated to one. 1 is added to the variable using  
the non-shorthand notation.

```csharp
a += 5;
```

Using a `+=` compound operator, we add 5 to the `a` variable. The  
statement is equal to `a = a + 5;`.

```csharp
a *= 3;
```

Using the `*=` operator, the `a` is multiplied by 3. The statement is  
equal to `a = a * 3;`.

```
$ dotnet run
2
7
21
```

## C# new Operator

The `new` operator is used to create objects and invoke constructors.

```csharp
var words = new string[] {"sky", "cup", "odd", "car"};

foreach (string word in words)
{
    Console.WriteLine(word);
}
```

We create an array of strings with the `new` operator.

```
$ dotnet run
sky
cup
odd
car
```

## C# Member Access Operator

The dot `.` operator provides access to members of a namespace, a type,  
or an object.

```csharp
var now = DateTime.Now;
Console.WriteLine(now);

string name = "Jane";
Console.WriteLine(name.ToUpper());
Console.WriteLine(name.Length);
```

With the dot operator, we access the static `Now` property of the  
`DateTime` type. We also access the `ToUpper()` method and `Length`  
property of a string.

```
$ dotnet run
12/28/2024 10:30:45 AM
JANE
4
```

## C# Null-Conditional Operators

The null-conditional operators `?.` and `?[]` perform a member access  
or element access operation only if an operand is non-null; otherwise,  
they return null.

```csharp
string? text = null;
Console.WriteLine(text?.Length);

string text2 = "falcon";  
Console.WriteLine(text2?.Length);

int[]? vals = null;
Console.WriteLine(vals?[0]);

int[] vals2 = {1, 2, 3, 4, 5};
Console.WriteLine(vals2?[0]);
```

If the left-hand operand is null, then these operators return null  
instead of throwing a NullReferenceException.

```
$ dotnet run

6

1
```

## C# Index from End Operator ^

The `^` operator indicates the element position from the end of a  
sequence. For a sequence of length n, `^i` points to the element with  
offset `n - i` from the start of a sequence.

```csharp
string[] words = {"sky", "blue", "den", "cloud", "forest"};

Console.WriteLine(words[^1]);
Console.WriteLine(words[^2]);
Console.WriteLine(words[^3]);
```

We access elements from the end of the array.

```
$ dotnet run
forest
cloud
den
```

## C# Range Operator ..

The `..` operator specifies the start and end of a range of indices as  
its operands.

```csharp
int[] vals = {1, 2, 3, 4, 5, 6, 7};

int[] vals2 = vals[1..4];
Console.WriteLine(string.Join(",", vals2));

int[] vals3 = vals[..3];
Console.WriteLine(string.Join(",", vals3));

int[] vals4 = vals[2..];
Console.WriteLine(string.Join(",", vals4));
```

We create subarrays using the range operator.

```csharp
int[] vals2 = vals[1..4];
```

We get elements with indexes 1, 2, and 3. The end index is exclusive.

```csharp
int[] vals3 = vals[..3];
```

This gets elements from the beginning up to index 3 (exclusive).

```csharp
int[] vals4 = vals[2..];
```

This gets elements from index 2 to the end of the array.

```
$ dotnet run
2,3,4
1,2,3
3,4,5,6,7
```

## C# Type Information Operators

The `typeof` operator obtains the System.Type instance for a type.

```csharp
int x = 5;
float y = 3.2f;

Console.WriteLine(x.GetType());
Console.WriteLine(y.GetType());
Console.WriteLine(typeof(int));
Console.WriteLine(typeof(float));
```

We print type information for integers and floats.

```
$ dotnet run
System.Int32
System.Single
System.Int32
System.Single
```

We can see that the `int` type is an alias for `System.Int32` and the  
`float` is an alias for the `System.Single` type.

The `is` operator checks if an object is compatible with a given type.

```csharp
Base _base = new Base();
Derived derived = new Derived();

Console.WriteLine(_base is Base);
Console.WriteLine(_base is Object);
Console.WriteLine(derived is Base);
Console.WriteLine(_base is Derived);

class Base { }
class Derived : Base { }
```

We create two objects from user defined types.

```csharp
class Base {}
class Derived : Base {}
```

We have a `Base` class and a `Derived` class. The `Derived` class  
inherits from the `Base` class.

```csharp
Console.WriteLine(_base is Base);
Console.WriteLine(_base is Object);
Console.WriteLine(derived is Base);
Console.WriteLine(_base is Derived);
```

The first expression is `true` since the `_base` object is an instance  
of the `Base` type. The second is `true` because each object inherits  
from the base `Object`. The third is `true` because the `derived`  
object is a `Derived` type, and the `Derived` type inherits from the  
`Base` type. The fourth is `false` because the `_base` object is not  
an instance of the `Derived` type.

```
$ dotnet run
True
True
True
False
```

The `as` operator is used to perform conversions between compatible  
reference types. When the conversion is not possible, the operator  
returns null instead of raising an exception.

```csharp
object[] objects = new object[6];
objects[0] = new Base();
objects[1] = new Derived();
objects[2] = "ZetCode";
objects[3] = 12;
objects[4] = 1.4;
objects[5] = null;

for (int i = 0; i < objects.Length; i++)
{
    string? s = objects[i] as string;
    Console.Write($"{i}:");

    if (s != null)
    {
        Console.WriteLine(s);
    }
    else
    {
        Console.WriteLine("not a string");
    }
}

class Base { }
class Derived : Base { }
```

In the above example, we use the `as` operator to perform casting.

```csharp
string? s = objects[i] as string;
```

We try to cast various types to the string type. But only once the  
casting is valid.

```
$ dotnet run
0:not a string
1:not a string
2:ZetCode
3:not a string
4:not a string
5:not a string
```

## C# Operator Precedence

The *operator precedence* tells us which operators are evaluated first.  
The precedence level is necessary to avoid ambiguity in expressions.

What is the outcome of the following expression, 28 or 40?

```
3 + 5 * 5
```

Like in mathematics, the multiplication operator has a higher precedence  
than addition operator. So the outcome is 28.

```
(3 + 5) * 5
```

To change the order of evaluation, we can use parentheses. Expressions  
inside parentheses are always evaluated first.

The following table shows common C# operators ordered by precedence  
(highest precedence first):

| Operator(s) | Category | Associativity |
|-------------|----------|---------------|
| `x.y` `x?.y` `x?[y]` `f(x)` `a[x]` `x++` `x--` `new` `typeof` `default` `checked` `unchecked` | Primary | Left |
| `+` `-` `!` `~` `++x` `--x` `(T)x` | Unary | Left |
| `*` `/` `%` | Multiplicative | Left |
| `+` `-` | Additive | Left |
| `<<` `>>` | Shift | Left |
| `<` `>` `<=` `>=` `is` `as` | Relational and type-testing | Left |
| `==` `!=` | Equality | Left |
| `&` | Logical AND | Left |
| `^` | Logical XOR | Left |
| `|` | Logical OR | Left |
| `&&` | Conditional AND | Left |
| `||` | Conditional OR | Left |
| `??` | Null Coalescing | Left |
| `?:` | Ternary | Right |
| `=` `*=` `/=` `%=` `+=` `-=` `<<=` `>>=` `&=` `^=` `|=` `??=` `=>` | Assignment | Right |

Operators on the same row of the table have the same precedence.

```csharp
Console.WriteLine(3 + 5 * 5);
Console.WriteLine((3 + 5) * 5);

int a, b, c, d;
a = b = c = d = 0;

Console.WriteLine($"{a} {b} {c} {d}");

int j = 0;
j *= 3 + 1;

Console.WriteLine(j);
```

In the example, we have two cases where the associativity rule determines  
the expression.

```csharp
int a, b, c, d;
a = b = c = d = 0;
```

The assignment operator is right to left associated. If the associativity  
was left to right, the previous expression would not be possible.

```csharp
int j = 0;
j *= 3 + 1;
```

The compound assignment operators are right to left associated. We might  
expect the result to be 1. But the actual result is 0. Because of the  
associativity. The expression on the right is evaluated first and then  
the compound assignment operator is applied.

```
$ dotnet run
28
40
0 0 0 0
0
```

## C# Null-Coalescing Operator

The null-coalescing operator `??` returns the value of its left-hand  
operand if it isn't null; otherwise, it evaluates the right-hand operand  
and returns its result.

```csharp
string? name = null;
string? name2 = "Peter";

Console.WriteLine(name ?? "unknown");
Console.WriteLine(name2 ?? "unknown");
```

The `??` operator is useful for providing default values for nullable  
types.

```
$ dotnet run
unknown
Peter
```

## C# Null-Coalescing Assignment Operator

The null-coalescing assignment operator `??=` assigns the value of its  
right-hand operand to its left-hand operand only if the left-hand  
operand evaluates to null.

```csharp
string? message = null;
string? message2 = "old value";

message ??= "new value";
message2 ??= "new value";

Console.WriteLine(message);
Console.WriteLine(message2);
```

The `??=` operator doesn't evaluate its right-hand operand if the  
left-hand operand evaluates to non-null.

```
$ dotnet run
new value
old value
```

## C# Pattern Matching with is

The `is` operator supports pattern matching. Pattern matching allows  
you to test if a value has a certain shape and extract information  
from it when it does.

```csharp
object[] objects = { 1, "hello", 3.14, true, null };

foreach (var obj in objects)
{
    var result = obj switch
    {
        int i when i > 0 => $"Positive integer: {i}",
        string s when !string.IsNullOrEmpty(s) => $"Non-empty string: {s}",
        double d => $"Double: {d}",
        bool b => $"Boolean: {b}",
        null => "Null value",
        _ => "Unknown type"
    };
    
    Console.WriteLine(result);
}
```

This example demonstrates pattern matching with switch expressions.

```csharp
// Traditional is pattern matching
if (obj is string str && str.Length > 5)
{
    Console.WriteLine($"Long string: {str}");
}

// Type patterns
if (obj is int number)
{
    Console.WriteLine($"Integer: {number}");
}
```

```
$ dotnet run
Positive integer: 1
Non-empty string: hello
Double: 3.14
Boolean: True
Null value
```

## C# Target-Typed new Expressions

Starting with C# 9.0, you can omit the type in a `new` expression when  
the target type is known:

```csharp
// Traditional way
Dictionary<string, int> dict1 = new Dictionary<string, int>();

// Target-typed new (C# 9.0+)
Dictionary<string, int> dict2 = new();

// Works with fields, properties, and method parameters
List<string> names = new() { "Alice", "Bob", "Charlie" };

// In method calls
ProcessList(new() { 1, 2, 3, 4, 5 });

void ProcessList(List<int> numbers)
{
    Console.WriteLine($"Processing {numbers.Count} numbers");
}
```

This feature reduces redundancy when the type is already specified.

```
$ dotnet run
Processing 5 numbers
```

## C# Switch Expressions

Switch expressions provide a more concise way to write switch statements  
(available since C# 8.0):

```csharp
var day = DateTime.Now.DayOfWeek;

var dayType = day switch
{
    DayOfWeek.Monday => "Start of work week",
    DayOfWeek.Tuesday or DayOfWeek.Wednesday or DayOfWeek.Thursday => "Midweek",
    DayOfWeek.Friday => "End of work week",
    DayOfWeek.Saturday or DayOfWeek.Sunday => "Weekend",
    _ => "Unknown day"
};

Console.WriteLine(dayType);

// With pattern matching
static string DescribeNumber(object obj) => obj switch
{
    int i when i < 0 => "Negative integer",
    int i when i == 0 => "Zero",
    int i when i > 0 => "Positive integer",
    double d => $"Double: {d:F2}",
    string s => $"String with length {s.Length}",
    _ => "Unknown type"
};

Console.WriteLine(DescribeNumber(42));
Console.WriteLine(DescribeNumber(-5));
Console.WriteLine(DescribeNumber(3.14159));
Console.WriteLine(DescribeNumber("Hello"));
```

Switch expressions are more concise and functional in style compared to  
traditional switch statements.

```
$ dotnet run
Midweek
Positive integer
Negative integer
Double: 3.14
String with length 5
```

This concludes our overview of C# operators. Understanding operator  
precedence, associativity, and the various types of operators is  
essential for writing effective C# code. Modern C# continues to evolve  
with new operator features like pattern matching, null-coalescing  
assignment, and target-typed new expressions that make code more  
expressive and safer.
