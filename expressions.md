# Rust Expressions

Expressions are constructs that evaluate to a value in Rust. Unlike statements,  
which perform actions, expressions produce values and can be used wherever a  
value is expected. Understanding expressions is fundamental to Rust programming  
because almost everything in Rust that produces a value is an expression.  

In Rust, expressions can be as simple as a literal value or as complex as a  
function call or control flow construct. Expressions can be combined to create  
larger expressions, and most control flow constructs like `if`, `match`, and  
loops are expressions that return values.  

## Basic Literal Expressions

The simplest expressions are literal values that evaluate directly to themselves.  

```rust
fn main() {
    let number = 42;          // Integer literal expression
    let float = 3.14;         // Float literal expression
    let text = "Hello";       // String literal expression
    let boolean = true;       // Boolean literal expression
    let character = 'A';      // Character literal expression
    
    println!("Number: {}", number);
    println!("Float: {}", float);
    println!("Text: {}", text);
    println!("Boolean: {}", boolean);
    println!("Character: {}", character);
}
```

Literal expressions are the foundation of all other expressions. They directly  
represent values in your code and are evaluated at compile time when possible.  
Each literal has a specific type that Rust can infer automatically.  

## Arithmetic Expressions

Arithmetic expressions combine operands with mathematical operators to produce  
numeric results.  

```rust
fn main() {
    let a = 10;
    let b = 3;
    
    let addition = a + b;        // Addition expression
    let subtraction = a - b;     // Subtraction expression
    let multiplication = a * b;  // Multiplication expression
    let division = a / b;        // Division expression
    let remainder = a % b;       // Modulus expression
    
    println!("Addition: {} + {} = {}", a, b, addition);
    println!("Subtraction: {} - {} = {}", a, b, subtraction);
    println!("Multiplication: {} * {} = {}", a, b, multiplication);
    println!("Division: {} / {} = {}", a, b, division);
    println!("Remainder: {} % {} = {}", a, b, remainder);
    
    // Complex arithmetic expression
    let result = (a + b) * 2 - a / b;
    println!("Complex: ({} + {}) * 2 - {} / {} = {}", a, b, a, b, result);
}
```

Arithmetic expressions follow standard mathematical precedence rules. You can  
use parentheses to control evaluation order. The result type depends on the  
operand types, and Rust performs type checking to ensure compatibility.  

## Assignment Expressions

Assignment expressions assign values to variables and return the assigned value.  

```rust
fn main() {
    let mut x = 10;
    let mut y = 20;
    
    // Basic assignment expressions
    x = 15;                    // Assignment returns ()
    y = x + 5;                 // Expression result assigned to y
    
    // Compound assignment expressions
    x += 10;                   // Equivalent to x = x + 10
    y *= 2;                    // Equivalent to y = y * 2
    x -= 3;                    // Equivalent to x = x - 3
    y /= 4;                    // Equivalent to y = y / 4
    
    println!("x = {}, y = {}", x, y);
    
    // Multiple assignment in one expression
    let a;
    let b;
    a = b = 42;               // Both a and b get value 42
    println!("a = {}, b = {}", a, b);
}
```

Assignment expressions modify variables and can be chained together. Compound  
assignment operators provide convenient shortcuts for common operations like  
incrementing or scaling values.  

## Conditional Expressions

Conditional expressions evaluate different branches based on boolean conditions  
and return values.  

```rust
fn main() {
    let temperature = 25;
    
    // Simple if expression
    let weather = if temperature > 20 {
        "warm"
    } else {
        "cold"
    };
    
    println!("The weather is {}", weather);
    
    // Multi-branch if expression
    let description = if temperature < 0 {
        "freezing"
    } else if temperature < 10 {
        "cold"
    } else if temperature < 25 {
        "mild"
    } else {
        "hot"
    };
    
    println!("Temperature description: {}", description);
    
    // Conditional expression in calculation
    let adjusted_temp = temperature + if temperature < 20 { 5 } else { -2 };
    println!("Adjusted temperature: {}", adjusted_temp);
}
```

If expressions must have all branches return the same type. The condition must  
evaluate to a boolean value. These expressions are commonly used for making  
decisions and selecting between alternative values.  

## Block Expressions

Block expressions group statements and expressions together, evaluating to the  
value of their final expression.  

