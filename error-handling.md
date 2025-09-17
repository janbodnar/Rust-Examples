# Error handling

Error handling in Rust is designed to be explicit, safe, and efficient.  
Instead of exceptions, Rust uses the type system to represent errors through  
`Result<T, E>` and `Option<T>` types. This approach forces developers to  
handle errors explicitly, preventing runtime crashes and making code more  
reliable.  

Key principles of Rust error handling:  
- **No exceptions**: Rust doesn't have traditional exception handling  
- **Explicit errors**: All errors are represented in the type system  
- **Recoverable vs unrecoverable**: `Result` for recoverable, `panic!` for fatal  
- **Zero-cost abstractions**: Error handling has minimal runtime overhead  
- **Composable**: Error handling constructs can be easily combined  

Rust provides several mechanisms for handling errors: the `?` operator for  
propagation, `match` expressions for explicit handling, combinator methods  
for transformation, and `panic!` for unrecoverable situations. This approach  
leads to more robust and maintainable code by making error paths visible  
and ensuring they're properly handled.  

## Basic Result type

The Result type represents either success (Ok) or failure (Err) and forces  
explicit error handling.  

```rust
fn divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err("Division by zero is not allowed".to_string())
    } else {
        Ok(a / b)
    }
}

fn main() {
    let result1 = divide(10.0, 2.0);
    let result2 = divide(10.0, 0.0);
    
    // Pattern matching for explicit handling
    match &result1 {
        Ok(value) => println!("10.0 / 2.0 = {}", value),
        Err(error) => println!("Error: {}", error),
    }
    
    match &result2 {
        Ok(value) => println!("10.0 / 0.0 = {}", value),
        Err(error) => println!("Error: {}", error),
    }
    
    // Using is_ok() and is_err() methods
    println!("Result1 is ok: {}", result1.is_ok());
    println!("Result2 is error: {}", result2.is_err());
}
```

The Result type eliminates the need for null checks or exception handling.  
The Ok variant contains the successful value, while Err contains error  
information. Pattern matching ensures all cases are handled explicitly,  
making the code more predictable and preventing silent failures.  

## Basic Option type

The Option type represents either a value (Some) or the absence of a value  
(None), eliminating null pointer exceptions.  

```rust
fn find_in_array(arr: &[i32], target: i32) -> Option<usize> {
    for (index, &value) in arr.iter().enumerate() {
        if value == target {
            return Some(index);
        }
    }
    None
}

fn get_first_word(text: &str) -> Option<&str> {
    text.split_whitespace().next()
}

fn main() {
    let numbers = [1, 2, 3, 4, 5];
    
    // Searching for existing value
    match find_in_array(&numbers, 3) {
        Some(index) => println!("Found 3 at index {}", index),
        None => println!("3 not found"),
    }
    
    // Searching for non-existing value
    match find_in_array(&numbers, 10) {
        Some(index) => println!("Found 10 at index {}", index),
        None => println!("10 not found"),
    }
    
    // Working with strings
    let text = "Hello world";
    let empty = "";
    
    println!("First word: {:?}", get_first_word(text));
    println!("First word of empty: {:?}", get_first_word(empty));
    
    // Using Option methods
    let some_value = Some(42);
    let none_value: Option<i32> = None;
    
    println!("some_value is some: {}", some_value.is_some());
    println!("none_value is none: {}", none_value.is_none());
}
```

Option types replace null pointers and make absence explicit in the type  
system. The Some variant contains a value, while None represents absence.  
This prevents null pointer dereferences at compile time and forces explicit  
handling of missing values, leading to more robust code.  

## The ? operator

The ? operator provides concise error propagation, automatically returning  
early from functions when encountering errors.  

```rust
use std::fs;
use std::io;

fn read_file_contents(filename: &str) -> Result<String, io::Error> {
    let contents = fs::read_to_string(filename)?;
    Ok(contents)
}

fn get_file_length(filename: &str) -> Result<usize, io::Error> {
    let contents = fs::read_to_string(filename)?;
    Ok(contents.len())
}

fn process_file(filename: &str) -> Result<String, io::Error> {
    let contents = fs::read_to_string(filename)?;
    let trimmed = contents.trim();
    
    if trimmed.is_empty() {
        return Err(io::Error::new(
            io::ErrorKind::InvalidData,
            "File is empty or contains only whitespace"
        ));
    }
    
    Ok(format!("Processed: {} characters", trimmed.len()))
}

fn main() {
    // Create a test file
    fs::write("test.txt", "Hello, Rust!\nThis is a test file.").unwrap();
    
    // Using ? operator through function calls
    match read_file_contents("test.txt") {
        Ok(contents) => println!("File contents:\n{}", contents),
        Err(error) => println!("Error reading file: {}", error),
    }
    
    match get_file_length("test.txt") {
        Ok(length) => println!("File length: {} bytes", length),
        Err(error) => println!("Error getting length: {}", error),
    }
    
    match process_file("test.txt") {
        Ok(result) => println!("{}", result),
        Err(error) => println!("Processing error: {}", error),
    }
    
    // Test with non-existent file
    match read_file_contents("nonexistent.txt") {
        Ok(contents) => println!("Unexpected success: {}", contents),
        Err(error) => println!("Expected error: {}", error),
    }
    
    // Cleanup
    fs::remove_file("test.txt").unwrap();
}
```

The ? operator simplifies error propagation by automatically unwrapping Ok  
values or returning Err values early. This eliminates verbose match  
statements for simple error propagation, making code cleaner while  
maintaining explicit error handling. It only works in functions that return  
Result or Option types.  

## Custom error types

Creating custom error types provides more specific error information and  
better error handling for domain-specific operations.  

```rust
use std::fmt;

#[derive(Debug)]
enum MathError {
    DivisionByZero,
    NegativeLogarithm,
    InvalidInput(String),
}

impl fmt::Display for MathError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            MathError::DivisionByZero => write!(f, "Cannot divide by zero"),
            MathError::NegativeLogarithm => write!(f, "Cannot take logarithm of negative number"),
            MathError::InvalidInput(msg) => write!(f, "Invalid input: {}", msg),
        }
    }
}

impl std::error::Error for MathError {}

fn safe_divide(a: f64, b: f64) -> Result<f64, MathError> {
    if b == 0.0 {
        Err(MathError::DivisionByZero)
    } else {
        Ok(a / b)
    }
}

fn safe_sqrt(x: f64) -> Result<f64, MathError> {
    if x < 0.0 {
        Err(MathError::InvalidInput("Square root of negative number".to_string()))
    } else {
        Ok(x.sqrt())
    }
}

fn safe_log(x: f64) -> Result<f64, MathError> {
    if x <= 0.0 {
        Err(MathError::NegativeLogarithm)
    } else {
        Ok(x.ln())
    }
}

fn main() {
    let operations = [
        ("10.0 / 2.0", safe_divide(10.0, 2.0)),
        ("10.0 / 0.0", safe_divide(10.0, 0.0)),
        ("sqrt(16.0)", safe_sqrt(16.0)),
        ("sqrt(-4.0)", safe_sqrt(-4.0)),
        ("log(2.718)", safe_log(2.718)),
        ("log(-1.0)", safe_log(-1.0)),
    ];
    
    for (description, result) in operations {
        match result {
            Ok(value) => println!("{} = {:.3}", description, value),
            Err(error) => println!("{} failed: {}", description, error),
        }
    }
    
    // Error type checking
    match safe_divide(1.0, 0.0) {
        Err(MathError::DivisionByZero) => println!("Caught division by zero!"),
        Err(e) => println!("Other math error: {}", e),
        Ok(_) => println!("Unexpected success"),
    }
}
```

Custom error types make error handling more precise and informative. They  
allow pattern matching on specific error conditions and provide better  
debugging information. Implementing Display and Error traits makes custom  
errors integrate well with Rust's error ecosystem and standard libraries.  

## Unwrap methods

Various unwrap methods provide different strategies for extracting values  
from Option and Result types.  

```rust
fn main() {
    let some_value = Some(42);
    let none_value: Option<i32> = None;
    
    // unwrap() - panics on None/Err
    println!("Unwrapped value: {}", some_value.unwrap());
    // none_value.unwrap(); // This would panic!
    
    // unwrap_or() - provides default value
    println!("With default: {}", none_value.unwrap_or(0));
    println!("Some with default: {}", some_value.unwrap_or(0));
    
    // unwrap_or_else() - computes default with closure
    println!("With computed default: {}", none_value.unwrap_or_else(|| {
        println!("Computing default value...");
        100
    }));
    
    // expect() - panics with custom message
    println!("Expected value: {}", some_value.expect("Value should exist"));
    // none_value.expect("This should not be None"); // Would panic with message
    
    // Working with Result types
    let ok_result: Result<i32, &str> = Ok(42);
    let err_result: Result<i32, &str> = Err("Something went wrong");
    
    println!("Result unwrap: {}", ok_result.unwrap());
    println!("Result with default: {}", err_result.unwrap_or(-1));
    
    // unwrap_or_default() uses the type's default value
    let empty_vec: Option<Vec<i32>> = None;
    let vec_with_default = empty_vec.unwrap_or_default();
    println!("Default vector: {:?}", vec_with_default);
    
    // Conditional unwrapping
    if let Some(value) = some_value {
        println!("Found value: {}", value);
    }
    
    if let Ok(value) = ok_result {
        println!("Success value: {}", value);
    }
    
    // Safe unwrapping with match
    let safe_value = match none_value {
        Some(v) => v,
        None => {
            println!("No value found, using default");
            42
        }
    };
    println!("Safely extracted: {}", safe_value);
}
```

Unwrap methods provide different strategies for handling Option and Result  
types. Use unwrap() only when you're certain the value exists, unwrap_or()  
for simple defaults, and unwrap_or_else() for computed defaults. The expect()  
method provides better error messages for debugging than unwrap().  

## Result combinators

Result combinators allow chaining operations while handling errors elegantly  
without explicit pattern matching.  

