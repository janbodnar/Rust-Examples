# Rust Operators

Rust operators are special symbols that perform operations on variables and  
values. This tutorial covers the various types of operators and expressions  
in the Rust language. Expressions are constructed from operands and operators,  
where operators indicate which operations to apply to the operands.

## Rust Operator List

Rust provides a comprehensive set of operators organized into several categories:

| Category | Operators |
|----------|-----------|
| Arithmetic | `+` `-` `*` `/` `%` |
| Logical (boolean and bitwise) | `&` `|` `^` `!` `~` `&&` `||` |
| Comparison | `==` `!=` `<` `>` `<=` `>=` |
| Assignment | `=` `+=` `-=` `*=` `/=` `%=` `&=` `|=` `^=` `<<=` `>>=` |
| Borrowing and dereferencing | `&` `&mut` `*` |
| Range | `..` `..=` |
| Error handling | `?` |
| Field access | `.` |
| Indexing | `[]` |
| Function call | `()` |
| Type casting | `as` |
| Pattern matching | `match` expressions |
| Closure | `|args| expr` |

An operator typically works with one or two operands. Operators that work  
with only one operand are called *unary operators*. Those that work with  
two operands are called *binary operators*.

Certain operators may be used in different contexts. For example, the `+`  
operator adds numbers, concatenates strings, and indicates the sign of a  
number. We say that the operator is *overloaded*.

## Rust Unary Operators

Rust unary operators include: `+`, `-`, `*`, `&`, `&mut`, and negation `!`.

### Sign Operators

There are two sign operators: `+` and `-`. They are used to indicate or  
change the sign of a value.

```rust
fn main() {
    println!("{}", 2);
    println!("{}", +2);
    println!("{}", -2);
}
```

The `+` and `-` signs indicate the sign of a value. The plus sign can be  
used to indicate that we have a positive number. It can be omitted and  
usually is.

```rust
fn main() {
    let a = 1;
    println!("{}", -a);
    println!("{}", -(-a));
}
```

The minus sign changes the sign of a value.

```
$ cargo run
-1
1
```

### Reference and Dereference Operators

Rust uses `&` to create references (borrowing) and `*` to dereference.  
These are fundamental to Rust's ownership system.

```rust
fn main() {
    let x = 42;
    let x_ref = &x;        // Create reference
    let x_val = *x_ref;    // Dereference
    
    println!("x = {}", x);
    println!("x_ref points to {}", x_ref);
    println!("x_val = {}", x_val);
}
```

Mutable references use `&mut`:

```rust
fn main() {
    let mut x = 42;
    let x_mut_ref = &mut x;
    *x_mut_ref += 10;
    
    println!("x = {}", x);
}
```

```
$ cargo run
x = 42
x_ref points to 42
x_val = 42
x = 52
```

### Type Casting Operator

The `as` operator is used for explicit type casting between compatible types:

```rust
fn main() {
    let val = 3.2;
    let num = val as i32;
    
    println!("val = {}", val);
    println!("num = {}", num);
}
```

```
$ cargo run
val = 3.2
num = 3
```

### Negation Operator

The negation operator `!` reverses the meaning of its boolean operand:

```rust
fn main() {
    let val = true;
    println!("val = {}", val);
    println!("!val = {}", !val);
}
```

```
$ cargo run
val = true
!val = false
```

## Rust Assignment Operator

The assignment operator `=` assigns a value to a variable. In Rust, variables  
are immutable by default, so they must be declared with `mut` to be modified.

```rust
fn main() {
    let x = 1;
    println!("{}", x);
}
```

We assign number 1 to the `x` variable and print it to the console.

For multiple assignment, Rust uses pattern destructuring:

```rust
fn main() {
    let (a, b, c) = (5, 6, 7);
    println!("{} {} {}", a, b, c);
}
```

Unlike some languages, Rust doesn't support chained assignment like  
`a = b = c = 5`. Instead, use pattern destructuring as shown above.

```
$ cargo run
1
5 6 7
```

## Rust String Concatenation

Rust provides several ways to concatenate strings. The `+` operator can be  
used with String types, and the `format!` macro provides flexible formatting.

```rust
fn main() {
    let name = "Jane";
    let message = format!("Hello {}", name);
    println!("{}", message);
    
    // Using + operator with String
    let greeting = "Hello".to_string();
    let full_message = greeting + " " + &name.to_string();
    println!("{}", full_message);
}
```

The `format!` macro is often preferred for string interpolation as it's more  
readable and flexible than concatenation.

