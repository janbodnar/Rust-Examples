# Types

Rust has a powerful, static type system that ensures memory safety and  
prevents many common programming errors at compile time. The type system  
is based on ownership, borrowing, and lifetimes, making Rust both safe  
and performant without garbage collection.  

Key characteristics of Rust's type system:  
- **Static typing**: All types are known at compile time  
- **Type inference**: The compiler can often deduce types automatically  
- **Zero-cost abstractions**: High-level features don't impact runtime  
- **Memory safety**: Prevents null pointer dereferences and buffer overflows  
- **Ownership model**: Manages memory through compile-time checks  

Rust provides scalar types (integers, floats, booleans, characters),  
compound types (tuples, arrays), and user-defined types (structs, enums).  
The type system also includes powerful features like generics, traits,  
and associated types for building flexible, reusable code.  

## Ways of type declarations 

In Rust, there are several ways to declare variable types:

- suffix
- explicit type declaration
- type inference

```rust

fn main() {
    
    // Type suffix: Append the type to the literal (e.g., 32i8 for i8)
    let x = 32i8;
    // 2. Explicit type annotation: Use colon and type after variable name
    let y: f32 = 23.45;

    // 3. Type inference: Let Rust infer the type from the value (defaults to i32 for integers)
    let z = 2345;

    // String slice (&str) - immutable reference to a string
    let greeting: &str = "Hello, Rust!";
    // Owned String - heap-allocated, growable string
    let name = String::from("Alice");

    // Boolean values
    let is_active: bool = true;
    let is_finished = false; // type inferred

    // Character type (Unicode scalar value)
    let letter: char = 'A';
    let emoji = '😊'; // type inferred

    // Arrays - fixed-size collection of same-type elements
    let numbers: [i32; 3] = [1, 2, 3];
    let zeros = [0; 5]; // array of 5 zeros, type inferred

    // Tuples - fixed-size collection of potentially different types
    let person: (String, i32, bool) = ("Bob".to_string(), 30, true);
    let coordinates = (10.5, 20.3); // type inferred as (f64, f64)

    // Constants - must have explicit type and are always immutable
    const PI: f64 = 3.14159;
    const MAX_USERS: u32 = 1000;

    // Mutable variables
    let mut counter = 0; // starts as i32
    counter += 1;

    // Print the values to the console
    println!("Integers: x = {}, z = {}", x, z);
    println!("Float: y = {}", y);
    println!("String: {}", greeting);
    println!("Mutable string: {}", name);
    println!("Booleans: active = {}, finished = {}", is_active, is_finished);
    println!("Characters: letter = {}, emoji = {}", letter, emoji);
    println!("Array: {:?}", numbers);
    println!("Array of zeros: {:?}", zeros);
    println!("Tuple: {:?}", person);
    println!("Coordinates: {:?}", coordinates);
    println!("Constants: PI = {}, MAX_USERS = {}", PI, MAX_USERS);
    println!("Mutable counter: {}", counter);
}
```

The example demonstrates three ways to declare types in Rust: type suffix  
notation (32i8), explicit type annotation (y: f32), and type inference  
where Rust automatically determines the type. Type inference defaults to  
i32 for integers and f64 for floating-point numbers when the type cannot  
be determined from context.  

## Example 1: Basic scalar types

Understanding Rust's fundamental scalar types and their characteristics.  

```rust
fn main() {
    // Integer types with different bit sizes
    let small_int: i8 = 127;        // 8-bit signed integer (-128 to 127)
    let medium_int: i16 = 32_767;   // 16-bit signed integer
    let normal_int: i32 = 2_147_483_647; // 32-bit signed (default)
    let big_int: i64 = 9_223_372_036_854_775_807; // 64-bit signed
    
    // Unsigned integers (only positive values)
    let unsigned_small: u8 = 255;   // 8-bit unsigned (0 to 255)
    let unsigned_big: u64 = 18_446_744_073_709_551_615; // 64-bit unsigned
    
    // Floating-point types
    let precise_float: f32 = 3.14159; // 32-bit float (single precision)
    let more_precise: f64 = 3.141592653589793; // 64-bit float (default)
    
    // Boolean type (true or false only)
    let is_learning: bool = true;
    let is_expert: bool = false;
    
    // Character type (Unicode scalar value, 4 bytes)
    let letter: char = 'R';         // ASCII character
    let unicode_char: char = '🦀';  // Unicode emoji (Rust mascot!)
    let chinese_char: char = '中';   // Unicode Chinese character
    
    // Display the values and their types
    println!("Integer types:");
    println!("i8: {}, i16: {}, i32: {}, i64: {}", 
             small_int, medium_int, normal_int, big_int);
    println!("u8: {}, u64: {}", unsigned_small, unsigned_big);
    
    println!("\nFloating-point types:");
    println!("f32: {}, f64: {}", precise_float, more_precise);
    
    println!("\nBoolean types:");
    println!("Learning Rust: {}, Expert: {}", is_learning, is_expert);
    
    println!("\nCharacter types:");
    println!("Letter: {}, Emoji: {}, Chinese: {}", 
             letter, unicode_char, chinese_char);
    
    // Demonstrate type sizes in bytes
    println!("\nType sizes in memory:");
    println!("i8: {} bytes", std::mem::size_of::<i8>());
    println!("i32: {} bytes", std::mem::size_of::<i32>());
    println!("f64: {} bytes", std::mem::size_of::<f64>());
    println!("bool: {} bytes", std::mem::size_of::<bool>());
    println!("char: {} bytes", std::mem::size_of::<char>());
}
```