```rust
fn parse_number(s: &str) -> Result<i32, std::num::ParseIntError> {
    s.parse()
}

fn double(x: i32) -> i32 {
    x * 2
}

fn is_even(x: i32) -> Result<i32, &'static str> {
    if x % 2 == 0 {
        Ok(x)
    } else {
        Err("Number is odd")
    }
}

fn main() {
    let input = "42";
    
    // map() - transforms the Ok value, leaves Err unchanged
    let doubled = parse_number(input).map(double);
    println!("Doubled: {:?}", doubled);
    
    // and_then() - chains operations that return Result
    let result = parse_number(input)
        .and_then(|x| Ok(double(x)))
        .and_then(is_even);
    println!("Chained operations: {:?}", result);
    
    // or_else() - provides alternative on error
    let bad_input = "abc";
    let with_fallback = parse_number(bad_input)
        .or_else(|_| Ok(0));
    println!("With fallback: {:?}", with_fallback);
    
    // map_err() - transforms the error
    let custom_error = parse_number(bad_input)
        .map_err(|_| "Failed to parse number");
    println!("Custom error: {:?}", custom_error);
    
    // Complex chaining
    let inputs = ["42", "13", "abc", "24"];
    
    for input in inputs {
        let result = parse_number(input)
            .map(double)
            .and_then(is_even)
            .map(|x| format!("Even number: {}", x))
            .unwrap_or_else(|e| format!("Error: {}", e));
        
        println!("{} -> {}", input, result);
    }
    
    // Using map_or for default values
    let results: Vec<String> = inputs.iter()
        .map(|&input| {
            parse_number(input)
                .map(|x| x.to_string())
                .map_or("invalid".to_string(), |x| x)
        })
        .collect();
    
    println!("Mapped results: {:?}", results);
}
```

Result combinators enable functional-style error handling by chaining  
operations without explicit match statements. The map() method transforms  
successful values, and_then() chains fallible operations, and or_else()  
provides error recovery. This approach leads to more readable and  
composable error handling code.  

## Panic handling

Panic represents unrecoverable errors and should be used sparingly for  
situations where the program cannot continue safely.  

```rust
use std::panic;

fn risky_operation(x: i32) -> i32 {
    if x < 0 {
        panic!("Negative numbers not allowed!");
    }
    if x == 13 {
        panic!("Unlucky number 13!");
    }
    x * 2
}

fn safe_risky_operation(x: i32) -> Result<i32, String> {
    let result = panic::catch_unwind(|| {
        risky_operation(x)
    });
    
    match result {
        Ok(value) => Ok(value),
        Err(_) => Err(format!("Operation panicked with input: {}", x)),
    }
}

fn assert_example(x: i32, expected: i32) {
    assert_eq!(x, expected, "Value {} doesn't match expected {}", x, expected);
    println!("Assertion passed for value {}", x);
}

fn main() {
    println!("=== Normal operations ===");
    println!("risky_operation(5) = {}", risky_operation(5));
    
    println!("\n=== Catching panics ===");
    let test_values = [10, -1, 13, 20];
    
    for value in test_values {
        match safe_risky_operation(value) {
            Ok(result) => println!("Safe operation({}) = {}", value, result),
            Err(error) => println!("Caught panic: {}", error),
        }
    }
    
    println!("\n=== Assertions ===");
    assert_example(5, 5);
    // assert_example(5, 10); // This would panic
    
    // Debug assertions (only in debug builds)
    debug_assert!(true, "This passes");
    debug_assert_eq!(2 + 2, 4);
    
    println!("\n=== Custom panic hook ===");
    let original_hook = panic::take_hook();
    
    panic::set_hook(Box::new(|panic_info| {
        println!("Custom panic handler: {}", panic_info);
    }));
    
    // Test the custom panic handler
    let _ = safe_risky_operation(-5);
    
    // Restore original hook
    panic::set_hook(original_hook);
    
    println!("Program completed successfully");
}
```

Panics should be reserved for truly unrecoverable errors where continuing  
execution would be unsafe. Use panic::catch_unwind() to recover from panics  
when calling potentially panicking code. Assertions help catch programming  
errors during development, and custom panic hooks allow controlled cleanup  
before termination.  

## Option combinators

Option combinators provide elegant ways to work with optional values  
without explicit pattern matching.  

```rust
fn main() {
    let numbers = vec![Some(1), None, Some(3), Some(4), None, Some(6)];
    
    // map() - transforms the Some value
    let doubled: Vec<Option<i32>> = numbers.iter()
        .map(|opt| opt.map(|x| x * 2))
        .collect();
    println!("Doubled: {:?}", doubled);
    
    // filter_map() - filters and transforms in one step
    let valid_doubled: Vec<i32> = numbers.iter()
        .filter_map(|opt| opt.map(|x| x * 2))
        .collect();
    println!("Valid doubled: {:?}", valid_doubled);
    
    // and_then() - chains optional operations
    fn safe_divide(x: i32, y: i32) -> Option<i32> {
        if y != 0 { Some(x / y) } else { None }
    }
    
    let some_number = Some(20);
    let chained_result = some_number
        .and_then(|x| safe_divide(x, 4))
        .and_then(|x| safe_divide(x, 2));
    println!("Chained division: {:?}", chained_result);
    
    // or() and or_else() - provide alternatives
    let none_value: Option<i32> = None;
    let alternative = none_value.or(Some(42));
    println!("With alternative: {:?}", alternative);
    
    let computed_alternative = none_value.or_else(|| {
        println!("Computing alternative...");
        Some(100)
    });
    println!("Computed alternative: {:?}", computed_alternative);
    
    // take() - takes the value out, leaving None
    let mut maybe_value = Some(42);
    let taken = maybe_value.take();
    println!("Taken: {:?}, remaining: {:?}", taken, maybe_value);
    
    // replace() - replaces the value, returning the old one
    let mut maybe_value = Some(42);
    let old = maybe_value.replace(100);
    println!("Old: {:?}, new: {:?}", old, maybe_value);
    
    // Working with references
    let opt_string = Some("hello".to_string());
    let length = opt_string.as_ref().map(|s| s.len());
    println!("String length: {:?}", length);
    println!("Original string still available: {:?}", opt_string);
    
    // zip() - combines two Options
    let opt1 = Some(1);
    let opt2 = Some(2);
    let opt3: Option<i32> = None;
    
    println!("Zip Some + Some: {:?}", opt1.zip(opt2));
    println!("Zip Some + None: {:?}", opt1.zip(opt3));
}
```

Option combinators enable functional programming patterns with optional  
values. The map() method transforms values when present, and_then() chains  
operations that might fail, and or() provides fallback values. These  
combinators eliminate the need for explicit match statements in many  
common scenarios.  

## Error conversion and mapping

Converting between different error types and transforming error information  
for better error handling across different modules.  

```rust
use std::fmt;
use std::num::ParseIntError;
use std::io;

#[derive(Debug)]
enum AppError {
    Parse(ParseIntError),
    Io(io::Error),
    Custom(String),
}

impl fmt::Display for AppError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            AppError::Parse(e) => write!(f, "Parse error: {}", e),
            AppError::Io(e) => write!(f, "IO error: {}", e),
            AppError::Custom(msg) => write!(f, "Custom error: {}", msg),
        }
    }
}

impl std::error::Error for AppError {}

// Automatic conversion from ParseIntError
impl From<ParseIntError> for AppError {
    fn from(error: ParseIntError) -> Self {
        AppError::Parse(error)
    }
}

// Automatic conversion from io::Error
impl From<io::Error> for AppError {
    fn from(error: io::Error) -> Self {
        AppError::Io(error)
    }
}

fn parse_and_double(s: &str) -> Result<i32, AppError> {
    let num: i32 = s.parse()?; // Automatic conversion via From trait
    Ok(num * 2)
}

fn read_and_parse_file(filename: &str) -> Result<Vec<i32>, AppError> {
    use std::fs;
    
    let contents = fs::read_to_string(filename)?; // Auto-converts io::Error
    let mut numbers = Vec::new();
    
    for line in contents.lines() {
        let trimmed = line.trim();
        if !trimmed.is_empty() {
            let num = trimmed.parse()?; // Auto-converts ParseIntError
            numbers.push(num);
        }
    }
    
    if numbers.is_empty() {
        return Err(AppError::Custom("No valid numbers found".to_string()));
    }
    
    Ok(numbers)
}

// Manual error mapping
fn validate_positive(x: i32) -> Result<i32, AppError> {
    if x > 0 {
        Ok(x)
    } else {
        Err(AppError::Custom(format!("Expected positive number, got {}", x)))
    }
}

fn main() {
    // Test parsing conversion
    match parse_and_double("42") {
        Ok(result) => println!("Parsed and doubled: {}", result),
        Err(e) => println!("Error: {}", e),
    }
    
    match parse_and_double("abc") {
        Ok(result) => println!("Unexpected success: {}", result),
        Err(e) => println!("Expected parse error: {}", e),
    }
    
    // Create test file
    std::fs::write("numbers.txt", "10\n20\n30\n").unwrap();
    
    // Test file reading with error conversion
    match read_and_parse_file("numbers.txt") {
        Ok(numbers) => println!("Parsed numbers: {:?}", numbers),
        Err(e) => println!("File error: {}", e),
    }
    
    // Test with non-existent file
    match read_and_parse_file("missing.txt") {
        Ok(numbers) => println!("Unexpected success: {:?}", numbers),
        Err(e) => println!("Expected IO error: {}", e),
    }
    
    // Manual error mapping with map_err
    let result = "42".parse::<i32>()
        .map_err(|e| AppError::Custom(format!("Failed to parse: {}", e)))
        .and_then(validate_positive);
    
    println!("Validation result: {:?}", result);
    
    // Cleanup
    std::fs::remove_file("numbers.txt").unwrap();
}
```

Error conversion allows seamless integration between different error types  
using the From trait. This enables the ? operator to automatically convert  
errors as they propagate up the call stack. The map_err() combinator  
provides manual error transformation when automatic conversion isn't  
available.  

## Early returns with try blocks

Using try blocks and the ? operator for early returns from complex  
operations without nested match statements.  