```rust
fn main() {
    // Simple block expression
    let result = {
        let x = 10;
        let y = 20;
        x + y                  // Final expression (no semicolon)
    };
    
    println!("Block result: {}", result);
    
    // Block with intermediate calculations
    let complex_calculation = {
        let base = 5;
        let multiplier = 3;
        let offset = 2;
        
        println!("Calculating: ({} * {}) + {}", base, multiplier, offset);
        base * multiplier + offset    // Returned value
    };
    
    println!("Complex calculation: {}", complex_calculation);
    
    // Conditional block expressions
    let category = {
        let score = 85;
        if score >= 90 {
            "Excellent"
        } else if score >= 80 {
            "Good"
        } else {
            "Needs improvement"
        }
    };
    
    println!("Category: {}", category);
}
```

Block expressions create new scopes and can contain multiple statements. The  
final expression (without semicolon) becomes the value of the entire block.  
This enables complex calculations while maintaining clean, readable code.  

## Function Call Expressions

Function call expressions invoke functions and evaluate to their return values.  

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

fn multiply_by_two(x: i32) -> i32 {
    x * 2
}

fn greet(name: &str) -> String {
    format!("Hello, {}!", name)
}

fn main() {
    // Simple function call expressions
    let sum = add(5, 3);
    println!("Sum: {}", sum);
    
    // Chained function call expressions
    let doubled_sum = multiply_by_two(add(4, 6));
    println!("Doubled sum: {}", doubled_sum);
    
    // Function call in complex expression
    let result = add(10, 20) * multiply_by_two(3) - 15;
    println!("Complex result: {}", result);
    
    // Function returning string
    let greeting = greet("Alice");
    println!("{}", greeting);
    
    // Function call as argument to another function
    let nested_greeting = greet(&format!("User{}", 42));
    println!("{}", nested_greeting);
}
```

Function call expressions can be nested and combined with other expressions.  
The arguments to functions are expressions themselves, allowing for complex  
parameter calculations. Functions can return any type, making them versatile  
building blocks for larger expressions.  

## Method Call Expressions

Method call expressions invoke methods on objects using dot notation.  

```rust
fn main() {
    let text = "Hello, World!";
    let numbers = vec![1, 2, 3, 4, 5];
    let mut mutable_vec = vec![10, 20, 30];
    
    // String method call expressions
    let length = text.len();
    let uppercase = text.to_uppercase();
    let contains_world = text.contains("World");
    
    println!("Text: '{}'", text);
    println!("Length: {}", length);
    println!("Uppercase: '{}'", uppercase);
    println!("Contains 'World': {}", contains_world);
    
    // Vector method call expressions
    let first = numbers.first();
    let sum: i32 = numbers.iter().sum();
    let doubled: Vec<i32> = numbers.iter().map(|x| x * 2).collect();
    
    println!("Numbers: {:?}", numbers);
    println!("First element: {:?}", first);
    println!("Sum: {}", sum);
    println!("Doubled: {:?}", doubled);
    
    // Mutable method calls
    mutable_vec.push(40);
    mutable_vec.sort();
    let popped = mutable_vec.pop();
    
    println!("After operations: {:?}", mutable_vec);
    println!("Popped: {:?}", popped);
}
```

Method calls are expressions that operate on specific instances of types. They  
can be chained together for fluent programming style. Some methods modify the  
object (mutable methods), while others return new values without changing the  
original object.  

## Closure Expressions

Closure expressions create anonymous functions that can capture variables from  
their surrounding scope.  

```rust
fn main() {
    let multiplier = 3;
    let numbers = vec![1, 2, 3, 4, 5];
    
    // Simple closure expression
    let double = |x| x * 2;
    let result = double(5);
    println!("Double of 5: {}", result);
    
    // Closure capturing environment
    let multiply_by_factor = |x| x * multiplier;
    let scaled = multiply_by_factor(7);
    println!("7 * {} = {}", multiplier, scaled);
    
    // Closure with explicit types
    let add_one: fn(i32) -> i32 = |x: i32| -> i32 { x + 1 };
    println!("Add one to 10: {}", add_one(10));
    
    // Closures in iterator chains
    let doubled_evens: Vec<i32> = numbers
        .iter()
        .filter(|&&x| x % 2 == 0)    // Filter even numbers
        .map(|&x| x * 2)             // Double them
        .collect();
    
    println!("Original: {:?}", numbers);
    println!("Doubled evens: {:?}", doubled_evens);
    
    // Multi-statement closure
    let complex_operation = |x: i32| {
        let doubled = x * 2;
        let plus_ten = doubled + 10;
        plus_ten * plus_ten
    };
    
    println!("Complex operation on 3: {}", complex_operation(3));
}
```

Closures are powerful expressions that create functions on-the-fly. They can  
capture variables from their environment and are commonly used with iterators  
and functional programming patterns. The syntax allows for both simple and  
complex function definitions inline.  

## Array and Tuple Indexing Expressions

Indexing expressions access elements from arrays, tuples, and other indexable  
collections.  

```rust
fn main() {
    let array = [10, 20, 30, 40, 50];
    let tuple = (1, "hello", 3.14, true);
    let nested = [[1, 2], [3, 4], [5, 6]];
    
    // Array indexing expressions
    let first = array[0];
    let third = array[2];
    let last = array[array.len() - 1];
    
    println!("Array: {:?}", array);
    println!("First: {}, Third: {}, Last: {}", first, third, last);
    
    // Tuple indexing expressions
    let number = tuple.0;
    let text = tuple.1;
    let pi = tuple.2;
    let flag = tuple.3;
    
    println!("Tuple elements: {}, '{}', {}, {}", number, text, pi, flag);
    
    // Nested indexing expressions
    let element = nested[1][0];
    let another = nested[2][1];
    
    println!("Nested array element [1][0]: {}", element);
    println!("Nested array element [2][1]: {}", another);
    
    // Dynamic indexing with variables
    let index = 3;
    let dynamic_access = array[index];
    println!("Element at index {}: {}", index, dynamic_access);
    
    // Indexing in expressions
    let sum = array[0] + array[1] + array[2];
    println!("Sum of first three elements: {}", sum);
}
```

Indexing expressions provide direct access to elements in collections. Array  
indices must be valid at runtime or the program will panic. Tuple indexing  
uses numeric literals and is checked at compile time for valid field access.  

## Range Expressions

Range expressions create iterator-like objects that represent sequences of values.  

```rust
fn main() {
    // Exclusive range expressions
    let range1 = 1..5;          // 1, 2, 3, 4 (excludes 5)
    let range2 = 0..10;         // 0 through 9
    
    println!("Range 1..5:");
    for n in range1 {
        print!("{} ", n);
    }
    println!();
    
    // Inclusive range expressions
    let inclusive_range = 1..=5;    // 1, 2, 3, 4, 5 (includes 5)
    println!("Inclusive range 1..=5:");
    for n in inclusive_range {
        print!("{} ", n);
    }
    println!();
    
    // Range expressions in collections
    let slice = &[10, 20, 30, 40, 50, 60];
    let partial = &slice[1..4];     // Elements at indices 1, 2, 3
    println!("Original slice: {:?}", slice);
    println!("Partial slice [1..4]: {:?}", partial);
    
    // Open-ended ranges
    let from_start = &slice[..3];   // From beginning to index 2
    let to_end = &slice[2..];       // From index 2 to end
    let full = &slice[..];          // Full slice
    
    println!("From start [..3]: {:?}", from_start);
    println!("To end [2..]: {:?}", to_end);
    println!("Full slice [..]: {:?}", full);
    
    // Range expressions in match
    let number = 42;
    let category = match number {
        1..=10 => "small",
        11..=50 => "medium",
        51..=100 => "large",
        _ => "very large",
    };
    println!("Number {} is {}", number, category);
}
```

Range expressions are versatile for creating sequences, slicing collections,  
and pattern matching. They come in exclusive (`..`) and inclusive (`..=`)  
variants, and can be open-ended for slicing operations.  

## Reference and Dereference Expressions

Reference and dereference expressions work with borrowed values and pointers.  

```rust
fn main() {
    let value = 42;
    let mut mutable_value = 100;
    
    // Reference expressions
    let reference = &value;              // Immutable reference
    let mutable_reference = &mut mutable_value;  // Mutable reference
    
    println!("Original value: {}", value);
    println!("Reference: {}", reference);
    println!("Mutable reference: {}", mutable_reference);
    
    // Dereference expressions
    let dereferenced = *reference;
    println!("Dereferenced value: {}", dereferenced);
    
    // Modifying through mutable reference
    *mutable_reference += 50;
    println!("Modified through reference: {}", mutable_value);
    
    // Reference to reference
    let ref_to_ref = &reference;
    let double_deref = **ref_to_ref;
    println!("Double dereference: {}", double_deref);
    
    // References in function calls
    fn add_ten(x: &mut i32) {
        *x += 10;
    }
    
    add_ten(&mut mutable_value);
    println!("After function call: {}", mutable_value);
    
    // Array element references
    let array = [1, 2, 3, 4, 5];
    let element_ref = &array[2];
    println!("Reference to array element: {}", element_ref);
    
    // Reference patterns
    let numbers = vec![10, 20, 30];
    for item_ref in &numbers {
        println!("Item reference: {}", item_ref);
    }
}
```

Reference expressions create borrowed pointers to values, while dereference  
expressions access the values those pointers refer to. This system enables  
safe memory management and allows functions to work with borrowed data without  
taking ownership.  

## Type Casting Expressions

Type casting expressions convert values from one type to another using the  
`as` keyword.  

```rust
fn main() {
    // Numeric type casting
    let integer = 42i32;
    let float = integer as f64;
    let byte = integer as u8;
    let large_int = integer as i64;
    
    println!("Original integer: {} (i32)", integer);
    println!("As float: {} (f64)", float);
    println!("As byte: {} (u8)", byte);
    println!("As large int: {} (i64)", large_int);
    
    // Floating point to integer casting
    let pi = 3.14159f64;
    let truncated = pi as i32;          // Truncates decimal part
    let rounded = (pi + 0.5) as i32;    // Manual rounding
    
    println!("Pi: {}", pi);
    println!("Truncated: {}", truncated);
    println!("Rounded: {}", rounded);
    
    // Character and byte casting
    let character = 'A';
    let ascii_value = character as u8;
    let back_to_char = ascii_value as char;
    
    println!("Character: '{}'", character);
    println!("ASCII value: {}", ascii_value);
    println!("Back to char: '{}'", back_to_char);
    
    // Pointer casting (unsafe)
    let number = 42i32;
    let ptr = &number as *const i32;
    
    println!("Number address: {:p}", ptr);
    
    // Casting in expressions
    let calculation = (10.7 as i32) + (3.9 as i32) * 2;
    println!("Calculation with casting: {}", calculation);
    
    // Size casting for array indexing
    let array = [1, 2, 3, 4, 5];
    let index = 2.5f32 as usize;
    println!("Array element at cast index: {}", array[index]);
}
```

Type casting expressions allow explicit conversion between compatible types.  
Numeric conversions may truncate or change precision. Some casts require  
unsafe blocks when converting between pointer types or performing potentially  
dangerous operations.  

## Logical and Comparison Expressions

Logical and comparison expressions evaluate boolean conditions and combine  
boolean values.  

```rust
fn main() {
    let a = 10;
    let b = 20;
    let c = 10;
    
    // Comparison expressions
    let equal = a == c;              // Equality
    let not_equal = a != b;          // Inequality
    let less_than = a < b;           // Less than
    let greater_than = b > a;        // Greater than
    let less_equal = a <= c;         // Less than or equal
    let greater_equal = b >= a;      // Greater than or equal
    
    println!("a = {}, b = {}, c = {}", a, b, c);
    println!("a == c: {}", equal);
    println!("a != b: {}", not_equal);
    println!("a < b: {}", less_than);
    println!("b > a: {}", greater_than);
    println!("a <= c: {}", less_equal);
    println!("b >= a: {}", greater_equal);
    
    // Logical expressions
    let and_result = (a < b) && (b > c);     // Logical AND
    let or_result = (a == b) || (a == c);    // Logical OR
    let not_result = !(a == b);              // Logical NOT
    
    println!("(a < b) && (b > c): {}", and_result);
    println!("(a == b) || (a == c): {}", or_result);
    println!("!(a == b): {}", not_result);
    
    // Short-circuit evaluation
    let x = 5;
    let result = (x > 0) && (10 / x > 1);   // Safe because first condition true
    println!("Short-circuit result: {}", result);
    
    // Complex logical expressions
    let age = 25;
    let has_license = true;
    let has_insurance = true;
    
    let can_drive = (age >= 18) && has_license && has_insurance;
    let needs_supervision = (age >= 16) && (age < 18) && has_license;
    
    println!("Can drive independently: {}", can_drive);
    println!("Needs supervision: {}", needs_supervision);
}
```

Comparison expressions test relationships between values, while logical  
expressions combine boolean values. Rust uses short-circuit evaluation for  
`&&` and `||` operators, meaning the second operand is only evaluated if  
necessary.  

## Pattern Matching Expressions

Pattern matching expressions use `match` to compare values against patterns  
and execute corresponding code blocks.  

```rust
enum Direction {
    North,
    South,
    East,
    West,
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    // Simple pattern matching
    let number = 7;
    let description = match number {
        1 => "one",
        2 | 3 | 5 | 7 | 11 => "prime",
        13..=19 => "teen",
        _ => "other",
    };
    println!("Number {} is {}", number, description);
    