This example introduces Rust's scalar types: integers (signed i8-i128,  
unsigned u8-u128), floating-point numbers (f32, f64), booleans (bool),  
and characters (char). Each type has specific size and range constraints.  
Integers default to i32, floats to f64. Characters are Unicode scalars  
stored in 4 bytes, supporting all Unicode characters including emojis.  

## Example 2: Type inference and explicit annotations

Exploring how Rust's type inference works and when explicit annotations  
are needed.  

```rust
fn main() {
    // Type inference - Rust determines types automatically
    let number = 42;           // Inferred as i32 (default integer type)
    let decimal = 3.14;        // Inferred as f64 (default float type)
    let text = "Hello";        // Inferred as &str (string slice)
    let flag = true;           // Inferred as bool
    
    // Explicit type annotations - telling Rust exactly what type to use
    let explicit_int: u64 = 42;      // Force to u64 instead of i32
    let explicit_float: f32 = 3.14;  // Force to f32 instead of f64
    let explicit_string: String = "Hello".to_string(); // Owned string
    
    // Type annotations required when inference is ambiguous
    let parsed_number: i32 = "123".parse().expect("Invalid number");
    // Without type annotation, Rust can't determine parse target type
    
    // Collections require type hints when empty
    let empty_vector: Vec<i32> = Vec::new(); // Empty vector of integers
    let empty_map: std::collections::HashMap<String, i32> = 
        std::collections::HashMap::new(); // Empty hash map
    
    // Inference works with method chaining
    let numbers = vec![1, 2, 3, 4, 5]; // Vec<i32> inferred from elements
    let doubled: Vec<i32> = numbers.iter().map(|x| x * 2).collect();
    
    // Multiple possible types - annotation resolves ambiguity
    let string_numbers = vec!["1", "2", "3"];
    let parsed: Vec<i32> = string_numbers.iter()
        .map(|s| s.parse().unwrap())
        .collect();
    
    // Turbofish syntax for specifying types in generic functions
    let parsed_turbofish = "456".parse::<u32>().unwrap();
    let collected_turbofish = (1..=5).collect::<Vec<i32>>();
    
    println!("Inferred types:");
    println!("number: {}, decimal: {}, text: {}, flag: {}", 
             number, decimal, text, flag);
    
    println!("\nExplicit types:");
    println!("u64: {}, f32: {}, String: {}", 
             explicit_int, explicit_float, explicit_string);
    
    println!("\nParsed number: {}", parsed_number);
    println!("Doubled numbers: {:?}", doubled);
    println!("Parsed from strings: {:?}", parsed);
    println!("Turbofish parsed: {}", parsed_turbofish);
    println!("Turbofish collected: {:?}", collected_turbofish);
    
    // Demonstrating type coercion limitations
    let integer: i32 = 100;
    let float: f64 = integer as f64; // Explicit cast required
    println!("Casted integer to float: {}", float);
}
```

Type inference reduces verbosity while maintaining type safety. Rust  
infers types from context, initial values, and usage patterns. Explicit  
annotations are required when types are ambiguous (like parsing strings  
or creating empty collections). The turbofish syntax (::<Type>) provides  
a way to specify generic type parameters explicitly.  

## Example 3: Arrays and their type signatures

Understanding fixed-size arrays, their type signatures, and operations.  

```rust
fn main() {
    // Array type signature: [element_type; length]
    let numbers: [i32; 5] = [1, 2, 3, 4, 5];
    let zeros: [i32; 3] = [0; 3]; // Initialize 3 elements to 0
    let mixed: [f64; 4] = [1.1, 2.2, 3.3, 4.4];
    
    // Arrays of different types
    let booleans: [bool; 3] = [true, false, true];
    let characters: [char; 4] = ['R', 'u', 's', 't'];
    let strings: [&str; 3] = ["hello", "world", "rust"];
    
    // Accessing array elements (zero-indexed)
    println!("First number: {}", numbers[0]);
    println!("Last number: {}", numbers[numbers.len() - 1]);
    
    // Array length and type information
    println!("Numbers array length: {}", numbers.len());
    println!("Numbers array size in bytes: {}", 
             std::mem::size_of_val(&numbers));
    
    // Iterating over arrays
    println!("\nNumbers in array:");
    for number in numbers {
        println!("{}", number);
    }
    
    // Iterating with indices
    println!("\nNumbers with indices:");
    for (index, value) in numbers.iter().enumerate() {
        println!("Index {}: {}", index, value);
    }
    
    // Array methods and operations
    println!("\nArray operations:");
    println!("Contains 3: {}", numbers.contains(&3));
    println!("First element: {:?}", numbers.first());
    println!("Last element: {:?}", numbers.last());
    
    // Slicing arrays (creates a slice &[T])
    let slice = &numbers[1..4]; // Elements at indices 1, 2, 3
    println!("Slice [1..4]: {:?}", slice);
    
    // Multi-dimensional arrays
    let matrix: [[i32; 3]; 2] = [[1, 2, 3], [4, 5, 6]];
    println!("Matrix:");
    for row in matrix {
        println!("{:?}", row);
    }
    
    // Array comparison and equality
    let array1: [i32; 3] = [1, 2, 3];
    let array2: [i32; 3] = [1, 2, 3];
    let array3: [i32; 3] = [1, 2, 4];
    
    println!("\nArray comparisons:");
    println!("array1 == array2: {}", array1 == array2);
    println!("array1 == array3: {}", array1 == array3);
    
    // Converting arrays to other types
    let vec_from_array: Vec<i32> = numbers.to_vec();
    println!("Array converted to Vec: {:?}", vec_from_array);
}
```