```rust
fn main() {
    // Simulated try block pattern using closures
    let result1 = (|| -> Result<i32, &'static str> {
        let a = Some(10).ok_or("No value for a")?;
        let b = Some(20).ok_or("No value for b")?;
        let c = if a + b > 25 { 
            Some(a + b) 
        } else { 
            None 
        }.ok_or("Sum too small")?;
        
        Ok(c)
    })();
    
    println!("Try block result 1: {:?}", result1);
    
    // Another example with file operations
    let result2 = (|| -> Result<String, Box<dyn std::error::Error>> {
        use std::fs;
        
        // Create test data
        fs::write("input.txt", "Hello\nWorld\n")?;
        
        let contents = fs::read_to_string("input.txt")?;
        let line_count = contents.lines().count();
        
        if line_count < 2 {
            return Err("Not enough lines".into());
        }
        
        let processed = contents.to_uppercase();
        fs::write("output.txt", &processed)?;
        
        Ok(format!("Processed {} lines", line_count))
    })();
    
    println!("Try block result 2: {:?}", result2);
    
    // Complex validation with early returns
    fn validate_user_data(name: &str, age: &str, email: &str) -> Result<(String, i32, String), String> {
        // Name validation
        let name = name.trim();
        if name.is_empty() {
            return Err("Name cannot be empty".to_string());
        }
        if name.len() < 2 {
            return Err("Name too short".to_string());
        }
        
        // Age validation
        let age: i32 = age.parse()
            .map_err(|_| "Age must be a number".to_string())?;
        if age < 0 || age > 150 {
            return Err("Age out of valid range".to_string());
        }
        
        // Email validation (simplified)
        let email = email.trim();
        if !email.contains('@') || !email.contains('.') {
            return Err("Invalid email format".to_string());
        }
        
        Ok((name.to_string(), age, email.to_string()))
    }
    
    let test_cases = [
        ("Alice", "25", "alice@example.com"),
        ("", "30", "bob@example.com"),
        ("Charlie", "abc", "charlie@example.com"),
        ("David", "35", "invalid-email"),
        ("Eve", "200", "eve@example.com"),
    ];
    
    for (name, age, email) in test_cases {
        match validate_user_data(name, age, email) {
            Ok((name, age, email)) => {
                println!("Valid user: {} ({}) - {}", name, age, email);
            },
            Err(error) => {
                println!("Validation failed for ({}, {}, {}): {}", name, age, email, error);
            }
        }
    }
    
    // Cleanup
    let _ = std::fs::remove_file("input.txt");
    let _ = std::fs::remove_file("output.txt");
}
```

Early returns with the ? operator eliminate deeply nested match statements  
and create more linear, readable code flow. This pattern is especially  
useful for validation functions and operations with multiple fallible  
steps. The closure-based try block pattern provides explicit error handling  
boundaries.  

## Error chaining

Chaining errors preserves the original error context while adding additional  
information as errors propagate through the call stack.  

```rust
use std::fmt;
use std::error::Error;

#[derive(Debug)]
struct DatabaseError {
    message: String,
    source: Option<Box<dyn Error>>,
}

impl fmt::Display for DatabaseError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "Database error: {}", self.message)
    }
}

impl Error for DatabaseError {
    fn source(&self) -> Option<&(dyn Error + 'static)> {
        self.source.as_ref().map(|e| e.as_ref())
    }
}

#[derive(Debug)]
struct ValidationError {
    field: String,
    message: String,
    source: Option<Box<dyn Error>>,
}

impl fmt::Display for ValidationError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "Validation error in field '{}': {}", self.field, self.message)
    }
}

impl Error for ValidationError {
    fn source(&self) -> Option<&(dyn Error + 'static)> {
        self.source.as_ref().map(|e| e.as_ref())
    }
}

fn parse_user_id(input: &str) -> Result<u32, ValidationError> {
    input.parse().map_err(|e| ValidationError {
        field: "user_id".to_string(),
        message: "Must be a valid number".to_string(),
        source: Some(Box::new(e)),
    })
}

fn validate_user_id(id: u32) -> Result<u32, ValidationError> {
    if id == 0 {
        Err(ValidationError {
            field: "user_id".to_string(),
            message: "Cannot be zero".to_string(),
            source: None,
        })
    } else if id > 1000000 {
        Err(ValidationError {
            field: "user_id".to_string(),
            message: "Too large".to_string(),
            source: None,
        })
    } else {
        Ok(id)
    }
}

fn lookup_user(id: u32) -> Result<String, DatabaseError> {
    // Simulate database lookup
    if id == 404 {
        Err(DatabaseError {
            message: "User not found".to_string(),
            source: None,
        })
    } else if id == 500 {
        Err(DatabaseError {
            message: "Connection timeout".to_string(),
            source: Some(Box::new(std::io::Error::new(
                std::io::ErrorKind::TimedOut,
                "Database connection timed out"
            ))),
        })
    } else {
        Ok(format!("User{}", id))
    }
}

fn get_user_by_input(input: &str) -> Result<String, Box<dyn Error>> {
    let id = parse_user_id(input)?;
    let validated_id = validate_user_id(id)?;
    let user = lookup_user(validated_id)?;
    Ok(user)
}

fn print_error_chain(error: &dyn Error) {
    println!("Error: {}", error);
    let mut source = error.source();
    let mut level = 1;
    
    while let Some(err) = source {
        println!("  {} Caused by: {}", "└─".repeat(level), err);
        source = err.source();
        level += 1;
    }
}

fn main() {
    let test_inputs = ["123", "0", "1000001", "abc", "404", "500"];
    
    for input in test_inputs {
        println!("\nTesting input: '{}'", input);
        match get_user_by_input(input) {
            Ok(user) => println!("Success: {}", user),
            Err(error) => print_error_chain(error.as_ref()),
        }
    }
}
```

Error chaining preserves the complete error context by linking errors through  
the source() method. This allows debugging complex error scenarios by  
tracing the full error path. The ? operator automatically propagates  
chained errors while maintaining the error hierarchy.  

## Recoverable vs unrecoverable errors

Distinguishing between errors that can be handled gracefully and those that  
require program termination.  

```rust
use std::fs;
use std::io;

#[derive(Debug)]
enum ConfigError {
    MissingFile,
    InvalidFormat(String),
    CorruptedData,
}

impl std::fmt::Display for ConfigError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            ConfigError::MissingFile => write!(f, "Configuration file is missing"),
            ConfigError::InvalidFormat(msg) => write!(f, "Invalid config format: {}", msg),
            ConfigError::CorruptedData => write!(f, "Configuration data is corrupted"),
        }
    }
}

impl std::error::Error for ConfigError {}

struct Config {
    app_name: String,
    port: u16,
    debug: bool,
}

impl Default for Config {
    fn default() -> Self {
        Config {
            app_name: "MyApp".to_string(),
            port: 8080,
            debug: false,
        }
    }
}

fn load_config(path: &str) -> Result<Config, ConfigError> {
    // Try to read the config file
    let contents = match fs::read_to_string(path) {
        Ok(contents) => contents,
        Err(ref e) if e.kind() == io::ErrorKind::NotFound => {
            return Err(ConfigError::MissingFile);
        }
        Err(_) => return Err(ConfigError::CorruptedData),
    };
    
    // Simple config parsing (key=value format)
    let mut config = Config::default();
    
    for line in contents.lines() {
        let line = line.trim();
        if line.is_empty() || line.starts_with('#') {
            continue;
        }
        
        let parts: Vec<&str> = line.split('=').collect();
        if parts.len() != 2 {
            return Err(ConfigError::InvalidFormat(
                format!("Invalid line: {}", line)
            ));
        }
        
        let key = parts[0].trim();
        let value = parts[1].trim();
        
        match key {
            "app_name" => config.app_name = value.to_string(),
            "port" => {
                config.port = value.parse().map_err(|_| {
                    ConfigError::InvalidFormat("Port must be a number".to_string())
                })?;
            }
            "debug" => {
                config.debug = value.parse().map_err(|_| {
                    ConfigError::InvalidFormat("Debug must be true/false".to_string())
                })?;
            }
            _ => {
                return Err(ConfigError::InvalidFormat(
                    format!("Unknown configuration key: {}", key)
                ));
            }
        }
    }
    
    Ok(config)
}

fn initialize_app() -> Config {
    // Try multiple config sources (recoverable errors)
    let config_paths = ["config.conf", "app.conf", "/etc/myapp/config.conf"];
    
    for path in &config_paths {
        match load_config(path) {
            Ok(config) => {
                println!("Loaded config from: {}", path);
                return config;
            }
            Err(ConfigError::MissingFile) => {
                println!("Config file {} not found, trying next...", path);
                continue;
            }
            Err(ConfigError::InvalidFormat(msg)) => {
                eprintln!("Warning: Invalid config in {}: {}", path, msg);
                continue;
            }
            Err(ConfigError::CorruptedData) => {
                // This might be recoverable, try next file
                eprintln!("Warning: Corrupted config file: {}", path);
                continue;
            }
        }
    }
    
    println!("No valid config found, using defaults");
    Config::default()
}

fn critical_system_check() -> Result<(), &'static str> {
    // Simulate checking system requirements
    let available_memory = 1024; // MB
    let required_memory = 512;
    
    if available_memory < required_memory {
        return Err("Insufficient memory to run application");
    }
    
    // Check for required system libraries
    if !std::path::Path::new("/lib/important_lib.so").exists() {
        return Err("Required system library not found");
    }
    
    Ok(())
}

fn main() {
    // Create test config files
    fs::write("config.conf", "app_name=TestApp\nport=3000\ndebug=true\n").unwrap();
    fs::write("app.conf", "invalid config\ncontent\n").unwrap();
    
    println!("=== Recoverable Error Handling ===");
    let config = initialize_app();
    println!("Final config: {:?}", format!("{}:{}:{}", config.app_name, config.port, config.debug));
    
    println!("\n=== Unrecoverable Error Handling ===");
    // For critical errors, we might need to panic or exit
    match critical_system_check() {
        Ok(()) => println!("System check passed"),
        Err(msg) => {
            eprintln!("CRITICAL ERROR: {}", msg);
            eprintln!("Application cannot continue safely");
            // In real application: std::process::exit(1);
            panic!("Critical system requirement not met");
        }
    }
    
    // Cleanup
    let _ = fs::remove_file("config.conf");
    let _ = fs::remove_file("app.conf");
}
```

Recoverable errors can be handled gracefully with fallbacks or retries,  
while unrecoverable errors indicate situations where the program cannot  
continue safely. Use Result types for recoverable errors and panic! or  
process::exit() for truly unrecoverable situations that compromise program  
integrity.  

## Error matching patterns

Advanced pattern matching techniques for handling different error scenarios  
and extracting error information.  