```
$ cargo run
Hello Jane
Hello Jane
```

## Rust Arithmetic Operators

The following is a table of arithmetic operators in Rust:

| Symbol | Name |
|--------|------|
| `+` | Addition |
| `-` | Subtraction |
| `*` | Multiplication |
| `/` | Division |
| `%` | Remainder |

The following example shows arithmetic operations:

```rust
fn main() {
    let a = 10;
    let b = 11;
    let c = 12;
    
    let add = a + b + c;
    let sub = c - a;
    let mult = a * b;
    let div = c / 3;
    let rem = c % a;
    
    println!("Addition: {}", add);
    println!("Subtraction: {}", sub);
    println!("Multiplication: {}", mult);
    println!("Division: {}", div);
    println!("Remainder: {}", rem);
}
```

In the preceding example, we use addition, subtraction, multiplication,  
division, and remainder operations.

```
$ cargo run
Addition: 33
Subtraction: 2
Multiplication: 110
Division: 4
Remainder: 2
```

### Integer Division in Rust

In Rust, integer division truncates toward zero, similar to other languages.  
When we divide two integers, the result is an integer:

```rust
fn main() {
    let c = 5 / 2;
    println!("{}", c);
}
```

The result is 2, not 2.5 as we might expect in floating-point division.

For floating-point division, at least one operand must be a float:

```rust
fn main() {
    let c = 5.0 / 2.0;
    println!("{}", c);
}
```

Now the result is 2.5 as expected.

```
$ cargo run
2
2.5
```

### Overflow Behavior

Rust has different behavior for overflow in debug vs release builds:

```rust
fn main() {
    let x: u8 = 255;
    // This will panic in debug mode, wrap in release mode
    // let y = x + 1;
    
    // For explicit wrapping behavior:
    let y = x.wrapping_add(1);
    println!("Wrapping add: {}", y);
    
    // For checked arithmetic:
    match x.checked_add(1) {
        Some(result) => println!("Result: {}", result),
        None => println!("Overflow occurred!"),
    }
}
```

```
$ cargo run
Wrapping add: 0
Overflow occurred!
```

## Rust Boolean Operators

In Rust, we have the following logical operators. Boolean operators work  
with boolean values.

| Symbol | Name |
|--------|------|
| `&&` | logical and |
| `||` | logical or |
| `!` | logical not |

The logical and `&&` operator evaluates to `true` only if both operands  
are `true`.

```rust
fn main() {
    let a = true;
    let b = true;
    let c = false;
    let d = false;
    
    println!("{}", a && b);
    println!("{}", a && c);
    println!("{}", c && d);
}
```

Only the first expression evaluates to `true`.

```
$ cargo run
true
false
false
```

The logical or `||` operator evaluates to `true` if either of the operands  
is `true`.

```rust
fn main() {
    let a = true;
    let b = true;
    let c = false;
    let d = false;
    
    println!("{}", a || b);
    println!("{}", a || c);
    println!("{}", c || d);
}
```

The first two expressions evaluate to `true`.

```
$ cargo run
true
true
false
```

The negation operator `!` makes `true` `false` and `false` `true`.

```rust
fn main() {
    let a = true;
    let b = false;
    
    println!("{}", !a);
    println!("{}", !b);
}
```

```
$ cargo run
false
true
```

The `&&` and `||` operators are short-circuited. Short-circuited means  
that the second operand is not evaluated if the first operand suffices  
to determine the expression. This behavior can improve performance and  
prevent potential errors.

```rust
fn main() {
    println!("{}", true || get_value());
    println!("{}", false && get_value());
}

fn get_value() -> bool {
    println!("get_value() function called");
    true
}
```

In this example, the `get_value()` function is never called.

```
$ cargo run
true
false
```

## Rust Comparison Operators

Comparison operators are used to compare values. These operators always  
result in boolean values.

| Symbol | Name |
|--------|------|
| `<` | strictly less than |
| `<=` | less than or equal to |
| `>` | strictly greater than |
| `>=` | greater than or equal to |
| `==` | equal to |
| `!=` | not equal to |

```rust
fn main() {
    println!("{}", 3 < 4);
    println!("{}", 3 == 4);
    println!("{}", 4 >= 3);
}
```

```
$ cargo run
true
false
true
```

Rust also provides ordering comparisons through the `Ord` trait:

```rust
use std::cmp::Ordering;

fn main() {
    let result = 3.cmp(&4);
    match result {
        Ordering::Less => println!("3 is less than 4"),
        Ordering::Equal => println!("3 equals 4"),
        Ordering::Greater => println!("3 is greater than 4"),
    }
}
```