Arrays in Rust have fixed size determined at compile time. The type  
signature [T; N] specifies element type T and length N. Arrays are  
stack-allocated and provide bounds checking. Unlike some languages,  
array length is part of the type, so [i32; 3] and [i32; 4] are  
different types. Arrays support iteration, slicing, and comparison.  

## Example 4: Tuples with mixed types

Exploring tuples as compound types that can hold different types together.  

```rust
fn main() {
    // Basic tuple with mixed types
    let person: (String, i32, bool, f64) = (
        "Alice".to_string(), 
        30, 
        true, 
        5.7
    );
    
    // Tuple destructuring (pattern matching)
    let (name, age, is_employed, height) = person;
    println!("Name: {}, Age: {}, Employed: {}, Height: {}m", 
             name, age, is_employed, height);
    
    // Accessing tuple elements by index
    let coordinates: (f64, f64, f64) = (10.5, 20.3, 5.8);
    println!("X: {}, Y: {}, Z: {}", coordinates.0, coordinates.1, coordinates.2);
    
    // Nested tuples
    let complex_data: ((i32, i32), (String, bool)) = (
        (100, 200), 
        ("status".to_string(), true)
    );
    println!("Point: ({}, {}), Status: {} is {}", 
             complex_data.0.0, complex_data.0.1,
             complex_data.1.0, complex_data.1.1);
    
    // Empty tuple (unit type)
    let unit: () = ();
    println!("Unit type size: {} bytes", std::mem::size_of_val(&unit));
    
    // Single element tuple (note the comma)
    let single: (i32,) = (42,);
    println!("Single element tuple: {:?}", single);
    
    // Tuples as function return values
    fn get_name_and_age() -> (String, u32) {
        ("Bob".to_string(), 25)
    }
    
    let (person_name, person_age) = get_name_and_age();
    println!("Returned from function: {} is {} years old", 
             person_name, person_age);
    
    // Tuple comparison and equality
    let point1: (i32, i32) = (10, 20);
    let point2: (i32, i32) = (10, 20);
    let point3: (i32, i32) = (15, 25);
    
    println!("\nTuple comparisons:");
    println!("point1 == point2: {}", point1 == point2);
    println!("point1 == point3: {}", point1 == point3);
    
    // Mutable tuples
    let mut mutable_tuple: (i32, String) = (1, "hello".to_string());
    mutable_tuple.0 = 42;
    mutable_tuple.1 = "world".to_string();
    println!("Modified tuple: {:?}", mutable_tuple);
    
    // Tuples with arrays and other compound types
    let mixed: ([i32; 3], Vec<String>, bool) = (
        [1, 2, 3],
        vec!["apple".to_string(), "banana".to_string()],
        false
    );
    println!("Mixed compound tuple: {:?}", mixed);
    
    // Using tuples for multiple assignment
    let mut x = 10;
    let mut y = 20;
    println!("Before swap: x = {}, y = {}", x, y);
    
    (x, y) = (y, x); // Swap values using tuple assignment
    println!("After swap: x = {}, y = {}", x, y);
}
```

Tuples are compound types that group multiple values of potentially  
different types. They're accessed by index (tuple.0, tuple.1) or  
destructured through pattern matching. The unit type () is a special  
zero-sized tuple. Tuples are useful for returning multiple values from  
functions and temporary grouping of related data.  

## Example 5: String types (&str vs String)

Understanding the difference between string slices and owned strings.  

```rust
fn main() {
    // String slice (&str) - borrowed, immutable reference to string data
    let string_literal: &str = "Hello, World!"; // Lives in program binary
    let slice_from_string: &str = "Rust programming"; // Also in binary
    
    // Owned String - heap-allocated, growable, mutable
    let owned_string: String = String::from("Hello, Rust!");
    let created_string: String = "Initial".to_string();
    let formatted_string: String = format!("Number: {}", 42);
    
    println!("String types demonstration:");
    println!("String literal: {}", string_literal);
    println!("Owned string: {}", owned_string);
    
    // String operations and mutability
    let mut mutable_string = String::new(); // Empty string
    mutable_string.push_str("Hello"); // Append string slice
    mutable_string.push(' '); // Append single character
    mutable_string.push_str("World");
    println!("Built string: {}", mutable_string);
    
    // Converting between types
    let from_str: String = string_literal.to_string(); // &str to String
    let to_str: &str = &owned_string; // String to &str (borrowing)
    let as_str: &str = owned_string.as_str(); // Explicit conversion
    
    println!("\nConversions:");
    println!("From &str to String: {}", from_str);
    println!("From String to &str: {}", to_str);
    
    // String slicing - creating &str from parts of String
    let full_string = String::from("Hello, beautiful World!");
    let hello_slice: &str = &full_string[0..5]; // "Hello"
    let world_slice: &str = &full_string[16..21]; // "World"
    
    println!("\nString slicing:");
    println!("Full string: {}", full_string);
    println!("Hello slice: {}", hello_slice);
    println!("World slice: {}", world_slice);
    
    // Memory and performance characteristics
    println!("\nMemory characteristics:");
    println!("&str size: {} bytes", std::mem::size_of::<&str>());
    println!("String size: {} bytes", std::mem::size_of::<String>());
    println!("String capacity: {} bytes", owned_string.capacity());
    println!("String length: {} bytes", owned_string.len());
    
    // Function parameters - best practices
    fn process_string_slice(s: &str) {
        println!("Processing slice: {}", s);
    }
    
    fn process_owned_string(s: String) {
        println!("Processing owned: {}", s);
        // String is moved here, can't use s after this function
    }
    
    // Function calls demonstrating flexibility
    process_string_slice(&owned_string); // String can be borrowed as &str
    process_string_slice(string_literal); // &str works directly
    
    let test_string = String::from("Test");
    process_string_slice(&test_string); // Borrow for slice function
    // test_string is still usable here
    
    let another_test = String::from("Another test");
    process_owned_string(another_test); // Move ownership
    // another_test is no longer usable here
    
    // String formatting and interpolation
    let name = "Alice";
    let age = 30;
    let formatted = format!("Name: {}, Age: {}", name, age);
    let interpolated = format!("Hello, {name}! You are {age} years old.");
    
    println!("\nFormatting:");
    println!("{}", formatted);
    println!("{}", interpolated);
    
    // Working with string methods
    let sample = "  Rust Programming Language  ";
    println!("\nString methods:");
    println!("Original: '{}'", sample);
    println!("Trimmed: '{}'", sample.trim());
    println!("Uppercase: '{}'", sample.trim().to_uppercase());
    println!("Replace: '{}'", sample.trim().replace("Rust", "Go"));
    println!("Contains 'Rust': {}", sample.contains("Rust"));
    println!("Starts with '  Rust': {}", sample.starts_with("  Rust"));
}
```