```rust
use std::fs;
use std::io;
use std::num::ParseIntError;

#[derive(Debug)]
enum AppError {
    Io(io::Error),
    Parse(ParseIntError),
    Network { code: u16, message: String },
    Validation { field: String, reason: String },
    Custom(String),
}

impl std::fmt::Display for AppError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            AppError::Io(e) => write!(f, "IO Error: {}", e),
            AppError::Parse(e) => write!(f, "Parse Error: {}", e),
            AppError::Network { code, message } => {
                write!(f, "Network Error {}: {}", code, message)
            }
            AppError::Validation { field, reason } => {
                write!(f, "Validation Error in {}: {}", field, reason)
            }
            AppError::Custom(msg) => write!(f, "Error: {}", msg),
        }
    }
}

impl std::error::Error for AppError {}

impl From<io::Error> for AppError {
    fn from(error: io::Error) -> Self {
        AppError::Io(error)
    }
}

impl From<ParseIntError> for AppError {
    fn from(error: ParseIntError) -> Self {
        AppError::Parse(error)
    }
}

fn simulate_operations() -> Vec<Result<String, AppError>> {
    vec![
        Ok("Success".to_string()),
        Err(AppError::Io(io::Error::new(io::ErrorKind::NotFound, "File not found"))),
        Err(AppError::Parse("invalid".parse::<i32>().unwrap_err())),
        Err(AppError::Network { code: 404, message: "Not Found".to_string() }),
        Err(AppError::Network { code: 500, message: "Internal Server Error".to_string() }),
        Err(AppError::Validation { 
            field: "email".to_string(), 
            reason: "Invalid format".to_string() 
        }),
        Err(AppError::Custom("Something went wrong".to_string())),
    ]
}

fn handle_error_detailed(error: &AppError) -> String {
    match error {
        // Pattern matching on IO error kinds
        AppError::Io(io_err) => match io_err.kind() {
            io::ErrorKind::NotFound => "File or resource not found - check path".to_string(),
            io::ErrorKind::PermissionDenied => "Access denied - check permissions".to_string(),
            io::ErrorKind::TimedOut => "Operation timed out - try again later".to_string(),
            io::ErrorKind::ConnectionRefused => "Connection refused - service may be down".to_string(),
            _ => format!("IO error: {}", io_err),
        },
        
        // Pattern matching on parse errors
        AppError::Parse(parse_err) => {
            format!("Failed to parse number: {}", parse_err)
        },
        
        // Pattern matching with guards
        AppError::Network { code, message } => match code {
            400..=499 => format!("Client error {}: {} - check your request", code, message),
            500..=599 => format!("Server error {}: {} - try again later", code, message),
            _ => format!("Network error {}: {}", code, message),
        },
        
        // Pattern matching on specific field values
        AppError::Validation { field, reason } => match field.as_str() {
            "email" => format!("Email validation failed: {}", reason),
            "password" => format!("Password validation failed: {}", reason),
            "age" => format!("Age validation failed: {}", reason),
            _ => format!("Validation failed for {}: {}", field, reason),
        },
        
        // Custom errors with message analysis
        AppError::Custom(msg) => {
            if msg.contains("timeout") {
                "Operation timed out - increase timeout or try again".to_string()
            } else if msg.contains("memory") {
                "Memory error - restart application or free up memory".to_string()
            } else {
                format!("Custom error: {}", msg)
            }
        }
    }
}

fn categorize_error(error: &AppError) -> &'static str {
    match error {
        AppError::Io(_) => "SYSTEM",
        AppError::Parse(_) => "DATA",
        AppError::Network { code, .. } if *code >= 500 => "SERVER",
        AppError::Network { .. } => "CLIENT",
        AppError::Validation { .. } => "USER_INPUT",
        AppError::Custom(_) => "APPLICATION",
    }
}

fn should_retry(error: &AppError) -> bool {
    match error {
        AppError::Io(io_err) => matches!(
            io_err.kind(),
            io::ErrorKind::TimedOut | io::ErrorKind::Interrupted
        ),
        AppError::Network { code, .. } => matches!(code, 429 | 500..=599),
        AppError::Custom(msg) => msg.contains("temporary") || msg.contains("retry"),
        _ => false,
    }
}

fn main() {
    let results = simulate_operations();
    
    println!("=== Error Pattern Matching ===");
    for (i, result) in results.iter().enumerate() {
        match result {
            Ok(value) => println!("Operation {}: Success - {}", i, value),
            Err(error) => {
                let category = categorize_error(error);
                let should_retry = should_retry(error);
                let detailed_msg = handle_error_detailed(error);
                
                println!("Operation {}: Error", i);
                println!("  Category: {}", category);
                println!("  Retry: {}", should_retry);
                println!("  Details: {}", detailed_msg);
                println!();
            }
        }
    }
    
    // Demonstrating nested pattern matching
    println!("=== Nested Pattern Matching ===");
    for result in &results[1..4] {
        let response = match result {
            Ok(_) => "Continue processing".to_string(),
            Err(AppError::Io(io_err)) if matches!(io_err.kind(), io::ErrorKind::NotFound) => {
                "Create missing file and retry".to_string()
            }
            Err(AppError::Parse(_)) => "Use default value and continue".to_string(),
            Err(AppError::Network { code: 404, .. }) => "Resource not found, skip".to_string(),
            Err(AppError::Network { code: 500..=599, .. }) => "Server error, retry later".to_string(),
            Err(_) => "Handle other errors gracefully".to_string(),
        };
        println!("Response: {}", response);
    }
}
```

Pattern matching on errors enables sophisticated error handling strategies  
based on error types, values, and conditions. Use guards to match specific  
error characteristics, nested patterns for complex error structures, and  
helper functions to categorize errors for different handling strategies.  

## File I/O error handling

Comprehensive error handling for file operations, including different types  
of I/O errors and recovery strategies.  

```rust
use std::fs::{self, File, OpenOptions};
use std::io::{self, BufRead, BufReader, Write};
use std::path::Path;

#[derive(Debug)]
enum FileError {
    NotFound(String),
    PermissionDenied(String),
    AlreadyExists(String),
    InvalidPath(String),
    IoError(io::Error),
}

impl std::fmt::Display for FileError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            FileError::NotFound(path) => write!(f, "File not found: {}", path),
            FileError::PermissionDenied(path) => write!(f, "Permission denied: {}", path),
            FileError::AlreadyExists(path) => write!(f, "File already exists: {}", path),
            FileError::InvalidPath(path) => write!(f, "Invalid path: {}", path),
            FileError::IoError(e) => write!(f, "IO error: {}", e),
        }
    }
}

impl std::error::Error for FileError {}

impl From<io::Error> for FileError {
    fn from(error: io::Error) -> Self {
        FileError::IoError(error)
    }
}

fn safe_read_file(path: &str) -> Result<String, FileError> {
    match fs::read_to_string(path) {
        Ok(contents) => Ok(contents),
        Err(error) => match error.kind() {
            io::ErrorKind::NotFound => Err(FileError::NotFound(path.to_string())),
            io::ErrorKind::PermissionDenied => Err(FileError::PermissionDenied(path.to_string())),
            _ => Err(FileError::IoError(error)),
        }
    }
}

fn safe_write_file(path: &str, contents: &str, overwrite: bool) -> Result<(), FileError> {
    // Check if file exists
    if Path::new(path).exists() && !overwrite {
        return Err(FileError::AlreadyExists(path.to_string()));
    }
    
    // Validate path
    if path.is_empty() || path.contains('\0') {
        return Err(FileError::InvalidPath(path.to_string()));
    }
    
    // Try to create parent directories
    if let Some(parent) = Path::new(path).parent() {
        if !parent.exists() {
            fs::create_dir_all(parent).map_err(|e| match e.kind() {
                io::ErrorKind::PermissionDenied => {
                    FileError::PermissionDenied(parent.to_string_lossy().to_string())
                }
                _ => FileError::IoError(e),
            })?;
        }
    }
    
    // Write the file
    fs::write(path, contents).map_err(|e| match e.kind() {
        io::ErrorKind::PermissionDenied => FileError::PermissionDenied(path.to_string()),
        _ => FileError::IoError(e),
    })
}

fn safe_copy_file(src: &str, dst: &str) -> Result<u64, FileError> {
    // Check source file exists
    if !Path::new(src).exists() {
        return Err(FileError::NotFound(src.to_string()));
    }
    
    // Check destination doesn't exist
    if Path::new(dst).exists() {
        return Err(FileError::AlreadyExists(dst.to_string()));
    }
    
    fs::copy(src, dst).map_err(|e| match e.kind() {
        io::ErrorKind::NotFound => FileError::NotFound(src.to_string()),
        io::ErrorKind::PermissionDenied => FileError::PermissionDenied(dst.to_string()),
        _ => FileError::IoError(e),
    })
}

fn read_lines_safely(path: &str) -> Result<Vec<String>, FileError> {
    let file = File::open(path).map_err(|e| match e.kind() {
        io::ErrorKind::NotFound => FileError::NotFound(path.to_string()),
        io::ErrorKind::PermissionDenied => FileError::PermissionDenied(path.to_string()),
        _ => FileError::IoError(e),
    })?;
    
    let reader = BufReader::new(file);
    let mut lines = Vec::new();
    
    for (line_num, line) in reader.lines().enumerate() {
        match line {
            Ok(content) => lines.push(content),
            Err(e) => {
                eprintln!("Warning: Error reading line {} in {}: {}", line_num + 1, path, e);
                // Continue reading other lines
                continue;
            }
        }
    }
    
    Ok(lines)
}

fn append_to_file(path: &str, content: &str) -> Result<(), FileError> {
    let mut file = OpenOptions::new()
        .create(true)
        .append(true)
        .open(path)
        .map_err(|e| match e.kind() {
            io::ErrorKind::PermissionDenied => FileError::PermissionDenied(path.to_string()),
            _ => FileError::IoError(e),
        })?;
    
    writeln!(file, "{}", content).map_err(FileError::IoError)
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    println!("=== File I/O Error Handling Demo ===");
    
    // Test writing a file
    match safe_write_file("test_data.txt", "Hello, World!\nThis is a test.", false) {
        Ok(()) => println!("✓ Successfully created test_data.txt"),
        Err(e) => println!("✗ Failed to create file: {}", e),
    }
    
    // Test reading the file
    match safe_read_file("test_data.txt") {
        Ok(contents) => println!("✓ File contents:\n{}", contents),
        Err(e) => println!("✗ Failed to read file: {}", e),
    }
    
    // Test reading non-existent file
    match safe_read_file("nonexistent.txt") {
        Ok(_) => println!("✗ Unexpected success"),
        Err(e) => println!("✓ Expected error: {}", e),
    }
    
    // Test writing file that already exists
    match safe_write_file("test_data.txt", "New content", false) {
        Ok(()) => println!("✗ Unexpected success"),
        Err(e) => println!("✓ Expected error: {}", e),
    }
    
    // Test overwriting existing file
    match safe_write_file("test_data.txt", "Overwritten content", true) {
        Ok(()) => println!("✓ Successfully overwritten file"),
        Err(e) => println!("✗ Failed to overwrite: {}", e),
    }
    
    // Test copying file
    match safe_copy_file("test_data.txt", "copy_data.txt") {
        Ok(bytes) => println!("✓ Copied {} bytes", bytes),
        Err(e) => println!("✗ Copy failed: {}", e),
    }
    
    // Test reading lines
    match read_lines_safely("test_data.txt") {
        Ok(lines) => {
            println!("✓ Read {} lines:", lines.len());
            for (i, line) in lines.iter().enumerate() {
                println!("  {}: {}", i + 1, line);
            }
        },
        Err(e) => println!("✗ Failed to read lines: {}", e),
    }
    
    // Test appending to file
    match append_to_file("test_data.txt", "Appended line") {
        Ok(()) => println!("✓ Successfully appended to file"),
        Err(e) => println!("✗ Failed to append: {}", e),
    }
    
    // Test creating file in subdirectory
    match safe_write_file("subdir/nested_file.txt", "Nested content", false) {
        Ok(()) => println!("✓ Created file in subdirectory"),
        Err(e) => println!("✗ Failed to create nested file: {}", e),
    }
    
    // Clean up
    let _ = fs::remove_file("test_data.txt");
    let _ = fs::remove_file("copy_data.txt");
    let _ = fs::remove_dir_all("subdir");
    
    Ok(())
}
```