    // Enum pattern matching
    let direction = Direction::North;
    let movement = match direction {
        Direction::North => (0, 1),
        Direction::South => (0, -1),
        Direction::East => (1, 0),
        Direction::West => (-1, 0),
    };
    println!("Direction movement: {:?}", movement);
    
    // Pattern matching with data extraction
    let message = Message::Move { x: 10, y: 20 };
    match message {
        Message::Quit => println!("Quit message"),
        Message::Move { x, y } => println!("Move to ({}, {})", x, y),
        Message::Write(text) => println!("Write: {}", text),
        Message::ChangeColor(r, g, b) => println!("Color: rgb({}, {}, {})", r, g, b),
    }
    
    // Pattern matching with guards
    let temperature = 25;
    let weather = match temperature {
        t if t < 0 => "freezing",
        t if t < 10 => "cold",
        t if t < 25 => "mild",
        t if t < 35 => "warm",
        _ => "hot",
    };
    println!("Temperature {} is {}", temperature, weather);
    
    // Tuple pattern matching
    let point = (3, 4);
    let location = match point {
        (0, 0) => "origin",
        (0, y) => "on y-axis",
        (x, 0) => "on x-axis",
        (x, y) if x == y => "diagonal",
        (x, y) => "general point",
    };
    println!("Point {:?} is at {}", point, location);
}
```

Pattern matching expressions provide powerful control flow that can destructure  
data and bind variables. They must be exhaustive, covering all possible cases.  
Guards allow additional conditions beyond pattern structure.  

## Loop Expressions

Loop expressions create iterative control flow and can return values when  
broken with a value.  

```rust
fn main() {
    // Basic loop expression with break value
    let result = loop {
        let random = 42; // Simulated random number
        if random > 30 {
            break random * 2;  // Break with value
        }
    };
    println!("Loop result: {}", result);
    
    // While loop expression
    let mut counter = 0;
    let limit = while counter < 5 {
        counter += 1;
        if counter == 3 {
            break counter * 10;  // While can also break with value
        }
    };
    println!("While loop result: {:?}", limit);
    
    // For loop with iterator
    let numbers = vec![1, 2, 3, 4, 5];
    let mut sum = 0;
    for num in &numbers {
        sum += num;
    }
    println!("Sum from for loop: {}", sum);
    
    // Nested loops with labeled breaks
    let mut found = None;
    'outer: for i in 1..=5 {
        for j in 1..=5 {
            if i * j == 12 {
                found = Some((i, j));
                break 'outer;
            }
        }
    }
    println!("Found pair: {:?}", found);
    
    // Loop expression returning Option
    let search_value = 3;
    let position = 'search: loop {
        for (index, &value) in numbers.iter().enumerate() {
            if value == search_value {
                break 'search Some(index);
            }
        }
        break None;
    };
    println!("Position of {}: {:?}", search_value, position);
    
    // Infinite loop with conditional break
    let mut attempts = 0;
    let success = loop {
        attempts += 1;
        println!("Attempt {}", attempts);
        
        if attempts == 3 {
            break true;
        }
        
        if attempts > 10 {
            break false;
        }
    };
    println!("Success after {} attempts: {}", attempts, success);
}
```

Loop expressions can return values using labeled breaks. The `loop` keyword  
creates infinite loops, while `while` and `for` provide conditional iteration.  
Labels allow breaking from nested loops and returning values from specific  
loop levels.  

## Error Handling Expressions

Error handling expressions manage `Result` and `Option` types using operators  
like `?`, `unwrap`, and `expect`.  

```rust
use std::fs;
use std::io;