The distinction between &str and String is fundamental in Rust. String  
slices (&str) are borrowed references to string data, often pointing to  
string literals in the program binary or slices of owned strings. String  
is heap-allocated and owns its data, allowing for mutation and growth.  
Use &str for function parameters when you don't need ownership.  

## Example 6: Option and Result types (basic)

Introducing Rust's approach to handling null values and errors safely.  

```rust
fn main() {
    // Option<T> - represents a value that might be present or absent
    let some_number: Option<i32> = Some(42);
    let no_number: Option<i32> = None;
    
    println!("Option types:");
    println!("Some number: {:?}", some_number);
    println!("No number: {:?}", no_number);
    
    // Pattern matching with Option
    match some_number {
        Some(value) => println!("Found value: {}", value),
        None => println!("No value found"),
    }
    
    match no_number {
        Some(value) => println!("Found value: {}", value),
        None => println!("No value found"),
    }
    
    // Using if let for concise Option handling
    if let Some(num) = some_number {
        println!("Number via if let: {}", num);
    }
    
    // Option methods
    println!("\nOption methods:");
    println!("Is some_number some? {}", some_number.is_some());
    println!("Is no_number none? {}", no_number.is_none());
    println!("Unwrap or default: {}", no_number.unwrap_or(0));
    println!("Unwrap or else: {}", no_number.unwrap_or_else(|| 100));
    
    // Result<T, E> - represents success (Ok) or failure (Err)
    fn divide(a: f64, b: f64) -> Result<f64, String> {
        if b == 0.0 {
            Err("Division by zero".to_string())
        } else {
            Ok(a / b)
        }
    }
    
    let success_result = divide(10.0, 2.0);
    let error_result = divide(10.0, 0.0);
    
    println!("\nResult types:");
    println!("Success: {:?}", success_result);
    println!("Error: {:?}", error_result);
    
    // Pattern matching with Result
    match success_result {
        Ok(value) => println!("Division result: {}", value),
        Err(error) => println!("Error: {}", error),
    }
    
    match error_result {
        Ok(value) => println!("Division result: {}", value),
        Err(error) => println!("Error occurred: {}", error),
    }
    
    // Result methods
    println!("\nResult methods:");
    println!("Is success ok? {}", success_result.is_ok());
    println!("Is error err? {}", error_result.is_err());
    println!("Unwrap or default: {}", error_result.unwrap_or(0.0));
    
    // Chaining operations with Option and Result
    let number_string: Option<&str> = Some("42");
    let parsed_number: Option<i32> = number_string
        .and_then(|s| s.parse().ok()); // Convert Result to Option
    
    println!("\nChained operations:");
    println!("Parsed number: {:?}", parsed_number);
    
    // Working with collections and Option
    let numbers = vec![1, 2, 3, 4, 5];
    let first: Option<&i32> = numbers.first();
    let last: Option<&i32> = numbers.last();
    let out_of_bounds: Option<&i32> = numbers.get(10);
    
    println!("\nCollection operations:");
    println!("First element: {:?}", first);
    println!("Last element: {:?}", last);
    println!("Out of bounds: {:?}", out_of_bounds);
    
    // Transforming Option values with map
    let doubled: Option<i32> = some_number.map(|x| x * 2);
    println!("Doubled value: {:?}", doubled);
    
    // Converting between Option and Result
    let option_to_result: Result<i32, &str> = some_number.ok_or("No value");
    let result_to_option: Option<f64> = success_result.ok();
    
    println!("\nConversions:");
    println!("Option to Result: {:?}", option_to_result);
    println!("Result to Option: {:?}", result_to_option);
    
    // Practical example: safe indexing
    fn safe_get(vec: &Vec<i32>, index: usize) -> Option<i32> {
        vec.get(index).copied()
    }
    
    let test_vec = vec![10, 20, 30];
    println!("\nSafe indexing:");
    println!("Index 1: {:?}", safe_get(&test_vec, 1));
    println!("Index 5: {:?}", safe_get(&test_vec, 5));
}
```

Option<T> replaces null pointers, forcing explicit handling of absent  
values. Result<T, E> represents operations that can fail, encoding both  
success and error cases in the type system. Both types provide rich  
methods for transformation and chaining operations. Pattern matching  
ensures all cases are handled, preventing null pointer exceptions and  
unhandled errors at compile time.  

## Example 7: Custom structs and type definitions

Creating custom data types using structs and type aliases.  

