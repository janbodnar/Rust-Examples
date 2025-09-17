# Enums

Enums (enumerations) in Rust are powerful algebraic data types that allow  
you to define a type by enumerating its possible variants. Unlike enums in  
many other languages, Rust enums are true sum types that can hold data of  
different types and sizes, making them incredibly versatile for modeling  
complex data structures and states.  

Rust enums enable pattern matching, exhaustive case handling, and zero-cost  
abstractions. They're fundamental to Rust's type system and are used  
extensively in the standard library (Option, Result) and throughout the  
ecosystem for representing alternatives, states, and hierarchical data.  

Each variant can be a unit variant (no data), tuple variant (with unnamed  
fields), or struct variant (with named fields). This flexibility makes  
enums perfect for state machines, error handling, and complex data modeling.  

## Basic unit enums

Simple enums with unit variants that represent distinct states or values  
without associated data.  

```rust
#[derive(Debug, PartialEq, Clone, Copy)]
enum Direction {
    North,
    South,
    East,
    West,
}

#[derive(Debug, PartialEq)]
enum Status {
    Active,
    Inactive,
    Pending,
    Suspended,
}

fn main() {
    let current_direction = Direction::North;
    let next_direction = Direction::East;
    
    println!("Current direction: {:?}", current_direction);
    println!("Next direction: {:?}", next_direction);
    
    // Comparing enum values
    if current_direction == Direction::North {
        println!("Heading north!");
    }
    
    let user_status = Status::Active;
    match user_status {
        Status::Active => println!("User is active"),
        Status::Inactive => println!("User is inactive"),
        Status::Pending => println!("User is pending approval"),
        Status::Suspended => println!("User is suspended"),
    }
    
    // Using enums in collections
    let directions = vec![Direction::North, Direction::East, Direction::South];
    for dir in directions {
        println!("Moving: {:?}", dir);
    }
}
```

This example shows basic enum definition with unit variants. The derive  
attributes add common traits: Debug for printing, PartialEq for comparison,  
Clone and Copy for easy duplication. Unit variants are accessed using  
double colon syntax and can be compared directly or used in match expressions.  

## Enums with tuple data

Enums where variants can hold data using tuple-like syntax, allowing each  
variant to carry different types and amounts of data.  

```rust
#[derive(Debug)]
enum IpAddress {
    V4(u8, u8, u8, u8),
    V6(String),
}

#[derive(Debug)]
enum Event {
    KeyPress(char),
    MouseClick(i32, i32),
    WindowResize(u32, u32),
    TextInput(String),
    Quit,
}

fn main() {
    let localhost_v4 = IpAddress::V4(127, 0, 0, 1);
    let localhost_v6 = IpAddress::V6(String::from("::1"));
    
    println!("IPv4: {:?}", localhost_v4);
    println!("IPv6: {:?}", localhost_v6);
    
    // Pattern matching to extract data
    match localhost_v4 {
        IpAddress::V4(a, b, c, d) => {
            println!("IPv4 address: {}.{}.{}.{}", a, b, c, d);
        }
        IpAddress::V6(addr) => {
            println!("IPv6 address: {}", addr);
        }
    }
    
    // Handling events
    let events = vec![
        Event::KeyPress('a'),
        Event::MouseClick(100, 200),
        Event::WindowResize(800, 600),
        Event::TextInput(String::from("Hello")),
        Event::Quit,
    ];
    
    for event in events {
        handle_event(event);
    }
}

fn handle_event(event: Event) {
    match event {
        Event::KeyPress(key) => println!("Key pressed: {}", key),
        Event::MouseClick(x, y) => println!("Mouse clicked at ({}, {})", x, y),
        Event::WindowResize(width, height) => {
            println!("Window resized to {}x{}", width, height);
        }
        Event::TextInput(text) => println!("Text input: {}", text),
        Event::Quit => println!("Quit event received"),
    }
}
```

Tuple variants allow enums to carry data of different types. Pattern  
matching with tuple syntax extracts the data for processing. This pattern  
is common for events, network protocols, and data parsing where each  
variant needs to carry specific associated information.  

## Enums with struct-like variants

Enums with named fields that provide clarity and structure when variants  
need multiple related pieces of data.  

```rust
#[derive(Debug)]
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write { text: String, color: String },
    ChangeColor { r: u8, g: u8, b: u8 },
}

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

fn main() {
    let messages = vec![
        Message::Quit,
        Message::Move { x: 10, y: 20 },
        Message::Write {
            text: String::from("Hello, World!"),
            color: String::from("blue"),
        },
        Message::ChangeColor { r: 255, g: 0, b: 128 },
    ];
    
    for message in messages {
        process_message(message);
    }
    
    let fs_entries = vec![
        FileSystemEntry::File {
            name: String::from("document.txt"),
            size: 1024,
            extension: String::from("txt"),
        },
        FileSystemEntry::Directory {
            name: String::from("projects"),
            entries: vec![
                String::from("rust_app"),
                String::from("web_server"),
            ],
        },
        FileSystemEntry::Symlink {
            name: String::from("latest"),
            target: String::from("/usr/bin/app-v2.1"),
        },
    ];
    
    for entry in fs_entries {
        print_fs_entry(&entry);
    }
}

fn process_message(msg: Message) {
    match msg {
        Message::Quit => println!("Quitting application"),
        Message::Move { x, y } => println!("Moving to position ({}, {})", x, y),
        Message::Write { text, color } => {
            println!("Writing '{}' in {} color", text, color);
        }
        Message::ChangeColor { r, g, b } => {
            println!("Changing color to RGB({}, {}, {})", r, g, b);
        }
    }
}

fn print_fs_entry(entry: &FileSystemEntry) {
    match entry {
        FileSystemEntry::File { name, size, extension } => {
            println!("File: {} ({} bytes, .{})", name, size, extension);
        }
        FileSystemEntry::Directory { name, entries } => {
            println!("Directory: {} ({} entries)", name, entries.len());
        }
        FileSystemEntry::Symlink { name, target } => {
            println!("Symlink: {} -> {}", name, target);
        }
    }
}
```

Struct-like variants use named fields for clarity and better documentation.  
The destructuring syntax in match expressions directly binds field names  
to variables. This approach is ideal when variants have multiple related  
fields that benefit from descriptive naming.  

## Pattern matching with guards

Advanced pattern matching techniques using guards, wildcards, and binding  
to handle complex enum scenarios.  

```rust
#[derive(Debug)]
enum Temperature {
    Celsius(f32),
    Fahrenheit(f32),
    Kelvin(f32),
}

#[derive(Debug)]
enum Response {
    Success { code: u16, data: String },
    Error { code: u16, message: String },
    Redirect { code: u16, location: String },
}

fn main() {
    let temperatures = vec![
        Temperature::Celsius(25.0),
        Temperature::Fahrenheit(77.0),
        Temperature::Kelvin(298.15),
        Temperature::Celsius(-10.0),
        Temperature::Fahrenheit(212.0),
    ];
    
    for temp in temperatures {
        analyze_temperature(temp);
    }
    
    let responses = vec![
        Response::Success {
            code: 200,
            data: String::from("User data"),
        },
        Response::Error {
            code: 404,
            message: String::from("Not found"),
        },
        Response::Redirect {
            code: 301,
            location: String::from("/new-page"),
        },
        Response::Error {
            code: 500,
            message: String::from("Internal server error"),
        },
    ];
    
    for response in responses {
        handle_response(response);
    }
}

fn analyze_temperature(temp: Temperature) {
    match temp {
        Temperature::Celsius(c) if c < 0.0 => {
            println!("Freezing! {}°C", c);
        }
        Temperature::Celsius(c) if c > 35.0 => {
            println!("Very hot! {}°C", c);
        }
        Temperature::Celsius(c) => {
            println!("Comfortable temperature: {}°C", c);
        }
        Temperature::Fahrenheit(f) if f > 100.0 => {
            println!("Hot! {}°F", f);
        }
        Temperature::Fahrenheit(f) => {
            println!("Temperature: {}°F", f);
        }
        Temperature::Kelvin(k) if k < 273.15 => {
            println!("Below freezing: {}K", k);
        }
        Temperature::Kelvin(k) => {
            println!("Temperature: {}K", k);
        }
    }
}

fn handle_response(response: Response) {
    match response {
        Response::Success { code: 200, data } => {
            println!("OK: {}", data);
        }
        Response::Success { code, data } => {
            println!("Success ({}): {}", code, data);
        }
        Response::Error { code: 404, .. } => {
            println!("Resource not found");
        }
        Response::Error { code, message } if code >= 500 => {
            println!("Server error ({}): {}", code, message);
        }
        Response::Error { code, message } => {
            println!("Client error ({}): {}", code, message);
        }
        Response::Redirect { location, .. } => {
            println!("Redirecting to: {}", location);
        }
    }
}
```

Pattern matching with guards (if conditions) enables fine-grained control  
over variant handling. The .. syntax ignores remaining fields, and specific  
value matching (like code: 200) filters variants precisely. Guards allow  
conditional logic within pattern matching for sophisticated control flow.  

## Option enum usage

