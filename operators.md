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

## Rust Indexing and Slicing

Rust provides array and slice indexing with the `[]` operator and slicing  
with range operators.

### Basic Indexing

```rust
fn main() {
    let words = ["sky", "blue", "den", "cloud", "forest"];
    
    println!("{}", words[0]);
    println!("{}", words[1]);
    println!("{}", words[4]);
    
    // Access from end using len()
    println!("{}", words[words.len() - 1]);
    println!("{}", words[words.len() - 2]);
}
```

Unlike some languages, Rust doesn't have a direct "index from end"  
operator, but you can use `len() - n` to access elements from the end.

```
$ cargo run
sky
blue
forest
cloud
```

### Safe Indexing

For safe indexing that doesn't panic, use the `get()` method:

```rust
fn main() {
    let words = ["sky", "blue", "den", "cloud", "forest"];
    
    // Safe indexing returns Option<T>
    match words.get(10) {
        Some(word) => println!("Word at index 10: {}", word),
        None => println!("No word at index 10"),
    }
    
    // Using unwrap_or for default values
    let word = words.get(10).unwrap_or(&"default");
    println!("Word: {}", word);
}
```

```
$ cargo run
No word at index 10
Word: default
```

## Rust Range Operators

Rust provides two range operators: `..` (exclusive end) and `..=` (inclusive end)  
for creating ranges and slicing collections.

### Range Types

```rust
fn main() {
    let vals = [1, 2, 3, 4, 5, 6, 7];
    
    // Slicing with exclusive range [start..end)
    let slice1 = &vals[1..4];
    println!("{:?}", slice1);
    
    // Slicing from beginning [..end)
    let slice2 = &vals[..3];
    println!("{:?}", slice2);
    
    // Slicing to end [start..]
    let slice3 = &vals[2..];
    println!("{:?}", slice3);
    
    // Full slice
    let slice4 = &vals[..];
    println!("{:?}", slice4);
}
```

We create slices using range operators. Note the use of `&` to create  
slice references.

```
$ cargo run
[2, 3, 4]
[1, 2, 3]
[3, 4, 5, 6, 7]
[1, 2, 3, 4, 5, 6, 7]
```

### Inclusive Ranges

```rust
fn main() {
    let vals = [1, 2, 3, 4, 5, 6, 7];
    
    // Inclusive range [start..=end]
    let slice1 = &vals[1..=4];
    println!("{:?}", slice1);
    
    // Inclusive range from beginning [..=end]
    let slice2 = &vals[..=2];
    println!("{:?}", slice2);
}
```

The `..=` operator includes the end value in the range.

```
$ cargo run
[2, 3, 4, 5]
[1, 2, 3]
```

### Ranges for Iteration

```rust
fn main() {
    // Exclusive range
    for i in 1..4 {
        println!("{}", i);
    }
    
    println!("---");
    
    // Inclusive range  
    for i in 1..=3 {
        println!("{}", i);
    }
    
    println!("---");
    
    // Range with step
    for i in (0..10).step_by(2) {
        println!("{}", i);
    }
}
```

Ranges are commonly used in for loops and iterator methods.

```
$ cargo run
1
2
3
---
1
2
3
---
0
2
4
6
8
```

## Rust Type Information and Casting

Rust provides several mechanisms for working with type information and  
performing type conversions.

### Type Annotations and Inference

```rust
fn main() {
    let x = 5i32;      // Explicit type suffix
    let y = 3.2f64;    // Explicit type suffix
    
    // Type annotations
    let a: i32 = 42;
    let b: f64 = 3.14;
    
    println!("x = {} ({})", x, std::any::type_name::<i32>());
    println!("y = {} ({})", y, std::any::type_name::<f64>());
    println!("a = {} ({})", a, std::any::type_name_of_val(&a));
    println!("b = {} ({})", b, std::any::type_name_of_val(&b));
}
```

Rust can often infer types, but explicit annotations are sometimes needed.

```
$ cargo run
x = 5 (i32)
y = 3.2 (f64)
a = 42 (i32)  
b = 3.14 (f64)
```

### Type Casting with as

The `as` operator performs explicit type conversion between compatible types:

```rust
fn main() {
    let x = 42i32;
    let y = x as f64;
    let z = x as u8;
    
    println!("x = {} (i32)", x);
    println!("y = {} (f64)", y);
    println!("z = {} (u8)", z);
    
    // Casting can truncate
    let large = 300i32;
    let truncated = large as u8;
    println!("300 as u8 = {}", truncated); // Wraps around
}
```