```rust
// Define custom structs
#[derive(Debug, Clone, PartialEq)]
struct Person {
    name: String,
    age: u32,
    email: String,
    is_active: bool,
}

#[derive(Debug)]
struct Point {
    x: f64,
    y: f64,
}

// Tuple struct (struct with unnamed fields)
#[derive(Debug)]
struct Color(u8, u8, u8); // RGB values

// Unit struct (no fields)
#[derive(Debug)]
struct UnitStruct;

// Type aliases for clarity and convenience
type UserId = u64;
type UserName = String;
type Temperature = f32;

fn main() {
    // Creating struct instances
    let person1 = Person {
        name: "Alice Johnson".to_string(),
        age: 28,
        email: "alice@example.com".to_string(),
        is_active: true,
    };
    
    // Struct update syntax (copying fields from another instance)
    let person2 = Person {
        name: "Bob Smith".to_string(),
        email: "bob@example.com".to_string(),
        ..person1.clone() // Copy remaining fields from person1
    };
    
    println!("Struct instances:");
    println!("Person 1: {:?}", person1);
    println!("Person 2: {:?}", person2);
    
    // Accessing struct fields
    println!("\nField access:");
    println!("{} is {} years old", person1.name, person1.age);
    println!("Active status: {}", person1.is_active);
    
    // Mutable structs
    let mut mutable_person = Person {
        name: "Charlie Brown".to_string(),
        age: 25,
        email: "charlie@example.com".to_string(),
        is_active: false,
    };
    
    // Modifying fields
    mutable_person.age = 26;
    mutable_person.is_active = true;
    println!("Modified person: {:?}", mutable_person);
    
    // Tuple structs
    let red = Color(255, 0, 0);
    let green = Color(0, 255, 0);
    let blue = Color(0, 0, 255);
    
    println!("\nTuple structs:");
    println!("Red: {:?}", red);
    println!("RGB values: ({}, {}, {})", red.0, red.1, red.2);
    
    // Points and geometric operations
    let origin = Point { x: 0.0, y: 0.0 };
    let point_a = Point { x: 3.0, y: 4.0 };
    
    println!("\nPoints:");
    println!("Origin: {:?}", origin);
    println!("Point A: {:?}", point_a);
    
    // Unit struct
    let unit = UnitStruct;
    println!("Unit struct: {:?}", unit);
    
    // Type aliases in action
    let user_id: UserId = 12345;
    let user_name: UserName = "developer".to_string();
    let temp: Temperature = 23.5;
    
    println!("\nType aliases:");
    println!("User ID: {}", user_id);
    println!("User Name: {}", user_name);
    println!("Temperature: {}°C", temp);
    
    // Functions working with custom types
    fn calculate_distance(p1: &Point, p2: &Point) -> f64 {
        let dx = p2.x - p1.x;
        let dy = p2.y - p1.y;
        (dx * dx + dy * dy).sqrt()
    }
    
    fn is_adult(person: &Person) -> bool {
        person.age >= 18
    }
    
    fn create_user(id: UserId, name: UserName) -> (UserId, UserName) {
        (id, name)
    }
    
    println!("\nFunctions with custom types:");
    let distance = calculate_distance(&origin, &point_a);
    println!("Distance from origin to point A: {:.2}", distance);
    
    println!("Is person1 an adult? {}", is_adult(&person1));
    
    let user = create_user(user_id, user_name);
    println!("Created user: {:?}", user);
    
    // Destructuring structs
    let Person { name, age, .. } = person1.clone();
    println!("\nDestructured: {} is {} years old", name, age);
    
    // Pattern matching with structs
    match person1.age {
        0..=17 => println!("{} is a minor", person1.name),
        18..=64 => println!("{} is an adult", person1.name),
        65.. => println!("{} is a senior", person1.name),
    }
    
    // Nested structs
    #[derive(Debug)]
    struct Address {
        street: String,
        city: String,
        zip: String,
    }
    
    #[derive(Debug)]
    struct PersonWithAddress {
        personal_info: Person,
        address: Address,
    }
    
    let full_person = PersonWithAddress {
        personal_info: person1.clone(),
        address: Address {
            street: "123 Main St".to_string(),
            city: "Anytown".to_string(),
            zip: "12345".to_string(),
        },
    };
    
    println!("\nNested struct: {:?}", full_person);
}
```

Structs define custom data types by grouping related data. Named structs  
use field names for clarity, tuple structs use positional access, and  
unit structs have no data. Type aliases create meaningful names for  
existing types, improving code readability. Structs support derivable  
traits like Debug, Clone, and PartialEq for common functionality.  

## Example 8: Enums with associated data

Exploring enums as sum types that can hold different variants with data.  