File I/O error handling requires distinguishing between different error  
types like permission denied, file not found, and disk full conditions.  
Use pattern matching on io::ErrorKind to provide specific error messages  
and recovery strategies. Always validate paths and handle partial failures  
gracefully in file operations.  

## Network error handling

Handling network-related errors with proper classification and retry  
strategies for robust network communication.  

```rust
use std::time::Duration;
use std::thread;

#[derive(Debug)]
enum NetworkError {
    ConnectionFailed(String),
    Timeout,
    InvalidResponse(String),
    ServerError(u16),
    ClientError(u16),
    UnknownError(String),
}

impl std::fmt::Display for NetworkError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            NetworkError::ConnectionFailed(host) => write!(f, "Failed to connect to {}", host),
            NetworkError::Timeout => write!(f, "Request timed out"),
            NetworkError::InvalidResponse(msg) => write!(f, "Invalid response: {}", msg),
            NetworkError::ServerError(code) => write!(f, "Server error: {}", code),
            NetworkError::ClientError(code) => write!(f, "Client error: {}", code),
            NetworkError::UnknownError(msg) => write!(f, "Unknown network error: {}", msg),
        }
    }
}

impl std::error::Error for NetworkError {}

// Simulated HTTP client
struct HttpClient {
    timeout: Duration,
    max_retries: u32,
}

impl HttpClient {
    fn new() -> Self {
        HttpClient {
            timeout: Duration::from_secs(30),
            max_retries: 3,
        }
    }
    
    fn with_timeout(mut self, timeout: Duration) -> Self {
        self.timeout = timeout;
        self
    }
    
    fn with_retries(mut self, retries: u32) -> Self {
        self.max_retries = retries;
        self
    }
    
    // Simulated network request
    fn get(&self, url: &str) -> Result<String, NetworkError> {
        println!("Making request to: {}", url);
        
        // Simulate different network scenarios based on URL
        match url {
            url if url.contains("timeout") => Err(NetworkError::Timeout),
            url if url.contains("404") => Err(NetworkError::ClientError(404)),
            url if url.contains("500") => Err(NetworkError::ServerError(500)),
            url if url.contains("invalid") => Err(NetworkError::InvalidResponse("Malformed JSON".to_string())),
            url if url.contains("unreachable") => Err(NetworkError::ConnectionFailed(url.to_string())),
            _ => Ok(format!("Response from {}", url)),
        }
    }
    
    fn get_with_retry(&self, url: &str) -> Result<String, NetworkError> {
        let mut last_error = NetworkError::UnknownError("No attempts made".to_string());
        
        for attempt in 1..=self.max_retries {
            println!("Attempt {}/{}", attempt, self.max_retries);
            
            match self.get(url) {
                Ok(response) => return Ok(response),
                Err(error) => {
                    last_error = error;
                    
                    // Determine if we should retry based on error type
                    let should_retry = match &last_error {
                        NetworkError::Timeout => true,
                        NetworkError::ConnectionFailed(_) => true,
                        NetworkError::ServerError(code) if *code >= 500 => true,
                        NetworkError::ClientError(429) => true, // Rate limited
                        _ => false,
                    };
                    
                    if !should_retry {
                        println!("Error not retryable: {}", last_error);
                        return Err(last_error);
                    }
                    
                    if attempt < self.max_retries {
                        let delay = Duration::from_millis(1000 * attempt as u64);
                        println!("Retrying in {:?}...", delay);
                        thread::sleep(delay);
                    }
                }
            }
        }
        
        Err(last_error)
    }
}

fn batch_requests(urls: &[&str], client: &HttpClient) -> Vec<Result<String, NetworkError>> {
    urls.iter()
        .map(|&url| client.get_with_retry(url))
        .collect()
}

fn analyze_network_errors(results: &[Result<String, NetworkError>]) {
    let mut success_count = 0;
    let mut error_counts = std::collections::HashMap::new();
    
    for result in results {
        match result {
            Ok(_) => success_count += 1,
            Err(error) => {
                let error_type = match error {
                    NetworkError::ConnectionFailed(_) => "Connection",
                    NetworkError::Timeout => "Timeout",
                    NetworkError::ServerError(_) => "Server",
                    NetworkError::ClientError(_) => "Client",
                    NetworkError::InvalidResponse(_) => "Response",
                    NetworkError::UnknownError(_) => "Unknown",
                };
                *error_counts.entry(error_type).or_insert(0) += 1;
            }
        }
    }
    
    println!("\n=== Network Error Analysis ===");
    println!("Successful requests: {}", success_count);
    for (error_type, count) in error_counts {
        println!("{} errors: {}", error_type, count);
    }
}

fn main() {
    let client = HttpClient::new()
        .with_timeout(Duration::from_secs(5))
        .with_retries(2);
    
    let test_urls = [
        "https://api.example.com/users",
        "https://api.example.com/timeout",
        "https://api.example.com/404",
        "https://api.example.com/500",
        "https://api.example.com/invalid",
        "https://unreachable.invalid/api",
    ];
    
    println!("=== Network Error Handling Demo ===");
    let results = batch_requests(&test_urls, &client);
    
    for (url, result) in test_urls.iter().zip(results.iter()) {
        match result {
            Ok(response) => println!("✓ {}: {}", url, response),
            Err(error) => println!("✗ {}: {}", url, error),
        }
        println!();
    }
    
    analyze_network_errors(&results);
    
    // Demonstrate circuit breaker pattern
    println!("\n=== Circuit Breaker Pattern ===");
    let mut failure_count = 0;
    let failure_threshold = 3;
    let mut circuit_open = false;
    
    for &url in &test_urls {
        if circuit_open {
            println!("Circuit breaker open - rejecting request to {}", url);
            continue;
        }
        
        match client.get(url) {
            Ok(response) => {
                println!("Success: {}", response);
                failure_count = 0; // Reset on success
            }
            Err(error) => {
                failure_count += 1;
                println!("Failure {}/{}: {}", failure_count, failure_threshold, error);
                
                if failure_count >= failure_threshold {
                    circuit_open = true;
                    println!("Circuit breaker opened due to consecutive failures");
                }
            }
        }
    }
}
```

Network error handling requires distinguishing between transient errors  
(timeouts, connection failures) that can be retried and permanent errors  
(404, authentication) that should not. Implement exponential backoff for  
retries and circuit breaker patterns to prevent cascading failures in  
distributed systems.  

## Error aggregation

Collecting and combining multiple errors from batch operations while  
preserving individual error information.  

```rust
use std::collections::HashMap;

#[derive(Debug, Clone)]
struct ValidationError {
    field: String,
    message: String,
}

impl std::fmt::Display for ValidationError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "{}: {}", self.field, self.message)
    }
}

impl std::error::Error for ValidationError {}

#[derive(Debug)]
struct AggregatedError {
    errors: Vec<ValidationError>,
}

impl std::fmt::Display for AggregatedError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "Validation failed with {} errors:", self.errors.len())?;
        for error in &self.errors {
            write!(f, "\n  - {}", error)?;
        }
        Ok(())
    }
}

impl std::error::Error for AggregatedError {}

impl AggregatedError {
    fn new() -> Self {
        AggregatedError {
            errors: Vec::new(),
        }
    }
    
    fn add_error(&mut self, error: ValidationError) {
        self.errors.push(error);
    }
    
    fn is_empty(&self) -> bool {
        self.errors.is_empty()
    }
    
    fn into_result<T>(self, value: T) -> Result<T, AggregatedError> {
        if self.is_empty() {
            Ok(value)
        } else {
            Err(self)
        }
    }
}

#[derive(Debug)]
struct User {
    name: String,
    email: String,
    age: u8,
}

fn validate_name(name: &str) -> Result<String, ValidationError> {
    let name = name.trim();
    if name.is_empty() {
        Err(ValidationError {
            field: "name".to_string(),
            message: "Name cannot be empty".to_string(),
        })
    } else if name.len() < 2 {
        Err(ValidationError {
            field: "name".to_string(),
            message: "Name must be at least 2 characters".to_string(),
        })
    } else if name.len() > 50 {
        Err(ValidationError {
            field: "name".to_string(),
            message: "Name cannot exceed 50 characters".to_string(),
        })
    } else {
        Ok(name.to_string())
    }
}

fn validate_email(email: &str) -> Result<String, ValidationError> {
    let email = email.trim().to_lowercase();
    if email.is_empty() {
        Err(ValidationError {
            field: "email".to_string(),
            message: "Email cannot be empty".to_string(),
        })
    } else if !email.contains('@') || !email.contains('.') {
        Err(ValidationError {
            field: "email".to_string(),
            message: "Email must be in valid format".to_string(),
        })
    } else if email.len() > 100 {
        Err(ValidationError {
            field: "email".to_string(),
            message: "Email cannot exceed 100 characters".to_string(),
        })
    } else {
        Ok(email)
    }
}

fn validate_age(age: &str) -> Result<u8, ValidationError> {
    match age.parse::<u8>() {
        Ok(age) if age > 0 && age < 150 => Ok(age),
        Ok(_) => Err(ValidationError {
            field: "age".to_string(),
            message: "Age must be between 1 and 149".to_string(),
        }),
        Err(_) => Err(ValidationError {
            field: "age".to_string(),
            message: "Age must be a valid number".to_string(),
        }),
    }
}

// Validate all fields and collect errors
fn validate_user(name: &str, email: &str, age: &str) -> Result<User, AggregatedError> {
    let mut errors = AggregatedError::new();
    
    let validated_name = match validate_name(name) {
        Ok(name) => Some(name),
        Err(error) => {
            errors.add_error(error);
            None
        }
    };
    
    let validated_email = match validate_email(email) {
        Ok(email) => Some(email),
        Err(error) => {
            errors.add_error(error);
            None
        }
    };
    
    let validated_age = match validate_age(age) {
        Ok(age) => Some(age),
        Err(error) => {
            errors.add_error(error);
            None
        }
    };
    
    // Only create user if all validations passed
    match (validated_name, validated_email, validated_age) {
        (Some(name), Some(email), Some(age)) => Ok(User { name, email, age }),
        _ => Err(errors),
    }
}

// Process multiple users and collect all errors
fn process_user_batch(user_data: &[(&str, &str, &str)]) -> (Vec<User>, Vec<AggregatedError>) {
    let mut valid_users = Vec::new();
    let mut all_errors = Vec::new();
    
    for (i, &(name, email, age)) in user_data.iter().enumerate() {
        println!("Processing user {}...", i + 1);
        match validate_user(name, email, age) {
            Ok(user) => {
                println!("✓ Valid user: {:?}", user);
                valid_users.push(user);
            }
            Err(errors) => {
                println!("✗ Validation errors for user {}:", i + 1);
                for error in &errors.errors {
                    println!("  - {}", error);
                }
                all_errors.push(errors);
            }
        }
        println!();
    }
    
    (valid_users, all_errors)
}

// Aggregate errors by field type
fn analyze_errors(error_batches: &[AggregatedError]) -> HashMap<String, usize> {
    let mut field_error_counts = HashMap::new();
    
    for error_batch in error_batches {
        for error in &error_batch.errors {
            *field_error_counts.entry(error.field.clone()).or_insert(0) += 1;
        }
    }
    
    field_error_counts
}

fn main() {
    let user_data = [
        ("Alice Smith", "alice@example.com", "25"),
        ("", "bob@example.com", "30"),                    // Empty name
        ("Charlie", "invalid-email", "35"),               // Invalid email
        ("David Johnson", "david@example.com", "200"),    // Invalid age
        ("", "", "abc"),                                  // Multiple errors
        ("Eve Brown", "eve@example.com", "28"),           // Valid
        ("F", "f@test.com", "0"),                        // Name too short, age invalid
    ];
    
    println!("=== Error Aggregation Demo ===");
    let (valid_users, error_batches) = process_user_batch(&user_data);
    
    println!("=== Summary ===");
    println!("Valid users created: {}", valid_users.len());
    println!("Users with errors: {}", error_batches.len());
    
    let field_errors = analyze_errors(&error_batches);
    println!("\nError analysis by field:");
    for (field, count) in field_errors {
        println!("  {}: {} errors", field, count);
    }
    
    // Demonstrate error filtering
    println!("\n=== Error Filtering ===");
    let critical_errors: Vec<&ValidationError> = error_batches
        .iter()
        .flat_map(|batch| &batch.errors)
        .filter(|error| error.message.contains("empty"))
        .collect();
    
    println!("Critical errors (empty fields): {}", critical_errors.len());
    for error in critical_errors {
        println!("  - {}", error);
    }
    
    // Demonstrate error transformation
    let error_summary: Vec<String> = error_batches
        .iter()
        .enumerate()
        .map(|(i, batch)| {
            format!("User {}: {} error(s)", i + 1, batch.errors.len())
        })
        .collect();
    
    println!("\nError summary:");
    for summary in error_summary {
        println!("  {}", summary);
    }
}
```