```
$ cargo run
x = 42 (i32)
y = 42 (f64)
z = 42 (u8)
300 as u8 = 44
```

### Pattern Matching with Type Checking

Instead of C#'s `is` operator, Rust uses pattern matching:

```rust
use std::any::Any;

fn describe_value(value: &dyn Any) {
    if let Some(s) = value.downcast_ref::<String>() {
        println!("String: {}", s);
    } else if let Some(i) = value.downcast_ref::<i32>() {
        println!("Integer: {}", i);
    } else if let Some(f) = value.downcast_ref::<f64>() {
        println!("Float: {}", f);
    } else {
        println!("Unknown type");
    }
}

fn main() {
    let values: Vec<Box<dyn Any>> = vec![
        Box::new("Hello".to_string()),
        Box::new(42i32),
        Box::new(3.14f64),
        Box::new(true),
    ];
    
    for value in &values {
        describe_value(value.as_ref());
    }
}
```

This example demonstrates runtime type checking using trait objects  
and downcasting.

```
$ cargo run
String: Hello
Integer: 42
Float: 3.14
Unknown type
```

### Safe Conversions with TryFrom/TryInto

For fallible conversions, use `TryFrom` and `TryInto`:

```rust
use std::convert::TryInto;

fn main() {
    let large_number: i32 = 1000;
    
    // Safe conversion that can fail
    let result: Result<u8, _> = large_number.try_into();
    match result {
        Ok(small) => println!("Converted: {}", small),
        Err(e) => println!("Conversion failed: {}", e),
    }
    
    let small_number: i32 = 100;
    let converted: u8 = small_number.try_into().unwrap();
    println!("Successfully converted: {}", converted);
}
```

```
$ cargo run
Conversion failed: out of range integral type conversion attempted
Successfully converted: 100
```

## Rust Operator Precedence

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

The following table shows common Rust operators ordered by precedence  
(highest precedence first):

| Operator(s) | Category | Associativity |
|-------------|----------|---------------|
| Paths | Paths | N/A |
| Method calls, field access, indexing, function calls | Postfix | Left to right |
| `?` | Error propagation | Left to right |
| `-` `!` `*` `&` `&mut` (unary) | Unary prefix | N/A |
| `as` | Type cast | Left to right |
| `*` `/` `%` | Multiplicative | Left to right |
| `+` `-` | Additive | Left to right |
| `<<` `>>` | Shift | Left to right |
| `&` | Bitwise AND | Left to right |
| `^` | Bitwise XOR | Left to right |
| `|` | Bitwise OR | Left to right |
| `==` `!=` `<` `>` `<=` `>=` | Comparison | Left to right |
| `&&` | Logical AND | Left to right |
| `||` | Logical OR | Left to right |
| `..` `..=` | Range | N/A |
| `=` `+=` `-=` `*=` `/=` `%=` etc. | Assignment | Right to left |

Operators on the same row of the table have the same precedence.

```rust
fn main() {
    println!("{}", 3 + 5 * 5);
    println!("{}", (3 + 5) * 5);
    
    let mut a = 0;
    let mut b = 0;
    let mut c = 0;
    let mut d = 0;
    
    a = b = c = d = 5; // This doesn't work in Rust
    // Instead use:
    d = 5;
    c = d;
    b = c;
    a = b;
    
    println!("{} {} {} {}", a, b, c, d);
    
    let mut j = 0;
    j *= 3 + 1;
    
    println!("{}", j);
}
```

Unlike some languages, Rust doesn't support chained assignment.  
Assignment operators are right to left associated.

```
$ cargo run
28
40
5 5 5 5
0
```

### Borrowing and Ownership Precedence

Rust's borrow checker adds additional rules that affect operator precedence:

```rust
fn main() {
    let mut x = 5;
    let y = &mut x;
    
    // The * operator has higher precedence than +
    *y += 1;  // Same as *y = *y + 1;
    
    println!("{}", x);
    
    let arr = [1, 2, 3, 4];
    let slice = &arr[1..3];
    
    // Method calls have higher precedence than indexing
    println!("{}", slice.len()); // Methods called first
    println!("{}", slice[0]);    // Then indexing
}
```

```
$ cargo run
6
2
2
```

## Rust Option Unwrapping and Default Values