fn read_file_length(filename: &str) -> Result<usize, io::Error> {
    let contents = fs::read_to_string(filename)?;  // ? operator
    Ok(contents.len())
}

fn safe_divide(a: f64, b: f64) -> Option<f64> {
    if b != 0.0 {
        Some(a / b)
    } else {
        None
    }
}

fn main() {
    // Using ? operator in expressions
    let result = match read_file_length("example.txt") {
        Ok(length) => format!("File length: {}", length),
        Err(error) => format!("Error: {}", error),
    };
    println!("{}", result);
    
    // Option handling expressions
    let numbers = vec![Some(1), None, Some(3), Some(4)];
    let valid_numbers: Vec<i32> = numbers
        .into_iter()
        .filter_map(|x| x)  // Extract Some values, filter out None
        .collect();
    println!("Valid numbers: {:?}", valid_numbers);
    
    // Unwrap expressions (use with caution)
    let some_value = Some(42);
    let unwrapped = some_value.unwrap();  // Panics if None
    println!("Unwrapped value: {}", unwrapped);
    
    // Expect expressions with custom messages
    let another_value = Some("hello");
    let expected = another_value.expect("Value should exist");
    println!("Expected value: {}", expected);
    
    // Unwrap_or expressions with defaults
    let maybe_number: Option<i32> = None;
    let with_default = maybe_number.unwrap_or(0);
    println!("Value with default: {}", with_default);
    
    // Chaining error handling expressions
    let calculation_result = safe_divide(10.0, 2.0)
        .and_then(|x| safe_divide(x, 0.0))  // This will be None
        .unwrap_or(0.0);
    println!("Calculation result: {}", calculation_result);
    
    // Match expressions for comprehensive error handling
    let division = safe_divide(15.0, 3.0);
    let message = match division {
        Some(result) => format!("Division successful: {}", result),
        None => "Division by zero".to_string(),
    };
    println!("{}", message);
}
```

Error handling expressions provide safe ways to work with fallible operations.  
The `?` operator propagates errors up the call stack, while `unwrap` family  
methods extract values with various fallback strategies. Pattern matching  
gives complete control over error scenarios.  

## Struct and Enum Construction Expressions

Construction expressions create instances of structs and enums with their  
associated data.  

```rust
// Define structs and enums
#[derive(Debug)]
struct Point {
    x: f64,
    y: f64,
}