Working with Rust's Option enum to handle nullable values safely without  
null pointer risks.  

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];
    
    // Safe indexing that returns Option
    println!("Element at index 2: {:?}", safe_get(&numbers, 2));
    println!("Element at index 10: {:?}", safe_get(&numbers, 10));
    
    // Using Option methods
    let some_number = Some(42);
    let no_number: Option<i32> = None;
    
    println!("Some number doubled: {:?}", some_number.map(|x| x * 2));
    println!("No number doubled: {:?}", no_number.map(|x| x * 2));
    
    // Chaining operations
    let result = Some(10)
        .map(|x| x * 2)
        .filter(|&x| x > 15)
        .map(|x| x + 5);
    println!("Chained result: {:?}", result);
    
    // Working with Option in practice
    let user_input = "42";
    let parsed_number = parse_number(user_input);
    
    match parsed_number {
        Some(num) => println!("Parsed number: {}", num),
        None => println!("Invalid number format"),
    }
    
    // Using unwrap_or for defaults
    let default_value = no_number.unwrap_or(0);
    println!("Default value: {}", default_value);
    
    // Using and_then for chaining Options
    let maybe_string = Some(String::from("123"));
    let maybe_doubled = maybe_string
        .as_ref()
        .and_then(|s| s.parse::<i32>().ok())
        .map(|n| n * 2);
    println!("Maybe doubled: {:?}", maybe_doubled);
    
    // Combining Options
    let x = Some(3);
    let y = Some(4);
    let sum = combine_options(x, y);
    println!("Combined: {:?}", sum);
}

fn safe_get<T>(vec: &Vec<T>, index: usize) -> Option<&T> {
    if index < vec.len() {
        Some(&vec[index])
    } else {
        None
    }
}

fn parse_number(input: &str) -> Option<i32> {
    input.parse().ok()
}

fn combine_options(a: Option<i32>, b: Option<i32>) -> Option<i32> {
    match (a, b) {
        (Some(x), Some(y)) => Some(x + y),
        _ => None,
    }
}
```

Option<T> replaces null pointers in Rust, forcing explicit handling of  
potentially absent values. The rich method set (map, filter, and_then,  
unwrap_or) enables functional programming patterns. Pattern matching on  
Option variants ensures all cases are handled at compile time.  

## Result enum for error handling

Using Result enum to handle operations that can fail, providing structured  
error handling without exceptions.  

```rust
use std::fs::File;
use std::io::{self, Read};

#[derive(Debug)]
enum MathError {
    DivisionByZero,
    NegativeSquareRoot,
    Overflow,
}

#[derive(Debug)]
enum ParseError {
    InvalidFormat,
    OutOfRange,
    Empty,
}

fn main() {
    // Basic Result usage
    println!("Division result: {:?}", safe_divide(10.0, 2.0));
    println!("Division by zero: {:?}", safe_divide(10.0, 0.0));
    
    // Chaining Result operations
    let input = "42";
    let result = parse_and_double(input);
    match result {
        Ok(value) => println!("Parsed and doubled: {}", value),
        Err(e) => println!("Error: {:?}", e),
    }
    
    // Using Result methods
    let success: Result<i32, &str> = Ok(42);
    let failure: Result<i32, &str> = Err("Something went wrong");
    
    println!("Success mapped: {:?}", success.map(|x| x * 2));
    println!("Failure mapped: {:?}", failure.map(|x| x * 2));
    
    // Error propagation with ?
    let math_result = complex_calculation(16.0);
    match math_result {
        Ok(value) => println!("Calculation result: {}", value),
        Err(e) => println!("Math error: {:?}", e),
    }
    
    // Converting between Result and Option
    let maybe_file = read_file_content("nonexistent.txt");
    let content_length = maybe_file
        .map(|content| content.len())
        .unwrap_or(0);
    println!("Content length: {}", content_length);
    
    // Multiple error types
    demonstrate_error_handling();
}

fn safe_divide(a: f64, b: f64) -> Result<f64, MathError> {
    if b == 0.0 {
        Err(MathError::DivisionByZero)
    } else if a.is_infinite() || b.is_infinite() {
        Err(MathError::Overflow)
    } else {
        Ok(a / b)
    }
}

fn safe_sqrt(x: f64) -> Result<f64, MathError> {
    if x < 0.0 {
        Err(MathError::NegativeSquareRoot)
    } else {
        Ok(x.sqrt())
    }
}

fn parse_and_double(input: &str) -> Result<i32, ParseError> {
    if input.is_empty() {
        return Err(ParseError::Empty);
    }
    
    let number = input.parse::<i32>()
        .map_err(|_| ParseError::InvalidFormat)?;
    
    number.checked_mul(2)
        .ok_or(ParseError::OutOfRange)
}

fn complex_calculation(x: f64) -> Result<f64, MathError> {
    let sqrt_result = safe_sqrt(x)?;
    let division_result = safe_divide(sqrt_result, 2.0)?;
    Ok(division_result)
}

fn read_file_content(filename: &str) -> Result<String, io::Error> {
    let mut file = File::open(filename)?;
    let mut content = String::new();
    file.read_to_string(&mut content)?;
    Ok(content)
}

fn demonstrate_error_handling() {
    let operations = vec![
        ("Valid", || safe_divide(10.0, 2.0)),
        ("Division by zero", || safe_divide(10.0, 0.0)),
        ("Square root", || safe_sqrt(-4.0)),
    ];
    
    for (name, operation) in operations {
        match operation() {
            Ok(result) => println!("{}: {}", name, result),
            Err(error) => println!("{}: Error - {:?}", name, error),
        }
    }
}
```

Result<T, E> provides structured error handling without exceptions. The ?  
operator enables clean error propagation, and the rich method set allows  
functional error handling patterns. Custom error types make error handling  
domain-specific and self-documenting.  

## Custom enums with methods

Implementing methods on enums to encapsulate behavior and create  
self-contained, object-oriented designs.  

```rust
#[derive(Debug, Clone)]
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

#[derive(Debug)]
enum Shape {
    Circle { radius: f64 },
    Rectangle { width: f64, height: f64 },
    Triangle { a: f64, b: f64, c: f64 },
}

#[derive(Debug)]
enum BankAccount {
    Checking { balance: f64, overdraft_limit: f64 },
    Savings { balance: f64, interest_rate: f64 },
    Certificate { balance: f64, maturity_months: u32 },
}

impl TrafficLight {
    fn next(&self) -> TrafficLight {
        match self {
            TrafficLight::Red => TrafficLight::Green,
            TrafficLight::Yellow => TrafficLight::Red,
            TrafficLight::Green => TrafficLight::Yellow,
        }
    }
    
    fn can_proceed(&self) -> bool {
        matches!(self, TrafficLight::Green)
    }
    
    fn duration_seconds(&self) -> u32 {
        match self {
            TrafficLight::Red => 60,
            TrafficLight::Yellow => 5,
            TrafficLight::Green => 30,
        }
    }
}

impl Shape {
    fn area(&self) -> f64 {
        match self {
            Shape::Circle { radius } => std::f64::consts::PI * radius * radius,
            Shape::Rectangle { width, height } => width * height,
            Shape::Triangle { a, b, c } => {
                let s = (a + b + c) / 2.0;
                (s * (s - a) * (s - b) * (s - c)).sqrt()
            }
        }
    }
    
    fn perimeter(&self) -> f64 {
        match self {
            Shape::Circle { radius } => 2.0 * std::f64::consts::PI * radius,
            Shape::Rectangle { width, height } => 2.0 * (width + height),
            Shape::Triangle { a, b, c } => a + b + c,
        }
    }
    
    fn scale(&mut self, factor: f64) {
        match self {
            Shape::Circle { radius } => *radius *= factor,
            Shape::Rectangle { width, height } => {
                *width *= factor;
                *height *= factor;
            }
            Shape::Triangle { a, b, c } => {
                *a *= factor;
                *b *= factor;
                *c *= factor;
            }
        }
    }
}

impl BankAccount {
    fn balance(&self) -> f64 {
        match self {
            BankAccount::Checking { balance, .. } => *balance,
            BankAccount::Savings { balance, .. } => *balance,
            BankAccount::Certificate { balance, .. } => *balance,
        }
    }
    
    fn withdraw(&mut self, amount: f64) -> Result<f64, String> {
        match self {
            BankAccount::Checking { balance, overdraft_limit } => {
                if *balance + *overdraft_limit >= amount {
                    *balance -= amount;
                    Ok(*balance)
                } else {
                    Err("Insufficient funds".to_string())
                }
            }
            BankAccount::Savings { balance, .. } => {
                if *balance >= amount {
                    *balance -= amount;
                    Ok(*balance)
                } else {
                    Err("Insufficient funds".to_string())
                }
            }
            BankAccount::Certificate { .. } => {
                Err("Cannot withdraw from certificate account".to_string())
            }
        }
    }
    
    fn deposit(&mut self, amount: f64) {
        match self {
            BankAccount::Checking { balance, .. } => *balance += amount,
            BankAccount::Savings { balance, .. } => *balance += amount,
            BankAccount::Certificate { balance, .. } => *balance += amount,
        }
    }
}