```rust
// Basic enum with unit variants
#[derive(Debug, PartialEq)]
enum Direction {
    North,
    South,
    East,
    West,
}

// Enum with associated data
#[derive(Debug)]
enum IpAddress {
    V4(u8, u8, u8, u8),           // IPv4 with 4 octets
    V6(String),                   // IPv6 as string
}

// Enum with different types of associated data
#[derive(Debug)]
enum Message {
    Quit,                         // Unit variant
    Move { x: i32, y: i32 },     // Struct-like variant
    Write(String),                // Tuple variant
    ChangeColor(u8, u8, u8),     // Tuple variant with multiple values
}

// Complex enum representing a file system entry
#[derive(Debug)]
enum FileSystemEntry {
    File {
        name: String,
        size: u64,
        extension: String,
    },
    Directory {
        name: String,
        entries: Vec<String>,
    },
    Symlink {
        name: String,
        target: String,
    },
}

// Enum with generic type parameter
#[derive(Debug)]
enum Operation<T> {
    Add(T, T),
    Subtract(T, T),
    Multiply(T, T),
    Divide(T, T),
}

fn main() {
    // Using basic enum
    let current_direction = Direction::North;
    let next_direction = Direction::East;
    
    println!("Directions:");
    println!("Current: {:?}", current_direction);
    println!("Next: {:?}", next_direction);
    
    // Comparing enum values
    if current_direction == Direction::North {
        println!("Heading north!");
    }
    
    // Enums with associated data
    let localhost_v4 = IpAddress::V4(127, 0, 0, 1);
    let localhost_v6 = IpAddress::V6("::1".to_string());
    
    println!("\nIP Addresses:");
    println!("IPv4 localhost: {:?}", localhost_v4);
    println!("IPv6 localhost: {:?}", localhost_v6);
    
    // Pattern matching with enums
    match localhost_v4 {
        IpAddress::V4(a, b, c, d) => {
            println!("IPv4 address: {}.{}.{}.{}", a, b, c, d);
        }
        IpAddress::V6(addr) => {
            println!("IPv6 address: {}", addr);
        }
    }
    
    // Using enum with different variant types
    let messages = vec![
        Message::Quit,
        Message::Move { x: 10, y: 20 },
        Message::Write("Hello, Rust!".to_string()),
        Message::ChangeColor(255, 0, 128),
    ];
    
    println!("\nProcessing messages:");
    for message in messages {
        match message {
            Message::Quit => println!("Received quit command"),
            Message::Move { x, y } => println!("Move to coordinates ({}, {})", x, y),
            Message::Write(text) => println!("Display text: {}", text),
            Message::ChangeColor(r, g, b) => {
                println!("Change color to RGB({}, {}, {})", r, g, b);
            }
        }
    }
    
    // File system entries
    let file = FileSystemEntry::File {
        name: "document.txt".to_string(),
        size: 1024,
        extension: "txt".to_string(),
    };
    
    let directory = FileSystemEntry::Directory {
        name: "project".to_string(),
        entries: vec![
            "main.rs".to_string(),
            "lib.rs".to_string(),
            "Cargo.toml".to_string(),
        ],
    };
    
    let symlink = FileSystemEntry::Symlink {
        name: "link_to_file".to_string(),
        target: "/path/to/original".to_string(),
    };
    
    let fs_entries = vec![file, directory, symlink];
    
    println!("\nFile system entries:");
    for entry in fs_entries {
        match entry {
            FileSystemEntry::File { name, size, extension } => {
                println!("File: {} ({} bytes, .{})", name, size, extension);
            }
            FileSystemEntry::Directory { name, entries } => {
                println!("Directory: {} ({} items)", name, entries.len());
                for item in entries {
                    println!("  - {}", item);
                }
            }
            FileSystemEntry::Symlink { name, target } => {
                println!("Symlink: {} -> {}", name, target);
            }
        }
    }
    
    // Generic enums
    let int_op = Operation::Add(5, 3);
    let float_op = Operation::Multiply(3.14, 2.0);
    let string_op = Operation::Add("Hello, ".to_string(), "World!".to_string());
    
    println!("\nGeneric operations:");
    
    fn evaluate_int_operation(op: Operation<i32>) -> Option<i32> {
        match op {
            Operation::Add(a, b) => Some(a + b),
            Operation::Subtract(a, b) => Some(a - b),
            Operation::Multiply(a, b) => Some(a * b),
            Operation::Divide(a, b) => if b != 0 { Some(a / b) } else { None },
        }
    }
    
    if let Some(result) = evaluate_int_operation(int_op) {
        println!("Integer operation result: {}", result);
    }
    
    // Using if let for specific pattern matching
    if let Operation::Multiply(a, b) = float_op {
        println!("Float multiplication: {} * {} = {}", a, b, a * b);
    }
    
    // Enum methods (implementing on enums)
    impl Direction {
        fn opposite(&self) -> Direction {
            match self {
                Direction::North => Direction::South,
                Direction::South => Direction::North,
                Direction::East => Direction::West,
                Direction::West => Direction::East,
            }
        }
    }
    
    let opposite = current_direction.opposite();
    println!("\nOpposite of {:?} is {:?}", current_direction, opposite);
}
```

Enums in Rust are algebraic data types that can represent one of several  
variants. Unlike enums in some languages, Rust enums can carry associated  
data with each variant. This makes them powerful for modeling states,  
messages, and complex data structures. Pattern matching ensures  
exhaustive handling of all variants, preventing runtime errors.  

## Example 9: Generic types and type parameters

Understanding how to write flexible, reusable code with generic types.  