#[derive(Debug)]
struct Rectangle {
    top_left: Point,
    bottom_right: Point,
}

#[derive(Debug)]
enum Shape {
    Circle { center: Point, radius: f64 },
    Rectangle(Rectangle),
    Triangle { a: Point, b: Point, c: Point },
}

impl Point {
    fn new(x: f64, y: f64) -> Self {
        Point { x, y }
    }
    
    fn distance_from_origin(&self) -> f64 {
        (self.x * self.x + self.y * self.y).sqrt()
    }
}

fn main() {
    // Struct construction expressions
    let origin = Point { x: 0.0, y: 0.0 };
    let point1 = Point { x: 3.0, y: 4.0 };
    
    println!("Origin: {:?}", origin);
    println!("Point 1: {:?}", point1);
    
    // Struct construction with method call
    let point2 = Point::new(5.0, 12.0);
    println!("Point 2: {:?}", point2);
    
    // Struct update syntax
    let point3 = Point { x: 1.0, ..origin };
    println!("Point 3: {:?}", point3);
    
    // Nested struct construction
    let rect = Rectangle {
        top_left: Point { x: 0.0, y: 5.0 },
        bottom_right: Point { x: 10.0, y: 0.0 },
    };
    println!("Rectangle: {:?}", rect);
    
    // Enum construction expressions
    let circle = Shape::Circle {
        center: Point { x: 2.0, y: 3.0 },
        radius: 5.0,
    };
    
    let rectangle_shape = Shape::Rectangle(rect);
    
    let triangle = Shape::Triangle {
        a: origin,
        b: point1,
        c: point2,
    };
    
    println!("Circle: {:?}", circle);
    println!("Rectangle shape: {:?}", rectangle_shape);
    println!("Triangle: {:?}", triangle);
    
    // Construction in expressions
    let shapes = vec![
        Shape::Circle { center: Point::new(0.0, 0.0), radius: 1.0 },
        Shape::Rectangle(Rectangle {
            top_left: Point::new(-1.0, 1.0),
            bottom_right: Point::new(1.0, -1.0),
        }),
    ];
    
    println!("Shapes: {:?}", shapes);
    
    // Using constructed values in calculations
    let distance = point1.distance_from_origin();
    println!("Distance from origin: {:.2}", distance);
}
```

Construction expressions create new instances of user-defined types. Struct  
construction uses field names and values, while enum construction specifies  
the variant and associated data. The update syntax allows copying fields from  
existing instances when creating new ones.  

## Complex Expression Combinations

Complex expressions combine multiple expression types to create sophisticated  
computations and control flow.  

```rust
use std::collections::HashMap;