Rust uses `Option<T>` instead of nullable types, and provides several  
methods for handling optional values safely.

### unwrap_or and unwrap_or_else

The `unwrap_or` method returns the contained value or a default:

```rust
fn main() {
    let name: Option<&str> = None;
    let name2: Option<&str> = Some("Peter");
    
    println!("{}", name.unwrap_or("unknown"));
    println!("{}", name2.unwrap_or("unknown"));
}
```

The `unwrap_or` method is useful for providing default values for optional  
types.

```
$ cargo run
unknown
Peter
```

### or and or_else for Option Chaining

```rust
fn main() {
    let primary: Option<&str> = None;
    let secondary: Option<&str> = Some("backup");
    let default = Some("default");
    
    let result = primary.or(secondary).or(default);
    println!("{:?}", result);
    
    // Using or_else with a closure
    let result2 = primary.or_else(|| {
        println!("Primary was None, trying secondary");
        secondary
    });
    println!("{:?}", result2);
}
```

```
$ cargo run
Some("backup")
Primary was None, trying secondary
Some("backup")
```

## Rust Option Assignment Patterns

Rust provides several patterns for conditionally assigning values to  
Option types.

### get_or_insert and get_or_insert_with

```rust
fn main() {
    let mut message: Option<String> = None;
    let mut message2: Option<String> = Some("old value".to_string());
    
    // Insert value if None
    message.get_or_insert("new value".to_string());
    message2.get_or_insert("new value".to_string());
    
    println!("{:?}", message);
    println!("{:?}", message2);
}
```

The `get_or_insert` method only assigns if the Option is None.

```
$ cargo run
Some("new value")
Some("old value")
```

### Using if let for Conditional Assignment

```rust
fn main() {
    let mut value: Option<i32> = None;
    
    // Assign only if None
    if value.is_none() {
        value = Some(42);
    }
    
    println!("{:?}", value);
    
    // More idiomatic approach
    let mut value2: Option<i32> = Some(10);
    value2 = value2.or(Some(42));
    
    println!("{:?}", value2);
}
```

```
$ cargo run
Some(42)
Some(10)
```

### Pattern Matching for Complex Assignment

```rust
fn main() {
    let mut config: Option<String> = None;
    
    // Complex conditional assignment
    config = match config {
        Some(existing) => Some(existing), // Keep existing value
        None => Some("default config".to_string()), // Set default
    };
    
    println!("{:?}", config);
}
```

```
$ cargo run
Some("default config")
```

## Rust Pattern Matching

Rust has powerful pattern matching with `match` expressions and `if let`  
statements. Pattern matching allows you to test values against patterns  
and extract information from them.

### Basic Pattern Matching

```rust
fn main() {
    let values: Vec<Box<dyn std::any::Any>> = vec![
        Box::new(1i32),
        Box::new("hello".to_string()),
        Box::new(3.14f64),
        Box::new(true),
    ];
    
    for (i, value) in values.iter().enumerate() {
        print!("Value {}: ", i);
        
        if let Some(int_val) = value.downcast_ref::<i32>() {
            println!("Integer: {}", int_val);
        } else if let Some(string_val) = value.downcast_ref::<String>() {
            println!("String: {}", string_val);
        } else if let Some(float_val) = value.downcast_ref::<f64>() {
            println!("Float: {}", float_val);
        } else if let Some(bool_val) = value.downcast_ref::<bool>() {
            println!("Boolean: {}", bool_val);
        } else {
            println!("Unknown type");
        }
    }
}
```

This example demonstrates pattern matching with type checking using  
trait objects and downcasting.

```
$ cargo run
Value 0: Integer: 1
Value 1: String: hello
Value 2: Float: 3.14
Value 3: Boolean: true
```

### Pattern Matching with Enums

```rust
#[derive(Debug)]
enum Value {
    Integer(i32),
    Text(String),
    Float(f64),
    Boolean(bool),
    Nothing,
}

fn describe_value(value: &Value) -> String {
    match value {
        Value::Integer(i) if *i > 0 => format!("Positive integer: {}", i),
        Value::Integer(i) if *i < 0 => format!("Negative integer: {}", i),
        Value::Integer(_) => "Zero".to_string(),
        Value::Text(s) if !s.is_empty() => format!("Non-empty string: {}", s),
        Value::Text(_) => "Empty string".to_string(),
        Value::Float(f) => format!("Float: {:.2}", f),
        Value::Boolean(b) => format!("Boolean: {}", b),
        Value::Nothing => "Nothing".to_string(),
    }
}

fn main() {
    let values = vec![
        Value::Integer(1),
        Value::Text("hello".to_string()),
        Value::Float(3.14159),
        Value::Boolean(true),
        Value::Nothing,
        Value::Integer(-5),
    ];
    
    for value in &values {
        println!("{}", describe_value(value));
    }
}
```