```rust
// Generic struct with one type parameter
#[derive(Debug)]
struct Container<T> {
    value: T,
}

// Generic struct with multiple type parameters
#[derive(Debug)]
struct Pair<T, U> {
    first: T,
    second: U,
}

// Generic enum (similar to Option<T>)
#[derive(Debug)]
enum Result<T, E> {
    Success(T),
    Failure(E),
}

// Generic function
fn swap<T>(a: T, b: T) -> (T, T) {
    (b, a)
}

// Generic function with multiple type parameters
fn combine<T, U, V>(first: T, second: U, combiner: impl Fn(T, U) -> V) -> V {
    combiner(first, second)
}

// Generic function with type constraints (bounds)
fn print_if_display<T: std::fmt::Display>(value: T) {
    println!("Value: {}", value);
}

// Generic function with multiple bounds
fn compare_and_print<T>(a: T, b: T) 
where 
    T: std::cmp::PartialOrd + std::fmt::Display + Copy,
{
    if a > b {
        println!("{} is greater than {}", a, b);
    } else if a < b {
        println!("{} is less than {}", a, b);
    } else {
        println!("{} equals {}", a, b);
    }
}

fn main() {
    // Creating generic struct instances
    let int_container = Container { value: 42 };
    let string_container = Container { value: "Hello".to_string() };
    let float_container = Container { value: 3.14 };
    
    println!("Generic containers:");
    println!("Integer: {:?}", int_container);
    println!("String: {:?}", string_container);
    println!("Float: {:?}", float_container);
    
    // Generic struct with multiple parameters
    let mixed_pair = Pair {
        first: 100,
        second: "items".to_string(),
    };
    let coordinate_pair = Pair {
        first: 10.5,
        second: 20.3,
    };
    
    println!("\nGeneric pairs:");
    println!("Mixed: {:?}", mixed_pair);
    println!("Coordinates: {:?}", coordinate_pair);
    
    // Using generic functions
    println!("\nGeneric functions:");
    let (x, y) = swap(1, 2);
    println!("Swapped integers: ({}, {})", x, y);
    
    let (a, b) = swap("hello", "world");
    println!("Swapped strings: ({}, {})", a, b);
    
    // Generic function with closure
    let sum = combine(5, 10, |a, b| a + b);
    let concatenated = combine("Hello, ", "World!", |a, b| format!("{}{}", a, b));
    
    println!("Combined sum: {}", sum);
    println!("Combined string: {}", concatenated);
    
    // Functions with type constraints
    print_if_display(42);
    print_if_display("Hello, generics!");
    print_if_display(3.14159);
    
    compare_and_print(10, 5);
    compare_and_print(2.5, 7.8);
    compare_and_print('a', 'z');
    
    // Generic collections
    let mut int_vec: Vec<i32> = Vec::new();
    int_vec.push(1);
    int_vec.push(2);
    int_vec.push(3);
    
    let string_vec: Vec<String> = vec![
        "apple".to_string(),
        "banana".to_string(),
        "cherry".to_string(),
    ];
    
    println!("\nGeneric collections:");
    println!("Integer vector: {:?}", int_vec);
    println!("String vector: {:?}", string_vec);
    
    // Generic HashMap
    use std::collections::HashMap;
    let mut scores: HashMap<String, i32> = HashMap::new();
    scores.insert("Alice".to_string(), 95);
    scores.insert("Bob".to_string(), 87);
    
    println!("Score map: {:?}", scores);
    
    // Implementing methods on generic types
    impl<T> Container<T> {
        fn new(value: T) -> Container<T> {
            Container { value }
        }
        
        fn get(&self) -> &T {
            &self.value
        }
        
        fn set(&mut self, value: T) {
            self.value = value;
        }
    }
    
    // Methods with additional type parameters
    impl<T> Container<T> {
        fn map<U, F>(self, f: F) -> Container<U>
        where
            F: FnOnce(T) -> U,
        {
            Container { value: f(self.value) }
        }
    }
    
    println!("\nGeneric methods:");
    let mut container = Container::new(42);
    println!("Original: {:?}", container.get());
    
    container.set(100);
    println!("After set: {:?}", container.get());
    
    let mapped = container.map(|x| x as f64 * 2.5);
    println!("Mapped: {:?}", mapped);
    
    // Generic struct with default type parameters
    #[derive(Debug)]
    struct DefaultContainer<T = i32> {
        value: T,
    }
    
    let default_container = DefaultContainer { value: 42 }; // Uses i32 default
    let explicit_container: DefaultContainer<String> = DefaultContainer {
        value: "explicit".to_string(),
    };
    
    println!("\nDefault type parameters:");
    println!("Default (i32): {:?}", default_container);
    println!("Explicit (String): {:?}", explicit_container);
    
    // Associated functions on generic types
    impl<T, U> Pair<T, U> {
        fn new(first: T, second: U) -> Pair<T, U> {
            Pair { first, second }
        }
        
        fn swap(self) -> Pair<U, T> {
            Pair {
                first: self.second,
                second: self.first,
            }
        }
    }
    
    let original_pair = Pair::new(1, "one");
    let swapped_pair = original_pair.swap();
    
    println!("\nPair operations:");
    println!("Swapped pair: {:?}", swapped_pair);
}
```

Generics enable writing flexible, reusable code that works with multiple  
types. Type parameters are specified in angle brackets <T, U>. Generic  
functions and types are monomorphized at compile time, creating specific  
versions for each type used. Type bounds constrain generic parameters to  
types that implement specific traits, ensuring type safety.  

## Example 10: Advanced traits and associated types

Exploring traits as contracts for shared behavior and advanced features.  