fn main() {
    // Traffic light example
    let mut light = TrafficLight::Red;
    for _ in 0..4 {
        println!("Light: {:?}, Can proceed: {}, Duration: {}s", 
                 light, light.can_proceed(), light.duration_seconds());
        light = light.next();
    }
    
    // Shape calculations
    let mut shapes = vec![
        Shape::Circle { radius: 5.0 },
        Shape::Rectangle { width: 4.0, height: 6.0 },
        Shape::Triangle { a: 3.0, b: 4.0, c: 5.0 },
    ];
    
    for shape in &shapes {
        println!("Shape: {:?}", shape);
        println!("  Area: {:.2}", shape.area());
        println!("  Perimeter: {:.2}", shape.perimeter());
    }
    
    // Scale shapes
    for shape in &mut shapes {
        shape.scale(2.0);
    }
    println!("\nAfter scaling by 2:");
    for shape in &shapes {
        println!("  Area: {:.2}", shape.area());
    }
    
    // Bank account operations
    let mut accounts = vec![
        BankAccount::Checking { balance: 1000.0, overdraft_limit: 500.0 },
        BankAccount::Savings { balance: 5000.0, interest_rate: 0.025 },
        BankAccount::Certificate { balance: 10000.0, maturity_months: 12 },
    ];
    
    for account in &mut accounts {
        println!("Account: {:?}", account);
        println!("  Balance: ${:.2}", account.balance());
        
        account.deposit(100.0);
        println!("  After $100 deposit: ${:.2}", account.balance());
        
        match account.withdraw(200.0) {
            Ok(new_balance) => println!("  After $200 withdrawal: ${:.2}", new_balance),
            Err(e) => println!("  Withdrawal failed: {}", e),
        }
    }
}
```

Methods on enums enable object-oriented patterns while maintaining the  
safety of pattern matching. Methods can access and modify enum data,  
implement state transitions, and provide domain-specific behavior. This  
approach creates clean, encapsulated APIs for complex enum types.  

## Generic enums

Creating flexible enums that work with multiple types using generics,  
enabling code reuse and type safety.  

```rust
#[derive(Debug, Clone)]
enum Container<T> {
    Empty,
    Single(T),
    Multiple(Vec<T>),
}

#[derive(Debug)]
enum Either<L, R> {
    Left(L),
    Right(R),
}

#[derive(Debug)]
enum Tree<T> {
    Leaf,
    Node {
        value: T,
        left: Box<Tree<T>>,
        right: Box<Tree<T>>,
    },
}

#[derive(Debug)]
enum Operation<T> {
    Add(T, T),
    Subtract(T, T),
    Multiply(T, T),
    Divide(T, T),
}

impl<T> Container<T> {
    fn new() -> Self {
        Container::Empty
    }
    
    fn add(&mut self, item: T) {
        match self {
            Container::Empty => *self = Container::Single(item),
            Container::Single(existing) => {
                let old_item = std::mem::replace(existing, item);
                *self = Container::Multiple(vec![old_item, item]);
            }
            Container::Multiple(items) => items.push(item),
        }
    }
    
    fn len(&self) -> usize {
        match self {
            Container::Empty => 0,
            Container::Single(_) => 1,
            Container::Multiple(items) => items.len(),
        }
    }
    
    fn is_empty(&self) -> bool {
        matches!(self, Container::Empty)
    }
}

impl<L, R> Either<L, R> {
    fn is_left(&self) -> bool {
        matches!(self, Either::Left(_))
    }
    
    fn is_right(&self) -> bool {
        matches!(self, Either::Right(_))
    }
    
    fn left(self) -> Option<L> {
        match self {
            Either::Left(l) => Some(l),
            Either::Right(_) => None,
        }
    }
    
    fn right(self) -> Option<R> {
        match self {
            Either::Left(_) => None,
            Either::Right(r) => Some(r),
        }
    }
}

impl<T> Tree<T> {
    fn new() -> Self {
        Tree::Leaf
    }
    
    fn insert(self, value: T) -> Self
    where
        T: PartialOrd,
    {
        match self {
            Tree::Leaf => Tree::Node {
                value,
                left: Box::new(Tree::Leaf),
                right: Box::new(Tree::Leaf),
            },
            Tree::Node { value: node_value, left, right } => {
                if value <= node_value {
                    Tree::Node {
                        value: node_value,
                        left: Box::new(left.as_ref().clone().insert(value)),
                        right,
                    }
                } else {
                    Tree::Node {
                        value: node_value,
                        left,
                        right: Box::new(right.as_ref().clone().insert(value)),
                    }
                }
            }
        }
    }
}

impl<T> Operation<T>
where
    T: std::ops::Add<Output = T> + std::ops::Sub<Output = T> 
     + std::ops::Mul<Output = T> + std::ops::Div<Output = T> + Copy,
{
    fn execute(&self) -> T {
        match self {
            Operation::Add(a, b) => *a + *b,
            Operation::Subtract(a, b) => *a - *b,
            Operation::Multiply(a, b) => *a * *b,
            Operation::Divide(a, b) => *a / *b,
        }
    }
}

fn main() {
    // Container example
    let mut int_container = Container::new();
    println!("Empty container: {:?}", int_container);
    
    int_container.add(42);
    println!("After adding 42: {:?}", int_container);
    
    int_container.add(24);
    int_container.add(100);
    println!("After adding more: {:?}", int_container);
    println!("Container length: {}", int_container.len());
    
    // String container
    let mut string_container = Container::new();
    string_container.add("Hello".to_string());
    string_container.add("World".to_string());
    println!("String container: {:?}", string_container);
    
    // Either example
    let either_int: Either<i32, String> = Either::Left(42);
    let either_string: Either<i32, String> = Either::Right("Hello".to_string());
    
    println!("Either int is left: {}", either_int.is_left());
    println!("Either string is right: {}", either_string.is_right());
    
    // Operations
    let int_ops = vec![
        Operation::Add(10, 5),
        Operation::Subtract(10, 3),
        Operation::Multiply(4, 7),
        Operation::Divide(20, 4),
    ];
    
    for op in int_ops {
        println!("{:?} = {}", op, op.execute());
    }
    
    let float_ops = vec![
        Operation::Add(3.14, 2.86),
        Operation::Multiply(2.5, 4.0),
    ];
    
    for op in float_ops {
        println!("{:?} = {:.2}", op, op.execute());
    }
    
    // Tree example
    let mut tree = Tree::new();
    let values = vec![5, 3, 7, 1, 9, 4, 6];
    
    for value in values {
        tree = tree.insert(value);
    }
    
    println!("Binary tree: {:?}", tree);
}
```

Generic enums provide type-safe, reusable data structures. Type parameters  
enable the same enum to work with different types while maintaining  
compile-time type checking. Trait bounds on implementations allow  
type-specific behavior while preserving generality.  

## Enums with associated constants

Defining constants within enum implementations to provide variant-specific  
metadata and configuration values.  

```rust
#[derive(Debug, PartialEq)]
enum Planet {
    Mercury,
    Venus,
    Earth,
    Mars,
    Jupiter,
    Saturn,
    Uranus,
    Neptune,
}

#[derive(Debug)]
enum HttpStatus {
    Ok,
    NotFound,
    InternalServerError,
    BadRequest,
    Unauthorized,
}

#[derive(Debug)]
enum LogLevel {
    Error,
    Warn,
    Info,
    Debug,
    Trace,
}

impl Planet {
    const EARTH_MASS: f64 = 5.972e24; // kg
    const EARTH_RADIUS: f64 = 6.371e6; // meters
    
    fn mass(&self) -> f64 {
        match self {
            Planet::Mercury => 3.303e23,
            Planet::Venus => 4.869e24,
            Planet::Earth => Self::EARTH_MASS,
            Planet::Mars => 6.421e23,
            Planet::Jupiter => 1.900e27,
            Planet::Saturn => 5.688e26,
            Planet::Uranus => 8.686e25,
            Planet::Neptune => 1.024e26,
        }
    }
    
    fn radius(&self) -> f64 {
        match self {
            Planet::Mercury => 2.440e6,
            Planet::Venus => 6.052e6,
            Planet::Earth => Self::EARTH_RADIUS,
            Planet::Mars => 3.390e6,
            Planet::Jupiter => 6.991e7,
            Planet::Saturn => 5.823e7,
            Planet::Uranus => 2.556e7,
            Planet::Neptune => 2.464e7,
        }
    }
    
    fn surface_gravity(&self) -> f64 {
        const G: f64 = 6.67430e-11; // gravitational constant
        G * self.mass() / (self.radius() * self.radius())
    }
    
    fn surface_weight(&self, mass: f64) -> f64 {
        mass * self.surface_gravity()
    }
}

impl HttpStatus {
    const OK_CODE: u16 = 200;
    const NOT_FOUND_CODE: u16 = 404;
    const INTERNAL_ERROR_CODE: u16 = 500;
    const BAD_REQUEST_CODE: u16 = 400;
    const UNAUTHORIZED_CODE: u16 = 401;
    
    fn code(&self) -> u16 {
        match self {
            HttpStatus::Ok => Self::OK_CODE,
            HttpStatus::NotFound => Self::NOT_FOUND_CODE,
            HttpStatus::InternalServerError => Self::INTERNAL_ERROR_CODE,
            HttpStatus::BadRequest => Self::BAD_REQUEST_CODE,
            HttpStatus::Unauthorized => Self::UNAUTHORIZED_CODE,
        }
    }
    
    fn message(&self) -> &'static str {
        match self {
            HttpStatus::Ok => "OK",
            HttpStatus::NotFound => "Not Found",
            HttpStatus::InternalServerError => "Internal Server Error",
            HttpStatus::BadRequest => "Bad Request",
            HttpStatus::Unauthorized => "Unauthorized",
        }
    }
    
    fn is_success(&self) -> bool {
        self.code() >= 200 && self.code() < 300
    }
    
    fn is_client_error(&self) -> bool {
        self.code() >= 400 && self.code() < 500
    }
    
    fn is_server_error(&self) -> bool {
        self.code() >= 500 && self.code() < 600
    }
}