Pattern matching with guards (conditions) allows for complex matching logic.

```
$ cargo run
Positive integer: 1
Non-empty string: hello
Float: 3.14
Boolean: true
Nothing
Negative integer: -5
```

### Destructuring Patterns

```rust
fn main() {
    let tuple = (1, "hello", 3.14);
    
    // Destructuring in match
    match tuple {
        (1, text, pi) => println!("Found 1, '{}', and {}", text, pi),
        (x, "world", _) => println!("Found {} and 'world'", x),
        _ => println!("Something else"),
    }
    
    // Destructuring in if let
    if let (1, text, _) = tuple {
        println!("First element is 1, text is '{}'", text);
    }
    
    // Destructuring structs
    #[derive(Debug)]
    struct Point { x: i32, y: i32 }
    
    let point = Point { x: 0, y: 7 };
    
    match point {
        Point { x: 0, y } => println!("On the y axis at {}", y),
        Point { x, y: 0 } => println!("On the x axis at {}", x),
        Point { x, y } => println!("Point at ({}, {})", x, y),
    }
}
```

```
$ cargo run
Found 1, 'hello', and 3.14
First element is 1, text is 'hello'
On the y axis at 7
```

## Rust Type Inference and Constructor Patterns

Rust has powerful type inference that allows you to omit type annotations  
in many cases. The compiler can often determine types from context.

### Type Inference

```rust
use std::collections::HashMap;

fn main() {
    // Type inferred from usage
    let mut map = HashMap::new();
    map.insert("key1", 42);  // Compiler infers HashMap<&str, i32>
    
    // Explicit type annotation
    let mut explicit_map: HashMap<String, i32> = HashMap::new();
    explicit_map.insert("key2".to_string(), 100);
    
    // Type inferred from return type
    let numbers = vec![1, 2, 3, 4, 5];  // Vec<i32> inferred
    
    // Turbofish syntax for explicit type parameters
    let parsed: i32 = "42".parse().unwrap();
    let parsed_explicit = "42".parse::<i32>().unwrap();
    
    println!("Map: {:?}", map);
    println!("Explicit map: {:?}", explicit_map);
    println!("Numbers: {:?}", numbers);
    println!("Parsed: {}, {}", parsed, parsed_explicit);
}
```

Rust's type inference reduces verbosity while maintaining type safety.

```
$ cargo run
Map: {"key1": 42}
Explicit map: {"key2": 100}
Numbers: [1, 2, 3, 4, 5]
Parsed: 42, 42
```

### Struct and Enum Construction

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u32,
}

#[derive(Debug)]
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn create_person(name: &str, age: u32) -> Person {
    Person {
        name: name.to_string(),
        age,  // Shorthand when field name matches variable name
    }
}

fn main() {
    // Direct struct construction
    let person1 = Person {
        name: "Alice".to_string(),
        age: 30,
    };
    
    // Using constructor function
    let person2 = create_person("Bob", 25);
    
    // Enum construction
    let messages = vec![
        Message::Quit,
        Message::Move { x: 10, y: 20 },
        Message::Write("Hello".to_string()),
        Message::ChangeColor(255, 0, 0),
    ];
    
    println!("Person 1: {:?}", person1);
    println!("Person 2: {:?}", person2);
    println!("Messages: {:?}", messages);
}
```

```
$ cargo run
Person 1: Person { name: "Alice", age: 30 }
Person 2: Person { name: "Bob", age: 25 }
Messages: [Quit, Move { x: 10, y: 20 }, Write("Hello"), ChangeColor(255, 0, 0)]
```

### Builder Pattern and Method Chaining

```rust
#[derive(Debug, Default)]
struct Config {
    host: String,
    port: u16,
    ssl: bool,
}

impl Config {
    fn new() -> Self {
        Self::default()
    }
    
    fn host(mut self, host: &str) -> Self {
        self.host = host.to_string();
        self
    }
    
    fn port(mut self, port: u16) -> Self {
        self.port = port;
        self
    }
    