Error aggregation allows collecting multiple validation errors to provide  
comprehensive feedback instead of failing on the first error. Use error  
collectors to accumulate errors during batch processing and provide  
detailed analysis of error patterns for better user experience and  
debugging.  

## Default value strategies

Implementing various strategies for providing default values when  
operations fail or values are missing.  

```rust
use std::collections::HashMap;
use std::fs;

#[derive(Debug, Clone)]
struct Config {
    database_url: String,
    port: u16,
    max_connections: u32,
    debug_mode: bool,
    api_key: Option<String>,
}

impl Default for Config {
    fn default() -> Self {
        Config {
            database_url: "localhost:5432".to_string(),
            port: 8080,
            max_connections: 100,
            debug_mode: false,
            api_key: None,
        }
    }
}

#[derive(Debug)]
enum ConfigSource {
    File(String),
    Environment,
    CommandLine,
    Default,
}

// Strategy 1: Fallback chain
fn load_config_with_fallback() -> (Config, Vec<(ConfigSource, String)>) {
    let mut attempts = Vec::new();
    
    // Try loading from file
    if let Ok(content) = fs::read_to_string("app.config") {
        attempts.push((ConfigSource::File("app.config".to_string()), "Found".to_string()));
        if let Ok(config) = parse_config(&content) {
            return (config, attempts);
        }
        attempts.push((ConfigSource::File("app.config".to_string()), "Parse failed".to_string()));
    } else {
        attempts.push((ConfigSource::File("app.config".to_string()), "Not found".to_string()));
    }
    
    // Try environment variables
    match load_from_environment() {
        Ok(config) => {
            attempts.push((ConfigSource::Environment, "Loaded".to_string()));
            return (config, attempts);
        }
        Err(msg) => {
            attempts.push((ConfigSource::Environment, msg));
        }
    }
    
    // Fall back to defaults
    attempts.push((ConfigSource::Default, "Using defaults".to_string()));
    (Config::default(), attempts)
}

// Strategy 2: Partial defaults with override
fn load_config_with_partial_defaults() -> Config {
    let mut config = Config::default();
    
    // Override with file values if available
    if let Ok(file_config) = load_from_file("app.config") {
        config.database_url = file_config.database_url;
        config.port = file_config.port;
        // Keep defaults for other fields if not specified
    }
    
    // Override with environment values
    if let Ok(env_port) = std::env::var("APP_PORT") {
        if let Ok(port) = env_port.parse() {
            config.port = port;
        }
    }
    
    if let Ok(debug) = std::env::var("DEBUG") {
        config.debug_mode = debug.to_lowercase() == "true";
    }
    
    config
}

// Strategy 3: Computed defaults
fn get_optimal_connection_count() -> u32 {
    // Simulate computing based on system resources
    let cpu_cores = 4; // Would use num_cpus crate in real code
    let available_memory = 8192; // MB
    
    std::cmp::min(cpu_cores * 25, available_memory / 32)
}

fn get_default_database_url() -> String {
    if cfg!(debug_assertions) {
        "localhost:5432/dev_db".to_string()
    } else {
        "production-db:5432/app_db".to_string()
    }
}

fn load_config_with_computed_defaults() -> Config {
    Config {
        database_url: std::env::var("DATABASE_URL")
            .unwrap_or_else(|_| get_default_database_url()),
        port: std::env::var("PORT")
            .ok()
            .and_then(|p| p.parse().ok())
            .unwrap_or(8080),
        max_connections: get_optimal_connection_count(),
        debug_mode: cfg!(debug_assertions),
        api_key: std::env::var("API_KEY").ok(),
    }
}

// Strategy 4: Validation with defaults
fn validate_with_defaults(input: &str) -> Config {
    let mut config = Config::default();
    let mut errors = Vec::new();
    
    // Parse and validate each field with fallbacks
    for line in input.lines() {
        let parts: Vec<&str> = line.split('=').collect();
        if parts.len() != 2 {
            continue;
        }
        
        let key = parts[0].trim();
        let value = parts[1].trim();
        
        match key {
            "database_url" => {
                if value.is_empty() {
                    errors.push("Empty database_url, using default".to_string());
                } else {
                    config.database_url = value.to_string();
                }
            }
            "port" => {
                match value.parse::<u16>() {
                    Ok(port) if port > 1024 => config.port = port,
                    _ => {
                        errors.push(format!("Invalid port '{}', using default {}", value, config.port));
                    }
                }
            }
            "max_connections" => {
                match value.parse::<u32>() {
                    Ok(conn) if conn > 0 && conn <= 1000 => config.max_connections = conn,
                    _ => {
                        errors.push(format!("Invalid max_connections '{}', using default {}", 
                                          value, config.max_connections));
                    }
                }
            }
            "debug_mode" => {
                config.debug_mode = value.to_lowercase() == "true";
            }
            _ => {
                errors.push(format!("Unknown configuration key: {}", key));
            }
        }
    }
    
    if !errors.is_empty() {
        println!("Configuration warnings:");
        for error in errors {
            println!("  - {}", error);
        }
    }
    
    config
}

// Helper functions
fn parse_config(content: &str) -> Result<Config, String> {
    // Simplified config parser
    let mut config = Config::default();
    
    for line in content.lines() {
        let parts: Vec<&str> = line.split('=').collect();
        if parts.len() != 2 {
            return Err("Invalid config format".to_string());
        }
        
        match parts[0].trim() {
            "database_url" => config.database_url = parts[1].trim().to_string(),
            "port" => config.port = parts[1].trim().parse().map_err(|_| "Invalid port")?,
            _ => return Err("Unknown config key".to_string()),
        }
    }
    
    Ok(config)
}

fn load_from_environment() -> Result<Config, String> {
    let database_url = std::env::var("DATABASE_URL")
        .map_err(|_| "DATABASE_URL not set".to_string())?;
    
    let port = std::env::var("PORT")
        .map_err(|_| "PORT not set".to_string())?
        .parse()
        .map_err(|_| "Invalid PORT value".to_string())?;
    
    Ok(Config {
        database_url,
        port,
        ..Config::default()
    })
}

fn load_from_file(path: &str) -> Result<Config, String> {
    let content = fs::read_to_string(path)
        .map_err(|_| "File not found".to_string())?;
    parse_config(&content)
}

fn main() {
    println!("=== Default Value Strategies Demo ===");
    
    // Create test configuration files
    fs::write("app.config", "database_url=localhost:5432\nport=3000").unwrap();
    fs::write("invalid.config", "invalid content\nno equals signs").unwrap();
    
    // Strategy 1: Fallback chain
    println!("\n--- Strategy 1: Fallback Chain ---");
    let (config, attempts) = load_config_with_fallback();
    println!("Final config: {:?}", config);
    println!("Load attempts:");
    for (source, result) in attempts {
        println!("  {:?}: {}", source, result);
    }
    
    // Strategy 2: Partial defaults
    println!("\n--- Strategy 2: Partial Defaults ---");
    let config = load_config_with_partial_defaults();
    println!("Config with partial defaults: {:?}", config);
    
    // Strategy 3: Computed defaults
    println!("\n--- Strategy 3: Computed Defaults ---");
    let config = load_config_with_computed_defaults();
    println!("Config with computed defaults: {:?}", config);
    
    // Strategy 4: Validation with defaults
    println!("\n--- Strategy 4: Validation with Defaults ---");
    let test_config = "database_url=test_db\nport=abc\nmax_connections=2000\ndebug_mode=true\ninvalid_key=value";
    let config = validate_with_defaults(test_config);
    println!("Validated config: {:?}", config);
    
    // Demonstrate Option/Result default patterns
    println!("\n--- Option/Result Default Patterns ---");
    let optional_values: Vec<Option<i32>> = vec![Some(42), None, Some(100), None];
    
    // Pattern 1: unwrap_or
    let with_defaults: Vec<i32> = optional_values.iter()
        .map(|opt| opt.unwrap_or(0))
        .collect();
    println!("With unwrap_or(0): {:?}", with_defaults);
    
    // Pattern 2: unwrap_or_else with computation
    let with_computed: Vec<i32> = optional_values.iter()
        .enumerate()
        .map(|(i, opt)| opt.unwrap_or_else(|| (i + 1) as i32 * 10))
        .collect();
    println!("With computed defaults: {:?}", with_computed);
    
    // Pattern 3: map_or for transformation with default
    let doubled_or_default: Vec<i32> = optional_values.iter()
        .map(|opt| opt.map_or(-1, |x| x * 2))
        .collect();
    println!("Doubled or -1: {:?}", doubled_or_default);
    
    // Cleanup
    let _ = fs::remove_file("app.config");
    let _ = fs::remove_file("invalid.config");
}
```