impl LogLevel {
    const ERROR_PRIORITY: u8 = 1;
    const WARN_PRIORITY: u8 = 2;
    const INFO_PRIORITY: u8 = 3;
    const DEBUG_PRIORITY: u8 = 4;
    const TRACE_PRIORITY: u8 = 5;
    
    fn priority(&self) -> u8 {
        match self {
            LogLevel::Error => Self::ERROR_PRIORITY,
            LogLevel::Warn => Self::WARN_PRIORITY,
            LogLevel::Info => Self::INFO_PRIORITY,
            LogLevel::Debug => Self::DEBUG_PRIORITY,
            LogLevel::Trace => Self::TRACE_PRIORITY,
        }
    }
    
    fn color_code(&self) -> &'static str {
        match self {
            LogLevel::Error => "\x1b[31m", // Red
            LogLevel::Warn => "\x1b[33m",  // Yellow
            LogLevel::Info => "\x1b[32m",  // Green
            LogLevel::Debug => "\x1b[36m", // Cyan
            LogLevel::Trace => "\x1b[37m", // White
        }
    }
    
    fn should_log(&self, min_level: &LogLevel) -> bool {
        self.priority() <= min_level.priority()
    }
}

fn main() {
    // Planet calculations
    println!("Planet Information:");
    let planets = vec![
        Planet::Mercury, Planet::Venus, Planet::Earth, Planet::Mars,
        Planet::Jupiter, Planet::Saturn, Planet::Uranus, Planet::Neptune,
    ];
    
    let earth_mass = 70.0; // kg
    println!("Weight of {}kg person on different planets:", earth_mass);
    
    for planet in planets {
        let weight = planet.surface_weight(earth_mass);
        println!("  {:?}: {:.1} N (gravity: {:.2} m/s²)", 
                 planet, weight, planet.surface_gravity());
    }
    
    println!("\nEarth constants:");
    println!("  Mass: {:.3e} kg", Planet::EARTH_MASS);
    println!("  Radius: {:.3e} m", Planet::EARTH_RADIUS);
    
    // HTTP status examples
    println!("\nHTTP Status Information:");
    let statuses = vec![
        HttpStatus::Ok,
        HttpStatus::NotFound,
        HttpStatus::InternalServerError,
        HttpStatus::BadRequest,
        HttpStatus::Unauthorized,
    ];
    
    for status in statuses {
        println!("  {:?}: {} - {}", status, status.code(), status.message());
        println!("    Success: {}, Client Error: {}, Server Error: {}", 
                 status.is_success(), status.is_client_error(), status.is_server_error());
    }
    
    // Logging system
    println!("\nLogging System:");
    let current_level = LogLevel::Info;
    let log_messages = vec![
        (LogLevel::Error, "Critical system failure"),
        (LogLevel::Warn, "Deprecated API usage"),
        (LogLevel::Info, "User authentication successful"),
        (LogLevel::Debug, "Database query executed"),
        (LogLevel::Trace, "Function entry point"),
    ];
    
    println!("Current log level: {:?} (priority {})", 
             current_level, current_level.priority());
    
    for (level, message) in log_messages {
        if level.should_log(&current_level) {
            println!("  {}{:?}\x1b[0m: {}", 
                     level.color_code(), level, message);
        }
    }
}
```

Associated constants in enum implementations provide variant-specific  
metadata that remains constant across instances. This pattern is ideal  
for configuration values, lookup tables, and domain-specific constants  
that are logically associated with enum variants.  

## Complex nested enums

Building sophisticated data structures using enums with recursive and  
nested patterns for complex domain modeling.  

```rust
#[derive(Debug, Clone)]
enum JsonValue {
    Null,
    Bool(bool),
    Number(f64),
    String(String),
    Array(Vec<JsonValue>),
    Object(std::collections::HashMap<String, JsonValue>),
}

#[derive(Debug)]
enum Expression {
    Literal(i32),
    Variable(String),
    Binary {
        op: BinaryOp,
        left: Box<Expression>,
        right: Box<Expression>,
    },
    Unary {
        op: UnaryOp,
        operand: Box<Expression>,
    },
    Call {
        function: String,
        args: Vec<Expression>,
    },
}

#[derive(Debug)]
enum BinaryOp {
    Add,
    Subtract,
    Multiply,
    Divide,
    Equal,
    NotEqual,
    LessThan,
    GreaterThan,
}

#[derive(Debug)]
enum UnaryOp {
    Negate,
    Not,
}

#[derive(Debug)]
enum Html {
    Text(String),
    Element {
        tag: String,
        attributes: std::collections::HashMap<String, String>,
        children: Vec<Html>,
    },
    Comment(String),
}

impl JsonValue {
    fn type_name(&self) -> &'static str {
        match self {
            JsonValue::Null => "null",
            JsonValue::Bool(_) => "boolean",
            JsonValue::Number(_) => "number",
            JsonValue::String(_) => "string",
            JsonValue::Array(_) => "array",
            JsonValue::Object(_) => "object",
        }
    }
    
    fn is_truthy(&self) -> bool {
        match self {
            JsonValue::Null => false,
            JsonValue::Bool(b) => *b,
            JsonValue::Number(n) => *n != 0.0,
            JsonValue::String(s) => !s.is_empty(),
            JsonValue::Array(arr) => !arr.is_empty(),
            JsonValue::Object(obj) => !obj.is_empty(),
        }
    }
    
    fn size(&self) -> usize {
        match self {
            JsonValue::Array(arr) => arr.len(),
            JsonValue::Object(obj) => obj.len(),
            JsonValue::String(s) => s.len(),
            _ => 0,
        }
    }
}

impl Expression {
    fn evaluate(&self, variables: &std::collections::HashMap<String, i32>) -> Result<i32, String> {
        match self {
            Expression::Literal(value) => Ok(*value),
            Expression::Variable(name) => {
                variables.get(name)
                    .copied()
                    .ok_or_else(|| format!("Unknown variable: {}", name))
            }
            Expression::Binary { op, left, right } => {
                let left_val = left.evaluate(variables)?;
                let right_val = right.evaluate(variables)?;
                
                match op {
                    BinaryOp::Add => Ok(left_val + right_val),
                    BinaryOp::Subtract => Ok(left_val - right_val),
                    BinaryOp::Multiply => Ok(left_val * right_val),
                    BinaryOp::Divide => {
                        if right_val == 0 {
                            Err("Division by zero".to_string())
                        } else {
                            Ok(left_val / right_val)
                        }
                    }
                    BinaryOp::Equal => Ok((left_val == right_val) as i32),
                    BinaryOp::NotEqual => Ok((left_val != right_val) as i32),
                    BinaryOp::LessThan => Ok((left_val < right_val) as i32),
                    BinaryOp::GreaterThan => Ok((left_val > right_val) as i32),
                }
            }
            Expression::Unary { op, operand } => {
                let value = operand.evaluate(variables)?;
                match op {
                    UnaryOp::Negate => Ok(-value),
                    UnaryOp::Not => Ok((value == 0) as i32),
                }
            }
            Expression::Call { function, args: _ } => {
                Err(format!("Function calls not implemented: {}", function))
            }
        }
    }
}

impl Html {
    fn render(&self, indent: usize) -> String {
        let spaces = " ".repeat(indent);
        match self {
            Html::Text(content) => format!("{}{}", spaces, content),
            Html::Comment(content) => format!("{}<!-- {} -->", spaces, content),
            Html::Element { tag, attributes, children } => {
                let mut result = format!("{}<{}", spaces, tag);
                
                for (key, value) in attributes {
                    result.push_str(&format!(" {}=\"{}\"", key, value));
                }
                
                if children.is_empty() {
                    result.push_str(" />");
                } else {
                    result.push('>');
                    for child in children {
                        result.push('\n');
                        result.push_str(&child.render(indent + 2));
                    }
                    result.push('\n');
                    result.push_str(&format!("{}</{}>", spaces, tag));
                }
                
                result
            }
        }
    }
}

fn main() {
    // JSON example
    let mut obj = std::collections::HashMap::new();
    obj.insert("name".to_string(), JsonValue::String("John".to_string()));
    obj.insert("age".to_string(), JsonValue::Number(30.0));
    obj.insert("active".to_string(), JsonValue::Bool(true));
    
    let json = JsonValue::Object(obj);
    println!("JSON type: {}", json.type_name());
    println!("JSON is truthy: {}", json.is_truthy());
    println!("JSON size: {}", json.size());
    
    // Expression evaluation
    let expr = Expression::Binary {
        op: BinaryOp::Add,
        left: Box::new(Expression::Variable("x".to_string())),
        right: Box::new(Expression::Binary {
            op: BinaryOp::Multiply,
            left: Box::new(Expression::Literal(2)),
            right: Box::new(Expression::Variable("y".to_string())),
        }),
    };
    
    let mut vars = std::collections::HashMap::new();
    vars.insert("x".to_string(), 10);
    vars.insert("y".to_string(), 5);
    
    match expr.evaluate(&vars) {
        Ok(result) => println!("Expression result: {}", result),
        Err(e) => println!("Evaluation error: {}", e),
    }
    
    // HTML rendering
    let mut attrs = std::collections::HashMap::new();
    attrs.insert("id".to_string(), "main".to_string());
    attrs.insert("class".to_string(), "container".to_string());
    
    let html = Html::Element {
        tag: "div".to_string(),
        attributes: attrs,
        children: vec![
            Html::Comment("This is a comment".to_string()),
            Html::Element {
                tag: "h1".to_string(),
                attributes: std::collections::HashMap::new(),
                children: vec![Html::Text("Welcome".to_string())],
            },
            Html::Element {
                tag: "p".to_string(),
                attributes: std::collections::HashMap::new(),
                children: vec![Html::Text("Hello, world!".to_string())],
            },
        ],
    };
    
    println!("HTML:\n{}", html.render(0));
}
```

Complex nested enums enable modeling sophisticated data structures like  
JSON, HTML, or abstract syntax trees. Recursive patterns with Box allow  
arbitrary nesting depth, while pattern matching provides safe traversal  
and manipulation of complex hierarchical data.  

## State machine with enums

Implementing state machines using enums to model system states and  
transitions with compile-time safety.  

```rust
#[derive(Debug, Clone, PartialEq)]
enum ConnectionState {
    Disconnected,
    Connecting { attempt: u32 },
    Connected { session_id: String },
    Authenticated { user_id: u32, session_id: String },
    Error { message: String, recoverable: bool },
}