```
$ cargo run
3 is less than 4
```

## Rust Bitwise Operators

Decimal numbers are natural to humans. Binary numbers are native to  
computers. Binary, octal, decimal, or hexadecimal symbols are only  
notations of the same number. Bitwise operators work with bits of a  
binary number.

| Symbol | Name |
|--------|------|
| `!` | bitwise not |
| `^` | bitwise exclusive or |
| `&` | bitwise and |
| `|` | bitwise or |
| `<<` | left shift |
| `>>` | right shift |

The bitwise and operator `&` performs bit-by-bit comparison. The bit  
result is 1 only if both bits are 1.

```rust
fn main() {
    println!("{}", 6 & 3);
    println!("{}", 3 & 6);
}
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

```rust
fn main() {
    println!("{}", 6 | 3);
    println!("{}", 3 | 6);
}
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

```rust
fn main() {
    println!("{}", 6 ^ 3);
    println!("{}", 3 ^ 6);
}
```

The result is 101, which is 5 in decimal.

```
110
011
---
101
```

The bitwise not operator `!` reverses all bits of the number.

```rust
fn main() {
    let x: i8 = 7;
    println!("{}", !x);
    
    let y: i8 = -8;
    println!("{}", !y);
}
```

```
$ cargo run
-8
7
```

The left shift operator `<<` shifts bits to the left. The right operand  
specifies the number of positions to shift the left operand.

```rust
fn main() {
    println!("{}", 9 << 1);
    println!("{}", 9 << 2);
}
```

9 is 1001 in binary. 9 << 1 returns 10010, which is 18 in decimal.  
9 << 2 returns 100100, which is 36 in decimal.

```
$ cargo run
18
36
```

The right shift operator `>>` shifts bits to the right. The left operand  
is shifted the number of places specified by the right operand.

```rust
fn main() {
    println!("{}", 9 >> 1);
    println!("{}", 9 >> 2);
}
```

9 >> 1 returns 100, which is 4 in decimal. 9 >> 2 returns 10, which is  
2 in decimal.

```
$ cargo run
4
2
```

### Binary Literals and Formatting

Rust supports binary literals and formatting for easier bit manipulation:

```rust
fn main() {
    let x = 0b1010; // Binary literal for 10
    let y = 0b1100; // Binary literal for 12
    
    println!("x = {:04b} ({})", x, x);
    println!("y = {:04b} ({})", y, y);
    println!("x & y = {:04b} ({})", x & y, x & y);
    println!("x | y = {:04b} ({})", x | y, x | y);
    println!("x ^ y = {:04b} ({})", x ^ y, x ^ y);
}
```

```
$ cargo run
x = 1010 (10)
y = 1100 (12)
x & y = 1000 (8)
x | y = 1110 (14)
x ^ y = 0110 (6)
```

## Rust Compound Assignment Operators

The compound assignment operators consist of two operators. They are  
shorthand operators.

```rust
fn main() {
    let mut a = 1;
    a = a + 1;
    
    println!("{}", a);
    
    a += 5;
    println!("{}", a);
    
    a *= 3;
    println!("{}", a);
}
```

In the example, we use two compound operators. Note that variables must  
be declared as `mut` to use compound assignment operators.

```rust
let mut a = 1;
a = a + 1;
```

The `a` variable is initiated to one. 1 is added to the variable using  
the non-shorthand notation.

```rust
a += 5;
```

Using a `+=` compound operator, we add 5 to the `a` variable. The  
statement is equal to `a = a + 5;`.

```rust
a *= 3;
```

Using the `*=` operator, the `a` is multiplied by 3. The statement is  
equal to `a = a * 3;`.

```
$ cargo run
2
7
21
```

Rust supports all the standard compound assignment operators:

```rust
fn main() {
    let mut x = 10;
    
    x += 5;   // Addition assignment
    println!("After += 5: {}", x);
    
    x -= 3;   // Subtraction assignment  
    println!("After -= 3: {}", x);
    
    x *= 2;   // Multiplication assignment
    println!("After *= 2: {}", x);
    
    x /= 4;   // Division assignment
    println!("After /= 4: {}", x);
    
    x %= 3;   // Remainder assignment
    println!("After %= 3: {}", x);
    
    // Bitwise compound assignments
    x |= 0b1001;  // Bitwise OR assignment
    println!("After |= 0b1001: {:04b}", x);
    
    x &= 0b1010;  // Bitwise AND assignment
    println!("After &= 0b1010: {:04b}", x);
    
    x ^= 0b0110;  // Bitwise XOR assignment
    println!("After ^= 0b0110: {:04b}", x);
}
```