#[derive(Debug, Clone)]
struct Product {
    name: String,
    price: f64,
    category: String,
}

fn main() {
    // Create sample data using various expressions
    let products = vec![
        Product { name: "Laptop".to_string(), price: 999.99, category: "Electronics".to_string() },
        Product { name: "Book".to_string(), price: 19.99, category: "Education".to_string() },
        Product { name: "Phone".to_string(), price: 699.99, category: "Electronics".to_string() },
        Product { name: "Desk".to_string(), price: 299.99, category: "Furniture".to_string() },
    ];
    
    // Complex expression combining multiple operations
    let expensive_electronics: Vec<String> = products
        .iter()
        .filter(|product| product.category == "Electronics")
        .filter(|product| product.price > 500.0)
        .map(|product| format!("{} (${:.2})", product.name, product.price))
        .collect();
    
    println!("Expensive Electronics: {:?}", expensive_electronics);
    
    // Expression with conditional logic and calculations
    let total_value = products
        .iter()
        .map(|product| {
            let discount = if product.price > 500.0 { 0.1 } else { 0.05 };
            product.price * (1.0 - discount)
        })
        .sum::<f64>();
    
    println!("Total value after discounts: ${:.2}", total_value);
    
    // Complex match expression with destructuring
    let category_stats: HashMap<String, (usize, f64)> = products
        .iter()
        .fold(HashMap::new(), |mut acc, product| {
            let entry = acc.entry(product.category.clone()).or_insert((0, 0.0));
            entry.0 += 1;
            entry.1 += product.price;
            acc
        });
    
    println!("Category Statistics:");
    for (category, (count, total)) in &category_stats {
        let average = total / count as f64;
        println!("  {}: {} items, avg price ${:.2}", category, count, average);
    }
    
    // Nested expressions with error handling
    let most_expensive = products
        .iter()
        .max_by(|a, b| a.price.partial_cmp(&b.price).unwrap())
        .map(|product| format!("Most expensive: {} at ${:.2}", product.name, product.price))
        .unwrap_or_else(|| "No products found".to_string());
    
    println!("{}", most_expensive);
    
    // Complex conditional expression with multiple branches
    let recommendation = {
        let budget = 400.0;
        let preferred_category = "Electronics";
        
        products
            .iter()
            .filter(|p| p.price <= budget)
            .filter(|p| p.category == preferred_category)
            .min_by(|a, b| a.price.partial_cmp(&b.price).unwrap())
            .map(|p| format!("Recommended: {} for ${:.2}", p.name, p.price))
            .unwrap_or_else(|| {
                // Fallback: find cheapest in any category
                products
                    .iter()
                    .filter(|p| p.price <= budget)
                    .min_by(|a, b| a.price.partial_cmp(&b.price).unwrap())
                    .map(|p| format!("Alternative: {} for ${:.2}", p.name, p.price))
                    .unwrap_or_else(|| "No products within budget".to_string())
            })
    };
    
    println!("{}", recommendation);
}
```

Complex expressions demonstrate how Rust's expression-oriented nature enables  
powerful, readable code. By combining iterators, closures, pattern matching,  
and error handling, you can express sophisticated logic concisely while  
maintaining type safety and performance.  