Default value strategies provide graceful degradation when configuration  
or data is missing. Use fallback chains for hierarchical configuration  
sources, computed defaults for system-dependent values, and validation  
with defaults to handle invalid input gracefully while maintaining  
application functionality.  

## Logging errors

Implementing comprehensive error logging with different levels and  
structured information for debugging and monitoring.  

```rust
use std::time::{SystemTime, UNIX_EPOCH};
use std::collections::HashMap;

#[derive(Debug, Clone)]
enum LogLevel {
    Error,
    Warn,
    Info,
    Debug,
}

impl std::fmt::Display for LogLevel {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            LogLevel::Error => write!(f, "ERROR"),
            LogLevel::Warn => write!(f, "WARN"),
            LogLevel::Info => write!(f, "INFO"),
            LogLevel::Debug => write!(f, "DEBUG"),
        }
    }
}

#[derive(Debug)]
struct LogEntry {
    timestamp: u64,
    level: LogLevel,
    message: String,
    context: HashMap<String, String>,
    error_chain: Vec<String>,
}

impl LogEntry {
    fn new(level: LogLevel, message: String) -> Self {
        LogEntry {
            timestamp: SystemTime::now()
                .duration_since(UNIX_EPOCH)
                .unwrap()
                .as_secs(),
            level,
            message,
            context: HashMap::new(),
            error_chain: Vec::new(),
        }
    }
    
    fn with_context(mut self, key: &str, value: &str) -> Self {
        self.context.insert(key.to_string(), value.to_string());
        self
    }
    
    fn with_error_chain(mut self, errors: Vec<String>) -> Self {
        self.error_chain = errors;
        self
    }
}

struct Logger {
    entries: Vec<LogEntry>,
    min_level: LogLevel,
}

impl Logger {
    fn new() -> Self {
        Logger {
            entries: Vec::new(),
            min_level: LogLevel::Info,
        }
    }
    
    fn set_level(&mut self, level: LogLevel) {
        self.min_level = level;
    }
    
    fn log(&mut self, entry: LogEntry) {
        // Simple level filtering
        let should_log = match (&entry.level, &self.min_level) {
            (LogLevel::Error, _) => true,
            (LogLevel::Warn, LogLevel::Debug | LogLevel::Info | LogLevel::Warn) => true,
            (LogLevel::Info, LogLevel::Debug | LogLevel::Info) => true,
            (LogLevel::Debug, LogLevel::Debug) => true,
            _ => false,
        };
        
        if should_log {
            println!("{}", self.format_entry(&entry));
            self.entries.push(entry);
        }
    }
    
    fn format_entry(&self, entry: &LogEntry) -> String {
        let mut formatted = format!("[{}] {}: {}", 
                                   entry.timestamp, entry.level, entry.message);
        
        if !entry.context.is_empty() {
            formatted.push_str(" | Context: ");
            for (key, value) in &entry.context {
                formatted.push_str(&format!("{}={} ", key, value));
            }
        }
        
        if !entry.error_chain.is_empty() {
            formatted.push_str(" | Error chain:");
            for (i, error) in entry.error_chain.iter().enumerate() {
                formatted.push_str(&format!("\n  {}: {}", i + 1, error));
            }
        }
        
        formatted
    }
    
    fn error(&mut self, message: &str) -> LogEntryBuilder {
        LogEntryBuilder::new(self, LogLevel::Error, message)
    }
    
    fn warn(&mut self, message: &str) -> LogEntryBuilder {
        LogEntryBuilder::new(self, LogLevel::Warn, message)
    }
    
    fn info(&mut self, message: &str) -> LogEntryBuilder {
        LogEntryBuilder::new(self, LogLevel::Info, message)
    }
    
    fn debug(&mut self, message: &str) -> LogEntryBuilder {
        LogEntryBuilder::new(self, LogLevel::Debug, message)
    }
}

struct LogEntryBuilder<'a> {
    logger: &'a mut Logger,
    entry: LogEntry,
}

impl<'a> LogEntryBuilder<'a> {
    fn new(logger: &'a mut Logger, level: LogLevel, message: &str) -> Self {
        LogEntryBuilder {
            logger,
            entry: LogEntry::new(level, message.to_string()),
        }
    }
    
    fn context(mut self, key: &str, value: &str) -> Self {
        self.entry = self.entry.with_context(key, value);
        self
    }
    
    fn error_chain<E: std::error::Error>(mut self, error: &E) -> Self {
        let mut chain = Vec::new();
        let mut current_error: &dyn std::error::Error = error;
        
        loop {
            chain.push(current_error.to_string());
            if let Some(source) = current_error.source() {
                current_error = source;
            } else {
                break;
            }
        }
        
        self.entry = self.entry.with_error_chain(chain);
        self
    }
    
    fn commit(self) {
        self.logger.log(self.entry);
    }
}

// Example error types for demonstration
#[derive(Debug)]
struct DatabaseError {
    message: String,
    source: Option<Box<dyn std::error::Error>>,
}

impl std::fmt::Display for DatabaseError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "Database error: {}", self.message)
    }
}

impl std::error::Error for DatabaseError {
    fn source(&self) -> Option<&(dyn std::error::Error + 'static)> {
        self.source.as_ref().map(|e| e.as_ref())
    }
}

#[derive(Debug)]
struct ValidationError {
    field: String,
    value: String,
}

impl std::fmt::Display for ValidationError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "Invalid {}: '{}'", self.field, self.value)
    }
}

impl std::error::Error for ValidationError {}

// Example functions that generate errors
fn process_user_data(user_id: u32, email: &str) -> Result<String, Box<dyn std::error::Error>> {
    // Validate email
    if !email.contains('@') {
        return Err(Box::new(ValidationError {
            field: "email".to_string(),
            value: email.to_string(),
        }));
    }
    
    // Simulate database error
    if user_id == 404 {
        let io_error = std::io::Error::new(std::io::ErrorKind::NotFound, "User not found");
        return Err(Box::new(DatabaseError {
            message: "Failed to query user".to_string(),
            source: Some(Box::new(io_error)),
        }));
    }
    
    // Simulate network timeout
    if user_id == 500 {
        let io_error = std::io::Error::new(std::io::ErrorKind::TimedOut, "Connection timeout");
        return Err(Box::new(DatabaseError {
            message: "Database connection failed".to_string(),
            source: Some(Box::new(io_error)),
        }));
    }
    
    Ok(format!("Processed user {} with email {}", user_id, email))
}

fn handle_request(user_id: u32, email: &str, logger: &mut Logger) -> Result<String, String> {
    logger.info("Processing user request")
        .context("user_id", &user_id.to_string())
        .context("email", email)
        .commit();
    
    match process_user_data(user_id, email) {
        Ok(result) => {
            logger.info("Request processed successfully")
                .context("user_id", &user_id.to_string())
                .context("result", &result)
                .commit();
            Ok(result)
        }
        Err(error) => {
            // Log error with full context and chain
            logger.error("Request processing failed")
                .context("user_id", &user_id.to_string())
                .context("email", email)
                .context("operation", "process_user_data")
                .error_chain(error.as_ref())
                .commit();
            
            Err("Internal server error".to_string())
        }
    }
}

fn analyze_error_logs(logger: &Logger) {
    let mut error_count = 0;
    let mut warning_count = 0;
    let mut error_types = HashMap::new();
    
    for entry in &logger.entries {
        match entry.level {
            LogLevel::Error => {
                error_count += 1;
                // Analyze error types from error chain
                if let Some(first_error) = entry.error_chain.first() {
                    if first_error.contains("Database") {
                        *error_types.entry("Database").or_insert(0) += 1;
                    } else if first_error.contains("Invalid") {
                        *error_types.entry("Validation").or_insert(0) += 1;
                    } else {
                        *error_types.entry("Other").or_insert(0) += 1;
                    }
                }
            }
            LogLevel::Warn => warning_count += 1,
            _ => {}
        }
    }
    
    println!("\n=== Error Log Analysis ===");
    println!("Total log entries: {}", logger.entries.len());
    println!("Error count: {}", error_count);
    println!("Warning count: {}", warning_count);
    
    if !error_types.is_empty() {
        println!("Error types:");
        for (error_type, count) in error_types {
            println!("  {}: {}", error_type, count);
        }
    }
}

fn main() {
    let mut logger = Logger::new();
    logger.set_level(LogLevel::Debug);
    
    println!("=== Error Logging Demo ===");
    
    // Test different scenarios
    let test_cases = [
        (123, "user@example.com"),      // Success
        (456, "invalid-email"),         // Validation error
        (404, "missing@example.com"),   // Database error (not found)
        (500, "timeout@example.com"),   // Database error (timeout)
        (789, "valid@test.com"),        // Success
    ];
    
    for (user_id, email) in test_cases {
        println!("\n--- Processing user {} ---", user_id);
        match handle_request(user_id, email, &mut logger) {
            Ok(result) => println!("Success: {}", result),
            Err(error) => println!("Error: {}", error),
        }
    }
    
    // Demonstrate different log levels
    println!("\n--- Additional logging examples ---");
    
    logger.debug("Debug information")
        .context("component", "auth")
        .context("action", "token_refresh")
        .commit();
    
    logger.warn("Performance warning")
        .context("response_time", "2.5s")
        .context("threshold", "1.0s")
        .commit();
    
    logger.info("System status")
        .context("uptime", "72h")
        .context("active_connections", "42")
        .commit();
    
    // Analyze collected logs
    analyze_error_logs(&logger);
}
```