#[derive(Debug)]
enum ConnectionEvent {
    Connect,
    ConnectionEstablished(String),
    Authenticate(u32),
    Disconnect,
    Error(String, bool),
    Retry,
}

#[derive(Debug, Clone)]
enum OrderState {
    Draft { items: Vec<String> },
    Submitted { order_id: String, items: Vec<String> },
    Processing { order_id: String, estimated_time: u32 },
    Shipped { order_id: String, tracking_number: String },
    Delivered { order_id: String, delivery_time: String },
    Cancelled { order_id: String, reason: String },
}

#[derive(Debug)]
enum OrderEvent {
    AddItem(String),
    Submit,
    StartProcessing(u32),
    Ship(String),
    Deliver(String),
    Cancel(String),
}

impl ConnectionState {
    fn handle_event(self, event: ConnectionEvent) -> Result<ConnectionState, String> {
        match (self, event) {
            (ConnectionState::Disconnected, ConnectionEvent::Connect) => {
                Ok(ConnectionState::Connecting { attempt: 1 })
            }
            (ConnectionState::Connecting { attempt }, ConnectionEvent::ConnectionEstablished(session_id)) => {
                Ok(ConnectionState::Connected { session_id })
            }
            (ConnectionState::Connecting { attempt }, ConnectionEvent::Error(msg, recoverable)) => {
                if recoverable && attempt < 3 {
                    Ok(ConnectionState::Connecting { attempt: attempt + 1 })
                } else {
                    Ok(ConnectionState::Error { message: msg, recoverable })
                }
            }
            (ConnectionState::Connected { session_id }, ConnectionEvent::Authenticate(user_id)) => {
                Ok(ConnectionState::Authenticated { user_id, session_id })
            }
            (ConnectionState::Authenticated { .. }, ConnectionEvent::Disconnect) |
            (ConnectionState::Connected { .. }, ConnectionEvent::Disconnect) |
            (ConnectionState::Connecting { .. }, ConnectionEvent::Disconnect) => {
                Ok(ConnectionState::Disconnected)
            }
            (ConnectionState::Error { recoverable: true, .. }, ConnectionEvent::Retry) => {
                Ok(ConnectionState::Connecting { attempt: 1 })
            }
            (state, event) => {
                Err(format!("Invalid transition: {:?} + {:?}", state, event))
            }
        }
    }
    
    fn can_send_data(&self) -> bool {
        matches!(self, ConnectionState::Authenticated { .. })
    }
    
    fn is_connected(&self) -> bool {
        matches!(self, 
            ConnectionState::Connected { .. } | 
            ConnectionState::Authenticated { .. }
        )
    }
}

impl OrderState {
    fn handle_event(self, event: OrderEvent) -> Result<OrderState, String> {
        match (self, event) {
            (OrderState::Draft { mut items }, OrderEvent::AddItem(item)) => {
                items.push(item);
                Ok(OrderState::Draft { items })
            }
            (OrderState::Draft { items }, OrderEvent::Submit) => {
                if items.is_empty() {
                    return Err("Cannot submit empty order".to_string());
                }
                let order_id = format!("ORD-{}", rand::random::<u32>());
                Ok(OrderState::Submitted { order_id, items })
            }
            (OrderState::Submitted { order_id, items }, OrderEvent::StartProcessing(estimated_time)) => {
                Ok(OrderState::Processing { order_id, estimated_time })
            }
            (OrderState::Processing { order_id, .. }, OrderEvent::Ship(tracking_number)) => {
                Ok(OrderState::Shipped { order_id, tracking_number })
            }
            (OrderState::Shipped { order_id, .. }, OrderEvent::Deliver(delivery_time)) => {
                Ok(OrderState::Delivered { order_id, delivery_time })
            }
            (state, OrderEvent::Cancel(reason)) => {
                let order_id = match &state {
                    OrderState::Submitted { order_id, .. } |
                    OrderState::Processing { order_id, .. } |
                    OrderState::Shipped { order_id, .. } => order_id.clone(),
                    _ => return Err("Cannot cancel order in this state".to_string()),
                };
                Ok(OrderState::Cancelled { order_id, reason })
            }
            (state, event) => {
                Err(format!("Invalid transition: {:?} + {:?}", state, event))
            }
        }
    }
    
    fn is_final(&self) -> bool {
        matches!(self, 
            OrderState::Delivered { .. } | 
            OrderState::Cancelled { .. }
        )
    }
    
    fn order_id(&self) -> Option<&str> {
        match self {
            OrderState::Draft { .. } => None,
            OrderState::Submitted { order_id, .. } |
            OrderState::Processing { order_id, .. } |
            OrderState::Shipped { order_id, .. } |
            OrderState::Delivered { order_id, .. } |
            OrderState::Cancelled { order_id, .. } => Some(order_id),
        }
    }
}

fn main() {
    // Connection state machine
    println!("Connection State Machine:");
    let mut conn_state = ConnectionState::Disconnected;
    
    let events = vec![
        ConnectionEvent::Connect,
        ConnectionEvent::ConnectionEstablished("sess123".to_string()),
        ConnectionEvent::Authenticate(42),
        ConnectionEvent::Disconnect,
    ];
    
    for event in events {
        println!("Current state: {:?}", conn_state);
        println!("Processing event: {:?}", event);
        
        match conn_state.handle_event(event) {
            Ok(new_state) => {
                conn_state = new_state;
                println!("New state: {:?}", conn_state);
                println!("Can send data: {}", conn_state.can_send_data());
            }
            Err(e) => println!("Error: {}", e),
        }
        println!();
    }
    
    // Order state machine
    println!("Order State Machine:");
    let mut order_state = OrderState::Draft { items: vec![] };
    
    let order_events = vec![
        OrderEvent::AddItem("Laptop".to_string()),
        OrderEvent::AddItem("Mouse".to_string()),
        OrderEvent::Submit,
        OrderEvent::StartProcessing(120),
        OrderEvent::Ship("TRACK123".to_string()),
        OrderEvent::Deliver("2023-12-01 10:30".to_string()),
    ];
    
    for event in order_events {
        println!("Current state: {:?}", order_state);
        println!("Processing event: {:?}", event);
        
        match order_state.handle_event(event) {
            Ok(new_state) => {
                order_state = new_state;
                println!("New state: {:?}", order_state);
                if let Some(id) = order_state.order_id() {
                    println!("Order ID: {}", id);
                }
                println!("Is final: {}", order_state.is_final());
            }
            Err(e) => println!("Error: {}", e),
        }
        println!();
    }
}
```

State machine implementations with enums provide compile-time guarantees  
about valid state transitions. Each state carries its own data, and the  
type system prevents invalid transitions. This pattern is excellent for  
modeling protocols, user workflows, and system lifecycles.  

## Error handling with custom enums

Creating comprehensive error handling systems using enums to represent  
different error categories and contexts.  

```rust
use std::fs::File;
use std::io::{self, Read};

#[derive(Debug)]
enum DatabaseError {
    ConnectionFailed { host: String, port: u16 },
    QueryFailed { query: String, reason: String },
    TransactionAborted { transaction_id: String },
    Timeout { operation: String, duration_ms: u64 },
    PermissionDenied { user: String, operation: String },
}

#[derive(Debug)]
enum ValidationError {
    Required { field: String },
    InvalidFormat { field: String, expected: String },
    OutOfRange { field: String, min: i32, max: i32, actual: i32 },
    TooLong { field: String, max_length: usize, actual_length: usize },
    TooShort { field: String, min_length: usize, actual_length: usize },
}

#[derive(Debug)]
enum NetworkError {
    Dns { domain: String },
    Connection { address: String },
    Timeout { url: String, timeout_ms: u64 },
    HttpStatus { url: String, status: u16 },
    InvalidResponse { url: String, reason: String },
}

#[derive(Debug)]
enum ApplicationError {
    Database(DatabaseError),
    Validation(ValidationError),
    Network(NetworkError),
    Io(io::Error),
    Config { message: String },
    Internal { message: String, code: String },
}