```
$ cargo run
After += 5: 15
After -= 3: 12
After *= 2: 24
After /= 4: 6
After %= 3: 0
After |= 0b1001: 1001
After &= 0b1010: 1000
After ^= 0b0110: 1110
```

## Rust Struct Instantiation and Memory Allocation

Rust doesn't have a `new` keyword like some languages. Instead, structs  
are created using struct literal syntax, and heap allocation is done  
explicitly with `Box::new()`.

```rust
struct Person {
    name: String,
    age: u32,
}

fn main() {
    // Stack allocation
    let person1 = Person {
        name: "Alice".to_string(),
        age: 30,
    };
    
    // Heap allocation with Box
    let person2 = Box::new(Person {
        name: "Bob".to_string(),
        age: 25,
    });
    
    println!("{} is {} years old", person1.name, person1.age);
    println!("{} is {} years old", person2.name, person2.age);
    
    // Vector creation
    let numbers = vec![1, 2, 3, 4, 5];
    println!("Numbers: {:?}", numbers);
}
```

The `vec!` macro creates a new vector, which is allocated on the heap.

```
$ cargo run
Alice is 30 years old
Bob is 25 years old
Numbers: [1, 2, 3, 4, 5]
```

## Rust Member Access Operator

The dot `.` operator provides access to fields of structs, methods, and  
associated functions.

```rust
use std::time::SystemTime;

fn main() {
    let now = SystemTime::now();
    println!("{:?}", now);
    
    let name = "Jane";
    println!("{}", name.to_uppercase());
    println!("{}", name.len());
    
    // Struct field access
    struct Point {
        x: f64,
        y: f64,
    }
    
    let point = Point { x: 3.0, y: 4.0 };
    println!("Point: ({}, {})", point.x, point.y);
}
```

With the dot operator, we access the associated function `now()` of the  
`SystemTime` type. We also access methods on string slices and struct fields.

```
$ cargo run
SystemTime { tv_sec: 1693910445, tv_nsec: 123456789 }
JANE
4
Point: (3, 4)
```

### Method Chaining

Rust supports method chaining when methods return `Self` or other types  
that have methods:

```rust
fn main() {
    let result = "  hello world  "
        .trim()
        .to_uppercase()
        .replace("WORLD", "RUST");
        
    println!("{}", result);
    
    let numbers: Vec<i32> = (1..=5)
        .map(|x| x * x)
        .filter(|&x| x > 10)
        .collect();
        
    println!("{:?}", numbers);
}
```

```
$ cargo run
HELLO RUST
[16, 25]
```

## Rust Option and Error Handling Operators

Rust doesn't have null values, but uses `Option<T>` to represent optional  
values and the `?` operator for error propagation.

### Option Handling

```rust
fn main() {
    let text: Option<&str> = None;
    println!("{:?}", text);
    
    let text2: Option<&str> = Some("falcon");
    println!("{:?}", text2);
    
    // Safe access with pattern matching
    match text2 {
        Some(s) => println!("Length: {}", s.len()),
        None => println!("No text"),
    }
    
    // Using if let
    if let Some(s) = text2 {
        println!("Text: {}", s);
    }
    
    // Using unwrap_or for default values
    let length = text.map(|s| s.len()).unwrap_or(0);
    println!("Length: {}", length);
}
```

```
$ cargo run
None
Some("falcon")
Length: 6
Text: falcon
Length: 0
```

### The ? Operator

The `?` operator is used for error propagation and Option unwrapping:

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_file_contents(filename: &str) -> Result<String, io::Error> {
    let mut file = File::open(filename)?;  // ? propagates error
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;   // ? propagates error
    Ok(contents)
}

fn get_first_word(text: Option<&str>) -> Option<&str> {
    let text = text?;  // Return None if text is None
    text.split_whitespace().next()
}

fn main() {
    match read_file_contents("example.txt") {
        Ok(contents) => println!("File contents: {}", contents),
        Err(e) => println!("Error reading file: {}", e),
    }
    
    let text = Some("hello world");
    let first_word = get_first_word(text);
    println!("First word: {:?}", first_word);
}
```

The `?` operator provides concise error handling without explicit match  
statements for every operation that could fail.

```
$ cargo run
Error reading file: No such file or directory (os error 2)
First word: Some("hello")
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