Structured error logging captures essential debugging information including  
error chains, contextual data, and timestamps. Use log levels to control  
verbosity, add context for debugging, and preserve error chain information  
to trace root causes. This approach enables effective monitoring and  
troubleshooting in production systems.  

## Testing error scenarios

Comprehensive testing of error handling paths to ensure robust error  
behavior and proper error propagation.  

```rust
use std::panic;

#[derive(Debug, PartialEq)]
enum CalculatorError {
    DivisionByZero,
    Overflow,
    InvalidInput(String),
}

impl std::fmt::Display for CalculatorError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            CalculatorError::DivisionByZero => write!(f, "Division by zero"),
            CalculatorError::Overflow => write!(f, "Arithmetic overflow"),
            CalculatorError::InvalidInput(msg) => write!(f, "Invalid input: {}", msg),
        }
    }
}

impl std::error::Error for CalculatorError {}

struct Calculator;

impl Calculator {
    fn divide(a: f64, b: f64) -> Result<f64, CalculatorError> {
        if b == 0.0 {
            Err(CalculatorError::DivisionByZero)
        } else if a.is_infinite() || b.is_infinite() {
            Err(CalculatorError::Overflow)
        } else {
            Ok(a / b)
        }
    }
    
    fn parse_and_divide(input: &str) -> Result<f64, CalculatorError> {
        let parts: Vec<&str> = input.split('/').collect();
        if parts.len() != 2 {
            return Err(CalculatorError::InvalidInput(
                "Expected format: 'number/number'".to_string()
            ));
        }
        
        let a: f64 = parts[0].trim().parse()
            .map_err(|_| CalculatorError::InvalidInput(
                format!("'{}' is not a valid number", parts[0])
            ))?;
        
        let b: f64 = parts[1].trim().parse()
            .map_err(|_| CalculatorError::InvalidInput(
                format!("'{}' is not a valid number", parts[1])
            ))?;
        
        Self::divide(a, b)
    }
    
    fn safe_operation<F>(operation: F) -> Result<f64, CalculatorError> 
    where
        F: FnOnce() -> f64 + panic::UnwindSafe,
    {
        match panic::catch_unwind(operation) {
            Ok(result) => {
                if result.is_infinite() {
                    Err(CalculatorError::Overflow)
                } else if result.is_nan() {
                    Err(CalculatorError::InvalidInput("Result is NaN".to_string()))
                } else {
                    Ok(result)
                }
            }
            Err(_) => Err(CalculatorError::InvalidInput("Operation panicked".to_string()))
        }
    }
}

// Test framework simulation
struct TestResult {
    name: String,
    passed: bool,
    error_message: Option<String>,
}

impl TestResult {
    fn success(name: &str) -> Self {
        TestResult {
            name: name.to_string(),
            passed: true,
            error_message: None,
        }
    }
    
    fn failure(name: &str, error: &str) -> Self {
        TestResult {
            name: name.to_string(),
            passed: false,
            error_message: Some(error.to_string()),
        }
    }
}

macro_rules! assert_error {
    ($result:expr, $expected_error:pat) => {
        match $result {
            Err($expected_error) => true,
            Err(other) => {
                panic!("Expected {:?}, got {:?}", stringify!($expected_error), other);
            }
            Ok(value) => {
                panic!("Expected error {:?}, got Ok({:?})", stringify!($expected_error), value);
            }
        }
    };
}

fn test_division_by_zero() -> TestResult {
    let result = Calculator::divide(10.0, 0.0);
    
    if assert_error!(result, CalculatorError::DivisionByZero) {
        TestResult::success("test_division_by_zero")
    } else {
        TestResult::failure("test_division_by_zero", "Did not get expected DivisionByZero error")
    }
}

fn test_valid_division() -> TestResult {
    match Calculator::divide(10.0, 2.0) {
        Ok(result) => {
            if (result - 5.0).abs() < f64::EPSILON {
                TestResult::success("test_valid_division")
            } else {
                TestResult::failure("test_valid_division", &format!("Expected 5.0, got {}", result))
            }
        }
        Err(e) => TestResult::failure("test_valid_division", &format!("Unexpected error: {}", e))
    }
}

fn test_overflow_detection() -> TestResult {
    let result = Calculator::divide(f64::INFINITY, 1.0);
    
    if assert_error!(result, CalculatorError::Overflow) {
        TestResult::success("test_overflow_detection")
    } else {
        TestResult::failure("test_overflow_detection", "Did not detect overflow")
    }
}

fn test_parse_invalid_format() -> TestResult {
    let result = Calculator::parse_and_divide("invalid input");
    
    match result {
        Err(CalculatorError::InvalidInput(msg)) if msg.contains("Expected format") => {
            TestResult::success("test_parse_invalid_format")
        }
        _ => TestResult::failure("test_parse_invalid_format", "Did not get expected format error")
    }
}

fn test_parse_invalid_numbers() -> TestResult {
    let test_cases = [
        "abc/5",
        "10/xyz",
        "not/numbers",
    ];
    
    for input in test_cases {
        let result = Calculator::parse_and_divide(input);
        
        match result {
            Err(CalculatorError::InvalidInput(_)) => continue,
            _ => {
                return TestResult::failure(
                    "test_parse_invalid_numbers", 
                    &format!("Input '{}' should have failed", input)
                );
            }
        }
    }
    
    TestResult::success("test_parse_invalid_numbers")
}

fn test_error_propagation() -> TestResult {
    // Test that parse errors propagate correctly
    let result = Calculator::parse_and_divide("10/0");
    
    if assert_error!(result, CalculatorError::DivisionByZero) {
        TestResult::success("test_error_propagation")
    } else {
        TestResult::failure("test_error_propagation", "Error did not propagate correctly")
    }
}

fn test_panic_recovery() -> TestResult {
    let result = Calculator::safe_operation(|| {
        panic!("Intentional panic for testing");
    });
    
    match result {
        Err(CalculatorError::InvalidInput(msg)) if msg.contains("panicked") => {
            TestResult::success("test_panic_recovery")
        }
        _ => TestResult::failure("test_panic_recovery", "Did not handle panic correctly")
    }
}

fn test_boundary_conditions() -> TestResult {
    let boundary_tests = [
        (f64::MIN, 1.0, true),          // Very small number
        (f64::MAX, 1.0, false),         // Very large number (might overflow)
        (0.0, f64::MIN_POSITIVE, true), // Division by very small number
        (1.0, f64::NAN, false),         // Division by NaN
    ];
    
    for (a, b, should_succeed) in boundary_tests {
        let result = Calculator::divide(a, b);
        
        match (result.is_ok(), should_succeed) {
            (true, true) | (false, false) => continue,
            (true, false) => {
                return TestResult::failure(
                    "test_boundary_conditions",
                    &format!("Expected error for {}/{}, got success", a, b)
                );
            }
            (false, true) => {
                return TestResult::failure(
                    "test_boundary_conditions",
                    &format!("Expected success for {}/{}, got error", a, b)
                );
            }
        }
    }
    
    TestResult::success("test_boundary_conditions")
}

fn test_error_message_quality() -> TestResult {
    let result = Calculator::parse_and_divide("hello/world");
    
    match result {
        Err(CalculatorError::InvalidInput(msg)) => {
            // Check that error message is informative
            if msg.contains("hello") && !msg.is_empty() {
                TestResult::success("test_error_message_quality")
            } else {
                TestResult::failure(
                    "test_error_message_quality",
                    "Error message not informative enough"
                )
            }
        }
        _ => TestResult::failure("test_error_message_quality", "Wrong error type")
    }
}

fn test_multiple_error_scenarios() -> TestResult {
    let test_scenarios = [
        ("10/2", true, None),
        ("0/5", true, None),
        ("10/0", false, Some("DivisionByZero")),
        ("abc/5", false, Some("InvalidInput")),
        ("10/def", false, Some("InvalidInput")),
        ("", false, Some("InvalidInput")),
    ];
    
    for (input, should_succeed, expected_error_type) in test_scenarios {
        let result = Calculator::parse_and_divide(input);
        
        match (result, should_succeed) {
            (Ok(_), true) => continue,
            (Err(error), false) => {
                if let Some(expected) = expected_error_type {
                    let error_name = match error {
                        CalculatorError::DivisionByZero => "DivisionByZero",
                        CalculatorError::Overflow => "Overflow",
                        CalculatorError::InvalidInput(_) => "InvalidInput",
                    };
                    
                    if error_name != expected {
                        return TestResult::failure(
                            "test_multiple_error_scenarios",
                            &format!("Expected {} for '{}', got {}", expected, input, error_name)
                        );
                    }
                }
                continue;
            }
            (Ok(_), false) => {
                return TestResult::failure(
                    "test_multiple_error_scenarios",
                    &format!("Expected error for '{}', got success", input)
                );
            }
            (Err(error), true) => {
                return TestResult::failure(
                    "test_multiple_error_scenarios",
                    &format!("Expected success for '{}', got error: {}", input, error)
                );
            }
        }
    }
    
    TestResult::success("test_multiple_error_scenarios")
}

fn run_test_suite() -> Vec<TestResult> {
    vec![
        test_division_by_zero(),
        test_valid_division(),
        test_overflow_detection(),
        test_parse_invalid_format(),
        test_parse_invalid_numbers(),
        test_error_propagation(),
        test_panic_recovery(),
        test_boundary_conditions(),
        test_error_message_quality(),
        test_multiple_error_scenarios(),
    ]
}

fn main() {
    println!("=== Error Scenario Testing Demo ===");
    
    let results = run_test_suite();
    let mut passed = 0;
    let mut failed = 0;
    
    for result in &results {
        if result.passed {
            println!("✓ {}", result.name);
            passed += 1;
        } else {
            println!("✗ {}: {}", result.name, 
                   result.error_message.as_ref().unwrap_or(&"Unknown error".to_string()));
            failed += 1;
        }
    }
    
    println!("\n=== Test Summary ===");
    println!("Total tests: {}", results.len());
    println!("Passed: {}", passed);
    println!("Failed: {}", failed);
    println!("Success rate: {:.1}%", (passed as f64 / results.len() as f64) * 100.0);
    
    if failed == 0 {
        println!("🎉 All tests passed!");
    } else {
        println!("⚠️  Some tests failed. Review error handling implementation.");
    }
}
```

Testing error scenarios ensures robust error handling by validating all  
error paths, boundary conditions, and error propagation. Use dedicated  
test functions for each error type, test boundary conditions, verify  
error message quality, and ensure proper error propagation through the  
call stack. This comprehensive testing approach builds confidence in  
error handling reliability.  