impl std::fmt::Display for DatabaseError {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self {
            DatabaseError::ConnectionFailed { host, port } => {
                write!(f, "Failed to connect to database at {}:{}", host, port)
            }
            DatabaseError::QueryFailed { query, reason } => {
                write!(f, "Query failed: {} (reason: {})", query, reason)
            }
            DatabaseError::TransactionAborted { transaction_id } => {
                write!(f, "Transaction {} was aborted", transaction_id)
            }
            DatabaseError::Timeout { operation, duration_ms } => {
                write!(f, "Operation '{}' timed out after {}ms", operation, duration_ms)
            }
            DatabaseError::PermissionDenied { user, operation } => {
                write!(f, "User '{}' denied permission for operation '{}'", user, operation)
            }
        }
    }
}

impl std::fmt::Display for ValidationError {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self {
            ValidationError::Required { field } => {
                write!(f, "Field '{}' is required", field)
            }
            ValidationError::InvalidFormat { field, expected } => {
                write!(f, "Field '{}' has invalid format, expected {}", field, expected)
            }
            ValidationError::OutOfRange { field, min, max, actual } => {
                write!(f, "Field '{}' value {} is out of range [{}, {}]", field, actual, min, max)
            }
            ValidationError::TooLong { field, max_length, actual_length } => {
                write!(f, "Field '{}' is too long: {} characters (max: {})", 
                       field, actual_length, max_length)
            }
            ValidationError::TooShort { field, min_length, actual_length } => {
                write!(f, "Field '{}' is too short: {} characters (min: {})", 
                       field, actual_length, min_length)
            }
        }
    }
}

impl std::fmt::Display for NetworkError {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self {
            NetworkError::Dns { domain } => {
                write!(f, "DNS resolution failed for domain: {}", domain)
            }
            NetworkError::Connection { address } => {
                write!(f, "Failed to connect to: {}", address)
            }
            NetworkError::Timeout { url, timeout_ms } => {
                write!(f, "Request to {} timed out after {}ms", url, timeout_ms)
            }
            NetworkError::HttpStatus { url, status } => {
                write!(f, "HTTP request to {} failed with status: {}", url, status)
            }
            NetworkError::InvalidResponse { url, reason } => {
                write!(f, "Invalid response from {}: {}", url, reason)
            }
        }
    }
}

impl std::fmt::Display for ApplicationError {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self {
            ApplicationError::Database(e) => write!(f, "Database error: {}", e),
            ApplicationError::Validation(e) => write!(f, "Validation error: {}", e),
            ApplicationError::Network(e) => write!(f, "Network error: {}", e),
            ApplicationError::Io(e) => write!(f, "IO error: {}", e),
            ApplicationError::Config { message } => write!(f, "Configuration error: {}", message),
            ApplicationError::Internal { message, code } => {
                write!(f, "Internal error [{}]: {}", code, message)
            }
        }
    }
}

impl From<io::Error> for ApplicationError {
    fn from(error: io::Error) -> Self {
        ApplicationError::Io(error)
    }
}

impl From<DatabaseError> for ApplicationError {
    fn from(error: DatabaseError) -> Self {
        ApplicationError::Database(error)
    }
}

impl From<ValidationError> for ApplicationError {
    fn from(error: ValidationError) -> Self {
        ApplicationError::Validation(error)
    }
}

impl From<NetworkError> for ApplicationError {
    fn from(error: NetworkError) -> Self {
        ApplicationError::Network(error)
    }
}

impl ApplicationError {
    fn is_recoverable(&self) -> bool {
        match self {
            ApplicationError::Database(DatabaseError::Timeout { .. }) => true,
            ApplicationError::Database(DatabaseError::ConnectionFailed { .. }) => true,
            ApplicationError::Network(NetworkError::Timeout { .. }) => true,
            ApplicationError::Network(NetworkError::Connection { .. }) => true,
            ApplicationError::Validation(_) => false,
            ApplicationError::Config { .. } => false,
            ApplicationError::Internal { .. } => false,
            ApplicationError::Io(_) => false,
            _ => false,
        }
    }
    
    fn error_code(&self) -> &'static str {
        match self {
            ApplicationError::Database(_) => "DB_ERROR",
            ApplicationError::Validation(_) => "VALIDATION_ERROR", 
            ApplicationError::Network(_) => "NETWORK_ERROR",
            ApplicationError::Io(_) => "IO_ERROR",
            ApplicationError::Config { .. } => "CONFIG_ERROR",
            ApplicationError::Internal { .. } => "INTERNAL_ERROR",
        }
    }
    
    fn severity(&self) -> ErrorSeverity {
        match self {
            ApplicationError::Internal { .. } => ErrorSeverity::Critical,
            ApplicationError::Database(_) => ErrorSeverity::High,
            ApplicationError::Network(_) => ErrorSeverity::Medium,
            ApplicationError::Validation(_) => ErrorSeverity::Low,
            ApplicationError::Config { .. } => ErrorSeverity::High,
            ApplicationError::Io(_) => ErrorSeverity::Medium,
        }
    }
}

#[derive(Debug)]
enum ErrorSeverity {
    Low,
    Medium,
    High,
    Critical,
}

fn validate_user_input(name: &str, age: i32, email: &str) -> Result<(), ValidationError> {
    if name.is_empty() {
        return Err(ValidationError::Required { field: "name".to_string() });
    }
    
    if name.len() > 50 {
        return Err(ValidationError::TooLong { 
            field: "name".to_string(),
            max_length: 50,
            actual_length: name.len()
        });
    }
    
    if age < 0 || age > 150 {
        return Err(ValidationError::OutOfRange {
            field: "age".to_string(),
            min: 0,
            max: 150,
            actual: age
        });
    }
    
    if !email.contains('@') {
        return Err(ValidationError::InvalidFormat {
            field: "email".to_string(),
            expected: "valid email address".to_string()
        });
    }
    
    Ok(())
}

fn simulate_database_operation() -> Result<String, DatabaseError> {
    // Simulate various database errors
    use std::collections::hash_map::DefaultHasher;
    use std::hash::{Hash, Hasher};
    
    let mut hasher = DefaultHasher::new();
    std::time::SystemTime::now().hash(&mut hasher);
    let random = hasher.finish() % 4;
    
    match random {
        0 => Ok("Operation successful".to_string()),
        1 => Err(DatabaseError::ConnectionFailed { 
            host: "localhost".to_string(), 
            port: 5432 
        }),
        2 => Err(DatabaseError::QueryFailed { 
            query: "SELECT * FROM users".to_string(),
            reason: "Table does not exist".to_string()
        }),
        3 => Err(DatabaseError::Timeout { 
            operation: "user_lookup".to_string(),
            duration_ms: 5000
        }),
        _ => unreachable!(),
    }
}

fn main() {
    // Validation errors
    println!("Validation Examples:");
    let test_cases = vec![
        ("", 25, "john@example.com"),
        ("John Doe", -5, "john@example.com"),
        ("John Doe", 25, "invalid-email"),
        ("John Doe", 25, "john@example.com"),
    ];
    
    for (name, age, email) in test_cases {
        match validate_user_input(name, age, email) {
            Ok(()) => println!("✓ Valid input: {} ({}), {}", name, age, email),
            Err(e) => println!("✗ {}", e),
        }
    }
    
    // Database operation examples
    println!("\nDatabase Operation Examples:");
    for i in 0..5 {
        match simulate_database_operation() {
            Ok(result) => println!("✓ Operation {}: {}", i + 1, result),
            Err(e) => {
                let app_error = ApplicationError::from(e);
                println!("✗ Operation {}: {} [{}] (Severity: {:?}, Recoverable: {})",
                         i + 1, app_error, app_error.error_code(), 
                         app_error.severity(), app_error.is_recoverable());
            }
        }
    }
    
    // Error conversion and handling
    println!("\nError Conversion Examples:");
    let errors: Vec<ApplicationError> = vec![
        ValidationError::Required { field: "username".to_string() }.into(),
        DatabaseError::ConnectionFailed { 
            host: "db.example.com".to_string(), 
            port: 3306 
        }.into(),
        NetworkError::Timeout { 
            url: "https://api.example.com".to_string(),
            timeout_ms: 10000
        }.into(),
        ApplicationError::Config { 
            message: "Missing API key".to_string() 
        },
    ];
    
    for error in errors {
        println!("Error: {} [{}] (Severity: {:?})", 
                 error, error.error_code(), error.severity());
    }
}
```

Comprehensive error handling with enums provides structured, type-safe  
error management. Different error types carry relevant context data,  
implement Display for user-friendly messages, and support conversion  
between error types. This enables robust error handling strategies.  

## Enum iterators and conversions

Implementing iterator patterns and type conversions for enums to enable  
functional programming and interoperability.  

```rust
use std::str::FromStr;

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
enum Color {
    Red,
    Green,
    Blue,
    Yellow,
    Purple,
    Orange,
}

#[derive(Debug, Clone, PartialEq)]
enum Priority {
    Low,
    Medium,
    High,
    Critical,
}

#[derive(Debug)]
enum TaskStatus {
    Todo,
    InProgress { assignee: String },
    Review { reviewer: String },
    Done,
    Cancelled { reason: String },
}

impl Color {
    fn all() -> impl Iterator<Item = Color> {
        [Color::Red, Color::Green, Color::Blue, Color::Yellow, Color::Purple, Color::Orange]
            .iter()
            .copied()
    }
    
    fn rgb(&self) -> (u8, u8, u8) {
        match self {
            Color::Red => (255, 0, 0),
            Color::Green => (0, 255, 0),
            Color::Blue => (0, 0, 255),
            Color::Yellow => (255, 255, 0),
            Color::Purple => (128, 0, 128),
            Color::Orange => (255, 165, 0),
        }
    }
    
