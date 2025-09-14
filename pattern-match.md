# Pattern matching

Pattern matching is a powerful control flow construct in Rust that allows you  
to compare a value against a series of patterns and execute code based on which  
pattern matches. It's more powerful than simple if-else statements because it  
can destructure data and extract values from complex types like enums, structs,  
and tuples.  

The `match` expression is exhaustive, meaning it must cover all possible cases  
for the type being matched. This ensures that your code handles every scenario,  
making it safer and more reliable. Rust's pattern matching is inspired by  
functional programming languages and provides a clean, readable way to handle  
complex branching logic.  

When a pattern matches, the corresponding code block executes, and the match  
expression returns the value of that block. This makes it both a control flow  
mechanism and an expression that can produce values, making your code more  
concise and functional in style.  

## Basic String Matching

This example demonstrates simple pattern matching with string literals and the  
use of the pipe operator (`|`) to match multiple patterns in a single arm.  

```rust
fn main() {
    let grades = vec!["A", "B", "C", "D", "E", "F", "FX"];
  
    for grade in grades {
        match grade {
            "A" | "B" | "C" | "D" | "E" | "F" => println!("passed"),
            "FX" => println!("failed"),
            _ => println!("invalid"),
        }
    }
}
```

The pipe operator (`|`) allows multiple patterns to be matched in a single arm.  
The underscore (`_`) is a wildcard that matches any value not covered by  
previous patterns, ensuring the match is exhaustive.  

## Guard Patterns with Numeric Ranges

This example shows how to use guard patterns with the `if` keyword to add  
additional conditions to pattern matching arms.  

```rust
use rand::Rng;

fn main() {
    let mut rng = rand::rng();
    let r = rng.random_range(-5..=5);

    match r {
        x if x > 0 => println!("The number {} is positive", x),
        0 => println!("The number is zero"),
        _ => println!("The number {} is negative", r),
    }
}
```

Guard patterns combine pattern matching with boolean conditions. The variable  
`x` binds to the matched value, and the `if` clause provides an additional  
condition that must be true for the pattern to match.  

## Example 1: Simple Value Matching

This example demonstrates basic pattern matching with integer literals and  
ranges, showing how to match exact values and ranges of values.  

```rust
fn main() {
    let number = 7;
    
    match number {
        1 => println!("One!"),
        2 | 3 | 5 | 7 | 11 => println!("This is a prime"),
        13..=19 => println!("A teen"),
        _ => println!("Ain't special"),
    }
    
    let boolean = true;
    match boolean {
        false => println!("The boolean is false"),
        true => println!("The boolean is true"),
    }
}
```

This example shows exact value matching, multiple value matching with the pipe  
operator, range matching with `..=`, and boolean matching. The range pattern  
`13..=19` matches any number from 13 to 19 inclusive.  

## Example 2: Option Enum Matching

This example demonstrates pattern matching with Rust's `Option` enum, showing  
how to safely handle potentially absent values.  

```rust
fn main() {
    let some_number = Some(5);
    let some_string = Some("hello");
    let absent_number: Option<i32> = None;
    
    match some_number {
        Some(x) if x < 10 => println!("Got a small number: {}", x),
        Some(x) => println!("Got a big number: {}", x),
        None => println!("No number provided"),
    }
    
    // Using match as an expression to return values
    let message = match some_string {
        Some(text) => format!("Message: {}", text),
        None => String::from("No message"),
    };
    println!("{}", message);
    
    // Demonstrating pattern matching with None
    match absent_number {
        Some(value) => println!("Found value: {}", value),
        None => println!("No value found"),
    }
}
```

This example shows how to destructure `Option` values to extract the inner  
data when present. The `Some(x)` pattern binds the inner value to `x`, while  
`None` handles the absence of a value. Notice how match can be used as an  
expression to return values, making the code more functional.  

## Example 3: Destructuring Tuples and Structs

This example shows how to destructure complex data types like tuples and  
structs to extract their individual components.  

```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    // Tuple destructuring
    let tuple = (1, "hello", 3.14);
    
    match tuple {
        (1, text, _) => println!("First element is 1, text is: {}", text),
        (_, "world", pi) => println!("Second element is 'world', pi is: {}", pi),
        (x, y, z) => println!("Tuple contents: {}, {}, {}", x, y, z),
    }
    
    // Struct destructuring
    let point = Point { x: 10, y: 20 };
    
    match point {
        Point { x: 0, y: 0 } => println!("Origin point"),
        Point { x: 0, y } => println!("On Y-axis at y = {}", y),
        Point { x, y: 0 } => println!("On X-axis at x = {}", x),
        Point { x, y } if x == y => println!("On diagonal at ({}, {})", x, y),
        Point { x, y } => println!("Point at ({}, {})", x, y),
    }
    
    // Partial destructuring with ..
    let complex_tuple = (1, 2, 3, 4, 5);
    match complex_tuple {
        (first, .., last) => {
            println!("First: {}, Last: {}", first, last);
        }
    }
}
```