```rust
// Basic trait definition
trait Describable {
    fn describe(&self) -> String;
}

// Trait with associated types
trait Iterator {
    type Item; // Associated type
    
    fn next(&mut self) -> Option<Self::Item>;
}

// Trait with default implementations
trait Greetable {
    fn name(&self) -> &str;
    
    // Default implementation
    fn greet(&self) -> String {
        format!("Hello, {}!", self.name())
    }
    
    // Default implementation calling other methods
    fn formal_greeting(&self) -> String {
        format!("Good day, {}. Pleased to meet you.", self.name())
    }
}

// Trait with associated functions (no self parameter)
trait Constructible {
    fn new() -> Self;
    fn with_name(name: String) -> Self;
}

// Trait for conversion between types
trait From<T> {
    fn from(value: T) -> Self;
}

trait Into<T> {
    fn into(self) -> T;
}

// Complex trait with multiple associated types and bounds
trait Collect<T> {
    type Output;
    type Error;
    
    fn collect(self) -> Result<Self::Output, Self::Error>
    where
        Self: Sized,
        T: Clone;
}

// Structs for demonstration
#[derive(Debug, Clone)]
struct Person {
    name: String,
    age: u32,
}

#[derive(Debug)]
struct Animal {
    name: String,
    species: String,
}

#[derive(Debug)]
struct Book {
    title: String,
    author: String,
}

// Implementing basic traits
impl Describable for Person {
    fn describe(&self) -> String {
        format!("{} is {} years old", self.name, self.age)
    }
}

impl Describable for Animal {
    fn describe(&self) -> String {
        format!("{} is a {}", self.name, self.species)
    }
}

impl Greetable for Person {
    fn name(&self) -> &str {
        &self.name
    }
    
    // Override default implementation
    fn greet(&self) -> String {
        format!("Hi there, {}! How are you?", self.name)
    }
}

impl Greetable for Animal {
    fn name(&self) -> &str {
        &self.name
    }
    // Uses default implementations for greet() and formal_greeting()
}

impl Constructible for Person {
    fn new() -> Self {
        Person {
            name: "Anonymous".to_string(),
            age: 0,
        }
    }
    
    fn with_name(name: String) -> Self {
        Person { name, age: 0 }
    }
}

// Custom iterator implementation
struct CountUp {
    current: i32,
    max: i32,
}

impl Iterator for CountUp {
    type Item = i32;
    
    fn next(&mut self) -> Option<Self::Item> {
        if self.current < self.max {
            let current = self.current;
            self.current += 1;
            Some(current)
        } else {
            None
        }
    }
}

impl CountUp {
    fn new(max: i32) -> CountUp {
        CountUp { current: 0, max }
    }
}

// Trait objects and dynamic dispatch
trait Draw {
    fn draw(&self);
}

impl Draw for Person {
    fn draw(&self) {
        println!("Drawing person: {}", self.describe());
    }
}

impl Draw for Animal {
    fn draw(&self) {
        println!("Drawing animal: {}", self.describe());
    }
}

// Generic functions with trait bounds
fn print_description<T: Describable>(item: &T) {
    println!("Description: {}", item.describe());
}

fn greet_all<T: Greetable>(items: &[T]) {
    for item in items {
        println!("{}", item.greet());
    }
}

// Multiple trait bounds
fn describe_and_greet<T>(item: &T) 
where 
    T: Describable + Greetable,
{
    println!("{}", item.describe());
    println!("{}", item.greet());
}

// Trait bounds with lifetime parameters
fn compare_names<'a, T, U>(first: &'a T, second: &'a U) -> bool
where
    T: Greetable,
    U: Greetable,
{
    first.name() == second.name()
}

fn main() {
    // Creating instances
    let person = Person {
        name: "Alice".to_string(),
        age: 30,
    };
    
    let animal = Animal {
        name: "Buddy".to_string(),
        species: "Golden Retriever".to_string(),
    };
    
    // Using traits
    println!("Trait implementations:");
    print_description(&person);
    print_description(&animal);
    
    println!("\nGreetings:");
    println!("{}", person.greet());
    println!("{}", animal.greet());
    println!("{}", animal.formal_greeting());
    
    // Associated functions
    let default_person = Person::new();
    let named_person = Person::with_name("Bob".to_string());
    
    println!("\nConstructed instances:");
    println!("Default: {:?}", default_person);
    println!("Named: {:?}", named_person);
    
    // Custom iterator
    println!("\nCustom iterator:");
    let mut counter = CountUp::new(5);
    while let Some(num) = counter.next() {
        println!("Count: {}", num);
    }
    
    // Trait objects for dynamic dispatch
    let drawable_items: Vec<Box<dyn Draw>> = vec![
        Box::new(person.clone()),
        Box::new(animal),
    ];
    
    println!("\nTrait objects:");
    for item in drawable_items {
        item.draw();
    }
    
    // Multiple trait bounds
    println!("\nMultiple bounds:");
    describe_and_greet(&person);
    
    // Using where clauses
    let person2 = Person {
        name: "Alice".to_string(),
        age: 25,
    };
    
    let same_name = compare_names(&person, &person2);
    println!("Same name: {}", same_name);
    
    // Trait implementations for built-in types
    trait Double {
        fn double(&self) -> Self;
    }
    
    impl Double for i32 {
        fn double(&self) -> Self {
            self * 2
        }
    }
    
    impl Double for String {
        fn double(&self) -> Self {
            format!("{}{}", self, self)
        }
    }
    
    let num = 21;
    let text = "Hello".to_string();
    
    println!("\nTrait for built-in types:");
    println!("Double of {} is {}", num, num.double());
    println!("Double of '{}' is '{}'", text, text.double());
    
    // Supertraits (trait inheritance)
    trait Named {
        fn name(&self) -> &str;
    }
    
    trait FullyDescribed: Named + Describable {
        fn full_description(&self) -> String {
            format!("Name: {}, Details: {}", self.name(), self.describe())
        }
    }
    
    impl Named for Person {
        fn name(&self) -> &str {
            &self.name
        }
    }
    
    impl FullyDescribed for Person {}
    
    println!("\nSupertraits:");
    println!("{}", person.full_description());
    
    // Associated constants in traits
    trait MathConstants {
        const PI: f64 = 3.14159265359;
        const E: f64 = 2.71828182846;
        
        fn circle_area(radius: f64) -> f64 {
            Self::PI * radius * radius
        }
    }
    
    struct Calculator;
    impl MathConstants for Calculator {}
    
    println!("\nTrait constants:");
    println!("PI: {}", Calculator::PI);
    println!("Circle area (r=2): {}", Calculator::circle_area(2.0));
}
```

Traits define shared behavior that types can implement. Associated types  
provide a way to define placeholder types within trait definitions.  
Default implementations reduce code duplication while allowing  
customization. Trait objects enable dynamic dispatch for runtime  
polymorphism. Supertraits create trait hierarchies, and associated  
constants provide compile-time values within traits.  