    fn hex(&self) -> String {
        let (r, g, b) = self.rgb();
        format!("#{:02x}{:02x}{:02x}", r, g, b)
    }
    
    fn is_primary(&self) -> bool {
        matches!(self, Color::Red | Color::Green | Color::Blue)
    }
}

impl FromStr for Color {
    type Err = String;
    
    fn from_str(s: &str) -> Result<Self, Self::Err> {
        match s.to_lowercase().as_str() {
            "red" => Ok(Color::Red),
            "green" => Ok(Color::Green),
            "blue" => Ok(Color::Blue),
            "yellow" => Ok(Color::Yellow),
            "purple" => Ok(Color::Purple),
            "orange" => Ok(Color::Orange),
            _ => Err(format!("Unknown color: {}", s)),
        }
    }
}

impl std::fmt::Display for Color {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        let name = match self {
            Color::Red => "Red",
            Color::Green => "Green",
            Color::Blue => "Blue",
            Color::Yellow => "Yellow",
            Color::Purple => "Purple",
            Color::Orange => "Orange",
        };
        write!(f, "{}", name)
    }
}

impl Priority {
    fn all() -> impl Iterator<Item = Priority> {
        [Priority::Low, Priority::Medium, Priority::High, Priority::Critical]
            .iter()
            .cloned()
    }
    
    fn level(&self) -> u8 {
        match self {
            Priority::Low => 1,
            Priority::Medium => 2,
            Priority::High => 3,
            Priority::Critical => 4,
        }
    }
    
    fn color(&self) -> Color {
        match self {
            Priority::Low => Color::Green,
            Priority::Medium => Color::Yellow,
            Priority::High => Color::Orange,
            Priority::Critical => Color::Red,
        }
    }
}

impl From<u8> for Priority {
    fn from(level: u8) -> Self {
        match level {
            1 => Priority::Low,
            2 => Priority::Medium,
            3 => Priority::High,
            4.. => Priority::Critical,
            0 => Priority::Low,
        }
    }
}

impl From<Priority> for u8 {
    fn from(priority: Priority) -> Self {
        priority.level()
    }
}

impl TaskStatus {
    fn all_simple() -> impl Iterator<Item = TaskStatus> {
        [
            TaskStatus::Todo,
            TaskStatus::InProgress { assignee: "TBD".to_string() },
            TaskStatus::Review { reviewer: "TBD".to_string() },
            TaskStatus::Done,
            TaskStatus::Cancelled { reason: "TBD".to_string() },
        ]
        .into_iter()
    }
    
    fn is_active(&self) -> bool {
        !matches!(self, TaskStatus::Done | TaskStatus::Cancelled { .. })
    }
    
    fn requires_assignee(&self) -> bool {
        matches!(self, TaskStatus::InProgress { .. })
    }
    
    fn status_name(&self) -> &'static str {
        match self {
            TaskStatus::Todo => "Todo",
            TaskStatus::InProgress { .. } => "In Progress",
            TaskStatus::Review { .. } => "In Review",
            TaskStatus::Done => "Done",
            TaskStatus::Cancelled { .. } => "Cancelled",
        }
    }
}

// Custom iterator for cycling through enum variants
struct ColorCycle {
    colors: Vec<Color>,
    current: usize,
}

impl ColorCycle {
    fn new() -> Self {
        ColorCycle {
            colors: Color::all().collect(),
            current: 0,
        }
    }
}

impl Iterator for ColorCycle {
    type Item = Color;
    
    fn next(&mut self) -> Option<Self::Item> {
        let color = self.colors[self.current];
        self.current = (self.current + 1) % self.colors.len();
        Some(color)
    }
}

// Filtering iterator for priorities
struct PriorityFilter<I> {
    iter: I,
    min_level: u8,
}

impl<I> PriorityFilter<I> {
    fn new(iter: I, min_priority: Priority) -> Self {
        PriorityFilter {
            iter,
            min_level: min_priority.level(),
        }
    }
}

impl<I> Iterator for PriorityFilter<I>
where
    I: Iterator<Item = Priority>,
{
    type Item = Priority;
    
    fn next(&mut self) -> Option<Self::Item> {
        self.iter.find(|priority| priority.level() >= self.min_level)
    }
}

trait PriorityIteratorExt: Iterator<Item = Priority> {
    fn filter_by_min_priority(self, min_priority: Priority) -> PriorityFilter<Self>
    where
        Self: Sized,
    {
        PriorityFilter::new(self, min_priority)
    }
}

impl<I: Iterator<Item = Priority>> PriorityIteratorExt for I {}

fn main() {
    // Color iteration and conversion
    println!("All Colors:");
    for color in Color::all() {
        println!("  {}: {} (RGB: {:?}) Primary: {}", 
                 color, color.hex(), color.rgb(), color.is_primary());
    }
    
    // String to color conversion
    println!("\nColor Parsing:");
    let color_strings = ["red", "GREEN", "Blue", "yellow", "invalid"];
    for color_str in color_strings {
        match Color::from_str(color_str) {
            Ok(color) => println!("  '{}' -> {}", color_str, color),
            Err(e) => println!("  '{}' -> Error: {}", color_str, e),
        }
    }
    
    // Priority level conversion
    println!("\nPriority Conversions:");
    for level in 0..=5 {
        let priority = Priority::from(level);
        let back_to_level: u8 = priority.into();
        println!("  {} -> {:?} -> {} (Color: {})", 
                 level, priority, back_to_level, priority.color());
    }
    
    // Custom color cycle iterator
    println!("\nColor Cycle (first 10):");
    let mut color_cycle = ColorCycle::new();
    for i in 0..10 {
        if let Some(color) = color_cycle.next() {
            println!("  {}: {}", i + 1, color);
        }
    }
    
    // Priority filtering
    println!("\nHigh Priority Tasks:");
    let all_priorities = vec![Priority::Low, Priority::High, Priority::Medium, 
                             Priority::Critical, Priority::Low, Priority::High];
    
    let high_priorities: Vec<_> = all_priorities
        .into_iter()
        .filter_by_min_priority(Priority::High)
        .collect();
    
    for priority in high_priorities {
        println!("  {:?} (Level: {})", priority, priority.level());
    }
    
    // Task status analysis
    println!("\nTask Status Analysis:");
    let statuses = vec![
        TaskStatus::Todo,
        TaskStatus::InProgress { assignee: "Alice".to_string() },
        TaskStatus::Review { reviewer: "Bob".to_string() },
        TaskStatus::Done,
        TaskStatus::Cancelled { reason: "Deprecated".to_string() },
    ];
    
    let active_count = statuses.iter().filter(|s| s.is_active()).count();
    let requiring_assignee: Vec<_> = statuses.iter()
        .filter(|s| s.requires_assignee())
        .collect();
    
    println!("  Total statuses: {}", statuses.len());
    println!("  Active tasks: {}", active_count);
    println!("  Tasks requiring assignee: {}", requiring_assignee.len());
    
    for status in statuses {
        println!("  {} - Active: {}", status.status_name(), status.is_active());
    }
    
    // Functional composition example
    println!("\nFunctional Composition:");
    let color_names: Vec<String> = Color::all()
        .filter(|c| c.is_primary())
        .map(|c| c.to_string().to_uppercase())
        .collect();
    
    println!("  Primary colors: {:?}", color_names);
    
    let priority_levels: Vec<u8> = Priority::all()
        .map(|p| p.level())
        .filter(|&level| level > 2)
        .collect();
    
    println!("  High priority levels: {:?}", priority_levels);
}
```

Iterator implementations for enums enable functional programming patterns  
and seamless integration with Rust's iterator ecosystem. Custom iterators,  
filtering, and conversion traits provide powerful composition capabilities  
while maintaining type safety and zero-cost abstractions.  

## Advanced enum patterns

Sophisticated enum usage patterns including discriminant control,  
memory optimization, and advanced type system features.  

```rust
use std::mem;

// Enum with explicit discriminants
#[repr(u8)]
#[derive(Debug, Clone, Copy, PartialEq)]
enum StatusCode {
    Ok = 200,
    NotFound = 404,
    InternalError = 500,
    BadRequest = 400,
    Unauthorized = 401,
}

// Memory-optimized enum using NonNull
#[derive(Debug)]
enum OptimizedOption<T> {
    None,
    Some(std::ptr::NonNull<T>),
}

// Enum with phantom types for compile-time state tracking
#[derive(Debug)]
enum Connection<S> {
    State(S),
}

#[derive(Debug)]
struct Disconnected;
#[derive(Debug)]
struct Connected { session_id: String }
#[derive(Debug)]
struct Authenticated { user_id: u32, session_id: String }

// Enum with associated types through traits
trait Command {
    type Output;
    type Error;
    
    fn execute(self) -> Result<Self::Output, Self::Error>;
}

#[derive(Debug)]
enum DatabaseCommand {
    Query { sql: String },
    Insert { table: String, data: Vec<(String, String)> },
    Update { table: String, id: u32, data: Vec<(String, String)> },
    Delete { table: String, id: u32 },
}

#[derive(Debug)]
enum QueryResult {
    Rows(Vec<std::collections::HashMap<String, String>>),
    RowsAffected(u32),
    Empty,
}

#[derive(Debug)]
enum DatabaseError {
    SqlError(String),
    ConnectionError,
    PermissionDenied,
}