This example demonstrates destructuring patterns that extract values from  
tuples and structs. The underscore `_` ignores values, while `..` ignores  
multiple values in sequences. Struct patterns can match specific field values  
or bind field values to variables.  

## Example 4: Advanced Guard Patterns and References

This example shows more complex guard patterns and how to work with references  
in pattern matching.  

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];
    let reference = &numbers[2];
    
    // Matching references and dereferencing
    match reference {
        &val if val > 2 => println!("Reference to value greater than 2: {}", val),
        &val => println!("Reference to value: {}", val),
    }
    
    // Complex guard patterns with multiple conditions
    let pair = (3i32, -2i32);
    match pair {
        (x, y) if x > 0 && y < 0 => {
            println!("First positive ({}), second negative ({})", x, y);
        }
        (x, y) if x == y => println!("Equal values: {}", x),
        (x, y) if x.abs() == y.abs() => {
            println!("Same absolute value: {} and {}", x, y);
        }
        (x, y) => println!("Other case: ({}, {})", x, y),
    }
    
    // Pattern matching with different data types
    let data: Result<i32, &str> = Ok(42);
    match data {
        Ok(value) if value > 40 => println!("Large success value: {}", value),
        Ok(value) => println!("Success with value: {}", value),
        Err(error) => println!("Error occurred: {}", error),
    }
}
```

This example shows advanced guard patterns with multiple conditions using  
logical operators. It also demonstrates pattern matching with references using  
the `&` pattern, and shows how to match `Result` types for error handling.  

## Example 5: Nested Pattern Matching with Enums

This final example demonstrates the most advanced pattern matching features  
with custom enums and nested destructuring.  

```rust
#[derive(Debug)]
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
    Nested(Box<Message>),
}

#[derive(Debug)]
enum Shape {
    Circle(f64),
    Rectangle { width: f64, height: f64 },
    Triangle(f64, f64, f64),
}

fn main() {
    // Complex enum pattern matching
    let messages = vec![
        Message::Quit,
        Message::Move { x: 10, y: 20 },
        Message::Write(String::from("Hello, World!")),
        Message::ChangeColor(255, 0, 128),
        Message::Nested(Box::new(Message::Write(String::from("Nested!")))),
    ];
    
    for msg in messages {
        match msg {
            Message::Quit => println!("Received quit message"),
            Message::Move { x, y } if x > 0 && y > 0 => {
                println!("Moving to positive coordinates: ({}, {})", x, y);
            }
            Message::Move { x, y } => {
                println!("Moving to coordinates: ({}, {})", x, y);
            }
            Message::Write(text) if text.len() > 10 => {
                println!("Long message: '{}'", text);
            }
            Message::Write(text) => println!("Short message: '{}'", text),
            Message::ChangeColor(r, g, b) => {
                println!("Changing color to RGB({}, {}, {})", r, g, b);
            }
            Message::Nested(inner_msg) => {
                println!("Processing nested message:");
                match *inner_msg {
                    Message::Write(ref text) => {
                        println!("  Nested text: '{}'", text);
                    }
                    other => println!("  Other nested message: {:?}", other),
                }
            }
        }
    }
    
    // Pattern matching with calculated values
    let shapes = vec![
        Shape::Circle(5.0),
        Shape::Rectangle { width: 10.0, height: 5.0 },
        Shape::Triangle(3.0, 4.0, 5.0),
    ];
    
    for shape in shapes {
        let area = match shape {
            Shape::Circle(radius) => std::f64::consts::PI * radius * radius,
            Shape::Rectangle { width, height } => width * height,
            Shape::Triangle(a, b, c) => {
                // Using Heron's formula
                let s = (a + b + c) / 2.0;
                (s * (s - a) * (s - b) * (s - c)).sqrt()
            }
        };
        println!("Shape: {:?}, Area: {:.2}", shape, area);
    }
}
```

This advanced example demonstrates nested pattern matching with custom enums,  
destructuring of struct-like enum variants, matching with `Box` types, and  
using pattern matching to calculate values. It shows how pattern matching can  
handle complex nested structures and combine destructuring with computation.  