    fn ssl(mut self, ssl: bool) -> Self {
        self.ssl = ssl;
        self
    }
}

fn main() {
    let config = Config::new()
        .host("localhost")
        .port(8080)
        .ssl(true);
    
    println!("Config: {:?}", config);
}
```

This pattern provides fluent construction similar to builder patterns  
in other languages.

```
$ cargo run
Config: Config { host: "localhost", port: 8080, ssl: true }
```

## Rust Match Expressions

Match expressions are Rust's primary control flow construct for pattern  
matching. They are more powerful and safer than switch statements in  
many languages.

### Basic Match Expressions

```rust
use std::time::{SystemTime, UNIX_EPOCH};

#[derive(Debug)]
enum Day {
    Monday,
    Tuesday,
    Wednesday,
    Thursday,
    Friday,
    Saturday,
    Sunday,
}

fn main() {
    let today = Day::Wednesday;
    
    let day_type = match today {
        Day::Monday => "Start of work week",
        Day::Tuesday | Day::Wednesday | Day::Thursday => "Midweek",
        Day::Friday => "End of work week",
        Day::Saturday | Day::Sunday => "Weekend",
    };
    
    println!("{}", day_type);
    
    // Match with guards and complex patterns
    let number = 42;
    let description = match number {
        n if n < 0 => "Negative",
        0 => "Zero",
        1..=10 => "Small positive",
        11..=100 => "Medium positive",
        _ => "Large positive",
    };
    
    println!("Number {} is {}", number, description);
}
```

Match expressions must be exhaustive - they must handle all possible values.

```
$ cargo run
Midweek
Number 42 is Medium positive
```

### Matching with Data Extraction

```rust
#[derive(Debug)]
enum Shape {
    Circle { radius: f64 },
    Rectangle { width: f64, height: f64 },
    Triangle { base: f64, height: f64 },
}

fn calculate_area(shape: &Shape) -> f64 {
    match shape {
        Shape::Circle { radius } => std::f64::consts::PI * radius * radius,
        Shape::Rectangle { width, height } => width * height,
        Shape::Triangle { base, height } => 0.5 * base * height,
    }
}

fn describe_shape(shape: &Shape) -> String {
    match shape {
        Shape::Circle { radius } if *radius > 10.0 => {
            format!("Large circle with radius {:.1}", radius)
        },
        Shape::Circle { radius } => {
            format!("Small circle with radius {:.1}", radius)
        },
        Shape::Rectangle { width, height } if width == height => {
            format!("Square with side {:.1}", width)
        },
        Shape::Rectangle { width, height } => {
            format!("Rectangle {}x{}", width, height)
        },
        Shape::Triangle { base, height } => {
            format!("Triangle with base {:.1} and height {:.1}", base, height)
        },
    }
}

fn main() {
    let shapes = vec![
        Shape::Circle { radius: 5.0 },
        Shape::Rectangle { width: 4.0, height: 4.0 },
        Shape::Triangle { base: 3.0, height: 6.0 },
    ];
    
    for shape in &shapes {
        println!("{}: area = {:.2}", describe_shape(shape), calculate_area(shape));
    }
}
```

Match expressions can extract data from complex patterns and apply guards  
for additional conditions.

```
$ cargo run
Small circle with radius 5.0: area = 78.54
Square with side 4.0: area = 16.00
Triangle with base 3.0 and height 6.0: area = 9.00
```

### if let for Simple Matches

For simple matches with only one pattern, `if let` is more concise:

```rust
fn main() {
    let maybe_number: Option<i32> = Some(42);
    
    // Using match
    match maybe_number {
        Some(n) => println!("Got number: {}", n),
        None => println!("No number"),
    }
    
    // Using if let (more concise for single pattern)
    if let Some(n) = maybe_number {
        println!("Got number with if let: {}", n);
    } else {
        println!("No number");
    }
    
    // while let for iterating until pattern fails
    let mut stack = vec![1, 2, 3];
    while let Some(top) = stack.pop() {
        println!("Popped: {}", top);
    }
}
```

```
$ cargo run
Got number: 42
Got number with if let: 42
Popped: 3
Popped: 2
Popped: 1
```

This concludes our overview of Rust operators. Understanding operator  
precedence, pattern matching, and the various types of operators is  
essential for writing effective Rust code. Rust's type system, ownership  
model, and pattern matching make it both safe and expressive, allowing  
developers to write efficient code with strong compile-time guarantees.