// Const generic enum
#[derive(Debug)]
enum FixedArray<T, const N: usize> {
    Empty,
    Partial { data: [Option<T>; N], count: usize },
    Full([T; N]),
}

// Enum with custom drop behavior
#[derive(Debug)]
enum Resource {
    File { handle: std::fs::File, path: String },
    Network { connection: String, port: u16 },
    Memory { ptr: *mut u8, size: usize },
}

impl StatusCode {
    fn is_success(self) -> bool {
        (self as u16) >= 200 && (self as u16) < 300
    }
    
    fn is_client_error(self) -> bool {
        (self as u16) >= 400 && (self as u16) < 500
    }
    
    fn is_server_error(self) -> bool {
        (self as u16) >= 500 && (self as u16) < 600
    }
    
    fn from_u16(code: u16) -> Option<Self> {
        match code {
            200 => Some(StatusCode::Ok),
            400 => Some(StatusCode::BadRequest),
            401 => Some(StatusCode::Unauthorized),
            404 => Some(StatusCode::NotFound),
            500 => Some(StatusCode::InternalError),
            _ => None,
        }
    }
}

impl<T> OptimizedOption<T> {
    fn new() -> Self {
        OptimizedOption::None
    }
    
    fn some(value: T) -> Self {
        let boxed = Box::new(value);
        let ptr = Box::into_raw(boxed);
        OptimizedOption::Some(unsafe { std::ptr::NonNull::new_unchecked(ptr) })
    }
    
    fn is_some(&self) -> bool {
        matches!(self, OptimizedOption::Some(_))
    }
    
    fn is_none(&self) -> bool {
        matches!(self, OptimizedOption::None)
    }
    
    fn take(&mut self) -> Self {
        mem::replace(self, OptimizedOption::None)
    }
    
    fn as_ref(&self) -> OptimizedOption<&T> {
        match self {
            OptimizedOption::None => OptimizedOption::None,
            OptimizedOption::Some(ptr) => {
                OptimizedOption::Some(unsafe { std::ptr::NonNull::new_unchecked(ptr.as_ref()) })
            }
        }
    }
}

impl<T> Drop for OptimizedOption<T> {
    fn drop(&mut self) {
        if let OptimizedOption::Some(ptr) = self {
            unsafe {
                let _ = Box::from_raw(ptr.as_ptr());
            }
        }
    }
}

impl Connection<Disconnected> {
    fn new() -> Self {
        Connection::State(Disconnected)
    }
    
    fn connect(self, session_id: String) -> Connection<Connected> {
        Connection::State(Connected { session_id })
    }
}

impl Connection<Connected> {
    fn authenticate(self, user_id: u32) -> Connection<Authenticated> {
        if let Connection::State(Connected { session_id }) = self {
            Connection::State(Authenticated { user_id, session_id })
        } else {
            unreachable!()
        }
    }
    
    fn disconnect(self) -> Connection<Disconnected> {
        Connection::State(Disconnected)
    }
}

impl Connection<Authenticated> {
    fn send_data(&self, data: &str) -> Result<(), String> {
        if let Connection::State(Authenticated { user_id, session_id }) = self {
            println!("Sending data for user {}, session {}: {}", user_id, session_id, data);
            Ok(())
        } else {
            Err("Not authenticated".to_string())
        }
    }
    
    fn disconnect(self) -> Connection<Disconnected> {
        Connection::State(Disconnected)
    }
}

impl Command for DatabaseCommand {
    type Output = QueryResult;
    type Error = DatabaseError;
    
    fn execute(self) -> Result<Self::Output, Self::Error> {
        match self {
            DatabaseCommand::Query { sql } => {
                if sql.to_lowercase().starts_with("select") {
                    // Simulate query execution
                    let mut rows = Vec::new();
                    let mut row = std::collections::HashMap::new();
                    row.insert("id".to_string(), "1".to_string());
                    row.insert("name".to_string(), "John".to_string());
                    rows.push(row);
                    Ok(QueryResult::Rows(rows))
                } else {
                    Err(DatabaseError::SqlError("Invalid SELECT query".to_string()))
                }
            }
            DatabaseCommand::Insert { table: _, data: _ } => {
                Ok(QueryResult::RowsAffected(1))
            }
            DatabaseCommand::Update { table: _, id: _, data: _ } => {
                Ok(QueryResult::RowsAffected(1))
            }
            DatabaseCommand::Delete { table: _, id: _ } => {
                Ok(QueryResult::RowsAffected(1))
            }
        }
    }
}

impl<T, const N: usize> FixedArray<T, N> {
    fn new() -> Self {
        FixedArray::Empty
    }
    
    fn push(&mut self, item: T) -> Result<(), T> {
        match self {
            FixedArray::Empty => {
                let mut data = [None; N];
                data[0] = Some(item);
                *self = FixedArray::Partial { data, count: 1 };
                Ok(())
            }
            FixedArray::Partial { data, count } => {
                if *count < N {
                    data[*count] = Some(item);
                    *count += 1;
                    
                    if *count == N {
                        // Convert to Full variant
                        let mut full_data = unsafe { mem::zeroed::<[T; N]>() };
                        for (i, slot) in data.iter_mut().enumerate() {
                            if let Some(value) = slot.take() {
                                unsafe { std::ptr::write(&mut full_data[i], value) };
                            }
                        }
                        *self = FixedArray::Full(full_data);
                    }
                    Ok(())
                } else {
                    Err(item)
                }
            }
            FixedArray::Full(_) => Err(item),
        }
    }
    
    fn len(&self) -> usize {
        match self {
            FixedArray::Empty => 0,
            FixedArray::Partial { count, .. } => *count,
            FixedArray::Full(_) => N,
        }
    }
    
    fn is_full(&self) -> bool {
        matches!(self, FixedArray::Full(_))
    }
}

impl Drop for Resource {
    fn drop(&mut self) {
        match self {
            Resource::File { path, .. } => {
                println!("Closing file: {}", path);
            }
            Resource::Network { connection, port } => {
                println!("Closing network connection: {}:{}", connection, port);
            }
            Resource::Memory { ptr, size } => {
                println!("Freeing memory: {} bytes at {:?}", size, ptr);
                // In real code, you would properly deallocate memory here
            }
        }
    }
}

fn main() {
    // Status code discriminants
    println!("Status Code Examples:");
    let codes = [200, 404, 500, 999];
    for code in codes {
        match StatusCode::from_u16(code) {
            Some(status) => {
                println!("  {}: {:?} (Success: {}, Client Error: {}, Server Error: {})",
                         code, status, status.is_success(), 
                         status.is_client_error(), status.is_server_error());
            }
            None => println!("  {}: Unknown status code", code),
        }
    }
    
    // Memory size comparison
    println!("\nMemory Usage:");
    println!("  Option<Box<i32>>: {} bytes", mem::size_of::<Option<Box<i32>>>());
    println!("  OptimizedOption<i32>: {} bytes", mem::size_of::<OptimizedOption<i32>>());
    println!("  Standard Result<i32, String>: {} bytes", mem::size_of::<Result<i32, String>>());
    
    // Type-state pattern
    println!("\nType-State Connection:");
    let conn = Connection::new();
    println!("  Created: {:?}", conn);
    
    let conn = conn.connect("session123".to_string());
    println!("  Connected: {:?}", conn);
    
    let conn = conn.authenticate(42);
    println!("  Authenticated: {:?}", conn);
    
    if let Err(e) = conn.send_data("Hello, World!") {
        println!("  Error: {}", e);
    }
    
    // Database command pattern
    println!("\nDatabase Commands:");
    let commands = vec![
        DatabaseCommand::Query { sql: "SELECT * FROM users".to_string() },
        DatabaseCommand::Insert { 
            table: "users".to_string(), 
            data: vec![("name".to_string(), "Alice".to_string())]
        },
        DatabaseCommand::Query { sql: "DROP TABLE users".to_string() },
    ];
    
    for cmd in commands {
        println!("  Executing: {:?}", cmd);
        match cmd.execute() {
            Ok(result) => println!("    Result: {:?}", result),
            Err(error) => println!("    Error: {:?}", error),
        }
    }
    
    // Fixed array example
    println!("\nFixed Array Example:");
    let mut arr: FixedArray<i32, 3> = FixedArray::new();
    println!("  Initial: len = {}, full = {}", arr.len(), arr.is_full());
    
    for i in 1..=4 {
        match arr.push(i) {
            Ok(()) => println!("  Pushed {}: len = {}, full = {}", i, arr.len(), arr.is_full()),
            Err(value) => println!("  Failed to push {}: array is full", value),
        }
    }
    
    // Resource management
    println!("\nResource Management:");
    {
        let _file_resource = Resource::File {
            handle: std::fs::File::open("Cargo.toml").unwrap_or_else(|_| {
                std::fs::File::create("/tmp/dummy.txt").expect("Could not create dummy file")
            }),
            path: "/tmp/test.txt".to_string(),
        };
        
        let _network_resource = Resource::Network {
            connection: "localhost".to_string(),
            port: 8080,
        };
        
        println!("  Resources created and will be dropped...");
    } // Resources are automatically dropped here
    
    println!("  Resources have been cleaned up");
}
```

Advanced enum patterns demonstrate sophisticated type system usage  
including explicit discriminants, memory optimization, phantom types for  
compile-time state tracking, and integration with const generics. These  
patterns enable zero-cost abstractions and compile-time guarantees.  