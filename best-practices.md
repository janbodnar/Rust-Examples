# Best Practices and Idiomatic Rust

Writing idiomatic Rust code involves following established patterns that  
leverage Rust's unique features like ownership, borrowing, and type safety.  
These practices help create code that is not only safe and performant but  
also readable and maintainable. This guide covers common idioms, potential  
pitfalls, and proven patterns for writing clean, efficient Rust code.  

Key principles of idiomatic Rust:  
- **Embrace ownership**: Use borrowing instead of cloning when possible  
- **Leverage the type system**: Make invalid states unrepresentable  
- **Handle errors explicitly**: Use Result and Option types properly  
- **Write self-documenting code**: Use descriptive names and clear patterns  
- **Follow community conventions**: Adhere to established style guidelines  

These examples progress from basic naming and formatting conventions to  
advanced patterns for API design, error handling, and performance  
optimization. Each practice is demonstrated with clear examples showing  
both what to do and what to avoid.  

## Use descriptive and consistent naming

Choose clear, descriptive names that follow Rust naming conventions.  
Use snake_case for variables and functions, PascalCase for types.  

```rust
// Good: Clear, descriptive names following conventions
struct UserAccount {
    username: String,
    email_address: String,
    is_verified: bool,
}

impl UserAccount {
    fn send_verification_email(&self) -> Result<(), EmailError> {
        // Implementation here
        Ok(())
    }
    
    fn calculate_account_score(&self) -> u32 {
        // Implementation here
        42
    }
}

// Avoid: Unclear abbreviations and inconsistent naming
struct UsrAcc {
    usr_nm: String,
    emailAddr: String,  // inconsistent casing
    verified: bool,
}

fn main() {
    let user_account = UserAccount {
        username: "alice_smith".to_string(),
        email_address: "alice@example.com".to_string(),
        is_verified: false,
    };
    
    println!("User: {}", user_account.username);
    println!("Score: {}", user_account.calculate_account_score());
}

#[derive(Debug)]
enum EmailError {
    NetworkFailure,
    InvalidAddress,
}
```

Descriptive naming makes code self-documenting and reduces the need for  
comments. Following Rust conventions ensures consistency across the  
ecosystem and makes code familiar to other Rust developers.  

## Prefer borrowing over cloning

Use references and borrowing instead of cloning data when possible to  
avoid unnecessary memory allocations and improve performance.  

```rust
// Good: Use borrowing for read-only access
fn calculate_total_length(strings: &[String]) -> usize {
    strings.iter().map(|s| s.len()).sum()
}

fn print_user_info(user: &UserAccount) {
    println!("Username: {}", user.username);
    println!("Email: {}", user.email_address);
}

// Good: Use mutable borrowing for modifications
fn capitalize_strings(strings: &mut [String]) {
    for string in strings {
        if let Some(first_char) = string.chars().next() {
            *string = first_char.to_uppercase().collect::<String>() 
                + &string[1..];
        }
    }
}

// Avoid: Unnecessary cloning
fn calculate_total_length_bad(strings: Vec<String>) -> usize {
    strings.iter().map(|s| s.len()).sum()  // strings is moved, can't be used again
}

fn print_user_info_bad(user: UserAccount) {
    println!("Username: {}", user.username);  // user is consumed
}

struct UserAccount {
    username: String,
    email_address: String,
    is_verified: bool,
}

fn main() {
    let mut strings = vec![
        "hello".to_string(),
        "world".to_string(),
        "rust".to_string(),
    ];
    
    // Using borrowing - strings can be reused
    let total_len = calculate_total_length(&strings);
    println!("Total length: {}", total_len);
    
    // Modify in place
    capitalize_strings(&mut strings);
    println!("Capitalized: {:?}", strings);
    
    // strings is still available for use
    println!("Final strings: {:?}", strings);
}
```

Borrowing allows multiple parts of your code to access data without  
transferring ownership. This reduces memory usage and allows for more  
flexible code design while maintaining Rust's safety guarantees.  

## Use Option and Result for error handling

Embrace Rust's type system for error handling instead of panicking or  
using sentinel values like -1 or null.  

```rust
use std::collections::HashMap;

// Good: Use Option for values that might not exist
fn find_user_by_id(users: &HashMap<u32, String>, id: u32) -> Option<&String> {
    users.get(&id)
}

// Good: Use Result for operations that can fail
fn parse_age(input: &str) -> Result<u32, String> {
    input.parse::<u32>()
        .map_err(|_| format!("'{}' is not a valid age", input))
}

fn validate_email(email: &str) -> Result<(), String> {
    if email.contains('@') && email.contains('.') {
        Ok(())
    } else {
        Err("Invalid email format".to_string())
    }
}

// Good: Chain operations with combinators
fn process_user_input(input: &str) -> Result<String, String> {
    parse_age(input)
        .and_then(|age| {
            if age >= 18 {
                Ok(format!("User is {} years old and eligible", age))
            } else {
                Err("User must be 18 or older".to_string())
            }
        })
}

// Avoid: Using panic for recoverable errors
fn parse_age_bad(input: &str) -> u32 {
    input.parse().expect("Failed to parse age")  // Will crash on invalid input
}

fn main() {
    let mut users = HashMap::new();
    users.insert(1, "Alice".to_string());
    users.insert(2, "Bob".to_string());
    
    // Handle Option gracefully
    match find_user_by_id(&users, 1) {
        Some(name) => println!("Found user: {}", name),
        None => println!("User not found"),
    }
    
    // Handle Result with pattern matching
    match parse_age("25") {
        Ok(age) => println!("Parsed age: {}", age),
        Err(error) => println!("Error: {}", error),
    }
    
    // Chain operations
    match process_user_input("20") {
        Ok(message) => println!("Success: {}", message),
        Err(error) => println!("Error: {}", error),
    }
}
```

Option and Result types make error handling explicit and force you to  
consider failure cases. This leads to more robust code that gracefully  
handles edge cases instead of crashing.  

## Use iterators instead of manual loops

Rust's iterator adapters are both more expressive and often more  
efficient than manual loops due to compiler optimizations.  

```rust
// Good: Use iterator methods for common operations
fn process_numbers(numbers: &[i32]) -> Vec<String> {
    numbers
        .iter()
        .filter(|&&n| n > 0)
        .map(|n| format!("positive: {}", n))
        .collect()
}

fn find_largest_even(numbers: &[i32]) -> Option<i32> {
    numbers
        .iter()
        .filter(|&&n| n % 2 == 0)
        .max()
        .copied()
}

fn calculate_stats(numbers: &[f64]) -> (f64, f64, usize) {
    let sum: f64 = numbers.iter().sum();
    let count = numbers.len();
    let average = if count > 0 { sum / count as f64 } else { 0.0 };
    (sum, average, count)
}

// Good: Use enumerate for index access
fn find_indices_of_value(slice: &[i32], target: i32) -> Vec<usize> {
    slice
        .iter()
        .enumerate()
        .filter_map(|(i, &value)| {
            if value == target { Some(i) } else { None }
        })
        .collect()
}

// Avoid: Manual loops for simple operations
fn process_numbers_bad(numbers: &[i32]) -> Vec<String> {
    let mut result = Vec::new();
    for i in 0..numbers.len() {
        if numbers[i] > 0 {
            result.push(format!("positive: {}", numbers[i]));
        }
    }
    result
}

fn main() {
    let numbers = vec![-2, -1, 0, 1, 2, 3, 4, 5, 6];
    let floats = vec![1.5, 2.7, 3.1, 4.9];
    
    let processed = process_numbers(&numbers);
    println!("Processed: {:?}", processed);
    
    if let Some(largest_even) = find_largest_even(&numbers) {
        println!("Largest even: {}", largest_even);
    }
    
    let (sum, avg, count) = calculate_stats(&floats);
    println!("Sum: {:.2}, Average: {:.2}, Count: {}", sum, avg, count);
    
    let indices = find_indices_of_value(&numbers, 2);
    println!("Indices of value 2: {:?}", indices);
}
```

Iterator methods are often more readable and less error-prone than manual  
loops. They also enable powerful functional programming patterns and can  
be optimized better by the compiler.  

## Use pattern matching effectively

Leverage Rust's powerful pattern matching to write concise and clear  
control flow logic that handles all cases explicitly.  

```rust
#[derive(Debug)]
enum ProcessingResult {
    Success(String),
    Warning(String, u32),
    Error(String),
}

#[derive(Debug)]
struct Config {
    debug_mode: bool,
    max_retries: Option<u32>,
    timeout_seconds: u32,
}

// Good: Comprehensive pattern matching
fn handle_result(result: ProcessingResult) -> String {
    match result {
        ProcessingResult::Success(message) => {
            format!("✓ {}", message)
        }
        ProcessingResult::Warning(message, code) => {
            format!("⚠ {} (code: {})", message, code)
        }
        ProcessingResult::Error(message) => {
            format!("✗ Error: {}", message)
        }
    }
}

// Good: Pattern matching with guards
fn categorize_number(n: i32) -> &'static str {
    match n {
        x if x < 0 => "negative",
        0 => "zero",
        1..=10 => "small positive",
        11..=100 => "medium positive",
        _ => "large positive",
    }
}

// Good: Destructuring in pattern matching
fn process_config(config: Config) -> String {
    match config {
        Config { debug_mode: true, max_retries: Some(retries), timeout_seconds } => {
            format!("Debug mode: {} retries, {} sec timeout", retries, timeout_seconds)
        }
        Config { debug_mode: false, max_retries: None, timeout_seconds } => {
            format!("Production mode: no retries, {} sec timeout", timeout_seconds)
        }
        Config { debug_mode, max_retries, timeout_seconds } => {
            format!("Mixed mode: debug={}, retries={:?}, timeout={}", 
                   debug_mode, max_retries, timeout_seconds)
        }
    }
}

// Good: Using if let for simple cases
fn extract_success_message(result: ProcessingResult) -> Option<String> {
    if let ProcessingResult::Success(message) = result {
        Some(message)
    } else {
        None
    }
}

fn main() {
    let results = vec![
        ProcessingResult::Success("Data processed successfully".to_string()),
        ProcessingResult::Warning("Deprecated API used".to_string(), 301),
        ProcessingResult::Error("Network connection failed".to_string()),
    ];
    
    for result in results {
        println!("{}", handle_result(result));
    }
    
    let numbers = vec![-5, 0, 3, 15, 150];
    for num in numbers {
        println!("{} is {}", num, categorize_number(num));
    }
    
    let configs = vec![
        Config { debug_mode: true, max_retries: Some(3), timeout_seconds: 30 },
        Config { debug_mode: false, max_retries: None, timeout_seconds: 60 },
        Config { debug_mode: true, max_retries: None, timeout_seconds: 10 },
    ];
    
    for config in configs {
        println!("{}", process_config(config));
    }
}
```

Pattern matching makes code more readable and ensures all cases are  
handled. It's particularly powerful for working with enums and complex  
data structures, reducing the chance of logic errors.  

## Write self-documenting code with types

Use the type system to encode invariants and make invalid states  
unrepresentable, reducing the need for runtime checks.  

```rust
use std::collections::HashMap;

// Good: Use newtype pattern for type safety
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
struct UserId(u32);

#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
struct ProductId(u32);

#[derive(Debug, Clone)]
struct Price {
    cents: u32,  // Store as cents to avoid float precision issues
}

impl Price {
    fn new(dollars: f64) -> Self {
        Self {
            cents: (dollars * 100.0).round() as u32,
        }
    }
    
    fn dollars(&self) -> f64 {
        self.cents as f64 / 100.0
    }
}

// Good: Use enums to represent states
#[derive(Debug)]
enum OrderStatus {
    Pending,
    Confirmed { tracking_number: String },
    Shipped { carrier: String, tracking_number: String },
    Delivered { delivery_date: String },
    Cancelled { reason: String },
}

#[derive(Debug)]
struct Order {
    id: UserId,
    product_id: ProductId,
    quantity: u32,
    unit_price: Price,
    status: OrderStatus,
}

impl Order {
    fn new(user_id: UserId, product_id: ProductId, quantity: u32, unit_price: Price) -> Self {
        Self {
            id: user_id,
            product_id,
            quantity,
            unit_price,
            status: OrderStatus::Pending,
        }
    }
    
    fn total_price(&self) -> Price {
        Price {
            cents: self.unit_price.cents * self.quantity,
        }
    }
    
    fn can_be_cancelled(&self) -> bool {
        matches!(self.status, OrderStatus::Pending | OrderStatus::Confirmed { .. })
    }
}

// Good: Use builder pattern for complex construction
struct OrderBuilder {
    user_id: Option<UserId>,
    product_id: Option<ProductId>,
    quantity: u32,
    unit_price: Option<Price>,
}

impl OrderBuilder {
    fn new() -> Self {
        Self {
            user_id: None,
            product_id: None,
            quantity: 1,
            unit_price: None,
        }
    }
    
    fn user_id(mut self, id: UserId) -> Self {
        self.user_id = Some(id);
        self
    }
    
    fn product_id(mut self, id: ProductId) -> Self {
        self.product_id = Some(id);
        self
    }
    
    fn quantity(mut self, qty: u32) -> Self {
        self.quantity = qty;
        self
    }
    
    fn unit_price(mut self, price: Price) -> Self {
        self.unit_price = Some(price);
        self
    }
    
    fn build(self) -> Result<Order, String> {
        let user_id = self.user_id.ok_or("User ID is required")?;
        let product_id = self.product_id.ok_or("Product ID is required")?;
        let unit_price = self.unit_price.ok_or("Unit price is required")?;
        
        if self.quantity == 0 {
            return Err("Quantity must be greater than 0".to_string());
        }
        
        Ok(Order::new(user_id, product_id, self.quantity, unit_price))
    }
}

fn main() {
    let user_id = UserId(12345);
    let product_id = ProductId(67890);
    let price = Price::new(29.99);
    
    // Type safety prevents mixing up IDs
    let mut orders: HashMap<UserId, Vec<Order>> = HashMap::new();
    
    // Using builder pattern for complex construction
    let order = OrderBuilder::new()
        .user_id(user_id)
        .product_id(product_id)
        .quantity(2)
        .unit_price(price)
        .build()
        .expect("Failed to build order");
    
    println!("Order: {:?}", order);
    println!("Total price: ${:.2}", order.total_price().dollars());
    println!("Can be cancelled: {}", order.can_be_cancelled());
    
    orders.entry(user_id).or_insert_with(Vec::new).push(order);
    
    // Type system prevents accidents like:
    // orders.insert(product_id, vec![order]);  // Compile error!
    
    println!("Orders: {:?}", orders);
}
```

Using distinct types prevents common mistakes like mixing up different  
kinds of IDs. Enums model state machines clearly, and builder patterns  
make complex object construction safe and readable.  

## Implement traits for common operations

Implement standard traits to make your types work seamlessly with Rust's  
ecosystem and enable idiomatic usage patterns.  

```rust
use std::fmt;
use std::ops::{Add, Sub};
use std::cmp::Ordering;

#[derive(Debug, Clone, Copy, PartialEq)]
struct Point {
    x: f64,
    y: f64,
}

impl Point {
    fn new(x: f64, y: f64) -> Self {
        Self { x, y }
    }
    
    fn distance_from_origin(&self) -> f64 {
        (self.x * self.x + self.y * self.y).sqrt()
    }
}

// Implement Display for user-friendly output
impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "({:.2}, {:.2})", self.x, self.y)
    }
}

// Implement arithmetic operations
impl Add for Point {
    type Output = Point;
    
    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

impl Sub for Point {
    type Output = Point;
    
    fn sub(self, other: Point) -> Point {
        Point {
            x: self.x - other.x,
            y: self.y - other.y,
        }
    }
}

// Implement ordering based on distance from origin
impl PartialOrd for Point {
    fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
        self.distance_from_origin()
            .partial_cmp(&other.distance_from_origin())
    }
}

// Custom trait for domain-specific operations
trait Geometric {
    fn distance_to(&self, other: &Self) -> f64;
    fn midpoint(&self, other: &Self) -> Self;
}

impl Geometric for Point {
    fn distance_to(&self, other: &Self) -> f64 {
        let dx = self.x - other.x;
        let dy = self.y - other.y;
        (dx * dx + dy * dy).sqrt()
    }
    
    fn midpoint(&self, other: &Self) -> Self {
        Point {
            x: (self.x + other.x) / 2.0,
            y: (self.y + other.y) / 2.0,
        }
    }
}

// Implement From trait for convenient conversions
impl From<(f64, f64)> for Point {
    fn from(tuple: (f64, f64)) -> Self {
        Point::new(tuple.0, tuple.1)
    }
}

impl From<[f64; 2]> for Point {
    fn from(array: [f64; 2]) -> Self {
        Point::new(array[0], array[1])
    }
}

// Make it iterable by implementing IntoIterator
impl IntoIterator for Point {
    type Item = f64;
    type IntoIter = std::array::IntoIter<f64, 2>;
    
    fn into_iter(self) -> Self::IntoIter {
        [self.x, self.y].into_iter()
    }
}

fn main() {
    let p1 = Point::new(3.0, 4.0);
    let p2 = Point::from((1.0, 2.0));
    let p3: Point = [5.0, 6.0].into();
    
    // Using Display trait
    println!("Points: {}, {}, {}", p1, p2, p3);
    
    // Using arithmetic operations
    let sum = p1 + p2;
    let diff = p3 - p1;
    println!("Sum: {}, Difference: {}", sum, diff);
    
    // Using comparison
    let mut points = vec![p1, p2, p3];
    points.sort_by(|a, b| a.partial_cmp(b).unwrap());
    println!("Sorted by distance from origin: {:?}", points);
    
    // Using custom trait
    println!("Distance between p1 and p2: {:.2}", p1.distance_to(&p2));
    println!("Midpoint of p1 and p3: {}", p1.midpoint(&p3));
    
    // Using IntoIterator
    for coordinate in p1 {
        println!("Coordinate: {:.2}", coordinate);
    }
    
    // Collecting coordinates
    let coordinates: Vec<f64> = p1.into_iter().collect();
    println!("Coordinates: {:?}", coordinates);
}
```

Implementing standard traits makes your types feel native to Rust and  
enables them to work with standard library functions, iterators, and  
collection operations seamlessly.  

## Use const and static appropriately

Use const for compile-time constants and static for global variables,  
choosing the right one based on your specific needs.  

```rust
use std::collections::HashMap;
use std::sync::Mutex;

// Good: Use const for compile-time constants
const MAX_BUFFER_SIZE: usize = 8192;
const PI: f64 = 3.14159265359;
const DEFAULT_TIMEOUT_MS: u64 = 5000;

// Good: Use const fn for compile-time computation
const fn calculate_buffer_segments(total_size: usize, segment_size: usize) -> usize {
    (total_size + segment_size - 1) / segment_size
}

const BUFFER_SEGMENTS: usize = calculate_buffer_segments(MAX_BUFFER_SIZE, 512);

// Good: Use static for global state (prefer avoiding when possible)
static GLOBAL_COUNTER: Mutex<u64> = Mutex::new(0);

// Good: Use static for expensive-to-compute constants that aren't compile-time
static ERROR_CODES: std::sync::LazyLock<HashMap<u32, &'static str>> = 
    std::sync::LazyLock::new(|| {
        let mut map = HashMap::new();
        map.insert(404, "Not Found");
        map.insert(500, "Internal Server Error");
        map.insert(200, "OK");
        map.insert(401, "Unauthorized");
        map
    });

struct Config {
    max_connections: usize,
    timeout_ms: u64,
    buffer_size: usize,
}

impl Config {
    // Good: Use const for default configurations
    const DEFAULT: Self = Self {
        max_connections: 100,
        timeout_ms: DEFAULT_TIMEOUT_MS,
        buffer_size: MAX_BUFFER_SIZE,
    };
    
    fn new() -> Self {
        Self::DEFAULT
    }
    
    fn with_timeout(mut self, timeout_ms: u64) -> Self {
        self.timeout_ms = timeout_ms;
        self
    }
}

// Good: Use const generics for compile-time parameters
struct FixedBuffer<const N: usize> {
    data: [u8; N],
    length: usize,
}

impl<const N: usize> FixedBuffer<N> {
    const fn new() -> Self {
        Self {
            data: [0; N],
            length: 0,
        }
    }
    
    fn push(&mut self, byte: u8) -> Result<(), &'static str> {
        if self.length >= N {
            Err("Buffer is full")
        } else {
            self.data[self.length] = byte;
            self.length += 1;
            Ok(())
        }
    }
    
    fn len(&self) -> usize {
        self.length
    }
    
    fn capacity(&self) -> usize {
        N
    }
}

fn get_error_message(code: u32) -> Option<&'static str> {
    ERROR_CODES.get(&code).copied()
}

fn increment_global_counter() -> u64 {
    let mut counter = GLOBAL_COUNTER.lock().unwrap();
    *counter += 1;
    *counter
}

fn main() {
    println!("Max buffer size: {}", MAX_BUFFER_SIZE);
    println!("Buffer segments: {}", BUFFER_SEGMENTS);
    
    let config = Config::new().with_timeout(10_000);
    println!("Config timeout: {} ms", config.timeout_ms);
    
    // Using const generic buffer
    let mut small_buffer: FixedBuffer<64> = FixedBuffer::new();
    let mut large_buffer: FixedBuffer<{ MAX_BUFFER_SIZE }> = FixedBuffer::new();
    
    small_buffer.push(42).unwrap();
    large_buffer.push(100).unwrap();
    
    println!("Small buffer: {}/{}", small_buffer.len(), small_buffer.capacity());
    println!("Large buffer: {}/{}", large_buffer.len(), large_buffer.capacity());
    
    // Using static error codes
    for &code in &[200, 404, 500, 999] {
        match get_error_message(code) {
            Some(message) => println!("Error {}: {}", code, message),
            None => println!("Unknown error code: {}", code),
        }
    }
    
    // Using global counter (generally discouraged, but sometimes necessary)
    for _ in 0..3 {
        println!("Counter: {}", increment_global_counter());
    }
}
```

Constants provide compile-time guarantees and zero runtime cost. Static  
variables should be used sparingly for global state, and when used,  
should be thread-safe with appropriate synchronization primitives.  

## Handle string types appropriately

Choose the right string type for your use case: &str for borrowed strings,  
String for owned strings, and understand when to convert between them.  

```rust
use std::collections::HashMap;

// Good: Use &str for parameters when you don't need ownership
fn process_name(name: &str) -> String {
    name.trim()
        .split_whitespace()
        .map(|word| {
            let mut chars: Vec<char> = word.chars().collect();
            if let Some(first) = chars.first_mut() {
                *first = first.to_uppercase().next().unwrap_or(*first);
            }
            chars.into_iter().collect::<String>()
        })
        .collect::<Vec<_>>()
        .join(" ")
}

// Good: Use String for return values when you need to own the data
fn create_greeting(name: &str, time_of_day: &str) -> String {
    format!("Good {}, {}!", time_of_day, name)
}

// Good: Use &str for lookups and comparisons
fn find_user_role(username: &str, roles: &HashMap<String, String>) -> Option<&String> {
    roles.get(username)
}

// Good: Use Cow for functions that might need to modify or not
use std::borrow::Cow;

fn normalize_path(path: &str) -> Cow<str> {
    if path.contains("\\") {
        // Need to modify - return owned String
        Cow::Owned(path.replace("\\", "/"))
    } else {
        // No modification needed - return borrowed &str
        Cow::Borrowed(path)
    }
}

// Good: Efficient string building for multiple concatenations
fn build_sql_query(table: &str, columns: &[&str], conditions: &[&str]) -> String {
    let mut query = String::with_capacity(256);  // Pre-allocate capacity
    
    query.push_str("SELECT ");
    query.push_str(&columns.join(", "));
    query.push_str(" FROM ");
    query.push_str(table);
    
    if !conditions.is_empty() {
        query.push_str(" WHERE ");
        query.push_str(&conditions.join(" AND "));
    }
    
    query
}

// Good: Use format! for simple cases, but avoid in loops
fn create_user_summary(name: &str, age: u32, city: &str) -> String {
    format!("{} is {} years old and lives in {}", name, age, city)
}

// Good: String validation and sanitization
fn validate_and_clean_input(input: &str) -> Result<String, String> {
    let trimmed = input.trim();
    
    if trimmed.is_empty() {
        return Err("Input cannot be empty".to_string());
    }
    
    if trimmed.len() > 100 {
        return Err("Input too long (max 100 characters)".to_string());
    }
    
    // Remove any control characters
    let cleaned: String = trimmed
        .chars()
        .filter(|c| !c.is_control() || c.is_whitespace())
        .collect();
    
    Ok(cleaned)
}

// Avoid: Taking String when &str would work
fn bad_process_name(name: String) -> String {  // Forces caller to give up ownership
    name.to_uppercase()
}

// Avoid: Unnecessary string allocations
fn bad_string_comparison(text: &str, target: &str) -> bool {
    text.to_string() == target.to_string()  // Should just be: text == target
}

fn main() {
    let raw_name = "  alice   bob  smith  ";
    let processed = process_name(raw_name);
    println!("Processed name: '{}'", processed);
    
    let greeting = create_greeting(&processed, "morning");
    println!("{}", greeting);
    
    // Using HashMap with string keys
    let mut roles = HashMap::new();
    roles.insert("alice".to_string(), "admin".to_string());
    roles.insert("bob".to_string(), "user".to_string());
    
    if let Some(role) = find_user_role("alice", &roles) {
        println!("Alice's role: {}", role);
    }
    
    // Using Cow for efficient path handling
    let windows_path = "C:\\Users\\Alice\\Documents";
    let unix_path = "/home/alice/documents";
    
    println!("Normalized paths:");
    println!("  {}", normalize_path(windows_path));
    println!("  {}", normalize_path(unix_path));
    
    // Efficient string building
    let columns = ["id", "name", "email"];
    let conditions = ["active = true", "age >= 18"];
    let query = build_sql_query("users", &columns, &conditions);
    println!("SQL Query: {}", query);
    
    // String validation
    let inputs = ["  valid input  ", "", "a".repeat(150).as_str()];
    for input in inputs {
        match validate_and_clean_input(input) {
            Ok(cleaned) => println!("Valid input: '{}'", cleaned),
            Err(error) => println!("Invalid input: {}", error),
        }
    }
}
```

Understanding string types is crucial for efficient Rust programming.  
Use &str for borrowing, String for ownership, and consider Cow when you  
might need either. Avoid unnecessary allocations and conversions.  

## Use Vec and slice operations effectively

Leverage Vec's capabilities and slice operations for efficient collection  
manipulation while understanding performance implications.  

```rust
// Good: Pre-allocate capacity when size is known
fn create_fibonacci_sequence(n: usize) -> Vec<u64> {
    if n == 0 {
        return Vec::new();
    }
    
    let mut fib = Vec::with_capacity(n);  // Avoid reallocations
    fib.push(0);
    
    if n > 1 {
        fib.push(1);
        
        for i in 2..n {
            let next = fib[i - 1] + fib[i - 2];
            fib.push(next);
        }
    }
    
    fib
}

// Good: Use slices for functions that don't need ownership
fn find_max_subarray_sum(arr: &[i32]) -> (i32, usize, usize) {
    if arr.is_empty() {
        return (0, 0, 0);
    }
    
    let mut max_sum = arr[0];
    let mut current_sum = arr[0];
    let mut start = 0;
    let mut end = 0;
    let mut temp_start = 0;
    
    for i in 1..arr.len() {
        if current_sum < 0 {
            current_sum = arr[i];
            temp_start = i;
        } else {
            current_sum += arr[i];
        }
        
        if current_sum > max_sum {
            max_sum = current_sum;
            start = temp_start;
            end = i;
        }
    }
    
    (max_sum, start, end)
}

// Good: Use slice methods for common operations
fn process_data(data: &mut [i32]) {
    // Sort in place
    data.sort_unstable();
    
    // Remove duplicates (requires sorted data)
    data.dedup();
}

fn analyze_numbers(numbers: &[i32]) -> Vec<String> {
    numbers
        .chunks(3)  // Process in groups of 3
        .enumerate()
        .map(|(i, chunk)| {
            let sum: i32 = chunk.iter().sum();
            let avg = sum as f64 / chunk.len() as f64;
            format!("Group {}: sum={}, avg={:.2}, values={:?}", i, sum, avg, chunk)
        })
        .collect()
}

// Good: Use split and window operations
fn find_patterns(text: &[char], pattern: &[char]) -> Vec<usize> {
    if pattern.is_empty() || text.len() < pattern.len() {
        return Vec::new();
    }
    
    text.windows(pattern.len())
        .enumerate()
        .filter_map(|(i, window)| {
            if window == pattern {
                Some(i)
            } else {
                None
            }
        })
        .collect()
}

// Good: Efficient batch processing
fn batch_process<T, F, R>(items: Vec<T>, batch_size: usize, mut processor: F) -> Vec<R>
where
    F: FnMut(&[T]) -> R,
{
    items
        .chunks(batch_size)
        .map(|batch| processor(batch))
        .collect()
}

// Good: Safe indexing with bounds checking
fn safe_get_range(vec: &[i32], start: usize, len: usize) -> Option<&[i32]> {
    if start + len <= vec.len() {
        Some(&vec[start..start + len])
    } else {
        None
    }
}

// Good: Using drain for efficient removal
fn remove_negative_numbers(numbers: &mut Vec<i32>) -> Vec<i32> {
    let mut removed = Vec::new();
    let mut i = 0;
    
    while i < numbers.len() {
        if numbers[i] < 0 {
            removed.push(numbers.remove(i));
        } else {
            i += 1;
        }
    }
    
    removed
}

// More efficient approach using retain
fn remove_negative_numbers_efficient(numbers: &mut Vec<i32>) -> Vec<i32> {
    let mut removed = Vec::new();
    
    let mut i = 0;
    numbers.retain(|&x| {
        if x < 0 {
            removed.push(x);
            false
        } else {
            true
        }
    });
    
    removed
}

fn main() {
    // Demonstrate Fibonacci generation
    let fib = create_fibonacci_sequence(10);
    println!("Fibonacci sequence: {:?}", fib);
    
    // Demonstrate max subarray
    let test_array = vec![-2, 1, -3, 4, -1, 2, 1, -5, 4];
    let (max_sum, start, end) = find_max_subarray_sum(&test_array);
    println!("Max subarray sum: {} from index {} to {}", max_sum, start, end);
    println!("Subarray: {:?}", &test_array[start..=end]);
    
    // Demonstrate data processing
    let mut data = vec![3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5];
    println!("Original data: {:?}", data);
    process_data(&mut data);
    println!("Processed data: {:?}", data);
    
    // Demonstrate analysis
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11];
    let analysis = analyze_numbers(&numbers);
    for result in analysis {
        println!("{}", result);
    }
    
    // Demonstrate pattern finding
    let text: Vec<char> = "hello world hello rust".chars().collect();
    let pattern: Vec<char> = "hello".chars().collect();
    let positions = find_patterns(&text, &pattern);
    println!("Pattern 'hello' found at positions: {:?}", positions);
    
    // Demonstrate batch processing
    let items = (1..=20).collect::<Vec<i32>>();
    let batch_results = batch_process(items, 5, |batch| {
        batch.iter().sum::<i32>()
    });
    println!("Batch sums: {:?}", batch_results);
    
    // Demonstrate safe indexing
    let test_vec = vec![1, 2, 3, 4, 5];
    match safe_get_range(&test_vec, 1, 3) {
        Some(slice) => println!("Safe slice: {:?}", slice),
        None => println!("Index out of bounds"),
    }
    
    // Demonstrate negative number removal
    let mut test_numbers = vec![-3, 5, -1, 8, -2, 10, -7];
    println!("Original: {:?}", test_numbers);
    let removed = remove_negative_numbers_efficient(&mut test_numbers);
    println!("Remaining: {:?}", test_numbers);
    println!("Removed: {:?}", removed);
}
```

Understanding Vec and slice operations enables efficient collection  
processing. Pre-allocating capacity, using slice methods, and choosing  
the right operation for your use case can significantly improve  
performance.  

## Design clean APIs with modules

Organize code into logical modules with clear public interfaces while  
keeping implementation details private.  

```rust
// lib.rs or main module

pub mod geometry {
    use std::f64::consts::PI;
    
    /// A 2D point with x and y coordinates
    #[derive(Debug, Clone, Copy, PartialEq)]
    pub struct Point {
        pub x: f64,
        pub y: f64,
    }
    
    /// Represents different shapes in 2D space
    #[derive(Debug, Clone)]
    pub enum Shape {
        Circle { center: Point, radius: f64 },
        Rectangle { top_left: Point, width: f64, height: f64 },
        Triangle { vertices: [Point; 3] },
    }
    
    impl Point {
        pub fn new(x: f64, y: f64) -> Self {
            Self { x, y }
        }
        
        pub fn origin() -> Self {
            Self::new(0.0, 0.0)
        }
        
        pub fn distance_to(&self, other: &Point) -> f64 {
            let dx = self.x - other.x;
            let dy = self.y - other.y;
            (dx * dx + dy * dy).sqrt()
        }
    }
    
    impl Shape {
        pub fn area(&self) -> f64 {
            match self {
                Shape::Circle { radius, .. } => PI * radius * radius,
                Shape::Rectangle { width, height, .. } => width * height,
                Shape::Triangle { vertices } => {
                    triangle_area(&vertices[0], &vertices[1], &vertices[2])
                }
            }
        }
        
        pub fn perimeter(&self) -> f64 {
            match self {
                Shape::Circle { radius, .. } => 2.0 * PI * radius,
                Shape::Rectangle { width, height, .. } => 2.0 * (width + height),
                Shape::Triangle { vertices } => {
                    vertices[0].distance_to(&vertices[1])
                        + vertices[1].distance_to(&vertices[2])
                        + vertices[2].distance_to(&vertices[0])
                }
            }
        }
        
        pub fn contains_point(&self, point: &Point) -> bool {
            match self {
                Shape::Circle { center, radius } => {
                    center.distance_to(point) <= *radius
                }
                Shape::Rectangle { top_left, width, height } => {
                    point.x >= top_left.x
                        && point.x <= top_left.x + width
                        && point.y >= top_left.y
                        && point.y <= top_left.y + height
                }
                Shape::Triangle { vertices } => {
                    point_in_triangle(point, &vertices[0], &vertices[1], &vertices[2])
                }
            }
        }
    }
    
    // Private helper functions - not exposed in public API
    fn triangle_area(p1: &Point, p2: &Point, p3: &Point) -> f64 {
        let a = p1.distance_to(p2);
        let b = p2.distance_to(p3);
        let c = p3.distance_to(p1);
        let s = (a + b + c) / 2.0;
        (s * (s - a) * (s - b) * (s - c)).sqrt()
    }
    
    fn point_in_triangle(p: &Point, a: &Point, b: &Point, c: &Point) -> bool {
        fn sign(p1: &Point, p2: &Point, p3: &Point) -> f64 {
            (p1.x - p3.x) * (p2.y - p3.y) - (p2.x - p3.x) * (p1.y - p3.y)
        }
        
        let d1 = sign(p, a, b);
        let d2 = sign(p, b, c);
        let d3 = sign(p, c, a);
        
        let has_neg = (d1 < 0.0) || (d2 < 0.0) || (d3 < 0.0);
        let has_pos = (d1 > 0.0) || (d2 > 0.0) || (d3 > 0.0);
        
        !(has_neg && has_pos)
    }
}

pub mod config {
    use std::collections::HashMap;
    use std::fs;
    use std::path::Path;
    
    #[derive(Debug, Clone)]
    pub struct AppConfig {
        settings: HashMap<String, String>,
    }
    
    impl AppConfig {
        pub fn new() -> Self {
            Self {
                settings: HashMap::new(),
            }
        }
        
        pub fn from_file<P: AsRef<Path>>(path: P) -> Result<Self, ConfigError> {
            let content = fs::read_to_string(path)
                .map_err(|e| ConfigError::IoError(e.to_string()))?;
            
            Self::from_string(&content)
        }
        
        pub fn from_string(content: &str) -> Result<Self, ConfigError> {
            let mut settings = HashMap::new();
            
            for (line_num, line) in content.lines().enumerate() {
                let line = line.trim();
                
                // Skip empty lines and comments
                if line.is_empty() || line.starts_with('#') {
                    continue;
                }
                
                if let Some((key, value)) = line.split_once('=') {
                    settings.insert(key.trim().to_string(), value.trim().to_string());
                } else {
                    return Err(ConfigError::ParseError {
                        line: line_num + 1,
                        content: line.to_string(),
                    });
                }
            }
            
            Ok(Self { settings })
        }
        
        pub fn get(&self, key: &str) -> Option<&str> {
            self.settings.get(key).map(|s| s.as_str())
        }
        
        pub fn get_or_default(&self, key: &str, default: &str) -> &str {
            self.get(key).unwrap_or(default)
        }
        
        pub fn set(&mut self, key: String, value: String) {
            self.settings.insert(key, value);
        }
        
        pub fn keys(&self) -> impl Iterator<Item = &String> {
            self.settings.keys()
        }
    }
    
    impl Default for AppConfig {
        fn default() -> Self {
            Self::new()
        }
    }
    
    #[derive(Debug)]
    pub enum ConfigError {
        IoError(String),
        ParseError { line: usize, content: String },
    }
    
    impl std::fmt::Display for ConfigError {
        fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
            match self {
                ConfigError::IoError(msg) => write!(f, "IO error: {}", msg),
                ConfigError::ParseError { line, content } => {
                    write!(f, "Parse error at line {}: '{}'", line, content)
                }
            }
        }
    }
    
    impl std::error::Error for ConfigError {}
}

pub mod utils {
    pub fn time_execution<F, R>(f: F) -> (R, std::time::Duration)
    where
        F: FnOnce() -> R,
    {
        let start = std::time::Instant::now();
        let result = f();
        let duration = start.elapsed();
        (result, duration)
    }
    
    pub fn retry_with_backoff<F, R, E>(
        mut operation: F,
        max_attempts: usize,
        initial_delay_ms: u64,
    ) -> Result<R, E>
    where
        F: FnMut() -> Result<R, E>,
    {
        let mut delay = initial_delay_ms;
        
        for attempt in 1..=max_attempts {
            match operation() {
                Ok(result) => return Ok(result),
                Err(e) => {
                    if attempt == max_attempts {
                        return Err(e);
                    }
                    std::thread::sleep(std::time::Duration::from_millis(delay));
                    delay *= 2;  // Exponential backoff
                }
            }
        }
        
        unreachable!()
    }
}

// Example usage
fn main() {
    use geometry::*;
    use config::*;
    use utils::*;
    
    // Using geometry module
    let circle = Shape::Circle {
        center: Point::new(0.0, 0.0),
        radius: 5.0,
    };
    
    let rectangle = Shape::Rectangle {
        top_left: Point::new(-2.0, -2.0),
        width: 4.0,
        height: 4.0,
    };
    
    let triangle = Shape::Triangle {
        vertices: [
            Point::new(0.0, 3.0),
            Point::new(-2.0, -1.0),
            Point::new(2.0, -1.0),
        ]
    };
    
    let shapes = vec![circle, rectangle, triangle];
    let test_point = Point::new(1.0, 1.0);
    
    for (i, shape) in shapes.iter().enumerate() {
        println!("Shape {}: area={:.2}, perimeter={:.2}, contains_point({:.1},{:.1})={}",
                 i,
                 shape.area(),
                 shape.perimeter(),
                 test_point.x,
                 test_point.y,
                 shape.contains_point(&test_point)
        );
    }
    
    // Using config module
    let config_content = r#"
        # Application configuration
        app_name=MyApp
        port=8080
        debug=true
        max_connections=100
    "#;
    
    match AppConfig::from_string(config_content) {
        Ok(config) => {
            println!("\\nConfiguration loaded:");
            for key in config.keys() {
                println!("  {}: {}", key, config.get(key).unwrap());
            }
            
            let port = config.get_or_default("port", "3000");
            println!("Server will run on port: {}", port);
        }
        Err(e) => {
            println!("Configuration error: {}", e);
        }
    }
    
    // Using utils module
    let (result, duration) = time_execution(|| {
        (1..=1000).sum::<u32>()
    });
    println!("\\nCalculation result: {} (took {:?})", result, duration);
    
    // Simulate a flaky operation
    let mut attempt_count = 0;
    let flaky_operation = || {
        attempt_count += 1;
        if attempt_count < 3 {
            Err("Temporary failure")
        } else {
            Ok("Success!")
        }
    };
    
    match retry_with_backoff(flaky_operation, 5, 100) {
        Ok(result) => println!("Retry operation succeeded: {}", result),
        Err(e) => println!("Retry operation failed: {}", e),
    }
}
```

Well-designed modules provide clear boundaries, hide implementation  
details, and offer intuitive public APIs. Use private functions for  
internal logic and carefully consider what should be exposed publicly.  

## Use proper error types and error handling

Create meaningful error types that provide context and enable proper error  
handling throughout your application.  

```rust
use std::fmt;
use std::fs;
use std::io;
use std::num::ParseIntError;

// Good: Define specific error types for your domain
#[derive(Debug)]
pub enum DatabaseError {
    ConnectionFailed(String),
    QueryFailed { query: String, reason: String },
    RecordNotFound { id: u64 },
    InvalidData { field: String, value: String },
}

impl fmt::Display for DatabaseError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            DatabaseError::ConnectionFailed(msg) => {
                write!(f, "Database connection failed: {}", msg)
            }
            DatabaseError::QueryFailed { query, reason } => {
                write!(f, "Query failed: '{}' - {}", query, reason)
            }
            DatabaseError::RecordNotFound { id } => {
                write!(f, "Record with ID {} not found", id)
            }
            DatabaseError::InvalidData { field, value } => {
                write!(f, "Invalid data for field '{}': '{}'", field, value)
            }
        }
    }
}

impl std::error::Error for DatabaseError {}

// Good: Chain errors with context
#[derive(Debug)]
pub enum AppError {
    Database(DatabaseError),
    Io(io::Error),
    Parse(ParseIntError),
    Configuration(String),
    Validation { message: String, field: String },
}

impl fmt::Display for AppError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            AppError::Database(e) => write!(f, "Database error: {}", e),
            AppError::Io(e) => write!(f, "IO error: {}", e),
            AppError::Parse(e) => write!(f, "Parse error: {}", e),
            AppError::Configuration(msg) => write!(f, "Configuration error: {}", msg),
            AppError::Validation { message, field } => {
                write!(f, "Validation error in field '{}': {}", field, message)
            }
        }
    }
}

impl std::error::Error for AppError {
    fn source(&self) -> Option<&(dyn std::error::Error + 'static)> {
        match self {
            AppError::Database(e) => Some(e),
            AppError::Io(e) => Some(e),
            AppError::Parse(e) => Some(e),
            _ => None,
        }
    }
}

// Implement From for easy error conversion
impl From<DatabaseError> for AppError {
    fn from(err: DatabaseError) -> Self {
        AppError::Database(err)
    }
}

impl From<io::Error> for AppError {
    fn from(err: io::Error) -> Self {
        AppError::Io(err)
    }
}

impl From<ParseIntError> for AppError {
    fn from(err: ParseIntError) -> Self {
        AppError::Parse(err)
    }
}

// Good: Use Result types consistently
fn read_config_file(path: &str) -> Result<String, AppError> {
    fs::read_to_string(path).map_err(AppError::from)
}

fn parse_port(config: &str) -> Result<u16, AppError> {
    config
        .lines()
        .find(|line| line.starts_with("port="))
        .and_then(|line| line.strip_prefix("port="))
        .ok_or_else(|| AppError::Configuration("Port not found in config".to_string()))?
        .parse::<u16>()
        .map_err(AppError::from)
        .and_then(|port| {
            if port > 0 {
                Ok(port)
            } else {
                Err(AppError::Validation {
                    message: "Port must be greater than 0".to_string(),
                    field: "port".to_string(),
                })
            }
        })
}

fn find_user(id: u64) -> Result<String, DatabaseError> {
    // Simulate database operation
    match id {
        1 => Ok("Alice".to_string()),
        2 => Ok("Bob".to_string()),
        _ => Err(DatabaseError::RecordNotFound { id }),
    }
}

// Good: Handle errors at appropriate levels
fn initialize_server() -> Result<u16, AppError> {
    let config = read_config_file("server.conf")?;
    let port = parse_port(&config)?;
    
    // Additional validation
    if port < 1024 {
        return Err(AppError::Validation {
            message: "Port should be >= 1024 for non-root users".to_string(),
            field: "port".to_string(),
        });
    }
    
    Ok(port)
}

fn main() {
    // Create a test config file
    let config_content = "app_name=TestApp\nport=8080\ndebug=true";
    fs::write("server.conf", config_content).unwrap();
    
    // Demonstrate error handling
    match initialize_server() {
        Ok(port) => println!("Server initialized on port {}", port),
        Err(e) => {
            eprintln!("Failed to initialize server: {}", e);
            
            // Print error chain
            let mut source = e.source();
            while let Some(err) = source {
                eprintln!("  Caused by: {}", err);
                source = err.source();
            }
        }
    }
    
    // Demonstrate database error handling
    let user_ids = vec![1, 2, 99];
    for id in user_ids {
        match find_user(id) {
            Ok(name) => println!("User {}: {}", id, name),
            Err(e) => println!("Error: {}", e),
        }
    }
    
    // Cleanup
    let _ = fs::remove_file("server.conf");
}
```

Proper error handling makes your code more reliable and easier to debug.  
Define specific error types, chain errors with context, and handle them  
at the appropriate level in your application.  

## Use appropriate collection types

Choose the right collection type based on your access patterns and  
performance requirements.  

```rust
use std::collections::{HashMap, HashSet, BTreeMap, BTreeSet, VecDeque, LinkedList};
use std::hash::{Hash, Hasher};

#[derive(Debug, Clone, PartialEq, Eq)]
struct User {
    id: u32,
    name: String,
    email: String,
}

impl Hash for User {
    fn hash<H: Hasher>(&self, state: &mut H) {
        self.id.hash(state);  // Hash only the unique identifier
    }
}

// Good: Use HashMap for fast key-value lookups
fn demonstrate_hashmap() {
    let mut user_cache: HashMap<u32, User> = HashMap::new();
    
    // Insert users
    let users = vec![
        User { id: 1, name: "Alice".to_string(), email: "alice@example.com".to_string() },
        User { id: 2, name: "Bob".to_string(), email: "bob@example.com".to_string() },
        User { id: 3, name: "Charlie".to_string(), email: "charlie@example.com".to_string() },
    ];
    
    for user in users {
        user_cache.insert(user.id, user);
    }
    
    // Fast O(1) lookup
    if let Some(user) = user_cache.get(&2) {
        println!("Found user: {:?}", user);
    }
    
    // Update user
    user_cache.entry(1).and_modify(|user| {
        user.email = "alice.new@example.com".to_string();
    });
    
    println!("User cache size: {}", user_cache.len());
}

// Good: Use BTreeMap when you need sorted keys
fn demonstrate_btreemap() {
    let mut scores: BTreeMap<String, u32> = BTreeMap::new();
    
    scores.insert("Alice".to_string(), 95);
    scores.insert("Bob".to_string(), 87);
    scores.insert("Charlie".to_string(), 92);
    scores.insert("Diana".to_string(), 98);
    
    println!("Scores in alphabetical order:");
    for (name, score) in &scores {
        println!("  {}: {}", name, score);
    }
    
    // Range queries
    let range: Vec<_> = scores.range("B".to_string().."D".to_string()).collect();
    println!("Names starting with B or C: {:?}", range);
}

// Good: Use HashSet for fast membership testing
fn demonstrate_hashset() {
    let mut active_users: HashSet<u32> = HashSet::new();
    active_users.extend([1, 3, 5, 7, 9]);
    
    let admin_users: HashSet<u32> = [1, 2, 3].into_iter().collect();
    
    // Fast membership testing
    println!("User 5 is active: {}", active_users.contains(&5));
    
    // Set operations
    let active_admins: HashSet<_> = active_users.intersection(&admin_users).collect();
    println!("Active admin users: {:?}", active_admins);
    
    let all_users: HashSet<_> = active_users.union(&admin_users).collect();
    println!("All users (union): {:?}", all_users);
}

// Good: Use VecDeque for efficient operations at both ends
fn demonstrate_vecdeque() {
    let mut task_queue: VecDeque<String> = VecDeque::new();
    
    // Add tasks to the back
    task_queue.push_back("Process data".to_string());
    task_queue.push_back("Send email".to_string());
    task_queue.push_back("Update database".to_string());
    
    // Add high-priority task to the front
    task_queue.push_front("Handle emergency".to_string());
    
    println!("Task queue: {:?}", task_queue);
    
    // Process tasks from the front
    while let Some(task) = task_queue.pop_front() {
        println!("Processing: {}", task);
    }
}

// Good: Use Vec for indexed access and when order matters
fn demonstrate_vec() {
    let mut timeline: Vec<(String, u64)> = Vec::new();
    
    // Events with timestamps
    timeline.push(("User login".to_string(), 1000));
    timeline.push(("File uploaded".to_string(), 1005));
    timeline.push(("Email sent".to_string(), 1010));
    
    // Sort by timestamp
    timeline.sort_by_key(|(_, timestamp)| *timestamp);
    
    // Access by index
    if let Some((event, time)) = timeline.get(1) {
        println!("Second event: {} at {}", event, time);
    }
    
    // Efficient iteration
    for (event, timestamp) in &timeline {
        println!("Event: {} at time {}", event, timestamp);
    }
}

// Performance comparison example
fn compare_lookup_performance() {
    let size = 100_000;
    
    // Prepare data
    let data: Vec<u32> = (0..size).collect();
    let mut hash_map: HashMap<u32, u32> = HashMap::new();
    let mut btree_map: BTreeMap<u32, u32> = BTreeMap::new();
    
    for &value in &data {
        hash_map.insert(value, value * 2);
        btree_map.insert(value, value * 2);
    }
    
    let search_keys: Vec<u32> = (0..1000).map(|_| fastrand::u32(0..size)).collect();
    
    // Time HashMap lookups
    let start = std::time::Instant::now();
    let mut hash_found = 0;
    for &key in &search_keys {
        if hash_map.contains_key(&key) {
            hash_found += 1;
        }
    }
    let hash_duration = start.elapsed();
    
    // Time BTreeMap lookups
    let start = std::time::Instant::now();
    let mut btree_found = 0;
    for &key in &search_keys {
        if btree_map.contains_key(&key) {
            btree_found += 1;
        }
    }
    let btree_duration = start.elapsed();
    
    println!("Lookup performance comparison:");
    println!("  HashMap: {} found in {:?}", hash_found, hash_duration);
    println!("  BTreeMap: {} found in {:?}", btree_found, btree_duration);
}

fn main() {
    println!("=== HashMap Demo ===");
    demonstrate_hashmap();
    
    println!("\n=== BTreeMap Demo ===");
    demonstrate_btreemap();
    
    println!("\n=== HashSet Demo ===");
    demonstrate_hashset();
    
    println!("\n=== VecDeque Demo ===");
    demonstrate_vecdeque();
    
    println!("\n=== Vec Demo ===");
    demonstrate_vec();
    
    println!("\n=== Performance Comparison ===");
    compare_lookup_performance();
}
```

Choose collections based on your access patterns: Vec for sequential  
access, HashMap for fast lookups, BTreeMap for sorted data, HashSet for  
membership testing, and VecDeque for queue-like operations.  

## Write efficient and safe concurrent code

Use Rust's concurrency primitives to write thread-safe code that avoids  
data races and common concurrency bugs.  

```rust
use std::sync::{Arc, Mutex, RwLock, mpsc, Barrier};
use std::thread;
use std::time::Duration;
use std::collections::HashMap;

// Good: Use Arc<Mutex<T>> for shared mutable state
fn demonstrate_shared_counter() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    
    for i in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            for _ in 0..100 {
                let mut num = counter.lock().unwrap();
                *num += 1;
            }
            println!("Thread {} finished", i);
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Final counter value: {}", *counter.lock().unwrap());
}

// Good: Use RwLock for read-heavy workloads
fn demonstrate_read_write_lock() {
    let data = Arc::new(RwLock::new(HashMap::new()));
    let mut handles = vec![];
    
    // Writer thread
    let data_writer = Arc::clone(&data);
    let writer_handle = thread::spawn(move || {
        for i in 0..100 {
            {
                let mut map = data_writer.write().unwrap();
                map.insert(format!("key_{}", i), i);
            }
            thread::sleep(Duration::from_millis(10));
        }
        println!("Writer finished");
    });
    
    // Multiple reader threads
    for thread_id in 0..5 {
        let data_reader = Arc::clone(&data);
        let handle = thread::spawn(move || {
            for _ in 0..50 {
                {
                    let map = data_reader.read().unwrap();
                    let count = map.len();
                    if count > 0 {
                        println!("Thread {} sees {} entries", thread_id, count);
                    }
                }
                thread::sleep(Duration::from_millis(20));
            }
            println!("Reader {} finished", thread_id);
        });
        handles.push(handle);
    }
    
    writer_handle.join().unwrap();
    for handle in handles {
        handle.join().unwrap();
    }
    
    let final_map = data.read().unwrap();
    println!("Final map size: {}", final_map.len());
}

// Good: Use channels for communication between threads
fn demonstrate_channels() {
    let (tx, rx) = mpsc::channel();
    
    // Producer threads
    let mut producers = vec![];
    for producer_id in 0..3 {
        let tx = tx.clone();
        let handle = thread::spawn(move || {
            for i in 0..5 {
                let message = format!("Message {} from producer {}", i, producer_id);
                tx.send(message).unwrap();
                thread::sleep(Duration::from_millis(100));
            }
            println!("Producer {} finished", producer_id);
        });
        producers.push(handle);
    }
    
    // Drop the original sender to close the channel when all producers are done
    drop(tx);
    
    // Consumer thread
    let consumer = thread::spawn(move || {
        let mut received_count = 0;
        while let Ok(message) = rx.recv() {
            println!("Received: {}", message);
            received_count += 1;
        }
        println!("Consumer received {} messages", received_count);
    });
    
    for handle in producers {
        handle.join().unwrap();
    }
    consumer.join().unwrap();
}

// Good: Use barriers for synchronization
fn demonstrate_barrier() {
    let barrier = Arc::new(Barrier::new(4));
    let mut handles = vec![];
    
    for i in 0..4 {
        let barrier = Arc::clone(&barrier);
        let handle = thread::spawn(move || {
            // Do some work
            let work_time = Duration::from_millis(100 + i * 50);
            thread::sleep(work_time);
            println!("Thread {} finished work phase", i);
            
            // Wait for all threads to reach this point
            barrier.wait();
            
            // All threads continue together
            println!("Thread {} starting next phase", i);
            thread::sleep(Duration::from_millis(50));
            println!("Thread {} completed", i);
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
}

// Good: Parallel processing with scoped threads
fn demonstrate_parallel_processing() {
    let data = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    let chunk_size = 3;
    let results = Arc::new(Mutex::new(Vec::new()));
    
    thread::scope(|s| {
        for chunk in data.chunks(chunk_size) {
            let results = Arc::clone(&results);
            s.spawn(move || {
                // Process chunk
                let sum: i32 = chunk.iter().sum();
                let processed = format!("Chunk {:?} sum: {}", chunk, sum);
                
                // Store result
                results.lock().unwrap().push(processed);
                
                println!("Processed chunk: {:?}", chunk);
            });
        }
    });
    
    let final_results = results.lock().unwrap();
    println!("All results:");
    for result in final_results.iter() {
        println!("  {}", result);
    }
}

// Pattern: Producer-Consumer with bounded channel
fn demonstrate_producer_consumer() {
    let (tx, rx) = mpsc::sync_channel(5);  // Bounded channel
    
    // Producer
    let producer = thread::spawn(move || {
        for i in 1..=20 {
            println!("Producing item {}", i);
            tx.send(i).unwrap();
            thread::sleep(Duration::from_millis(50));
        }
        println!("Producer finished");
    });
    
    // Consumer
    let consumer = thread::spawn(move || {
        while let Ok(item) = rx.recv() {
            println!("  Consuming item {}", item);
            thread::sleep(Duration::from_millis(100));  // Slower than producer
        }
        println!("Consumer finished");
    });
    
    producer.join().unwrap();
    consumer.join().unwrap();
}

fn main() {
    println!("=== Shared Counter Demo ===");
    demonstrate_shared_counter();
    
    println!("\n=== Read-Write Lock Demo ===");
    demonstrate_read_write_lock();
    
    println!("\n=== Channels Demo ===");
    demonstrate_channels();
    
    println!("\n=== Barrier Demo ===");
    demonstrate_barrier();
    
    println!("\n=== Parallel Processing Demo ===");
    demonstrate_parallel_processing();
    
    println!("\n=== Producer-Consumer Demo ===");
    demonstrate_producer_consumer();
}
```

Rust's ownership system prevents data races at compile time. Use Arc for  
shared ownership, Mutex/RwLock for synchronization, channels for  
communication, and barriers for coordination between threads.  

## Implement Debug and Display traits

Provide useful representations of your types for debugging and user-facing  
output by implementing Debug and Display traits appropriately.  

```rust
use std::fmt;

#[derive(Clone, PartialEq)]
struct User {
    id: u64,
    username: String,
    email: String,
    is_active: bool,
}

// Good: Implement Debug for development and debugging
impl fmt::Debug for User {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        f.debug_struct("User")
            .field("id", &self.id)
            .field("username", &self.username)
            .field("email", &self.email)
            .field("is_active", &self.is_active)
            .finish()
    }
}

// Good: Implement Display for user-facing output
impl fmt::Display for User {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "{} ({})", self.username, self.email)
    }
}

#[derive(Debug, Clone)]
struct BankAccount {
    account_number: String,
    balance: f64,
    currency: String,
}

// Good: Custom Display that's appropriate for the domain
impl fmt::Display for BankAccount {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "Account {}: {:.2} {}", 
               self.account_number, self.balance, self.currency)
    }
}

// Good: Enum with meaningful Debug and Display
#[derive(Debug, Clone)]
enum TransactionStatus {
    Pending,
    Completed { timestamp: u64 },
    Failed { reason: String },
    Cancelled,
}

impl fmt::Display for TransactionStatus {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            TransactionStatus::Pending => write!(f, "Pending"),
            TransactionStatus::Completed { timestamp } => {
                write!(f, "Completed at {}", timestamp)
            }
            TransactionStatus::Failed { reason } => {
                write!(f, "Failed: {}", reason)
            }
            TransactionStatus::Cancelled => write!(f, "Cancelled"),
        }
    }
}

// Good: Complex type with structured debug output
#[derive(Debug)]
struct Transaction {
    id: String,
    from_account: BankAccount,
    to_account: BankAccount,
    amount: f64,
    status: TransactionStatus,
}

impl fmt::Display for Transaction {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "Transaction {}: {:.2} {} from {} to {} - {}",
               self.id,
               self.amount,
               self.from_account.currency,
               self.from_account.account_number,
               self.to_account.account_number,
               self.status
        )
    }
}

// Good: Custom error type with helpful Display
#[derive(Debug)]
enum ValidationError {
    EmptyField(String),
    InvalidFormat { field: String, expected: String },
    OutOfRange { field: String, min: i32, max: i32, actual: i32 },
}

impl fmt::Display for ValidationError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            ValidationError::EmptyField(field) => {
                write!(f, "Field '{}' cannot be empty", field)
            }
            ValidationError::InvalidFormat { field, expected } => {
                write!(f, "Field '{}' has invalid format, expected: {}", field, expected)
            }
            ValidationError::OutOfRange { field, min, max, actual } => {
                write!(f, "Field '{}' value {} is out of range [{}, {}]", 
                       field, actual, min, max)
            }
        }
    }
}

impl std::error::Error for ValidationError {}

// Good: Pretty printing for collections
struct Report {
    title: String,
    users: Vec<User>,
    accounts: Vec<BankAccount>,
}

impl fmt::Debug for Report {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        f.debug_struct("Report")
            .field("title", &self.title)
            .field("user_count", &self.users.len())
            .field("account_count", &self.accounts.len())
            .finish()
    }
}

impl fmt::Display for Report {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        writeln!(f, "=== {} ===", self.title)?;
        writeln!(f, "Users ({}):", self.users.len())?;
        for user in &self.users {
            writeln!(f, "  - {}", user)?;
        }
        writeln!(f, "\\nAccounts ({}):", self.accounts.len())?;
        for account in &self.accounts {
            writeln!(f, "  - {}", account)?;
        }
        Ok(())
    }
}

// Good: Conditional formatting based on formatter options
struct DataPoint {
    x: f64,
    y: f64,
    label: Option<String>,
}

impl fmt::Debug for DataPoint {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        let mut debug_struct = f.debug_struct("DataPoint");
        debug_struct.field("x", &self.x);
        debug_struct.field("y", &self.y);
        if let Some(ref label) = self.label {
            debug_struct.field("label", label);
        }
        debug_struct.finish()
    }
}

impl fmt::Display for DataPoint {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        if let Some(ref label) = self.label {
            write!(f, "{}: ({:.2}, {:.2})", label, self.x, self.y)
        } else {
            write!(f, "({:.2}, {:.2})", self.x, self.y)
        }
    }
}

fn main() {
    let user = User {
        id: 1,
        username: "alice_smith".to_string(),
        email: "alice@example.com".to_string(),
        is_active: true,
    };
    
    // Debug output - for developers
    println!("Debug: {:?}", user);
    
    // Display output - for users
    println!("Display: {}", user);
    
    let account1 = BankAccount {
        account_number: "ACC-001".to_string(),
        balance: 1250.75,
        currency: "USD".to_string(),
    };
    
    let account2 = BankAccount {
        account_number: "ACC-002".to_string(),
        balance: 890.25,
        currency: "USD".to_string(),
    };
    
    println!("\\nAccount info: {}", account1);
    
    let transaction = Transaction {
        id: "TXN-12345".to_string(),
        from_account: account1.clone(),
        to_account: account2.clone(),
        amount: 100.0,
        status: TransactionStatus::Completed { timestamp: 1634567890 },
    };
    
    println!("\\nTransaction: {}", transaction);
    println!("Debug transaction: {:?}", transaction);
    
    // Error formatting
    let errors = vec![
        ValidationError::EmptyField("username".to_string()),
        ValidationError::InvalidFormat {
            field: "email".to_string(),
            expected: "user@domain.com".to_string(),
        },
        ValidationError::OutOfRange {
            field: "age".to_string(),
            min: 0,
            max: 120,
            actual: 150,
        },
    ];
    
    println!("\\nValidation errors:");
    for error in errors {
        println!("  Error: {}", error);
        println!("  Debug: {:?}", error);
    }
    
    // Report with pretty printing
    let report = Report {
        title: "Monthly Report".to_string(),
        users: vec![user],
        accounts: vec![account1, account2],
    };
    
    println!("\\n{}", report);
    println!("Report debug: {:?}", report);
    
    // Data points with conditional formatting
    let points = vec![
        DataPoint { x: 1.0, y: 2.0, label: Some("Start".to_string()) },
        DataPoint { x: 3.0, y: 4.0, label: None },
        DataPoint { x: 5.0, y: 6.0, label: Some("End".to_string()) },
    ];
    
    println!("\\nData points:");
    for point in points {
        println!("  Display: {}", point);
        println!("  Debug: {:?}", point);
    }
}
```

Good Debug implementations help with development and troubleshooting,  
while Display implementations provide user-friendly output. Consider  
your audience and use case when implementing these traits.  

## Use lifetimes correctly

Understand and use lifetimes to express borrowing relationships clearly  
while avoiding unnecessary lifetime annotations.  

```rust
// Good: Simple lifetime annotation when needed
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

// Good: Struct with lifetime parameters
#[derive(Debug)]
struct TextAnalyzer<'a> {
    text: &'a str,
    word_count: usize,
}

impl<'a> TextAnalyzer<'a> {
    fn new(text: &'a str) -> Self {
        let word_count = text.split_whitespace().count();
        Self { text, word_count }
    }
    
    fn get_words(&self) -> Vec<&'a str> {
        self.text.split_whitespace().collect()
    }
    
    fn find_longest_word(&self) -> Option<&'a str> {
        self.text
            .split_whitespace()
            .max_by_key(|word| word.len())
    }
    
    fn word_count(&self) -> usize {
        self.word_count
    }
}

// Good: Multiple lifetime parameters when relationships are different
fn compare_and_select<'a, 'b>(
    primary: &'a str,
    fallback: &'b str,
    use_primary: bool,
) -> &'a str where 'b: 'a {  // 'b outlives 'a
    if use_primary && !primary.is_empty() {
        primary
    } else {
        fallback  // This requires 'b: 'a bound
    }
}

// Alternative without lifetime bound - different return type pattern
fn compare_and_select_owned(
    primary: &str,
    fallback: &str,
    use_primary: bool,
) -> String {
    if use_primary && !primary.is_empty() {
        primary.to_string()
    } else {
        fallback.to_string()
    }
}

// Good: Lifetime elision - compiler can infer these
fn first_word(s: &str) -> &str {  // Actually: fn first_word<'a>(s: &'a str) -> &'a str
    s.split_whitespace().next().unwrap_or("")
}

fn process_line(line: &str) -> (&str, usize) {  // Returns borrow from input
    (line.trim(), line.len())
}

// Good: Static lifetime for constants
const GREETING: &'static str = "Hello, World!";

fn get_default_message() -> &'static str {
    "Default message"
}

// Good: Iterator with lifetimes
struct WordIterator<'a> {
    text: &'a str,
    position: usize,
}

impl<'a> WordIterator<'a> {
    fn new(text: &'a str) -> Self {
        Self { text, position: 0 }
    }
}

impl<'a> Iterator for WordIterator<'a> {
    type Item = &'a str;
    
    fn next(&mut self) -> Option<Self::Item> {
        let remaining = &self.text[self.position..];
        
        if remaining.is_empty() {
            return None;
        }
        
        // Skip whitespace
        let start = remaining.find(|c: char| !c.is_whitespace())?;
        let word_start = self.position + start;
        
        // Find end of word
        let word_remaining = &self.text[word_start..];
        let word_end = word_remaining
            .find(|c: char| c.is_whitespace())
            .map(|i| word_start + i)
            .unwrap_or(self.text.len());
        
        self.position = word_end;
        
        Some(&self.text[word_start..word_end])
    }
}

// Good: Configuration holder with borrowed data
#[derive(Debug)]
struct Config<'a> {
    app_name: &'a str,
    version: &'a str,
    features: Vec<&'a str>,
}

impl<'a> Config<'a> {
    fn new(app_name: &'a str, version: &'a str) -> Self {
        Self {
            app_name,
            version,
            features: Vec::new(),
        }
    }
    
    fn add_feature(&mut self, feature: &'a str) {
        self.features.push(feature);
    }
    
    fn display_info(&self) -> String {
        format!(
            "{} v{} with features: [{}]",
            self.app_name,
            self.version,
            self.features.join(", ")
        )
    }
}

// Good: Working with slices and lifetime bounds
fn find_common_prefix<'a>(strings: &[&'a str]) -> &'a str {
    if strings.is_empty() {
        return "";
    }
    
    let first = strings[0];
    let mut common_len = first.len();
    
    for &string in &strings[1..] {
        common_len = first.chars()
            .zip(string.chars())
            .take_while(|(a, b)| a == b)
            .count()
            .min(common_len);
    }
    
    &first[..common_len]
}

// Avoid: Unnecessary lifetime annotations (these would be inferred)
// fn bad_example<'a>(s: &'a str) -> usize {  // Lifetime not needed for usize return
//     s.len()
// }

fn main() {
    // Basic lifetime usage
    let string1 = "Hello, world!";
    let string2 = "Hi!";
    let result = longest(string1, string2);
    println!("Longest string: '{}'", result);
    
    // Text analyzer
    let text = "The quick brown fox jumps over the lazy dog";
    let analyzer = TextAnalyzer::new(text);
    
    println!("Text: '{}'", analyzer.text);
    println!("Word count: {}", analyzer.word_count());
    
    if let Some(longest_word) = analyzer.find_longest_word() {
        println!("Longest word: '{}'", longest_word);
    }
    
    let words = analyzer.get_words();
    println!("First few words: {:?}", &words[..3]);
    
    // String processing
    let line = "  Hello, Rust!  ";
    let (trimmed, original_len) = process_line(line);
    println!("Processed: '{}' (original length: {})", trimmed, original_len);
    
    // Custom iterator
    let sample_text = "Rust is fast and safe";
    let mut word_iter = WordIterator::new(sample_text);
    
    println!("Words from iterator:");
    while let Some(word) = word_iter.next() {
        println!("  '{}'", word);
    }
    
    // Configuration with borrowed strings
    let app_name = "MyApp";
    let version = "1.0.0";
    let feature1 = "authentication";
    let feature2 = "logging";
    
    let mut config = Config::new(app_name, version);
    config.add_feature(feature1);
    config.add_feature(feature2);
    
    println!("Config: {}", config.display_info());
    
    // Common prefix finding
    let strings = vec!["testing", "test", "tester", "tests"];
    let prefix = find_common_prefix(&strings);
    println!("Common prefix of {:?}: '{}'", strings, prefix);
    
    // Static lifetime
    println!("Static greeting: {}", GREETING);
    println!("Default message: {}", get_default_message());
}
```

Lifetimes express borrowing relationships explicitly when the compiler  
can't infer them. Use them judiciously - many cases work with lifetime  
elision, and over-annotation makes code harder to read.  

## Optimize performance thoughtfully

Write clear code first, then optimize based on actual performance  
measurements and profiling data.  

```rust
use std::collections::HashMap;
use std::time::Instant;

// Good: Optimize hot paths after profiling
struct ImageProcessor {
    width: usize,
    height: usize,
    pixels: Vec<u8>,
}

impl ImageProcessor {
    fn new(width: usize, height: usize) -> Self {
        Self {
            width,
            height,
            pixels: vec![0; width * height * 3],  // RGB
        }
    }
    
    // Optimized: Avoid bounds checking in hot loop
    fn apply_brightness_fast(&mut self, factor: f32) {
        // Pre-calculate to avoid repeated computation
        let len = self.pixels.len();
        
        // Use chunks for better cache locality and vectorization
        for chunk in self.pixels.chunks_exact_mut(3) {
            chunk[0] = ((chunk[0] as f32 * factor).min(255.0) as u8);
            chunk[1] = ((chunk[1] as f32 * factor).min(255.0) as u8);
            chunk[2] = ((chunk[2] as f32 * factor).min(255.0) as u8);
        }
    }
    
    // Alternative: Use unsafe for maximum performance (measure first!)
    fn apply_brightness_unsafe(&mut self, factor: f32) {
        let len = self.pixels.len();
        let ptr = self.pixels.as_mut_ptr();
        
        unsafe {
            for i in (0..len).step_by(3) {
                let r = (*ptr.add(i) as f32 * factor).min(255.0) as u8;
                let g = (*ptr.add(i + 1) as f32 * factor).min(255.0) as u8;
                let b = (*ptr.add(i + 2) as f32 * factor).min(255.0) as u8;
                
                *ptr.add(i) = r;
                *ptr.add(i + 1) = g;
                *ptr.add(i + 2) = b;
            }
        }
    }
}

// Good: Use const for compile-time computation
const LOOKUP_TABLE: [f32; 256] = {
    let mut table = [0.0; 256];
    let mut i = 0;
    while i < 256 {
        table[i] = (i as f32 / 255.0).powf(2.2);  // Gamma correction
        i += 1;
    }
    table
};

// Good: Pool expensive-to-create objects
struct StringPool {
    pool: Vec<String>,
    next_index: usize,
}

impl StringPool {
    fn new() -> Self {
        Self {
            pool: Vec::new(),
            next_index: 0,
        }
    }
    
    fn get_string(&mut self) -> String {
        if self.next_index < self.pool.len() {
            let mut string = std::mem::take(&mut self.pool[self.next_index]);
            string.clear();
            self.next_index += 1;
            string
        } else {
            String::new()
        }
    }
    
    fn return_string(&mut self, mut string: String) {
        if self.next_index > 0 {
            self.next_index -= 1;
            string.clear();
            self.pool[self.next_index] = string;
        } else {
            string.clear();
            self.pool.push(string);
        }
    }
}

// Good: Use appropriate data structures for the use case
fn benchmark_data_structures() {
    const SIZE: usize = 100_000;
    const LOOKUPS: usize = 10_000;
    
    // Prepare test data
    let keys: Vec<u32> = (0..SIZE as u32).collect();
    let lookup_keys: Vec<u32> = (0..LOOKUPS).map(|_| fastrand::u32(0..SIZE as u32)).collect();
    
    // HashMap benchmark
    let start = Instant::now();
    let mut hash_map: HashMap<u32, u32> = HashMap::with_capacity(SIZE);
    for &key in &keys {
        hash_map.insert(key, key * 2);
    }
    let hash_build_time = start.elapsed();
    
    let start = Instant::now();
    let mut hash_found = 0;
    for &key in &lookup_keys {
        if hash_map.contains_key(&key) {
            hash_found += 1;
        }
    }
    let hash_lookup_time = start.elapsed();
    
    // Vec benchmark (for comparison)
    let start = Instant::now();
    let vec_data: Vec<(u32, u32)> = keys.iter().map(|&k| (k, k * 2)).collect();
    let vec_build_time = start.elapsed();
    
    let start = Instant::now();
    let mut vec_found = 0;
    for &key in &lookup_keys {
        if vec_data.binary_search_by_key(&key, |(k, _)| *k).is_ok() {
            vec_found += 1;
        }
    }
    let vec_lookup_time = start.elapsed();
    
    println!("Performance comparison for {} items, {} lookups:", SIZE, LOOKUPS);
    println!("HashMap: build {:?}, lookup {:?}, found {}", 
             hash_build_time, hash_lookup_time, hash_found);
    println!("Vec:     build {:?}, lookup {:?}, found {}", 
             vec_build_time, vec_lookup_time, vec_found);
}

// Good: Memory-efficient string processing
fn process_large_text_efficiently(text: &str) -> HashMap<String, usize> {
    let mut word_counts: HashMap<String, usize> = HashMap::new();
    
    // Process without allocating for each word
    for line in text.lines() {
        for word in line.split_whitespace() {
            // Only allocate when inserting new entries
            *word_counts.entry(word.to_lowercase()).or_insert(0) += 1;
        }
    }
    
    word_counts
}

// Good: Batch processing for better cache performance
fn batch_process_numbers(numbers: &[i32], batch_size: usize) -> Vec<i32> {
    let mut results = Vec::with_capacity(numbers.len());
    
    for batch in numbers.chunks(batch_size) {
        // Process each batch together for better cache locality
        let mut batch_results: Vec<i32> = batch
            .iter()
            .map(|&n| {
                // Simulate expensive computation
                let mut result = n;
                for _ in 0..10 {
                    result = result.wrapping_mul(17).wrapping_add(1);
                }
                result
            })
            .collect();
        
        results.append(&mut batch_results);
    }
    
    results
}

// Good: Use iterators for lazy evaluation
fn find_prime_numbers(limit: usize) -> Vec<usize> {
    (2..=limit)
        .filter(|&n| {
            (2..=(n as f64).sqrt() as usize).all(|i| n % i != 0)
        })
        .collect()
}

// Optimized version with early termination
fn find_prime_numbers_optimized(limit: usize) -> Vec<usize> {
    if limit < 2 {
        return Vec::new();
    }
    
    let mut primes = Vec::new();
    let mut is_prime = vec![true; limit + 1];
    is_prime[0] = false;
    is_prime[1] = false;
    
    for i in 2..=limit {
        if is_prime[i] {
            primes.push(i);
            
            // Mark multiples as non-prime
            let mut multiple = i * i;
            while multiple <= limit {
                is_prime[multiple] = false;
                multiple += i;
            }
        }
    }
    
    primes
}

fn measure_performance<F, R>(name: &str, f: F) -> R
where
    F: FnOnce() -> R,
{
    let start = Instant::now();
    let result = f();
    let duration = start.elapsed();
    println!("{}: {:?}", name, duration);
    result
}

fn main() {
    // Image processing benchmark
    let mut image = ImageProcessor::new(1920, 1080);
    
    measure_performance("Image brightness (safe)", || {
        image.apply_brightness_fast(1.2);
    });
    
    measure_performance("Image brightness (unsafe)", || {
        image.apply_brightness_unsafe(1.2);
    });
    
    // Data structure comparison
    benchmark_data_structures();
    
    // String pool usage
    let mut pool = StringPool::new();
    measure_performance("String operations with pool", || {
        for _ in 0..1000 {
            let mut s = pool.get_string();
            s.push_str("Hello, ");
            s.push_str("world!");
            pool.return_string(s);
        }
    });
    
    // Text processing
    let sample_text = "The quick brown fox jumps over the lazy dog. ".repeat(1000);
    let word_counts = measure_performance("Text processing", || {
        process_large_text_efficiently(&sample_text)
    });
    println!("Found {} unique words", word_counts.len());
    
    // Batch processing
    let numbers: Vec<i32> = (1..=10000).collect();
    let _results = measure_performance("Batch processing", || {
        batch_process_numbers(&numbers, 1000)
    });
    
    // Prime number comparison
    const PRIME_LIMIT: usize = 10000;
    
    let primes1 = measure_performance("Prime finding (simple)", || {
        find_prime_numbers(PRIME_LIMIT)
    });
    
    let primes2 = measure_performance("Prime finding (optimized)", || {
        find_prime_numbers_optimized(PRIME_LIMIT)
    });
    
    println!("Found {} primes (simple), {} primes (optimized)", 
             primes1.len(), primes2.len());
    
    // Demonstrate lookup table usage
    measure_performance("Gamma correction with lookup", || {
        for i in 0..256 {
            let _gamma_corrected = LOOKUP_TABLE[i];
        }
    });
}
```

Performance optimization should be driven by measurement, not assumptions.  
Profile your code, identify bottlenecks, and optimize thoughtfully while  
maintaining code clarity and safety.  

## Write comprehensive tests

Write clear, maintainable tests that verify both happy paths and edge  
cases to ensure code reliability and facilitate refactoring.  

```rust
#[derive(Debug, PartialEq)]
pub struct Calculator {
    history: Vec<f64>,
}

impl Calculator {
    pub fn new() -> Self {
        Self {
            history: Vec::new(),
        }
    }
    
    pub fn add(&mut self, a: f64, b: f64) -> f64 {
        let result = a + b;
        self.history.push(result);
        result
    }
    
    pub fn divide(&mut self, a: f64, b: f64) -> Result<f64, String> {
        if b == 0.0 {
            Err("Division by zero".to_string())
        } else {
            let result = a / b;
            self.history.push(result);
            Ok(result)
        }
    }
    
    pub fn history(&self) -> &[f64] {
        &self.history
    }
    
    pub fn clear_history(&mut self) {
        self.history.clear();
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    
    // Good: Test basic functionality
    #[test]
    fn test_calculator_new() {
        let calc = Calculator::new();
        assert_eq!(calc.history().len(), 0);
    }
    
    #[test]
    fn test_addition() {
        let mut calc = Calculator::new();
        let result = calc.add(2.0, 3.0);
        assert_eq!(result, 5.0);
        assert_eq!(calc.history(), &[5.0]);
    }
    
    // Good: Test edge cases
    #[test]
    fn test_addition_with_negatives() {
        let mut calc = Calculator::new();
        assert_eq!(calc.add(-2.0, 3.0), 1.0);
        assert_eq!(calc.add(-2.0, -3.0), -5.0);
    }
    
    #[test]
    fn test_addition_with_zero() {
        let mut calc = Calculator::new();
        assert_eq!(calc.add(0.0, 0.0), 0.0);
        assert_eq!(calc.add(5.0, 0.0), 5.0);
    }
    
    // Good: Test error conditions
    #[test]
    fn test_division_by_zero() {
        let mut calc = Calculator::new();
        let result = calc.divide(5.0, 0.0);
        assert!(result.is_err());
        assert_eq!(result.unwrap_err(), "Division by zero");
        assert_eq!(calc.history().len(), 0);  // No result should be stored
    }
    
    #[test]
    fn test_successful_division() {
        let mut calc = Calculator::new();
        let result = calc.divide(6.0, 2.0).unwrap();
        assert_eq!(result, 3.0);
        assert_eq!(calc.history(), &[3.0]);
    }
    
    // Good: Test state management
    #[test]
    fn test_history_accumulation() {
        let mut calc = Calculator::new();
        calc.add(1.0, 2.0);
        calc.add(3.0, 4.0);
        calc.divide(10.0, 2.0).unwrap();
        
        assert_eq!(calc.history(), &[3.0, 7.0, 5.0]);
    }
    
    #[test]
    fn test_clear_history() {
        let mut calc = Calculator::new();
        calc.add(1.0, 2.0);
        calc.add(3.0, 4.0);
        assert_eq!(calc.history().len(), 2);
        
        calc.clear_history();
        assert_eq!(calc.history().len(), 0);
    }
    
    // Good: Property-based testing approach
    #[test]
    fn test_addition_properties() {
        let mut calc = Calculator::new();
        
        // Commutative property: a + b = b + a
        assert_eq!(calc.add(3.0, 5.0), calc.add(5.0, 3.0));
        
        // Identity property: a + 0 = a
        let a = 42.0;
        calc.clear_history();
        assert_eq!(calc.add(a, 0.0), a);
    }
    
    // Good: Test with realistic data
    #[test]
    fn test_calculator_workflow() {
        let mut calc = Calculator::new();
        
        // Simulate a realistic calculation workflow
        let tax_rate = 0.08;
        let price = 100.0;
        let tax = calc.add(price * tax_rate, 0.0);  // Calculate tax
        let total = calc.add(price, tax);           // Add tax to price
        
        assert_eq!(tax, 8.0);
        assert_eq!(total, 108.0);
        assert_eq!(calc.history().len(), 2);
    }
}

// Good: Integration test example (in a separate file: tests/integration_test.rs)
// This would normally be in tests/integration_test.rs
mod integration_tests {
    use super::*;
    
    #[test]
    fn test_calculator_complete_workflow() {
        let mut calc = Calculator::new();
        
        // Test a complete calculation workflow
        let initial_value = 100.0;
        let operations = vec![
            (initial_value, 25.0),
            (50.0, 10.0),
            (75.0, 5.0),
        ];
        
        let mut expected_history = Vec::new();
        for (a, b) in operations {
            let result = calc.add(a, b);
            expected_history.push(result);
        }
        
        assert_eq!(calc.history(), expected_history.as_slice());
        
        // Test error handling doesn't affect state
        let history_before = calc.history().to_vec();
        let _ = calc.divide(10.0, 0.0);  // This should fail
        assert_eq!(calc.history(), history_before.as_slice());
    }
}

// Good: Test helper functions
#[cfg(test)]
mod test_helpers {
    use super::*;
    
    pub fn create_calculator_with_history(values: &[f64]) -> Calculator {
        let mut calc = Calculator::new();
        for &value in values {
            calc.add(value, 0.0);  // Add to history
        }
        calc
    }
    
    #[test]
    fn test_helper_function() {
        let calc = create_calculator_with_history(&[1.0, 2.0, 3.0]);
        assert_eq!(calc.history(), &[1.0, 2.0, 3.0]);
    }
}

fn main() {
    let mut calc = Calculator::new();
    
    println!("Basic calculations:");
    println!("2 + 3 = {}", calc.add(2.0, 3.0));
    println!("10 / 2 = {:?}", calc.divide(10.0, 2.0));
    println!("5 / 0 = {:?}", calc.divide(5.0, 0.0));
    
    println!("History: {:?}", calc.history());
}
```

Good tests serve as documentation, catch regressions, and enable confident  
refactoring. Test both happy paths and edge cases, use descriptive names,  
and keep tests simple and focused.  

## Handle configuration and environment

Manage application configuration and environment variables in a structured,  
type-safe way with appropriate defaults and validation.  

```rust
use std::env;
use std::fs;
use std::path::Path;
use std::str::FromStr;

#[derive(Debug, Clone)]
pub struct AppConfig {
    pub server_host: String,
    pub server_port: u16,
    pub database_url: String,
    pub log_level: LogLevel,
    pub max_connections: usize,
    pub timeout_seconds: u64,
    pub debug_mode: bool,
}

#[derive(Debug, Clone, PartialEq)]
pub enum LogLevel {
    Error,
    Warn,
    Info,
    Debug,
    Trace,
}

impl FromStr for LogLevel {
    type Err = String;
    
    fn from_str(s: &str) -> Result<Self, Self::Err> {
        match s.to_lowercase().as_str() {
            "error" => Ok(LogLevel::Error),
            "warn" | "warning" => Ok(LogLevel::Warn),
            "info" => Ok(LogLevel::Info),
            "debug" => Ok(LogLevel::Debug),
            "trace" => Ok(LogLevel::Trace),
            _ => Err(format!("Invalid log level: {}", s)),
        }
    }
}

impl Default for AppConfig {
    fn default() -> Self {
        Self {
            server_host: "localhost".to_string(),
            server_port: 8080,
            database_url: "sqlite://database.db".to_string(),
            log_level: LogLevel::Info,
            max_connections: 100,
            timeout_seconds: 30,
            debug_mode: false,
        }
    }
}

#[derive(Debug)]
pub enum ConfigError {
    EnvVarError(String),
    ParseError { key: String, value: String, error: String },
    ValidationError(String),
    FileError(String),
}

impl std::fmt::Display for ConfigError {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self {
            ConfigError::EnvVarError(msg) => write!(f, "Environment variable error: {}", msg),
            ConfigError::ParseError { key, value, error } => {
                write!(f, "Failed to parse '{}' = '{}': {}", key, value, error)
            }
            ConfigError::ValidationError(msg) => write!(f, "Validation error: {}", msg),
            ConfigError::FileError(msg) => write!(f, "File error: {}", msg),
        }
    }
}

impl std::error::Error for ConfigError {}

impl AppConfig {
    // Good: Load configuration from multiple sources with priority
    pub fn load() -> Result<Self, ConfigError> {
        let mut config = Self::default();
        
        // 1. Load from config file if it exists
        if let Ok(file_config) = Self::from_file("config.toml") {
            config = file_config;
        }
        
        // 2. Override with environment variables (higher priority)
        config.load_from_env()?;
        
        // 3. Validate the final configuration
        config.validate()?;
        
        Ok(config)
    }
    
    // Good: Parse from TOML file
    pub fn from_file<P: AsRef<Path>>(path: P) -> Result<Self, ConfigError> {
        let content = fs::read_to_string(path)
            .map_err(|e| ConfigError::FileError(e.to_string()))?;
        
        Self::from_toml(&content)
    }
    
    pub fn from_toml(content: &str) -> Result<Self, ConfigError> {
        let mut config = Self::default();
        
        for line in content.lines() {
            let line = line.trim();
            if line.is_empty() || line.starts_with('#') {
                continue;
            }
            
            if let Some((key, value)) = line.split_once('=') {
                let key = key.trim();
                let value = value.trim().trim_matches('"');
                
                config.set_value(key, value)?;
            }
        }
        
        Ok(config)
    }
    
    // Good: Load from environment variables with prefixes
    pub fn load_from_env(&mut self) -> Result<(), ConfigError> {
        if let Ok(host) = env::var("APP_HOST") {
            self.server_host = host;
        }
        
        if let Ok(port_str) = env::var("APP_PORT") {
            self.server_port = port_str.parse()
                .map_err(|e| ConfigError::ParseError {
                    key: "APP_PORT".to_string(),
                    value: port_str,
                    error: e.to_string(),
                })?;
        }
        
        if let Ok(db_url) = env::var("DATABASE_URL") {
            self.database_url = db_url;
        }
        
        if let Ok(log_level_str) = env::var("LOG_LEVEL") {
            self.log_level = log_level_str.parse()
                .map_err(|e| ConfigError::ParseError {
                    key: "LOG_LEVEL".to_string(),
                    value: log_level_str,
                    error: e,
                })?;
        }
        
        if let Ok(max_conn_str) = env::var("MAX_CONNECTIONS") {
            self.max_connections = max_conn_str.parse()
                .map_err(|e| ConfigError::ParseError {
                    key: "MAX_CONNECTIONS".to_string(),
                    value: max_conn_str,
                    error: e.to_string(),
                })?;
        }
        
        if let Ok(timeout_str) = env::var("TIMEOUT_SECONDS") {
            self.timeout_seconds = timeout_str.parse()
                .map_err(|e| ConfigError::ParseError {
                    key: "TIMEOUT_SECONDS".to_string(),
                    value: timeout_str,
                    error: e.to_string(),
                })?;
        }
        
        if let Ok(debug_str) = env::var("DEBUG") {
            self.debug_mode = debug_str.parse()
                .map_err(|e| ConfigError::ParseError {
                    key: "DEBUG".to_string(),
                    value: debug_str,
                    error: e.to_string(),
                })?;
        }
        
        Ok(())
    }
    
    // Helper method to set individual values
    fn set_value(&mut self, key: &str, value: &str) -> Result<(), ConfigError> {
        match key {
            "host" => self.server_host = value.to_string(),
            "port" => {
                self.server_port = value.parse()
                    .map_err(|e| ConfigError::ParseError {
                        key: key.to_string(),
                        value: value.to_string(),
                        error: e.to_string(),
                    })?;
            }
            "database_url" => self.database_url = value.to_string(),
            "log_level" => {
                self.log_level = value.parse()
                    .map_err(|e| ConfigError::ParseError {
                        key: key.to_string(),
                        value: value.to_string(),
                        error: e,
                    })?;
            }
            "max_connections" => {
                self.max_connections = value.parse()
                    .map_err(|e| ConfigError::ParseError {
                        key: key.to_string(),
                        value: value.to_string(),
                        error: e.to_string(),
                    })?;
            }
            "timeout_seconds" => {
                self.timeout_seconds = value.parse()
                    .map_err(|e| ConfigError::ParseError {
                        key: key.to_string(),
                        value: value.to_string(),
                        error: e.to_string(),
                    })?;
            }
            "debug" => {
                self.debug_mode = value.parse()
                    .map_err(|e| ConfigError::ParseError {
                        key: key.to_string(),
                        value: value.to_string(),
                        error: e.to_string(),
                    })?;
            }
            _ => {} // Ignore unknown keys
        }
        Ok(())
    }
    
    // Good: Validate configuration values
    pub fn validate(&self) -> Result<(), ConfigError> {
        if self.server_port == 0 {
            return Err(ConfigError::ValidationError(
                "Server port cannot be 0".to_string()
            ));
        }
        
        if self.server_host.is_empty() {
            return Err(ConfigError::ValidationError(
                "Server host cannot be empty".to_string()
            ));
        }
        
        if self.database_url.is_empty() {
            return Err(ConfigError::ValidationError(
                "Database URL cannot be empty".to_string()
            ));
        }
        
        if self.max_connections == 0 {
            return Err(ConfigError::ValidationError(
                "Max connections must be greater than 0".to_string()
            ));
        }
        
        if self.timeout_seconds == 0 {
            return Err(ConfigError::ValidationError(
                "Timeout must be greater than 0".to_string()
            ));
        }
        
        Ok(())
    }
    
    // Good: Provide environment-specific configurations
    pub fn for_testing() -> Self {
        Self {
            server_host: "127.0.0.1".to_string(),
            server_port: 0,  // Let OS choose available port
            database_url: "sqlite://:memory:".to_string(),
            log_level: LogLevel::Debug,
            max_connections: 10,
            timeout_seconds: 5,
            debug_mode: true,
        }
    }
    
    pub fn for_production() -> Self {
        Self {
            server_host: "0.0.0.0".to_string(),
            server_port: 80,
            database_url: "postgresql://user:pass@localhost/db".to_string(),
            log_level: LogLevel::Info,
            max_connections: 1000,
            timeout_seconds: 30,
            debug_mode: false,
        }
    }
}

// Good: Environment detection utility
pub fn detect_environment() -> String {
    env::var("ENVIRONMENT")
        .or_else(|_| env::var("ENV"))
        .or_else(|_| env::var("RUST_ENV"))
        .unwrap_or_else(|_| "development".to_string())
        .to_lowercase()
}

// Good: Configuration builder for tests
#[cfg(test)]
pub struct ConfigBuilder {
    config: AppConfig,
}

#[cfg(test)]
impl ConfigBuilder {
    pub fn new() -> Self {
        Self {
            config: AppConfig::default(),
        }
    }
    
    pub fn host(mut self, host: &str) -> Self {
        self.config.server_host = host.to_string();
        self
    }
    
    pub fn port(mut self, port: u16) -> Self {
        self.config.server_port = port;
        self
    }
    
    pub fn debug(mut self, debug: bool) -> Self {
        self.config.debug_mode = debug;
        self
    }
    
    pub fn build(self) -> AppConfig {
        self.config
    }
}

fn main() {
    // Create a sample config file for demonstration
    let config_content = r#"
# Application configuration
host = "0.0.0.0"
port = 3000
database_url = "postgresql://user:pass@localhost/mydb"
log_level = "info"
max_connections = 200
timeout_seconds = 60
debug = false
"#;
    
    fs::write("config.toml", config_content).unwrap();
    
    // Set some environment variables for demonstration
    env::set_var("APP_PORT", "8080");
    env::set_var("LOG_LEVEL", "debug");
    env::set_var("DEBUG", "true");
    
    println!("Environment: {}", detect_environment());
    
    // Load configuration
    match AppConfig::load() {
        Ok(config) => {
            println!("Configuration loaded successfully:");
            println!("  Server: {}:{}", config.server_host, config.server_port);
            println!("  Database: {}", config.database_url);
            println!("  Log level: {:?}", config.log_level);
            println!("  Max connections: {}", config.max_connections);
            println!("  Timeout: {}s", config.timeout_seconds);
            println!("  Debug mode: {}", config.debug_mode);
        }
        Err(e) => {
            eprintln!("Failed to load configuration: {}", e);
        }
    }
    
    // Demonstrate environment-specific configs
    println!("\\nTesting configuration:");
    let test_config = AppConfig::for_testing();
    println!("Test database: {}", test_config.database_url);
    
    // Cleanup
    let _ = fs::remove_file("config.toml");
}

#[cfg(test)]
mod config_tests {
    use super::*;
    
    #[test]
    fn test_default_config() {
        let config = AppConfig::default();
        assert_eq!(config.server_host, "localhost");
        assert_eq!(config.server_port, 8080);
        assert_eq!(config.log_level, LogLevel::Info);
    }
    
    #[test]
    fn test_config_builder() {
        let config = ConfigBuilder::new()
            .host("example.com")
            .port(9000)
            .debug(true)
            .build();
        
        assert_eq!(config.server_host, "example.com");
        assert_eq!(config.server_port, 9000);
        assert!(config.debug_mode);
    }
    
    #[test]
    fn test_validation() {
        let mut config = AppConfig::default();
        config.server_port = 0;
        assert!(config.validate().is_err());
    }
}
```

Structured configuration management prevents runtime errors and makes  
applications more maintainable. Use defaults, validate inputs, and  
support multiple configuration sources with clear precedence.  

## Use appropriate synchronization primitives

Choose the right synchronization primitives based on your concurrency  
patterns and performance requirements.  

```rust
use std::sync::{Arc, Mutex, RwLock, Condvar, atomic::{AtomicBool, AtomicUsize, Ordering}};
use std::thread;
use std::time::{Duration, Instant};
use std::collections::VecDeque;

// Good: Use Mutex for simple shared mutable state
struct Counter {
    value: Arc<Mutex<usize>>,
}

impl Counter {
    fn new() -> Self {
        Self {
            value: Arc::new(Mutex::new(0)),
        }
    }
    
    fn increment(&self) {
        let mut val = self.value.lock().unwrap();
        *val += 1;
    }
    
    fn get(&self) -> usize {
        *self.value.lock().unwrap()
    }
    
    fn clone_handle(&self) -> Self {
        Self {
            value: Arc::clone(&self.value),
        }
    }
}

// Good: Use RwLock for read-heavy workloads
struct ReadHeavyData {
    data: Arc<RwLock<Vec<String>>>,
}

impl ReadHeavyData {
    fn new() -> Self {
        Self {
            data: Arc::new(RwLock::new(Vec::new())),
        }
    }
    
    fn add_item(&self, item: String) {
        let mut data = self.data.write().unwrap();
        data.push(item);
    }
    
    fn get_count(&self) -> usize {
        let data = self.data.read().unwrap();
        data.len()
    }
    
    fn contains(&self, item: &str) -> bool {
        let data = self.data.read().unwrap();
        data.iter().any(|s| s == item)
    }
    
    fn clone_handle(&self) -> Self {
        Self {
            data: Arc::clone(&self.data),
        }
    }
}

// Good: Use atomics for simple counters and flags
struct AtomicCounter {
    value: AtomicUsize,
    is_active: AtomicBool,
}

impl AtomicCounter {
    fn new() -> Self {
        Self {
            value: AtomicUsize::new(0),
            is_active: AtomicBool::new(true),
        }
    }
    
    fn increment(&self) -> usize {
        self.value.fetch_add(1, Ordering::SeqCst)
    }
    
    fn get(&self) -> usize {
        self.value.load(Ordering::SeqCst)
    }
    
    fn deactivate(&self) {
        self.is_active.store(false, Ordering::SeqCst);
    }
    
    fn is_active(&self) -> bool {
        self.is_active.load(Ordering::SeqCst)
    }
}

// Good: Use Condvar for complex synchronization
struct WorkQueue {
    items: Mutex<VecDeque<String>>,
    condvar: Condvar,
    shutdown: AtomicBool,
}

impl WorkQueue {
    fn new() -> Self {
        Self {
            items: Mutex::new(VecDeque::new()),
            condvar: Condvar::new(),
            shutdown: AtomicBool::new(false),
        }
    }
    
    fn push(&self, item: String) {
        if !self.shutdown.load(Ordering::SeqCst) {
            let mut queue = self.items.lock().unwrap();
            queue.push_back(item);
            self.condvar.notify_one();
        }
    }
    
    fn pop(&self) -> Option<String> {
        let mut queue = self.items.lock().unwrap();
        
        loop {
            if let Some(item) = queue.pop_front() {
                return Some(item);
            }
            
            if self.shutdown.load(Ordering::SeqCst) {
                return None;
            }
            
            queue = self.condvar.wait(queue).unwrap();
        }
    }
    
    fn shutdown(&self) {
        self.shutdown.store(true, Ordering::SeqCst);
        self.condvar.notify_all();
    }
}

// Good: Lock-free data structure using atomics
struct LockFreeStack<T> {
    head: Arc<atomic::AtomicPtr<Node<T>>>,
}

struct Node<T> {
    data: T,
    next: *mut Node<T>,
}

impl<T> LockFreeStack<T> {
    fn new() -> Self {
        Self {
            head: Arc::new(atomic::AtomicPtr::new(std::ptr::null_mut())),
        }
    }
    
    fn push(&self, data: T) {
        let new_node = Box::into_raw(Box::new(Node {
            data,
            next: std::ptr::null_mut(),
        }));
        
        loop {
            let current_head = self.head.load(Ordering::Acquire);
            unsafe {
                (*new_node).next = current_head;
            }
            
            match self.head.compare_exchange_weak(
                current_head,
                new_node,
                Ordering::Release,
                Ordering::Relaxed,
            ) {
                Ok(_) => break,
                Err(_) => continue,
            }
        }
    }
    
    fn pop(&self) -> Option<T> {
        loop {
            let current_head = self.head.load(Ordering::Acquire);
            if current_head.is_null() {
                return None;
            }
            
            let next = unsafe { (*current_head).next };
            
            match self.head.compare_exchange_weak(
                current_head,
                next,
                Ordering::Release,
                Ordering::Relaxed,
            ) {
                Ok(_) => {
                    let data = unsafe { Box::from_raw(current_head).data };
                    return Some(data);
                }
                Err(_) => continue,
            }
        }
    }
    
    fn clone_handle(&self) -> Self {
        Self {
            head: Arc::clone(&self.head),
        }
    }
}

// Good: Performance comparison between synchronization methods
fn benchmark_synchronization() {
    const ITERATIONS: usize = 100_000;
    const THREAD_COUNT: usize = 4;
    
    // Benchmark Mutex
    let mutex_counter = Counter::new();
    let mutex_handles: Vec<_> = (0..THREAD_COUNT)
        .map(|_| {
            let counter = mutex_counter.clone_handle();
            thread::spawn(move || {
                for _ in 0..ITERATIONS / THREAD_COUNT {
                    counter.increment();
                }
            })
        })
        .collect();
    
    let start = Instant::now();
    for handle in mutex_handles {
        handle.join().unwrap();
    }
    let mutex_duration = start.elapsed();
    
    // Benchmark AtomicUsize
    let atomic_counter = Arc::new(AtomicCounter::new());
    let atomic_handles: Vec<_> = (0..THREAD_COUNT)
        .map(|_| {
            let counter = Arc::clone(&atomic_counter);
            thread::spawn(move || {
                for _ in 0..ITERATIONS / THREAD_COUNT {
                    counter.increment();
                }
            })
        })
        .collect();
    
    let start = Instant::now();
    for handle in atomic_handles {
        handle.join().unwrap();
    }
    let atomic_duration = start.elapsed();
    
    println!("Synchronization benchmark ({} operations, {} threads):", ITERATIONS, THREAD_COUNT);
    println!("  Mutex:   {:?} - Final value: {}", mutex_duration, mutex_counter.get());
    println!("  Atomic:  {:?} - Final value: {}", atomic_duration, atomic_counter.get());
}

fn demonstrate_work_queue() {
    let queue = Arc::new(WorkQueue::new());
    
    // Producer thread
    let producer_queue = Arc::clone(&queue);
    let producer = thread::spawn(move || {
        for i in 1..=10 {
            producer_queue.push(format!("Task {}", i));
            thread::sleep(Duration::from_millis(100));
        }
        println!("Producer finished");
    });
    
    // Consumer threads
    let mut consumers = Vec::new();
    for worker_id in 1..=3 {
        let consumer_queue = Arc::clone(&queue);
        let consumer = thread::spawn(move || {
            while let Some(task) = consumer_queue.pop() {
                println!("Worker {} processing: {}", worker_id, task);
                thread::sleep(Duration::from_millis(200)); // Simulate work
            }
            println!("Worker {} finished", worker_id);
        });
        consumers.push(consumer);
    }
    
    // Wait for producer to finish
    producer.join().unwrap();
    
    // Give consumers time to process remaining tasks
    thread::sleep(Duration::from_millis(1000));
    
    // Shutdown the queue
    queue.shutdown();
    
    // Wait for all consumers to finish
    for consumer in consumers {
        consumer.join().unwrap();
    }
}

fn demonstrate_read_heavy_workload() {
    let data = ReadHeavyData::new();
    
    // Initial data setup
    for i in 1..=100 {
        data.add_item(format!("Item {}", i));
    }
    
    // Many reader threads
    let mut readers = Vec::new();
    for reader_id in 1..=10 {
        let data_handle = data.clone_handle();
        let reader = thread::spawn(move || {
            for _ in 0..1000 {
                let count = data_handle.get_count();
                let exists = data_handle.contains("Item 50");
                if reader_id == 1 && count > 0 {
                    println!("Reader {}: {} items, Item 50 exists: {}", reader_id, count, exists);
                }
            }
        });
        readers.push(reader);
    }
    
    // One writer thread
    let writer_data = data.clone_handle();
    let writer = thread::spawn(move || {
        for i in 101..=110 {
            writer_data.add_item(format!("Item {}", i));
            thread::sleep(Duration::from_millis(50));
        }
    });
    
    writer.join().unwrap();
    for reader in readers {
        reader.join().unwrap();
    }
    
    println!("Final item count: {}", data.get_count());
}

fn demonstrate_lock_free_stack() {
    let stack = LockFreeStack::new();
    
    // Multiple producer threads
    let mut producers = Vec::new();
    for thread_id in 1..=4 {
        let stack_handle = stack.clone_handle();
        let producer = thread::spawn(move || {
            for i in 1..=25 {
                stack_handle.push(format!("T{}-Item{}", thread_id, i));
            }
        });
        producers.push(producer);
    }
    
    // Wait for all producers
    for producer in producers {
        producer.join().unwrap();
    }
    
    // Consumer thread
    let mut items = Vec::new();
    while let Some(item) = stack.pop() {
        items.push(item);
    }
    
    println!("Lock-free stack processed {} items", items.len());
    println!("Sample items: {:?}", &items[..5.min(items.len())]);
}

fn main() {
    println!("=== Synchronization Primitives Demo ===\\n");
    
    println!("1. Performance Benchmark:");
    benchmark_synchronization();
    
    println!("\\n2. Work Queue with Condvar:");
    demonstrate_work_queue();
    
    println!("\\n3. Read-Heavy Workload with RwLock:");
    demonstrate_read_heavy_workload();
    
    println!("\\n4. Lock-Free Stack:");
    demonstrate_lock_free_stack();
}
```

Choose synchronization primitives based on access patterns: Mutex for  
simple shared state, RwLock for read-heavy workloads, atomics for simple  
counters, and Condvar for complex coordination between threads.  

## Implement proper logging

Set up structured logging that provides useful information for debugging  
and monitoring while being performant and configurable.  

```rust
use std::collections::HashMap;
use std::fs::{File, OpenOptions};
use std::io::{self, Write};
use std::sync::{Arc, Mutex};
use std::time::{SystemTime, UNIX_EPOCH};

// Good: Define log levels with ordering
#[derive(Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord)]
pub enum LogLevel {
    Error = 1,
    Warn = 2,
    Info = 3,
    Debug = 4,
    Trace = 5,
}

impl std::fmt::Display for LogLevel {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self {
            LogLevel::Error => write!(f, "ERROR"),
            LogLevel::Warn => write!(f, "WARN "),
            LogLevel::Info => write!(f, "INFO "),
            LogLevel::Debug => write!(f, "DEBUG"),
            LogLevel::Trace => write!(f, "TRACE"),
        }
    }
}

// Good: Structured log entry
#[derive(Debug, Clone)]
pub struct LogEntry {
    pub timestamp: u64,
    pub level: LogLevel,
    pub target: String,
    pub message: String,
    pub fields: HashMap<String, String>,
}

impl LogEntry {
    pub fn new(level: LogLevel, target: &str, message: String) -> Self {
        let timestamp = SystemTime::now()
            .duration_since(UNIX_EPOCH)
            .unwrap()
            .as_secs();
        
        Self {
            timestamp,
            level,
            target: target.to_string(),
            message,
            fields: HashMap::new(),
        }
    }
    
    pub fn with_field(mut self, key: &str, value: &str) -> Self {
        self.fields.insert(key.to_string(), value.to_string());
        self
    }
    
    pub fn format_json(&self) -> String {
        let mut json = format!(
            r#"{{"timestamp":{},"level":"{}","target":"{}","message":"{}""#,
            self.timestamp, self.level, self.target, self.message
        );
        
        for (key, value) in &self.fields {
            json.push_str(&format!(r#","{}":"{}""#, key, value));
        }
        
        json.push('}');
        json
    }
    
    pub fn format_pretty(&self) -> String {
        let datetime = chrono::DateTime::<chrono::Utc>::from_timestamp(self.timestamp as i64, 0)
            .unwrap_or_default()
            .format("%Y-%m-%d %H:%M:%S UTC");
        
        let mut formatted = format!(
            "[{}] {} {}: {}",
            datetime, self.level, self.target, self.message
        );
        
        if !self.fields.is_empty() {
            formatted.push_str(" {");
            for (i, (key, value)) in self.fields.iter().enumerate() {
                if i > 0 {
                    formatted.push_str(", ");
                }
                formatted.push_str(&format!("{}={}", key, value));
            }
            formatted.push('}');
        }
        
        formatted
    }
}

// Good: Flexible logger trait
pub trait LogWriter: Send + Sync {
    fn write_log(&self, entry: &LogEntry) -> io::Result<()>;
}

// Console logger
pub struct ConsoleLogger {
    min_level: LogLevel,
    use_json: bool,
}

impl ConsoleLogger {
    pub fn new(min_level: LogLevel, use_json: bool) -> Self {
        Self { min_level, use_json }
    }
}

impl LogWriter for ConsoleLogger {
    fn write_log(&self, entry: &LogEntry) -> io::Result<()> {
        if entry.level <= self.min_level {
            let formatted = if self.use_json {
                entry.format_json()
            } else {
                entry.format_pretty()
            };
            
            if entry.level == LogLevel::Error {
                eprintln!("{}", formatted);
            } else {
                println!("{}", formatted);
            }
        }
        Ok(())
    }
}

// File logger
pub struct FileLogger {
    file: Arc<Mutex<File>>,
    min_level: LogLevel,
    use_json: bool,
}

impl FileLogger {
    pub fn new(file_path: &str, min_level: LogLevel, use_json: bool) -> io::Result<Self> {
        let file = OpenOptions::new()
            .create(true)
            .append(true)
            .open(file_path)?;
        
        Ok(Self {
            file: Arc::new(Mutex::new(file)),
            min_level,
            use_json,
        })
    }
}

impl LogWriter for FileLogger {
    fn write_log(&self, entry: &LogEntry) -> io::Result<()> {
        if entry.level <= self.min_level {
            let formatted = if self.use_json {
                entry.format_json()
            } else {
                entry.format_pretty()
            };
            
            let mut file = self.file.lock().unwrap();
            writeln!(file, "{}", formatted)?;
            file.flush()?;
        }
        Ok(())
    }
}

// Good: Main logger with multiple writers
pub struct Logger {
    writers: Vec<Box<dyn LogWriter>>,
}

impl Logger {
    pub fn new() -> Self {
        Self {
            writers: Vec::new(),
        }
    }
    
    pub fn add_writer(mut self, writer: Box<dyn LogWriter>) -> Self {
        self.writers.push(writer);
        self
    }
    
    pub fn log(&self, entry: LogEntry) {
        for writer in &self.writers {
            let _ = writer.write_log(&entry);
        }
    }
    
    pub fn error(&self, target: &str, message: &str) {
        let entry = LogEntry::new(LogLevel::Error, target, message.to_string());
        self.log(entry);
    }
    
    pub fn warn(&self, target: &str, message: &str) {
        let entry = LogEntry::new(LogLevel::Warn, target, message.to_string());
        self.log(entry);
    }
    
    pub fn info(&self, target: &str, message: &str) {
        let entry = LogEntry::new(LogLevel::Info, target, message.to_string());
        self.log(entry);
    }
    
    pub fn debug(&self, target: &str, message: &str) {
        let entry = LogEntry::new(LogLevel::Debug, target, message.to_string());
        self.log(entry);
    }
    
    pub fn trace(&self, target: &str, message: &str) {
        let entry = LogEntry::new(LogLevel::Trace, target, message.to_string());
        self.log(entry);
    }
    
    // Good: Structured logging methods
    pub fn log_with_fields(&self, level: LogLevel, target: &str, message: &str, fields: HashMap<String, String>) {
        let mut entry = LogEntry::new(level, target, message.to_string());
        entry.fields = fields;
        self.log(entry);
    }
    
    pub fn error_with_context(&self, target: &str, message: &str, context: &[(&str, &str)]) {
        let mut entry = LogEntry::new(LogLevel::Error, target, message.to_string());
        for (key, value) in context {
            entry = entry.with_field(key, value);
        }
        self.log(entry);
    }
}

// Good: Global logger instance
static mut GLOBAL_LOGGER: Option<Logger> = None;
static INIT: std::sync::Once = std::sync::Once::new();

pub fn init_logger(logger: Logger) {
    INIT.call_once(|| {
        unsafe {
            GLOBAL_LOGGER = Some(logger);
        }
    });
}

pub fn get_logger() -> &'static Logger {
    unsafe {
        GLOBAL_LOGGER.as_ref().expect("Logger not initialized")
    }
}

// Good: Convenience macros (simplified versions)
macro_rules! log_error {
    ($target:expr, $message:expr) => {
        get_logger().error($target, $message)
    };
    ($target:expr, $($arg:tt)*) => {
        get_logger().error($target, &format!($($arg)*))
    };
}

macro_rules! log_info {
    ($target:expr, $message:expr) => {
        get_logger().info($target, $message)
    };
    ($target:expr, $($arg:tt)*) => {
        get_logger().info($target, &format!($($arg)*))
    };
}

macro_rules! log_debug {
    ($target:expr, $message:expr) => {
        get_logger().debug($target, $message)
    };
    ($target:expr, $($arg:tt)*) => {
        get_logger().debug($target, &format!($($arg)*))
    };
}

// Good: Application example with proper logging
struct UserService {
    users: HashMap<u32, String>,
}

impl UserService {
    fn new() -> Self {
        log_info!("user_service", "Initializing user service");
        Self {
            users: HashMap::new(),
        }
    }
    
    fn create_user(&mut self, id: u32, name: String) -> Result<(), String> {
        log_debug!("user_service", "Creating user with ID: {}", id);
        
        if self.users.contains_key(&id) {
            let error_msg = format!("User with ID {} already exists", id);
            log_error!("user_service", "{}", error_msg);
            return Err(error_msg);
        }
        
        self.users.insert(id, name.clone());
        
        get_logger().log_with_fields(
            LogLevel::Info,
            "user_service",
            "User created successfully",
            [
                ("user_id".to_string(), id.to_string()),
                ("user_name".to_string(), name),
                ("total_users".to_string(), self.users.len().to_string()),
            ].into_iter().collect(),
        );
        
        Ok(())
    }
    
    fn get_user(&self, id: u32) -> Option<&String> {
        log_trace!("user_service", "Looking up user with ID: {}", id);
        
        match self.users.get(&id) {
            Some(name) => {
                log_debug!("user_service", "User found: {} -> {}", id, name);
                Some(name)
            }
            None => {
                log_warn!("user_service", "User not found with ID: {}", id);
                None
            }
        }
    }
    
    fn delete_user(&mut self, id: u32) -> Result<String, String> {
        log_debug!("user_service", "Deleting user with ID: {}", id);
        
        match self.users.remove(&id) {
            Some(name) => {
                get_logger().error_with_context(
                    "user_service",
                    "User deleted",
                    &[
                        ("user_id", &id.to_string()),
                        ("user_name", &name),
                        ("remaining_users", &self.users.len().to_string()),
                    ],
                );
                Ok(name)
            }
            None => {
                let error_msg = format!("User with ID {} not found for deletion", id);
                log_error!("user_service", "{}", error_msg);
                Err(error_msg)
            }
        }
    }
}

macro_rules! log_warn {
    ($target:expr, $message:expr) => {
        get_logger().warn($target, $message)
    };
    ($target:expr, $($arg:tt)*) => {
        get_logger().warn($target, &format!($($arg)*))
    };
}

macro_rules! log_trace {
    ($target:expr, $message:expr) => {
        get_logger().trace($target, $message)
    };
    ($target:expr, $($arg:tt)*) => {
        get_logger().trace($target, &format!($($arg)*))
    };
}

fn main() -> io::Result<()> {
    // Initialize logger with multiple outputs
    let logger = Logger::new()
        .add_writer(Box::new(ConsoleLogger::new(LogLevel::Info, false)))
        .add_writer(Box::new(FileLogger::new("app.log", LogLevel::Debug, true)?));
    
    init_logger(logger);
    
    log_info!("main", "Application starting");
    
    // Demonstrate user service with logging
    let mut service = UserService::new();
    
    // Create some users
    service.create_user(1, "Alice".to_string()).unwrap();
    service.create_user(2, "Bob".to_string()).unwrap();
    
    // Try to create duplicate
    if let Err(e) = service.create_user(1, "Charlie".to_string()) {
        log_error!("main", "Failed to create user: {}", e);
    }
    
    // Lookup users
    service.get_user(1);
    service.get_user(999);  // Non-existent user
    
    // Delete user
    service.delete_user(2).unwrap();
    
    log_info!("main", "Application finished");
    
    Ok(())
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_log_entry_formatting() {
        let entry = LogEntry::new(LogLevel::Info, "test", "Test message".to_string())
            .with_field("key1", "value1")
            .with_field("key2", "value2");
        
        let json = entry.format_json();
        assert!(json.contains(r#""level":"INFO""#));
        assert!(json.contains(r#""message":"Test message""#));
        assert!(json.contains(r#""key1":"value1""#));
        
        let pretty = entry.format_pretty();
        assert!(pretty.contains("INFO"));
        assert!(pretty.contains("Test message"));
        assert!(pretty.contains("key1=value1"));
    }
    
    #[test]
    fn test_log_levels() {
        assert!(LogLevel::Error < LogLevel::Warn);
        assert!(LogLevel::Warn < LogLevel::Info);
        assert!(LogLevel::Info < LogLevel::Debug);
        assert!(LogLevel::Debug < LogLevel::Trace);
    }
}
```

Good logging provides visibility into application behavior while being  
performant and configurable. Use structured logging with appropriate  
levels and context to aid debugging and monitoring.  

## Avoid common pitfalls and antipatterns

Learn to recognize and avoid common Rust antipatterns that can lead to  
performance issues, safety problems, or maintenance difficulties.  

```rust
use std::collections::HashMap;
use std::rc::Rc;
use std::cell::RefCell;

// Pitfall 1: Unnecessary cloning
// Bad: Cloning when borrowing would work
fn process_strings_bad(strings: Vec<String>) -> Vec<String> {
    strings.into_iter()
        .map(|s| s.clone())  // Unnecessary clone
        .filter(|s| s.len() > 5)
        .collect()
}

// Good: Use references when possible
fn process_strings_good(strings: &[String]) -> Vec<String> {
    strings.iter()
        .filter(|s| s.len() > 5)
        .cloned()  // Only clone what we need to keep
        .collect()
}

// Pitfall 2: Fighting the borrow checker with Rc<RefCell<T>>
// Bad: Overusing Rc<RefCell<T>> when simple ownership would work
struct BadGraph {
    nodes: Vec<Rc<RefCell<Node>>>,
}

struct Node {
    value: i32,
    connections: Vec<Rc<RefCell<Node>>>,
}

// Good: Use indices for graph-like structures
struct GoodGraph {
    nodes: Vec<GraphNode>,
}

struct GraphNode {
    value: i32,
    connections: Vec<usize>,  // Indices into the nodes vector
}

impl GoodGraph {
    fn new() -> Self {
        Self { nodes: Vec::new() }
    }
    
    fn add_node(&mut self, value: i32) -> usize {
        let index = self.nodes.len();
        self.nodes.push(GraphNode {
            value,
            connections: Vec::new(),
        });
        index
    }
    
    fn connect(&mut self, from: usize, to: usize) -> Result<(), String> {
        if from >= self.nodes.len() || to >= self.nodes.len() {
            return Err("Invalid node index".to_string());
        }
        self.nodes[from].connections.push(to);
        Ok(())
    }
    
    fn get_connections(&self, node: usize) -> Option<&[usize]> {
        self.nodes.get(node).map(|n| n.connections.as_slice())
    }
}

// Pitfall 3: Using String when &str would suffice
// Bad: Taking ownership when borrowing is enough
fn format_greeting_bad(name: String, greeting: String) -> String {
    format!("{}, {}!", greeting, name)
}

// Good: Use string slices for parameters
fn format_greeting_good(name: &str, greeting: &str) -> String {
    format!("{}, {}!", greeting, name)
}

// Pitfall 4: Ignoring error handling
// Bad: Using unwrap() or expect() everywhere
fn read_config_bad(filename: &str) -> HashMap<String, String> {
    let content = std::fs::read_to_string(filename).unwrap();  // Can panic
    let mut config = HashMap::new();
    
    for line in content.lines() {
        let parts: Vec<&str> = line.split('=').collect();
        config.insert(parts[0].to_string(), parts[1].to_string());  // Can panic
    }
    
    config
}

// Good: Proper error handling
fn read_config_good(filename: &str) -> Result<HashMap<String, String>, String> {
    let content = std::fs::read_to_string(filename)
        .map_err(|e| format!("Failed to read file: {}", e))?;
    
    let mut config = HashMap::new();
    
    for (line_num, line) in content.lines().enumerate() {
        if let Some((key, value)) = line.split_once('=') {
            config.insert(key.trim().to_string(), value.trim().to_string());
        } else if !line.trim().is_empty() {
            return Err(format!("Invalid config line {}: {}", line_num + 1, line));
        }
    }
    
    Ok(config)
}

// Pitfall 5: Inappropriate use of Box<T>
// Bad: Boxing when it's not needed
fn create_numbers_bad() -> Vec<Box<i32>> {
    (1..=10).map(|i| Box::new(i)).collect()
}

// Good: Use Box only when you need heap allocation or trait objects
fn create_numbers_good() -> Vec<i32> {
    (1..=10).collect()
}

// Good use of Box for trait objects
trait Processor {
    fn process(&self, data: &str) -> String;
}

struct UppercaseProcessor;
struct ReverseProcessor;

impl Processor for UppercaseProcessor {
    fn process(&self, data: &str) -> String {
        data.to_uppercase()
    }
}

impl Processor for ReverseProcessor {
    fn process(&self, data: &str) -> String {
        data.chars().rev().collect()
    }
}

fn create_processors() -> Vec<Box<dyn Processor>> {
    vec![
        Box::new(UppercaseProcessor),
        Box::new(ReverseProcessor),
    ]
}

// Pitfall 6: Premature optimization
// Bad: Micro-optimizing without measuring
fn calculate_sum_micro_optimized(numbers: &[i32]) -> i32 {
    unsafe {
        let mut sum = 0;
        let ptr = numbers.as_ptr();
        for i in 0..numbers.len() {
            sum += *ptr.add(i);  // Unnecessary unsafe code
        }
        sum
    }
}

// Good: Clear, safe code first
fn calculate_sum_clear(numbers: &[i32]) -> i32 {
    numbers.iter().sum()  // Idiomatic and likely just as fast
}

// Pitfall 7: Overly complex generics
// Bad: Generic parameters that don't add value
fn print_value_bad<T: std::fmt::Debug + Clone + PartialEq>(value: T) {
    println!("{:?}", value);  // Only uses Debug, others are unnecessary
}

// Good: Only constrain what you actually use
fn print_value_good<T: std::fmt::Debug>(value: T) {
    println!("{:?}", value);
}

// Pitfall 8: Not using iterators idiomatically
// Bad: Collecting intermediate results unnecessarily
fn process_data_bad(data: Vec<i32>) -> Vec<String> {
    let filtered: Vec<i32> = data.into_iter()
        .filter(|&x| x > 0)
        .collect();  // Unnecessary intermediate collection
    
    let doubled: Vec<i32> = filtered.into_iter()
        .map(|x| x * 2)
        .collect();  // Another unnecessary collection
    
    doubled.into_iter()
        .map(|x| x.to_string())
        .collect()
}

// Good: Chain operations without intermediate collections
fn process_data_good(data: Vec<i32>) -> Vec<String> {
    data.into_iter()
        .filter(|&x| x > 0)
        .map(|x| x * 2)
        .map(|x| x.to_string())
        .collect()
}

// Pitfall 9: Using unwrap() in library code
// Bad: Library function that can panic
pub fn divide_library_bad(a: f64, b: f64) -> f64 {
    if b == 0.0 {
        panic!("Division by zero");  // Bad for library code
    }
    a / b
}

// Good: Library function that returns Result
pub fn divide_library_good(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err("Division by zero".to_string())
    } else {
        Ok(a / b)
    }
}

// Pitfall 10: Inappropriate sharing patterns
// Bad: Using Arc<Mutex<T>> when single ownership would work
use std::sync::{Arc, Mutex};
use std::thread;

fn bad_sharing_pattern() {
    let data = Arc::new(Mutex::new(vec![1, 2, 3, 4, 5]));
    
    let data_clone = Arc::clone(&data);
    let handle = thread::spawn(move || {
        let mut d = data_clone.lock().unwrap();
        d.push(6);
    });
    
    handle.join().unwrap();
    
    let final_data = data.lock().unwrap();
    println!("Data: {:?}", *final_data);
}

// Good: Pass owned data and get results back
fn good_ownership_pattern() {
    let mut data = vec![1, 2, 3, 4, 5];
    
    let handle = thread::spawn(move || {
        data.push(6);
        data  // Return the modified data
    });
    
    let final_data = handle.join().unwrap();
    println!("Data: {:?}", final_data);
}

fn main() {
    // Demonstrate good practices
    let strings = vec!["hello".to_string(), "world".to_string(), "rust".to_string()];
    let processed = process_strings_good(&strings);
    println!("Processed strings: {:?}", processed);
    
    // Graph example
    let mut graph = GoodGraph::new();
    let node1 = graph.add_node(10);
    let node2 = graph.add_node(20);
    graph.connect(node1, node2).unwrap();
    
    if let Some(connections) = graph.get_connections(node1) {
        println!("Node {} connects to: {:?}", node1, connections);
    }
    
    // Proper error handling
    match read_config_good("nonexistent.conf") {
        Ok(config) => println!("Config: {:?}", config),
        Err(e) => println!("Config error: {}", e),
    }
    
    // Good use of processors
    let processors = create_processors();
    let test_data = "Hello, World!";
    
    for processor in processors {
        println!("Processed: {}", processor.process(test_data));
    }
    
    // Iterator chaining
    let numbers = vec![-2, -1, 0, 1, 2, 3, 4, 5];
    let result = process_data_good(numbers);
    println!("Processed data: {:?}", result);
    
    // Proper division handling
    match divide_library_good(10.0, 2.0) {
        Ok(result) => println!("Division result: {}", result),
        Err(e) => println!("Division error: {}", e),
    }
    
    // Good ownership pattern
    good_ownership_pattern();
}

// Good: Write tests for error conditions
#[cfg(test)]
mod antipattern_tests {
    use super::*;
    
    #[test]
    fn test_division_by_zero() {
        assert!(divide_library_good(5.0, 0.0).is_err());
    }
    
    #[test]
    fn test_config_parsing_error() {
        // Test with invalid config content
        let result = read_config_good("definitely_nonexistent_file.conf");
        assert!(result.is_err());
    }
    
    #[test]
    fn test_graph_invalid_connection() {
        let mut graph = GoodGraph::new();
        let node1 = graph.add_node(1);
        
        // Try to connect to non-existent node
        assert!(graph.connect(node1, 999).is_err());
    }
}
```

Avoiding antipatterns leads to more maintainable, performant, and safe  
code. Focus on clear ownership, proper error handling, and idiomatic  
iterator usage rather than fighting the language.  

## Use workspace and project organization

Organize larger Rust projects using workspaces and clear module  
hierarchies to maintain code clarity and enable code reuse.  

```rust
// This example shows how to structure a workspace
// File: Cargo.toml (workspace root)
/*
[workspace]
members = [
    "core",
    "api", 
    "cli",
    "web"
]
resolver = "2"

[workspace.dependencies]
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1.0", features = ["full"] }
clap = { version = "4.0", features = ["derive"] }
*/

// File: core/Cargo.toml
/*
[package]
name = "myapp-core"
version = "0.1.0"
edition = "2021"

[dependencies]
serde = { workspace = true }
tokio = { workspace = true }
*/

// File: core/src/lib.rs
//! Core business logic and shared types

pub mod domain;
pub mod repository;
pub mod service;
pub mod error;

// Re-export commonly used types
pub use domain::{User, Product, Order};
pub use error::{AppError, Result};
pub use service::{UserService, ProductService};

// File: core/src/domain.rs
//! Domain models and business entities

use serde::{Deserialize, Serialize};

#[derive(Debug, Clone, Serialize, Deserialize, PartialEq)]
pub struct User {
    pub id: u64,
    pub username: String,
    pub email: String,
    pub created_at: u64,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Product {
    pub id: u64,
    pub name: String,
    pub description: String,
    pub price: f64,
    pub in_stock: u32,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Order {
    pub id: u64,
    pub user_id: u64,
    pub products: Vec<OrderItem>,
    pub total: f64,
    pub status: OrderStatus,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct OrderItem {
    pub product_id: u64,
    pub quantity: u32,
    pub unit_price: f64,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum OrderStatus {
    Pending,
    Processing,
    Shipped,
    Delivered,
    Cancelled,
}

impl User {
    pub fn new(id: u64, username: String, email: String) -> Self {
        Self {
            id,
            username,
            email,
            created_at: std::time::SystemTime::now()
                .duration_since(std::time::UNIX_EPOCH)
                .unwrap()
                .as_secs(),
        }
    }
    
    pub fn is_valid_email(&self) -> bool {
        self.email.contains('@') && self.email.contains('.')
    }
}

impl Order {
    pub fn calculate_total(&mut self) {
        self.total = self.products.iter()
            .map(|item| item.unit_price * item.quantity as f64)
            .sum();
    }
}

// File: core/src/error.rs
//! Error types for the application

use std::fmt;

pub type Result<T> = std::result::Result<T, AppError>;

#[derive(Debug)]
pub enum AppError {
    NotFound(String),
    ValidationError(String),
    DatabaseError(String),
    NetworkError(String),
    InternalError(String),
}

impl fmt::Display for AppError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            AppError::NotFound(msg) => write!(f, "Not found: {}", msg),
            AppError::ValidationError(msg) => write!(f, "Validation error: {}", msg),
            AppError::DatabaseError(msg) => write!(f, "Database error: {}", msg),
            AppError::NetworkError(msg) => write!(f, "Network error: {}", msg),
            AppError::InternalError(msg) => write!(f, "Internal error: {}", msg),
        }
    }
}

impl std::error::Error for AppError {}

// File: core/src/repository.rs
//! Data access layer traits

use crate::domain::{User, Product, Order};
use crate::error::Result;

#[async_trait::async_trait]
pub trait UserRepository: Send + Sync {
    async fn find_by_id(&self, id: u64) -> Result<Option<User>>;
    async fn find_by_username(&self, username: &str) -> Result<Option<User>>;
    async fn create(&self, user: User) -> Result<User>;
    async fn update(&self, user: User) -> Result<User>;
    async fn delete(&self, id: u64) -> Result<bool>;
    async fn list_all(&self) -> Result<Vec<User>>;
}

#[async_trait::async_trait]
pub trait ProductRepository: Send + Sync {
    async fn find_by_id(&self, id: u64) -> Result<Option<Product>>;
    async fn list_available(&self) -> Result<Vec<Product>>;
    async fn create(&self, product: Product) -> Result<Product>;
    async fn update_stock(&self, id: u64, new_stock: u32) -> Result<bool>;
}

#[async_trait::async_trait]
pub trait OrderRepository: Send + Sync {
    async fn find_by_id(&self, id: u64) -> Result<Option<Order>>;
    async fn find_by_user(&self, user_id: u64) -> Result<Vec<Order>>;
    async fn create(&self, order: Order) -> Result<Order>;
    async fn update_status(&self, id: u64, status: crate::domain::OrderStatus) -> Result<bool>;
}

// File: core/src/service.rs
//! Business logic services

use crate::domain::{User, Product, Order, OrderStatus};
use crate::repository::{UserRepository, ProductRepository, OrderRepository};
use crate::error::{AppError, Result};
use std::sync::Arc;

pub struct UserService {
    repository: Arc<dyn UserRepository>,
}

impl UserService {
    pub fn new(repository: Arc<dyn UserRepository>) -> Self {
        Self { repository }
    }
    
    pub async fn create_user(&self, username: String, email: String) -> Result<User> {
        // Validation
        if username.is_empty() {
            return Err(AppError::ValidationError("Username cannot be empty".to_string()));
        }
        
        if !email.contains('@') {
            return Err(AppError::ValidationError("Invalid email format".to_string()));
        }
        
        // Check if username already exists
        if let Some(_) = self.repository.find_by_username(&username).await? {
            return Err(AppError::ValidationError("Username already exists".to_string()));
        }
        
        // Create user
        let user = User::new(0, username, email);  // ID will be assigned by database
        self.repository.create(user).await
    }
    
    pub async fn get_user(&self, id: u64) -> Result<User> {
        self.repository.find_by_id(id).await?
            .ok_or_else(|| AppError::NotFound(format!("User with ID {}", id)))
    }
    
    pub async fn list_users(&self) -> Result<Vec<User>> {
        self.repository.list_all().await
    }
}

pub struct ProductService {
    repository: Arc<dyn ProductRepository>,
}

impl ProductService {
    pub fn new(repository: Arc<dyn ProductRepository>) -> Self {
        Self { repository }
    }
    
    pub async fn get_available_products(&self) -> Result<Vec<Product>> {
        self.repository.list_available().await
    }
    
    pub async fn check_availability(&self, product_id: u64, quantity: u32) -> Result<bool> {
        let product = self.repository.find_by_id(product_id).await?
            .ok_or_else(|| AppError::NotFound(format!("Product {}", product_id)))?;
        
        Ok(product.in_stock >= quantity)
    }
}

pub struct OrderService {
    order_repo: Arc<dyn OrderRepository>,
    product_repo: Arc<dyn ProductRepository>,
    user_repo: Arc<dyn UserRepository>,
}

impl OrderService {
    pub fn new(
        order_repo: Arc<dyn OrderRepository>,
        product_repo: Arc<dyn ProductRepository>,
        user_repo: Arc<dyn UserRepository>,
    ) -> Self {
        Self {
            order_repo,
            product_repo,
            user_repo,
        }
    }
    
    pub async fn create_order(&self, user_id: u64, items: Vec<(u64, u32)>) -> Result<Order> {
        // Verify user exists
        self.user_repo.find_by_id(user_id).await?
            .ok_or_else(|| AppError::NotFound(format!("User {}", user_id)))?;
        
        // Validate and build order items
        let mut order_items = Vec::new();
        for (product_id, quantity) in items {
            let product = self.product_repo.find_by_id(product_id).await?
                .ok_or_else(|| AppError::NotFound(format!("Product {}", product_id)))?;
            
            if product.in_stock < quantity {
                return Err(AppError::ValidationError(
                    format!("Insufficient stock for product {}", product_id)
                ));
            }
            
            order_items.push(crate::domain::OrderItem {
                product_id,
                quantity,
                unit_price: product.price,
            });
        }
        
        // Create order
        let mut order = Order {
            id: 0,  // Will be assigned by database
            user_id,
            products: order_items,
            total: 0.0,
            status: OrderStatus::Pending,
        };
        
        order.calculate_total();
        self.order_repo.create(order).await
    }
    
    pub async fn get_user_orders(&self, user_id: u64) -> Result<Vec<Order>> {
        self.order_repo.find_by_user(user_id).await
    }
}

// File: api/Cargo.toml
/*
[package]
name = "myapp-api"
version = "0.1.0"
edition = "2021"

[dependencies]
myapp-core = { path = "../core" }
tokio = { workspace = true }
serde = { workspace = true }
serde_json = "1.0"
axum = "0.7"
*/

// File: api/src/main.rs
//! HTTP API server

use myapp_core::{UserService, ProductService, OrderService};
use std::sync::Arc;

// This would contain HTTP handlers using the services
#[tokio::main]
async fn main() {
    println!("API server would start here");
    
    // Example of how services would be used
    demo_services().await;
}

async fn demo_services() {
    // In a real app, these would be actual database implementations
    struct MockUserRepo;
    struct MockProductRepo;
    struct MockOrderRepo;
    
    // Implement the traits with mock behavior for demo
    #[async_trait::async_trait]
    impl myapp_core::repository::UserRepository for MockUserRepo {
        async fn find_by_id(&self, _id: u64) -> myapp_core::Result<Option<myapp_core::User>> {
            Ok(None)
        }
        async fn find_by_username(&self, _username: &str) -> myapp_core::Result<Option<myapp_core::User>> {
            Ok(None)
        }
        async fn create(&self, mut user: myapp_core::User) -> myapp_core::Result<myapp_core::User> {
            user.id = 1; // Mock ID assignment
            Ok(user)
        }
        async fn update(&self, user: myapp_core::User) -> myapp_core::Result<myapp_core::User> {
            Ok(user)
        }
        async fn delete(&self, _id: u64) -> myapp_core::Result<bool> {
            Ok(true)
        }
        async fn list_all(&self) -> myapp_core::Result<Vec<myapp_core::User>> {
            Ok(vec![])
        }
    }
    
    #[async_trait::async_trait]
    impl myapp_core::repository::ProductRepository for MockProductRepo {
        async fn find_by_id(&self, _id: u64) -> myapp_core::Result<Option<myapp_core::Product>> {
            Ok(None)
        }
        async fn list_available(&self) -> myapp_core::Result<Vec<myapp_core::Product>> {
            Ok(vec![])
        }
        async fn create(&self, product: myapp_core::Product) -> myapp_core::Result<myapp_core::Product> {
            Ok(product)
        }
        async fn update_stock(&self, _id: u64, _new_stock: u32) -> myapp_core::Result<bool> {
            Ok(true)
        }
    }
    
    #[async_trait::async_trait]
    impl myapp_core::repository::OrderRepository for MockOrderRepo {
        async fn find_by_id(&self, _id: u64) -> myapp_core::Result<Option<myapp_core::Order>> {
            Ok(None)
        }
        async fn find_by_user(&self, _user_id: u64) -> myapp_core::Result<Vec<myapp_core::Order>> {
            Ok(vec![])
        }
        async fn create(&self, mut order: myapp_core::Order) -> myapp_core::Result<myapp_core::Order> {
            order.id = 1;
            Ok(order)
        }
        async fn update_status(&self, _id: u64, _status: myapp_core::domain::OrderStatus) -> myapp_core::Result<bool> {
            Ok(true)
        }
    }
    
    let user_service = UserService::new(Arc::new(MockUserRepo));
    
    match user_service.create_user("alice".to_string(), "alice@example.com".to_string()).await {
        Ok(user) => println!("Created user: {:?}", user),
        Err(e) => println!("Error creating user: {}", e),
    }
}

// File: cli/Cargo.toml
/*
[package]
name = "myapp-cli"
version = "0.1.0"
edition = "2021"

[[bin]]
name = "myapp"
path = "src/main.rs"

[dependencies]
myapp-core = { path = "../core" }
clap = { workspace = true }
tokio = { workspace = true }
*/

// File: cli/src/main.rs
//! Command-line interface

use clap::{Parser, Subcommand};
use myapp_core::User;

#[derive(Parser)]
#[command(name = "myapp")]
#[command(about = "A sample application CLI")]
struct Cli {
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]
enum Commands {
    /// User management commands
    User {
        #[command(subcommand)]
        action: UserCommands,
    },
    /// Product management commands  
    Product {
        #[command(subcommand)]
        action: ProductCommands,
    },
}

#[derive(Subcommand)]
enum UserCommands {
    /// Create a new user
    Create {
        username: String,
        email: String,
    },
    /// List all users
    List,
}

#[derive(Subcommand)]
enum ProductCommands {
    /// List available products
    List,
}

#[tokio::main]
async fn main() {
    let cli = Cli::parse();
    
    match cli.command {
        Commands::User { action } => {
            match action {
                UserCommands::Create { username, email } => {
                    let user = User::new(1, username, email);
                    println!("Created user: {:?}", user);
                }
                UserCommands::List => {
                    println!("Listing users...");
                }
            }
        }
        Commands::Product { action } => {
            match action {
                ProductCommands::List => {
                    println!("Listing products...");
                }
            }
        }
    }
}

fn main() {
    println!("Workspace organization example");
    println!("This demonstrates how to structure a multi-crate Rust workspace");
    println!("with shared core logic and multiple frontend applications.");
    
    // Create a sample user to show the core functionality
    let user = myapp_core::User::new(1, "alice".to_string(), "alice@example.com".to_string());
    println!("Sample user: {:?}", user);
    println!("Email valid: {}", user.is_valid_email());
}
```

Workspace organization enables code reuse, clear separation of concerns,  
and easier testing. Structure projects with a core library containing  
business logic and separate crates for different interfaces.  

## Follow Rust API design guidelines

Design APIs that feel natural to Rust developers by following established  
conventions and idioms for function signatures, error handling, and  
type design.  

```rust
use std::collections::HashMap;
use std::fmt;
use std::str::FromStr;

// Good: Use consistent naming conventions
pub struct ConfigBuilder {
    values: HashMap<String, String>,
}

impl ConfigBuilder {
    // Good: Constructor follows convention
    pub fn new() -> Self {
        Self {
            values: HashMap::new(),
        }
    }
    
    // Good: Builder methods consume self and return Self
    pub fn with_value(mut self, key: impl Into<String>, value: impl Into<String>) -> Self {
        self.values.insert(key.into(), value.into());
        self
    }
    
    // Good: Terminal method that consumes self
    pub fn build(self) -> Config {
        Config { values: self.values }
    }
}

impl Default for ConfigBuilder {
    fn default() -> Self {
        Self::new()
    }
}

pub struct Config {
    values: HashMap<String, String>,
}

impl Config {
    // Good: Provide both owned and borrowed accessors
    pub fn get(&self, key: &str) -> Option<&str> {
        self.values.get(key).map(|s| s.as_str())
    }
    
    pub fn get_owned(&self, key: &str) -> Option<String> {
        self.values.get(key).cloned()
    }
    
    // Good: Accept flexible input types with impl Into<T>
    pub fn insert(&mut self, key: impl Into<String>, value: impl Into<String>) {
        self.values.insert(key.into(), value.into());
    }
    
    // Good: Return iterators for collections
    pub fn keys(&self) -> impl Iterator<Item = &str> {
        self.values.keys().map(|s| s.as_str())
    }
    
    pub fn values(&self) -> impl Iterator<Item = &str> {
        self.values.values().map(|s| s.as_str())
    }
    
    pub fn iter(&self) -> impl Iterator<Item = (&str, &str)> {
        self.values.iter().map(|(k, v)| (k.as_str(), v.as_str()))
    }
}

// Good: Implement standard traits
impl FromStr for Config {
    type Err = ConfigError;
    
    fn from_str(s: &str) -> Result<Self, Self::Err> {
        let mut builder = ConfigBuilder::new();
        
        for (line_no, line) in s.lines().enumerate() {
            let line = line.trim();
            if line.is_empty() || line.starts_with('#') {
                continue;
            }
            
            if let Some((key, value)) = line.split_once('=') {
                builder = builder.with_value(key.trim(), value.trim());
            } else {
                return Err(ConfigError::ParseError {
                    line: line_no + 1,
                    content: line.to_string(),
                });
            }
        }
        
        Ok(builder.build())
    }
}

impl fmt::Display for Config {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        for (key, value) in &self.values {
            writeln!(f, "{}={}", key, value)?;
        }
        Ok(())
    }
}

// Good: Comprehensive error type
#[derive(Debug)]
pub enum ConfigError {
    ParseError { line: usize, content: String },
    ValidationError(String),
    IoError(std::io::Error),
}

impl fmt::Display for ConfigError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            ConfigError::ParseError { line, content } => {
                write!(f, "Parse error at line {}: '{}'", line, content)
            }
            ConfigError::ValidationError(msg) => write!(f, "Validation error: {}", msg),
            ConfigError::IoError(err) => write!(f, "IO error: {}", err),
        }
    }
}

impl std::error::Error for ConfigError {
    fn source(&self) -> Option<&(dyn std::error::Error + 'static)> {
        match self {
            ConfigError::IoError(err) => Some(err),
            _ => None,
        }
    }
}

impl From<std::io::Error> for ConfigError {
    fn from(err: std::io::Error) -> Self {
        ConfigError::IoError(err)
    }
}

// Good: Flexible input types for functions
pub fn merge_configs(primary: &Config, secondary: &Config) -> Config {
    let mut builder = ConfigBuilder::new();
    
    // Add secondary config first (lower priority)
    for (key, value) in secondary.iter() {
        builder = builder.with_value(key, value);
    }
    
    // Add primary config (higher priority, overwrites secondary)
    for (key, value) in primary.iter() {
        builder = builder.with_value(key, value);
    }
    
    builder.build()
}

// Good: Generic function with appropriate bounds
pub fn validate_config<T, F>(config: &Config, validator: F) -> Result<T, ConfigError>
where
    F: FnOnce(&Config) -> Result<T, String>,
{
    validator(config).map_err(ConfigError::ValidationError)
}

// Good: Extension trait for adding functionality
pub trait ConfigExt {
    fn get_bool(&self, key: &str) -> Option<bool>;
    fn get_int(&self, key: &str) -> Option<i32>;
    fn get_float(&self, key: &str) -> Option<f64>;
}

impl ConfigExt for Config {
    fn get_bool(&self, key: &str) -> Option<bool> {
        self.get(key)?.parse().ok()
    }
    
    fn get_int(&self, key: &str) -> Option<i32> {
        self.get(key)?.parse().ok()
    }
    
    fn get_float(&self, key: &str) -> Option<f64> {
        self.get(key)?.parse().ok()
    }
}

// Good: Implement IntoIterator for natural iteration
impl IntoIterator for Config {
    type Item = (String, String);
    type IntoIter = std::collections::hash_map::IntoIter<String, String>;
    
    fn into_iter(self) -> Self::IntoIter {
        self.values.into_iter()
    }
}

impl<'a> IntoIterator for &'a Config {
    type Item = (&'a str, &'a str);
    type IntoIter = Box<dyn Iterator<Item = (&'a str, &'a str)> + 'a>;
    
    fn into_iter(self) -> Self::IntoIter {
        Box::new(self.iter())
    }
}

// Good: Type-safe configuration sections
pub struct DatabaseConfig {
    pub host: String,
    pub port: u16,
    pub username: String,
    pub password: String,
    pub database: String,
}

impl DatabaseConfig {
    pub fn from_config(config: &Config) -> Result<Self, ConfigError> {
        let host = config.get("db_host")
            .ok_or_else(|| ConfigError::ValidationError("Missing db_host".to_string()))?
            .to_string();
        
        let port = config.get("db_port")
            .ok_or_else(|| ConfigError::ValidationError("Missing db_port".to_string()))?
            .parse()
            .map_err(|_| ConfigError::ValidationError("Invalid db_port".to_string()))?;
        
        let username = config.get("db_username")
            .ok_or_else(|| ConfigError::ValidationError("Missing db_username".to_string()))?
            .to_string();
        
        let password = config.get("db_password")
            .ok_or_else(|| ConfigError::ValidationError("Missing db_password".to_string()))?
            .to_string();
        
        let database = config.get("db_name")
            .ok_or_else(|| ConfigError::ValidationError("Missing db_name".to_string()))?
            .to_string();
        
        Ok(Self {
            host,
            port,
            username,
            password,
            database,
        })
    }
    
    pub fn connection_string(&self) -> String {
        format!("postgresql://{}:{}@{}:{}/{}", 
                self.username, self.password, self.host, self.port, self.database)
    }
}

// Good: Provide both high-level and low-level APIs
pub mod high_level {
    use super::*;
    
    /// High-level API for common use cases
    pub fn load_config_from_file(path: &str) -> Result<Config, ConfigError> {
        let content = std::fs::read_to_string(path)?;
        content.parse()
    }
    
    pub fn load_config_with_defaults(path: &str, defaults: &[(&str, &str)]) -> Result<Config, ConfigError> {
        let mut builder = ConfigBuilder::new();
        
        // Add defaults first
        for (key, value) in defaults {
            builder = builder.with_value(*key, *value);
        }
        
        // Try to load file, but don't fail if it doesn't exist
        if let Ok(file_config) = load_config_from_file(path) {
            // Merge file config (higher priority)
            for (key, value) in file_config.iter() {
                builder = builder.with_value(key, value);
            }
        }
        
        Ok(builder.build())
    }
}

pub mod low_level {
    use super::*;
    
    /// Low-level API for advanced users
    pub fn parse_config_line(line: &str) -> Result<Option<(String, String)>, ConfigError> {
        let line = line.trim();
        
        if line.is_empty() || line.starts_with('#') {
            return Ok(None);
        }
        
        if let Some((key, value)) = line.split_once('=') {
            Ok(Some((key.trim().to_string(), value.trim().to_string())))
        } else {
            Err(ConfigError::ParseError {
                line: 0,
                content: line.to_string(),
            })
        }
    }
    
    pub fn build_config_from_pairs<I>(pairs: I) -> Config
    where
        I: IntoIterator<Item = (String, String)>,
    {
        let mut builder = ConfigBuilder::new();
        for (key, value) in pairs {
            builder = builder.with_value(key, value);
        }
        builder.build()
    }
}

fn main() {
    // Demonstrate API usage
    let config = ConfigBuilder::new()
        .with_value("app_name", "MyApp")
        .with_value("version", "1.0.0")
        .with_value("debug", "true")
        .with_value("port", "8080")
        .build();
    
    println!("Config:");
    println!("{}", config);
    
    // Demonstrate extension trait
    println!("Debug mode: {:?}", config.get_bool("debug"));
    println!("Port: {:?}", config.get_int("port"));
    
    // Demonstrate iteration
    println!("All config values:");
    for (key, value) in &config {
        println!("  {} = {}", key, value);
    }
    
    // Demonstrate parsing from string
    let config_text = r#"
        # Sample configuration
        db_host = localhost
        db_port = 5432
        db_username = user
        db_password = secret
        db_name = myapp
    "#;
    
    match config_text.parse::<Config>() {
        Ok(parsed_config) => {
            match DatabaseConfig::from_config(&parsed_config) {
                Ok(db_config) => {
                    println!("Database connection: {}", db_config.connection_string());
                }
                Err(e) => println!("Database config error: {}", e),
            }
        }
        Err(e) => println!("Parse error: {}", e),
    }
    
    // Demonstrate high-level API
    let defaults = [
        ("timeout", "30"),
        ("retries", "3"),
        ("log_level", "info"),
    ];
    
    match high_level::load_config_with_defaults("nonexistent.conf", &defaults) {
        Ok(config_with_defaults) => {
            println!("Config with defaults:");
            for (key, value) in config_with_defaults.iter() {
                println!("  {} = {}", key, value);
            }
        }
        Err(e) => println!("Error loading config: {}", e),
    }
}

#[cfg(test)]
mod api_tests {
    use super::*;
    
    #[test]
    fn test_builder_pattern() {
        let config = ConfigBuilder::new()
            .with_value("key1", "value1")
            .with_value("key2", "value2")
            .build();
        
        assert_eq!(config.get("key1"), Some("value1"));
        assert_eq!(config.get("key2"), Some("value2"));
    }
    
    #[test]
    fn test_parsing() {
        let config_text = "key1=value1\nkey2=value2";
        let config: Config = config_text.parse().unwrap();
        
        assert_eq!(config.get("key1"), Some("value1"));
        assert_eq!(config.get("key2"), Some("value2"));
    }
    
    #[test]
    fn test_extension_trait() {
        let config = ConfigBuilder::new()
            .with_value("flag", "true")
            .with_value("number", "42")
            .build();
        
        assert_eq!(config.get_bool("flag"), Some(true));
        assert_eq!(config.get_int("number"), Some(42));
    }
}
```

Good API design makes libraries intuitive and pleasant to use. Follow  
Rust conventions for naming, provide flexible input types, implement  
standard traits, and offer both high-level and low-level interfaces.  

## Write documentation and examples

Provide comprehensive documentation with examples that help users  
understand how to use your code effectively.  

```rust
//! # Calculator Library
//! 
//! A simple calculator library demonstrating good documentation practices.
//! 
//! ## Quick Start
//! 
//! ```rust
//! use calculator::Calculator;
//! 
//! let mut calc = Calculator::new();
//! let result = calc.add(2.0, 3.0);
//! assert_eq!(result, 5.0);
//! ```
//! 
//! ## Features
//! 
//! - Basic arithmetic operations
//! - Memory functions
//! - History tracking
//! - Error handling for invalid operations

/// A calculator that performs basic arithmetic operations and maintains history.
/// 
/// # Examples
/// 
/// Basic usage:
/// 
/// ```
/// # use std::collections::VecDeque;
/// # pub struct Calculator { history: VecDeque<f64> }
/// # impl Calculator {
/// #     pub fn new() -> Self { Self { history: VecDeque::new() } }
/// #     pub fn add(&mut self, a: f64, b: f64) -> f64 { let result = a + b; self.history.push_back(result); result }
/// #     pub fn divide(&mut self, a: f64, b: f64) -> Result<f64, String> { 
/// #         if b == 0.0 { Err("Division by zero".to_string()) } else { let result = a / b; self.history.push_back(result); Ok(result) } 
/// #     }
/// #     pub fn get_history(&self) -> &VecDeque<f64> { &self.history }
/// # }
/// let mut calc = Calculator::new();
/// 
/// // Perform calculations
/// let sum = calc.add(10.0, 5.0);
/// let quotient = calc.divide(15.0, 3.0).unwrap();
/// 
/// assert_eq!(sum, 15.0);
/// assert_eq!(quotient, 5.0);
/// ```
/// 
/// Error handling:
/// 
/// ```
/// # use std::collections::VecDeque;
/// # pub struct Calculator { history: VecDeque<f64> }
/// # impl Calculator {
/// #     pub fn new() -> Self { Self { history: VecDeque::new() } }
/// #     pub fn divide(&mut self, a: f64, b: f64) -> Result<f64, String> { 
/// #         if b == 0.0 { Err("Division by zero".to_string()) } else { let result = a / b; self.history.push_back(result); Ok(result) } 
/// #     }
/// # }
/// let mut calc = Calculator::new();
/// 
/// match calc.divide(10.0, 0.0) {
///     Ok(result) => println!("Result: {}", result),
///     Err(e) => println!("Error: {}", e),
/// }
/// ```
pub struct Calculator {
    history: std::collections::VecDeque<f64>,
}

impl Calculator {
    /// Creates a new calculator instance.
    /// 
    /// # Examples
    /// 
    /// ```
    /// # use std::collections::VecDeque;
    /// # pub struct Calculator { history: VecDeque<f64> }
    /// # impl Calculator { pub fn new() -> Self { Self { history: VecDeque::new() } } }
    /// let calc = Calculator::new();
    /// ```
    pub fn new() -> Self {
        Self {
            history: std::collections::VecDeque::new(),
        }
    }
    
    /// Adds two numbers and stores the result in history.
    /// 
    /// # Arguments
    /// 
    /// * `a` - The first number
    /// * `b` - The second number
    /// 
    /// # Returns
    /// 
    /// The sum of `a` and `b`
    /// 
    /// # Examples
    /// 
    /// ```
    /// # use std::collections::VecDeque;
    /// # pub struct Calculator { history: VecDeque<f64> }
    /// # impl Calculator {
    /// #     pub fn new() -> Self { Self { history: VecDeque::new() } }
    /// #     pub fn add(&mut self, a: f64, b: f64) -> f64 { let result = a + b; self.history.push_back(result); result }
    /// # }
    /// let mut calc = Calculator::new();
    /// let result = calc.add(2.5, 3.5);
    /// assert_eq!(result, 6.0);
    /// ```
    pub fn add(&mut self, a: f64, b: f64) -> f64 {
        let result = a + b;
        self.history.push_back(result);
        result
    }
    
    /// Divides two numbers and stores the result in history.
    /// 
    /// # Arguments
    /// 
    /// * `a` - The dividend
    /// * `b` - The divisor
    /// 
    /// # Returns
    /// 
    /// * `Ok(result)` - The quotient of `a` divided by `b`
    /// * `Err(message)` - Error message if `b` is zero
    /// 
    /// # Errors
    /// 
    /// This function will return an error if the divisor `b` is zero.
    /// 
    /// # Examples
    /// 
    /// Successful division:
    /// 
    /// ```
    /// # use std::collections::VecDeque;
    /// # pub struct Calculator { history: VecDeque<f64> }
    /// # impl Calculator {
    /// #     pub fn new() -> Self { Self { history: VecDeque::new() } }
    /// #     pub fn divide(&mut self, a: f64, b: f64) -> Result<f64, String> { 
    /// #         if b == 0.0 { Err("Division by zero".to_string()) } else { let result = a / b; self.history.push_back(result); Ok(result) } 
    /// #     }
    /// # }
    /// let mut calc = Calculator::new();
    /// let result = calc.divide(10.0, 2.0).unwrap();
    /// assert_eq!(result, 5.0);
    /// ```
    /// 
    /// Division by zero:
    /// 
    /// ```
    /// # use std::collections::VecDeque;
    /// # pub struct Calculator { history: VecDeque<f64> }
    /// # impl Calculator {
    /// #     pub fn new() -> Self { Self { history: VecDeque::new() } }
    /// #     pub fn divide(&mut self, a: f64, b: f64) -> Result<f64, String> { 
    /// #         if b == 0.0 { Err("Division by zero".to_string()) } else { let result = a / b; self.history.push_back(result); Ok(result) } 
    /// #     }
    /// # }
    /// let mut calc = Calculator::new();
    /// assert!(calc.divide(5.0, 0.0).is_err());
    /// ```
    pub fn divide(&mut self, a: f64, b: f64) -> Result<f64, String> {
        if b == 0.0 {
            Err("Division by zero".to_string())
        } else {
            let result = a / b;
            self.history.push_back(result);
            Ok(result)
        }
    }
    
    /// Returns a reference to the calculation history.
    /// 
    /// The history contains all results from previous calculations in  
    /// chronological order.
    /// 
    /// # Examples
    /// 
    /// ```
    /// # use std::collections::VecDeque;
    /// # pub struct Calculator { history: VecDeque<f64> }
    /// # impl Calculator {
    /// #     pub fn new() -> Self { Self { history: VecDeque::new() } }
    /// #     pub fn add(&mut self, a: f64, b: f64) -> f64 { let result = a + b; self.history.push_back(result); result }
    /// #     pub fn get_history(&self) -> &VecDeque<f64> { &self.history }
    /// # }
    /// let mut calc = Calculator::new();
    /// calc.add(1.0, 2.0);
    /// calc.add(3.0, 4.0);
    /// 
    /// let history = calc.get_history();
    /// assert_eq!(history.len(), 2);
    /// assert_eq!(history[0], 3.0);
    /// assert_eq!(history[1], 7.0);
    /// ```
    pub fn get_history(&self) -> &std::collections::VecDeque<f64> {
        &self.history
    }
}

fn main() {
    let mut calc = Calculator::new();
    
    println!("Calculator Demo");
    println!("===============");
    
    // Demonstrate basic operations
    let sum = calc.add(10.0, 5.0);
    println!("10.0 + 5.0 = {}", sum);
    
    let difference = calc.add(20.0, -8.0);  // Using add for subtraction
    println!("20.0 - 8.0 = {}", difference);
    
    // Demonstrate error handling
    match calc.divide(15.0, 3.0) {
        Ok(result) => println!("15.0 / 3.0 = {}", result),
        Err(e) => println!("Error: {}", e),
    }
    
    match calc.divide(10.0, 0.0) {
        Ok(result) => println!("10.0 / 0.0 = {}", result),
        Err(e) => println!("Error: {}", e),
    }
    
    // Show history
    println!("\nCalculation History:");
    for (i, result) in calc.get_history().iter().enumerate() {
        println!("  {}: {}", i + 1, result);
    }
}
```

Good documentation includes module-level overviews, comprehensive examples,  
clear parameter descriptions, and examples of both success and error cases.  
Write documentation as you code, not as an afterthought.  

## Profile and benchmark code

Use profiling tools and benchmarks to understand performance characteristics  
and identify optimization opportunities based on real measurements.  

```rust
use std::time::{Duration, Instant};
use std::collections::HashMap;

// Simple benchmarking framework
pub struct Benchmark {
    name: String,
    iterations: usize,
    warmup_iterations: usize,
}

impl Benchmark {
    pub fn new(name: &str) -> Self {
        Self {
            name: name.to_string(),
            iterations: 1000,
            warmup_iterations: 100,
        }
    }
    
    pub fn iterations(mut self, count: usize) -> Self {
        self.iterations = count;
        self
    }
    
    pub fn warmup(mut self, count: usize) -> Self {
        self.warmup_iterations = count;
        self
    }
    
    pub fn run<F>(self, mut operation: F) -> BenchmarkResult
    where
        F: FnMut(),
    {
        // Warmup
        for _ in 0..self.warmup_iterations {
            operation();
        }
        
        // Actual benchmark
        let start = Instant::now();
        for _ in 0..self.iterations {
            operation();
        }
        let total_duration = start.elapsed();
        
        BenchmarkResult {
            name: self.name,
            total_duration,
            iterations: self.iterations,
            avg_duration: total_duration / self.iterations as u32,
        }
    }
}

pub struct BenchmarkResult {
    pub name: String,
    pub total_duration: Duration,
    pub iterations: usize,
    pub avg_duration: Duration,
}

impl BenchmarkResult {
    pub fn print(&self) {
        println!("Benchmark: {}", self.name);
        println!("  Total time: {:?}", self.total_duration);
        println!("  Iterations: {}", self.iterations);
        println!("  Average: {:?}", self.avg_duration);
        println!("  Ops/sec: {:.2}", 1_000_000_000.0 / self.avg_duration.as_nanos() as f64);
        println!();
    }
}

// Example: String concatenation performance comparison
fn benchmark_string_concat() {
    let words = vec!["hello", "world", "rust", "programming", "performance"];
    
    // Method 1: Using + operator
    let result1 = Benchmark::new("String concatenation with +")
        .iterations(10000)
        .run(|| {
            let mut result = String::new();
            for word in &words {
                result = result + word + " ";
            }
            std::hint::black_box(result);
        });
    
    // Method 2: Using push_str
    let result2 = Benchmark::new("String concatenation with push_str")
        .iterations(10000)
        .run(|| {
            let mut result = String::new();
            for word in &words {
                result.push_str(word);
                result.push(' ');
            }
            std::hint::black_box(result);
        });
    
    // Method 3: Using join
    let result3 = Benchmark::new("String concatenation with join")
        .iterations(10000)
        .run(|| {
            let result = words.join(" ");
            std::hint::black_box(result);
        });
    
    // Method 4: Using format!
    let result4 = Benchmark::new("String concatenation with format!")
        .iterations(10000)
        .run(|| {
            let result = format!("{} {} {} {} {}", words[0], words[1], words[2], words[3], words[4]);
            std::hint::black_box(result);
        });
    
    result1.print();
    result2.print();
    result3.print();
    result4.print();
}

// Example: Collection performance comparison
fn benchmark_collections() {
    const SIZE: usize = 10000;
    let data: Vec<i32> = (0..SIZE as i32).collect();
    
    // HashMap insertion
    let result1 = Benchmark::new("HashMap insertion")
        .iterations(100)
        .run(|| {
            let mut map = HashMap::new();
            for &value in &data {
                map.insert(value, value * 2);
            }
            std::hint::black_box(map);
        });
    
    // Vec creation and sorting
    let result2 = Benchmark::new("Vec creation and sorting")
        .iterations(100)
        .run(|| {
            let mut vec = data.clone();
            vec.sort_unstable();
            std::hint::black_box(vec);
        });
    
    // Binary search in sorted vec
    let sorted_data: Vec<i32> = {
        let mut v = data.clone();
        v.sort_unstable();
        v
    };
    
    let result3 = Benchmark::new("Binary search in Vec")
        .iterations(1000)
        .run(|| {
            for i in 0..100 {
                let key = i * 100;
                let result = sorted_data.binary_search(&key);
                std::hint::black_box(result);
            }
        });
    
    // HashMap lookup
    let map: HashMap<i32, i32> = data.iter().map(|&x| (x, x * 2)).collect();
    
    let result4 = Benchmark::new("HashMap lookup")
        .iterations(1000)
        .run(|| {
            for i in 0..100 {
                let key = i * 100;
                let result = map.get(&key);
                std::hint::black_box(result);
            }
        });
    
    result1.print();
    result2.print();
    result3.print();
    result4.print();
}

// Example: Algorithm comparison
fn fibonacci_recursive(n: u32) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci_recursive(n - 1) + fibonacci_recursive(n - 2),
    }
}

fn fibonacci_iterative(n: u32) -> u64 {
    if n <= 1 {
        return n as u64;
    }
    
    let mut prev = 0;
    let mut curr = 1;
    
    for _ in 2..=n {
        let next = prev + curr;
        prev = curr;
        curr = next;
    }
    
    curr
}

fn fibonacci_memoized(n: u32) -> u64 {
    fn fib_memo(n: u32, memo: &mut HashMap<u32, u64>) -> u64 {
        if let Some(&result) = memo.get(&n) {
            return result;
        }
        
        let result = match n {
            0 => 0,
            1 => 1,
            _ => fib_memo(n - 1, memo) + fib_memo(n - 2, memo),
        };
        
        memo.insert(n, result);
        result
    }
    
    let mut memo = HashMap::new();
    fib_memo(n, &mut memo)
}

fn benchmark_fibonacci() {
    const N: u32 = 30;
    
    // Don't benchmark recursive for large N - it's too slow!
    if N <= 35 {
        let result1 = Benchmark::new("Fibonacci recursive")
            .iterations(10)
            .run(|| {
                let result = fibonacci_recursive(N);
                std::hint::black_box(result);
            });
        result1.print();
    }
    
    let result2 = Benchmark::new("Fibonacci iterative")
        .iterations(10000)
        .run(|| {
            let result = fibonacci_iterative(N);
            std::hint::black_box(result);
        });
    
    let result3 = Benchmark::new("Fibonacci memoized")
        .iterations(1000)
        .run(|| {
            let result = fibonacci_memoized(N);
            std::hint::black_box(result);
        });
    
    result2.print();
    result3.print();
}

// Memory allocation profiling
fn benchmark_memory_allocation() {
    let result1 = Benchmark::new("Vec with known capacity")
        .iterations(1000)
        .run(|| {
            let mut vec = Vec::with_capacity(1000);
            for i in 0..1000 {
                vec.push(i);
            }
            std::hint::black_box(vec);
        });
    
    let result2 = Benchmark::new("Vec without capacity")
        .iterations(1000)
        .run(|| {
            let mut vec = Vec::new();
            for i in 0..1000 {
                vec.push(i);
            }
            std::hint::black_box(vec);
        });
    
    result1.print();
    result2.print();
}

fn main() {
    println!("Performance Benchmarks");
    println!("=====================\\n");
    
    println!("String Concatenation Comparison:");
    println!("-------------------------------");
    benchmark_string_concat();
    
    println!("Collection Operations Comparison:");
    println!("--------------------------------");
    benchmark_collections();
    
    println!("Algorithm Comparison (Fibonacci):");
    println!("---------------------------------");
    benchmark_fibonacci();
    
    println!("Memory Allocation Comparison:");
    println!("----------------------------");
    benchmark_memory_allocation();
    
    println!("Profiling Tips:");
    println!("- Use `cargo bench` for more sophisticated benchmarking");
    println!("- Use `perf` or `valgrind` for detailed profiling");
    println!("- Profile in release mode: `cargo build --release`");
    println!("- Use `std::hint::black_box()` to prevent optimization");
    println!("- Measure real workloads, not microbenchmarks only");
}
```

Regular profiling and benchmarking help identify performance bottlenecks  
and validate optimization efforts. Use tools like `cargo bench`, `perf`,  
and `flamegraph` for comprehensive performance analysis.  

## Use unsafe code judiciously

When unsafe code is necessary, isolate it carefully, document invariants  
thoroughly, and provide safe abstractions around it.  

```rust
/// A safe wrapper around a raw pointer with bounds checking
pub struct SafeBuffer {
    ptr: *mut u8,
    len: usize,
    capacity: usize,
}

impl SafeBuffer {
    /// Creates a new buffer with the specified capacity
    /// 
    /// # Panics
    /// 
    /// Panics if capacity is 0 or if memory allocation fails
    pub fn new(capacity: usize) -> Self {
        assert!(capacity > 0, "Capacity must be greater than 0");
        
        let layout = std::alloc::Layout::array::<u8>(capacity)
            .expect("Invalid layout");
        
        let ptr = unsafe { std::alloc::alloc(layout) };
        
        if ptr.is_null() {
            std::alloc::handle_alloc_error(layout);
        }
        
        Self {
            ptr,
            len: 0,
            capacity,
        }
    }
    
    /// Returns the current length of the buffer
    pub fn len(&self) -> usize {
        self.len
    }
    
    /// Returns the capacity of the buffer
    pub fn capacity(&self) -> usize {
        self.capacity
    }
    
    /// Checks if the buffer is empty
    pub fn is_empty(&self) -> bool {
        self.len == 0
    }
    
    /// Pushes a byte to the end of the buffer
    /// 
    /// # Panics
    /// 
    /// Panics if the buffer is at capacity
    pub fn push(&mut self, byte: u8) {
        assert!(self.len < self.capacity, "Buffer is at capacity");
        
        unsafe {
            // SAFETY: We just checked that len < capacity, so this write is safe
            *self.ptr.add(self.len) = byte;
        }
        self.len += 1;
    }
    
    /// Gets a byte at the specified index
    /// 
    /// # Panics
    /// 
    /// Panics if index is out of bounds
    pub fn get(&self, index: usize) -> u8 {
        assert!(index < self.len, "Index {} is out of bounds for length {}", index, self.len);
        
        unsafe {
            // SAFETY: We just checked bounds, so this read is safe
            *self.ptr.add(index)
        }
    }
    
    /// Sets a byte at the specified index
    /// 
    /// # Panics
    /// 
    /// Panics if index is out of bounds
    pub fn set(&mut self, index: usize, byte: u8) {
        assert!(index < self.len, "Index {} is out of bounds for length {}", index, self.len);
        
        unsafe {
            // SAFETY: We just checked bounds, so this write is safe
            *self.ptr.add(index) = byte;
        }
    }
    
    /// Returns a safe slice view of the buffer contents
    pub fn as_slice(&self) -> &[u8] {
        unsafe {
            // SAFETY: ptr is valid for len bytes, and we maintain this invariant
            std::slice::from_raw_parts(self.ptr, self.len)
        }
    }
    
    /// Unsafe method to get a raw pointer - marked as unsafe to warn callers
    /// 
    /// # Safety
    /// 
    /// Caller must ensure:
    /// - The pointer is not used beyond the buffer's lifetime
    /// - Writes don't exceed the buffer's capacity
    /// - The buffer is not moved while the pointer is in use
    pub unsafe fn as_ptr(&self) -> *const u8 {
        self.ptr
    }
    
    /// Unsafe method to extend the buffer length without initialization
    /// 
    /// # Safety
    /// 
    /// Caller must ensure:
    /// - new_len <= capacity
    /// - The memory between old len and new_len is properly initialized
    pub unsafe fn set_len(&mut self, new_len: usize) {
        assert!(new_len <= self.capacity, "New length exceeds capacity");
        self.len = new_len;
    }
}

impl Drop for SafeBuffer {
    fn drop(&mut self) {
        let layout = std::alloc::Layout::array::<u8>(self.capacity)
            .expect("Invalid layout");
        
        unsafe {
            // SAFETY: ptr was allocated with the same layout
            std::alloc::dealloc(self.ptr, layout);
        }
    }
}

// Unsafe Send/Sync implementations need careful consideration
unsafe impl Send for SafeBuffer {}
unsafe impl Sync for SafeBuffer {}

/// Example of unsafe transmutation with safety checks
fn safe_transmute_u32_to_bytes(value: u32) -> [u8; 4] {
    unsafe {
        // SAFETY: u32 and [u8; 4] have the same size and alignment
        std::mem::transmute(value.to_le_bytes())
    }
}

/// Example of working with uninitialized memory safely
fn create_initialized_vec(size: usize, init_value: u8) -> Vec<u8> {
    let mut vec = Vec::with_capacity(size);
    
    unsafe {
        // Get raw pointer to the vec's buffer
        let ptr = vec.as_mut_ptr();
        
        // Initialize the memory
        for i in 0..size {
            // SAFETY: We have capacity for 'size' elements, and we're writing within bounds
            ptr.add(i).write(init_value);
        }
        
        // SAFETY: We just initialized 'size' elements
        vec.set_len(size);
    }
    
    vec
}

/// Example of FFI with proper error handling
extern "C" {
    fn strlen(s: *const std::os::raw::c_char) -> usize;
}

/// Safe wrapper around C's strlen function
/// 
/// # Safety
/// 
/// The input must be a valid, null-terminated C string
pub unsafe fn safe_strlen(s: *const std::os::raw::c_char) -> Option<usize> {
    if s.is_null() {
        return None;
    }
    
    // SAFETY: Caller guarantees s is a valid null-terminated string
    Some(strlen(s))
}

/// Convert C string to Rust string safely
pub unsafe fn c_str_to_string(s: *const std::os::raw::c_char) -> Option<String> {
    if s.is_null() {
        return None;
    }
    
    // SAFETY: Caller guarantees s is a valid null-terminated string
    let c_str = std::ffi::CStr::from_ptr(s);
    c_str.to_str().ok().map(|s| s.to_string())
}

/// Example of performance-critical unsafe code with extensive documentation
/// 
/// Copies non-overlapping memory regions efficiently
/// 
/// # Safety
/// 
/// Caller must ensure:
/// - src is valid for reads of len bytes
/// - dst is valid for writes of len bytes  
/// - src and dst do not overlap
/// - Both pointers are properly aligned for their types
pub unsafe fn fast_copy_nonoverlapping<T>(src: *const T, dst: *mut T, len: usize) {
    // SAFETY: Caller guarantees the safety requirements
    std::ptr::copy_nonoverlapping(src, dst, len);
}

/// Safe wrapper that checks for overlaps
pub fn safe_copy<T: Copy>(src: &[T], dst: &mut [T]) -> Result<(), &'static str> {
    if src.len() != dst.len() {
        return Err("Source and destination slices must have the same length");
    }
    
    let src_ptr = src.as_ptr();
    let dst_ptr = dst.as_mut_ptr();
    let len = src.len();
    
    // Check for overlaps
    let src_start = src_ptr as usize;
    let src_end = src_start + len * std::mem::size_of::<T>();
    let dst_start = dst_ptr as usize;
    let dst_end = dst_start + len * std::mem::size_of::<T>();
    
    if (src_start < dst_end && dst_start < src_end) {
        return Err("Source and destination regions overlap");
    }
    
    unsafe {
        // SAFETY: We checked for overlaps and length equality
        fast_copy_nonoverlapping(src_ptr, dst_ptr, len);
    }
    
    Ok(())
}

fn main() {
    // Demonstrate safe buffer usage
    println!("SafeBuffer Demo:");
    let mut buffer = SafeBuffer::new(10);
    
    for i in 0..5 {
        buffer.push(i * 10);
    }
    
    println!("Buffer contents: {:?}", buffer.as_slice());
    
    buffer.set(2, 99);
    println!("After setting index 2 to 99: {:?}", buffer.as_slice());
    
    // Demonstrate safe copy
    println!("\\nSafe Copy Demo:");
    let src = vec![1, 2, 3, 4, 5];
    let mut dst = vec![0; 5];
    
    match safe_copy(&src, &mut dst) {
        Ok(()) => println!("Copied successfully: {:?}", dst),
        Err(e) => println!("Copy failed: {}", e),
    }
    
    // Demonstrate initialized vec creation
    println!("\\nInitialized Vec Demo:");
    let initialized = create_initialized_vec(5, 42);
    println!("Initialized vec: {:?}", initialized);
    
    // Demonstrate transmutation
    println!("\\nTransmutation Demo:");
    let value = 0x12345678u32;
    let bytes = safe_transmute_u32_to_bytes(value);
    println!("u32 {} as bytes: {:?}", value, bytes);
    
    println!("\\nUnsafe Code Guidelines:");
    println!("- Always document safety requirements");
    println!("- Provide safe wrappers around unsafe code");
    println!("- Use assertions to check invariants");
    println!("- Minimize the scope of unsafe blocks");
    println!("- Test thoroughly, especially edge cases");
    println!("- Consider using tools like Miri for testing");
}

#[cfg(test)]
mod unsafe_tests {
    use super::*;
    
    #[test]
    fn test_safe_buffer() {
        let mut buffer = SafeBuffer::new(5);
        assert_eq!(buffer.len(), 0);
        assert_eq!(buffer.capacity(), 5);
        assert!(buffer.is_empty());
        
        buffer.push(10);
        buffer.push(20);
        assert_eq!(buffer.len(), 2);
        assert!(!buffer.is_empty());
        
        assert_eq!(buffer.get(0), 10);
        assert_eq!(buffer.get(1), 20);
        
        buffer.set(0, 99);
        assert_eq!(buffer.get(0), 99);
    }
    
    #[test]
    #[should_panic(expected = "Buffer is at capacity")]
    fn test_buffer_overflow() {
        let mut buffer = SafeBuffer::new(1);
        buffer.push(1);
        buffer.push(2); // Should panic
    }
    
    #[test]
    fn test_safe_copy() {
        let src = vec![1, 2, 3];
        let mut dst = vec![0; 3];
        
        assert!(safe_copy(&src, &mut dst).is_ok());
        assert_eq!(dst, vec![1, 2, 3]);
    }
    
    #[test]
    fn test_safe_copy_length_mismatch() {
        let src = vec![1, 2, 3];
        let mut dst = vec![0; 2];
        
        assert!(safe_copy(&src, &mut dst).is_err());
    }
}
```

Unsafe code should be used sparingly and wrapped in safe abstractions.  
Always document safety requirements, use assertions to check invariants,  
and test thoroughly including with tools like Miri.  

## Leverage the type system for correctness

Use Rust's type system to encode business rules and prevent invalid states  
at compile time rather than relying on runtime checks.  

```rust
/// Demonstrates using the type system to prevent invalid states

// Good: Use newtypes for different kinds of IDs
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub struct UserId(pub u64);

#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub struct OrderId(pub u64);

#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub struct ProductId(pub u64);

// Good: Use enums to represent state machines
#[derive(Debug, Clone, PartialEq)]
pub enum OrderState {
    Created { items: Vec<OrderItem> },
    Paid { items: Vec<OrderItem>, payment_id: String },
    Shipped { tracking_number: String },
    Delivered { delivery_date: String },
    Cancelled { reason: String },
}

#[derive(Debug, Clone, PartialEq)]
pub struct OrderItem {
    product_id: ProductId,
    quantity: u32,
    price_cents: u32,
}

// Good: Use phantom types for compile-time state tracking
use std::marker::PhantomData;

pub struct Database<State> {
    connection_string: String,
    _state: PhantomData<State>,
}

pub struct Connected;
pub struct Disconnected;

impl Database<Disconnected> {
    pub fn new(connection_string: String) -> Self {
        Self {
            connection_string,
            _state: PhantomData,
        }
    }
    
    pub fn connect(self) -> Result<Database<Connected>, String> {
        // Simulate connection logic
        if self.connection_string.is_empty() {
            Err("Invalid connection string".to_string())
        } else {
            Ok(Database {
                connection_string: self.connection_string,
                _state: PhantomData,
            })
        }
    }
}

impl Database<Connected> {
    pub fn query(&self, sql: &str) -> Result<Vec<String>, String> {
        // Only connected databases can execute queries
        if sql.trim().is_empty() {
            Err("Empty query".to_string())
        } else {
            Ok(vec![format!("Result for: {}", sql)])
        }
    }
    
    pub fn disconnect(self) -> Database<Disconnected> {
        Database {
            connection_string: self.connection_string,
            _state: PhantomData,
        }
    }
}

// Good: Use units and type-safe wrappers
#[derive(Debug, Clone, Copy, PartialEq, PartialOrd)]
pub struct Temperature(f64);

impl Temperature {
    pub fn celsius(value: f64) -> Self {
        Self(value)
    }
    
    pub fn fahrenheit(value: f64) -> Self {
        Self((value - 32.0) * 5.0 / 9.0)
    }
    
    pub fn kelvin(value: f64) -> Self {
        Self(value - 273.15)
    }
    
    pub fn to_celsius(&self) -> f64 {
        self.0
    }
    
    pub fn to_fahrenheit(&self) -> f64 {
        self.0 * 9.0 / 5.0 + 32.0
    }
    
    pub fn to_kelvin(&self) -> f64 {
        self.0 + 273.15
    }
}

// Good: Use const generics for compile-time validation
#[derive(Debug)]
pub struct FixedArray<T, const N: usize> {
    data: [T; N],
}

impl<T: Default + Copy, const N: usize> FixedArray<T, N> {
    pub fn new() -> Self {
        Self {
            data: [T::default(); N],
        }
    }
    
    pub fn get(&self, index: usize) -> Option<&T> {
        self.data.get(index)
    }
    
    pub fn set(&mut self, index: usize, value: T) -> Result<(), &'static str> {
        if index < N {
            self.data[index] = value;
            Ok(())
        } else {
            Err("Index out of bounds")
        }
    }
    
    pub fn len(&self) -> usize {
        N
    }
}

// Good: Use builder pattern with type states
pub struct QueryBuilder<State> {
    table: Option<String>,
    columns: Vec<String>,
    conditions: Vec<String>,
    _state: PhantomData<State>,
}

pub struct NoTable;
pub struct WithTable;

impl QueryBuilder<NoTable> {
    pub fn new() -> Self {
        Self {
            table: None,
            columns: Vec::new(),
            conditions: Vec::new(),
            _state: PhantomData,
        }
    }
    
    pub fn from_table(mut self, table: &str) -> QueryBuilder<WithTable> {
        QueryBuilder {
            table: Some(table.to_string()),
            columns: self.columns,
            conditions: self.conditions,
            _state: PhantomData,
        }
    }
}

impl QueryBuilder<WithTable> {
    pub fn select(mut self, column: &str) -> Self {
        self.columns.push(column.to_string());
        self
    }
    
    pub fn where_clause(mut self, condition: &str) -> Self {
        self.conditions.push(condition.to_string());
        self
    }
    
    pub fn build(self) -> String {
        let columns = if self.columns.is_empty() {
            "*".to_string()
        } else {
            self.columns.join(", ")
        };
        
        let mut query = format!("SELECT {} FROM {}", columns, self.table.unwrap());
        
        if !self.conditions.is_empty() {
            query.push_str(&format!(" WHERE {}", self.conditions.join(" AND ")));
        }
        
        query
    }
}

// Good: Use associated types for flexible APIs
trait Iterator2 {
    type Item;
    
    fn next(&mut self) -> Option<Self::Item>;
}

struct NumberIterator {
    current: u32,
    max: u32,
}

impl NumberIterator {
    fn new(max: u32) -> Self {
        Self { current: 0, max }
    }
}

impl Iterator2 for NumberIterator {
    type Item = u32;
    
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

fn main() {
    // Demonstrate ID type safety
    let user_id = UserId(123);
    let order_id = OrderId(456);
    
    // This would be a compile error:
    // let wrong = user_id == order_id;
    
    println!("User ID: {:?}, Order ID: {:?}", user_id, order_id);
    
    // Demonstrate state machine
    let order_items = vec![OrderItem {
        product_id: ProductId(1),
        quantity: 2,
        price_cents: 1999,
    }];
    
    let order = OrderState::Created { items: order_items.clone() };
    println!("Order state: {:?}", order);
    
    let paid_order = OrderState::Paid {
        items: order_items,
        payment_id: "payment_123".to_string(),
    };
    println!("Paid order: {:?}", paid_order);
    
    // Demonstrate phantom types
    let db = Database::new("postgresql://localhost/test".to_string());
    
    match db.connect() {
        Ok(connected_db) => {
            // Can only query on connected database
            match connected_db.query("SELECT * FROM users") {
                Ok(results) => println!("Query results: {:?}", results),
                Err(e) => println!("Query error: {}", e),
            }
        }
        Err(e) => println!("Connection error: {}", e),
    }
    
    // Demonstrate type-safe temperatures
    let temp_c = Temperature::celsius(25.0);
    let temp_f = Temperature::fahrenheit(77.0);
    
    println!("25°C = {:.1}°F", temp_c.to_fahrenheit());
    println!("77°F = {:.1}°C", temp_f.to_celsius());
    
    // Demonstrate const generics
    let mut array: FixedArray<i32, 5> = FixedArray::new();
    array.set(0, 42).unwrap();
    array.set(4, 100).unwrap();
    
    println!("Array length: {}", array.len());
    println!("Element at 0: {:?}", array.get(0));
    
    // Demonstrate builder with type states
    let query = QueryBuilder::new()
        .from_table("users")
        .select("id")
        .select("name")
        .where_clause("active = true")
        .where_clause("age >= 18")
        .build();
    
    println!("Generated query: {}", query);
    
    // Demonstrate custom iterator
    let mut iter = NumberIterator::new(5);
    while let Some(value) = iter.next() {
        println!("Iterator value: {}", value);
    }
}
```

Use the type system to make illegal states unrepresentable. This moves  
error detection from runtime to compile time, making your code more  
reliable and easier to reason about.  

## Optimize for readability and maintainability

Write code that clearly expresses intent and is easy to modify, even if  
it means sacrificing minor performance gains.  

```rust
use std::collections::HashMap;

// Good: Clear, self-documenting function names
fn calculate_total_price_including_tax(base_price: f64, tax_rate: f64) -> f64 {
    base_price * (1.0 + tax_rate)
}

fn is_eligible_for_discount(customer_age: u32, is_member: bool, order_total: f64) -> bool {
    const SENIOR_AGE_THRESHOLD: u32 = 65;
    const MEMBER_DISCOUNT_THRESHOLD: f64 = 100.0;
    const SENIOR_DISCOUNT_THRESHOLD: f64 = 50.0;
    
    (customer_age >= SENIOR_AGE_THRESHOLD && order_total >= SENIOR_DISCOUNT_THRESHOLD)
        || (is_member && order_total >= MEMBER_DISCOUNT_THRESHOLD)
}

// Good: Well-structured data types with meaningful names
#[derive(Debug, Clone)]
pub struct Customer {
    pub id: u64,
    pub name: String,
    pub email: String,
    pub age: u32,
    pub is_premium_member: bool,
    pub registration_date: String,
}

#[derive(Debug, Clone)]
pub struct Product {
    pub id: u64,
    pub name: String,
    pub description: String,
    pub base_price: f64,
    pub category: ProductCategory,
    pub is_available: bool,
}

#[derive(Debug, Clone, PartialEq)]
pub enum ProductCategory {
    Electronics,
    Clothing,
    Books,
    HomeAndGarden,
    Sports,
}

#[derive(Debug, Clone)]
pub struct OrderItem {
    pub product: Product,
    pub quantity: u32,
    pub applied_discount: Option<f64>,
}

#[derive(Debug, Clone)]
pub struct ShoppingCart {
    pub customer: Customer,
    pub items: Vec<OrderItem>,
    pub tax_rate: f64,
}

impl ShoppingCart {
    pub fn new(customer: Customer, tax_rate: f64) -> Self {
        Self {
            customer,
            items: Vec::new(),
            tax_rate,
        }
    }
    
    // Good: Small, focused methods that do one thing
    pub fn add_item(&mut self, product: Product, quantity: u32) -> Result<(), String> {
        if !product.is_available {
            return Err(format!("Product '{}' is not available", product.name));
        }
        
        if quantity == 0 {
            return Err("Quantity must be greater than zero".to_string());
        }
        
        // Check if item already exists and update quantity
        if let Some(existing_item) = self.find_item_mut(product.id) {
            existing_item.quantity += quantity;
        } else {
            self.items.push(OrderItem {
                product,
                quantity,
                applied_discount: None,
            });
        }
        
        Ok(())
    }
    
    pub fn remove_item(&mut self, product_id: u64) -> Result<(), String> {
        let initial_len = self.items.len();
        self.items.retain(|item| item.product.id != product_id);
        
        if self.items.len() == initial_len {
            Err(format!("Product with ID {} not found in cart", product_id))
        } else {
            Ok(())
        }
    }
    
    pub fn calculate_subtotal(&self) -> f64 {
        self.items
            .iter()
            .map(|item| self.calculate_item_total(item))
            .sum()
    }
    
    pub fn calculate_total_with_tax(&self) -> f64 {
        let subtotal = self.calculate_subtotal();
        calculate_total_price_including_tax(subtotal, self.tax_rate)
    }
    
    pub fn apply_discounts(&mut self) {
        let customer_qualifies_for_discount = is_eligible_for_discount(
            self.customer.age,
            self.customer.is_premium_member,
            self.calculate_subtotal(),
        );
        
        if customer_qualifies_for_discount {
            self.apply_customer_discount();
        }
        
        self.apply_category_discounts();
    }
    
    pub fn get_order_summary(&self) -> OrderSummary {
        let subtotal = self.calculate_subtotal();
        let tax_amount = subtotal * self.tax_rate;
        let total = subtotal + tax_amount;
        let total_items = self.items.iter().map(|item| item.quantity).sum();
        
        OrderSummary {
            customer_name: self.customer.name.clone(),
            total_items,
            subtotal,
            tax_amount,
            total,
            applied_discounts: self.get_applied_discounts(),
        }
    }
    
    // Private helper methods with clear purposes
    fn find_item_mut(&mut self, product_id: u64) -> Option<&mut OrderItem> {
        self.items.iter_mut().find(|item| item.product.id == product_id)
    }
    
    fn calculate_item_total(&self, item: &OrderItem) -> f64 {
        let base_total = item.product.base_price * item.quantity as f64;
        
        match item.applied_discount {
            Some(discount) => base_total * (1.0 - discount),
            None => base_total,
        }
    }
    
    fn apply_customer_discount(&mut self) {
        const CUSTOMER_DISCOUNT_RATE: f64 = 0.10; // 10% discount
        
        for item in &mut self.items {
            if item.applied_discount.is_none() {
                item.applied_discount = Some(CUSTOMER_DISCOUNT_RATE);
            }
        }
    }
    
    fn apply_category_discounts(&mut self) {
        let category_discounts = self.get_category_discounts();
        
        for item in &mut self.items {
            if let Some(&category_discount) = category_discounts.get(&item.product.category) {
                let current_discount = item.applied_discount.unwrap_or(0.0);
                let total_discount = current_discount + category_discount;
                item.applied_discount = Some(total_discount.min(0.5)); // Cap at 50%
            }
        }
    }
    
    fn get_category_discounts(&self) -> HashMap<ProductCategory, f64> {
        let mut discounts = HashMap::new();
        
        // Apply seasonal discounts based on business logic
        discounts.insert(ProductCategory::Clothing, 0.15);
        discounts.insert(ProductCategory::Electronics, 0.05);
        
        discounts
    }
    
    fn get_applied_discounts(&self) -> Vec<String> {
        let mut discounts = Vec::new();
        
        let has_customer_discount = self.items
            .iter()
            .any(|item| item.applied_discount.is_some());
        
        if has_customer_discount {
            if self.customer.age >= 65 {
                discounts.push("Senior citizen discount".to_string());
            }
            
            if self.customer.is_premium_member {
                discounts.push("Premium member discount".to_string());
            }
        }
        
        let category_discounts = self.get_category_discounts();
        for item in &self.items {
            if category_discounts.contains_key(&item.product.category) {
                discounts.push(format!("{:?} category discount", item.product.category));
            }
        }
        
        discounts.sort();
        discounts.dedup();
        discounts
    }
}

#[derive(Debug)]
pub struct OrderSummary {
    pub customer_name: String,
    pub total_items: u32,
    pub subtotal: f64,
    pub tax_amount: f64,
    pub total: f64,
    pub applied_discounts: Vec<String>,
}

impl OrderSummary {
    pub fn print_detailed_summary(&self) {
        println!("Order Summary for {}", self.customer_name);
        println!("=====================================");
        println!("Total Items: {}", self.total_items);
        println!("Subtotal: ${:.2}", self.subtotal);
        println!("Tax: ${:.2}", self.tax_amount);
        
        if !self.applied_discounts.is_empty() {
            println!("Applied Discounts:");
            for discount in &self.applied_discounts {
                println!("  - {}", discount);
            }
        }
        
        println!("Total: ${:.2}", self.total);
    }
}

// Good: Clear factory functions for test data
#[cfg(test)]
pub fn create_test_customer() -> Customer {
    Customer {
        id: 1,
        name: "Alice Johnson".to_string(),
        email: "alice@example.com".to_string(),
        age: 35,
        is_premium_member: true,
        registration_date: "2023-01-15".to_string(),
    }
}

#[cfg(test)]
pub fn create_test_product(category: ProductCategory, price: f64) -> Product {
    Product {
        id: 1,
        name: "Test Product".to_string(),
        description: "A test product".to_string(),
        base_price: price,
        category,
        is_available: true,
    }
}

fn main() {
    // Demonstrate the shopping cart system
    let customer = Customer {
        id: 1,
        name: "John Doe".to_string(),
        email: "john@example.com".to_string(),
        age: 68,
        is_premium_member: false,
        registration_date: "2022-06-15".to_string(),
    };
    
    let mut cart = ShoppingCart::new(customer, 0.08); // 8% tax rate
    
    // Add some products
    let laptop = Product {
        id: 101,
        name: "Gaming Laptop".to_string(),
        description: "High-performance gaming laptop".to_string(),
        base_price: 1299.99,
        category: ProductCategory::Electronics,
        is_available: true,
    };
    
    let shirt = Product {
        id: 102,
        name: "Cotton T-Shirt".to_string(),
        description: "Comfortable cotton t-shirt".to_string(),
        base_price: 29.99,
        category: ProductCategory::Clothing,
        is_available: true,
    };
    
    // Add items to cart
    cart.add_item(laptop, 1).unwrap();
    cart.add_item(shirt, 2).unwrap();
    
    println!("Cart before discounts:");
    println!("Subtotal: ${:.2}", cart.calculate_subtotal());
    
    // Apply discounts
    cart.apply_discounts();
    
    println!("\\nCart after discounts:");
    let summary = cart.get_order_summary();
    summary.print_detailed_summary();
}

#[cfg(test)]
mod readability_tests {
    use super::*;
    
    #[test]
    fn test_discount_eligibility() {
        // Test senior discount
        assert!(is_eligible_for_discount(70, false, 60.0));
        
        // Test member discount
        assert!(is_eligible_for_discount(30, true, 150.0));
        
        // Test no discount
        assert!(!is_eligible_for_discount(30, false, 50.0));
    }
    
    #[test]
    fn test_cart_operations() {
        let customer = create_test_customer();
        let mut cart = ShoppingCart::new(customer, 0.1);
        
        let product = create_test_product(ProductCategory::Books, 19.99);
        
        assert!(cart.add_item(product, 2).is_ok());
        assert_eq!(cart.items.len(), 1);
        assert_eq!(cart.items[0].quantity, 2);
        
        let subtotal = cart.calculate_subtotal();
        assert_eq!(subtotal, 39.98);
    }
}
```

Prioritize code clarity over cleverness. Use meaningful names, keep  
functions small and focused, and structure code in a way that tells a  
story about what the program does.  

## Establish coding standards and conventions

Maintain consistent code style and conventions across your project to  
improve readability and reduce cognitive load for developers.  

```rust
//! # Project Coding Standards Example
//! 
//! This module demonstrates established coding conventions for Rust projects.
//! Following consistent patterns makes code more maintainable and easier
//! for team members to understand and contribute to.

use std::collections::HashMap;
use std::fmt;
use std::error::Error;

// Convention: Use descriptive names with consistent casing
// - Types: PascalCase
// - Functions and variables: snake_case  
// - Constants: SCREAMING_SNAKE_CASE
// - Modules: snake_case

const DEFAULT_MAX_RETRIES: u32 = 3;
const CONNECTION_TIMEOUT_SECONDS: u64 = 30;

// Convention: Group related functionality in modules
pub mod user_management {
    use super::*;
    
    // Convention: Use doc comments for public APIs
    /// Represents a user in the system with authentication and profile information.
    /// 
    /// # Examples
    /// 
    /// ```
    /// # use std::collections::HashMap;
    /// # pub struct User { pub id: u64, pub username: String, pub email: String, pub metadata: HashMap<String, String> }
    /// # impl User { pub fn new(id: u64, username: String, email: String) -> Self { Self { id, username, email, metadata: HashMap::new() } } }
    /// let user = User::new(1, "alice".to_string(), "alice@example.com".to_string());
    /// ```
    #[derive(Debug, Clone, PartialEq)]
    pub struct User {
        pub id: u64,
        pub username: String,
        pub email: String,
        pub metadata: HashMap<String, String>,
    }
    
    impl User {
        /// Creates a new user with the specified ID, username, and email.
        /// 
        /// Metadata is initialized as an empty HashMap and can be populated
        /// using the `add_metadata` method.
        pub fn new(id: u64, username: String, email: String) -> Self {
            Self {
                id,
                username,
                email,
                metadata: HashMap::new(),
            }
        }
        
        /// Adds a key-value pair to the user's metadata.
        /// 
        /// Returns the previous value if the key already existed.
        pub fn add_metadata(&mut self, key: String, value: String) -> Option<String> {
            self.metadata.insert(key, value)
        }
        
        /// Validates that the user's email address has a basic valid format.
        /// 
        /// This is a simple validation and should not be relied upon for
        /// production email validation.
        pub fn has_valid_email(&self) -> bool {
            self.email.contains('@') && self.email.contains('.')
        }
    }
    
    // Convention: Implement standard traits when appropriate
    impl fmt::Display for User {
        fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
            write!(f, "User({}: {} <{}>)", self.id, self.username, self.email)
        }
    }
    
    // Convention: Create specific error types for different domains
    #[derive(Debug)]
    pub enum UserError {
        InvalidEmail(String),
        DuplicateUsername(String),
        NotFound(u64),
        DatabaseError(String),
    }
    
    impl fmt::Display for UserError {
        fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
            match self {
                UserError::InvalidEmail(email) => {
                    write!(f, "Invalid email address: {}", email)
                }
                UserError::DuplicateUsername(username) => {
                    write!(f, "Username '{}' is already taken", username)
                }
                UserError::NotFound(id) => {
                    write!(f, "User with ID {} not found", id)
                }
                UserError::DatabaseError(msg) => {
                    write!(f, "Database error: {}", msg)
                }
            }
        }
    }
    
    impl Error for UserError {}
    
    // Convention: Use type aliases for complex generic types
    pub type UserResult<T> = Result<T, UserError>;
    
    /// Service for managing users with consistent error handling patterns.
    pub struct UserService {
        users: HashMap<u64, User>,
        next_id: u64,
    }
    
    impl UserService {
        pub fn new() -> Self {
            Self {
                users: HashMap::new(),
                next_id: 1,
            }
        }
        
        /// Creates a new user after validating the input.
        /// 
        /// # Errors
        /// 
        /// Returns `UserError::InvalidEmail` if the email format is invalid.
        /// Returns `UserError::DuplicateUsername` if the username is already taken.
        pub fn create_user(
            &mut self,
            username: String,
            email: String,
        ) -> UserResult<User> {
            // Validation
            if !email.contains('@') || !email.contains('.') {
                return Err(UserError::InvalidEmail(email));
            }
            
            // Check for duplicate username
            if self.users.values().any(|u| u.username == username) {
                return Err(UserError::DuplicateUsername(username));
            }
            
            // Create user
            let user = User::new(self.next_id, username, email);
            self.users.insert(self.next_id, user.clone());
            self.next_id += 1;
            
            Ok(user)
        }
        
        pub fn get_user(&self, id: u64) -> UserResult<&User> {
            self.users.get(&id).ok_or(UserError::NotFound(id))
        }
        
        pub fn list_all_users(&self) -> Vec<&User> {
            self.users.values().collect()
        }
    }
    
    impl Default for UserService {
        fn default() -> Self {
            Self::new()
        }
    }
}

// Convention: Organize configuration in a dedicated module
pub mod config {
    use std::env;
    use std::str::FromStr;
    
    // Convention: Use enums for configuration options
    #[derive(Debug, Clone, Copy, PartialEq)]
    pub enum LogLevel {
        Error,
        Warn,
        Info,
        Debug,
        Trace,
    }
    
    impl FromStr for LogLevel {
        type Err = String;
        
        fn from_str(s: &str) -> Result<Self, Self::Err> {
            match s.to_lowercase().as_str() {
                "error" => Ok(LogLevel::Error),
                "warn" | "warning" => Ok(LogLevel::Warn),
                "info" => Ok(LogLevel::Info),
                "debug" => Ok(LogLevel::Debug),
                "trace" => Ok(LogLevel::Trace),
                _ => Err(format!("Unknown log level: {}", s)),
            }
        }
    }
    
    // Convention: Group related configuration in structs
    #[derive(Debug, Clone)]
    pub struct AppConfig {
        pub server_port: u16,
        pub database_url: String,
        pub log_level: LogLevel,
        pub max_connections: u32,
        pub enable_metrics: bool,
    }
    
    impl AppConfig {
        /// Loads configuration from environment variables with fallback defaults.
        pub fn from_env() -> Self {
            Self {
                server_port: get_env_var("PORT").unwrap_or(8080),
                database_url: get_env_var("DATABASE_URL")
                    .unwrap_or_else(|| "sqlite://app.db".to_string()),
                log_level: get_env_var("LOG_LEVEL").unwrap_or(LogLevel::Info),
                max_connections: get_env_var("MAX_CONNECTIONS").unwrap_or(100),
                enable_metrics: get_env_var("ENABLE_METRICS").unwrap_or(false),
            }
        }
    }
    
    impl Default for AppConfig {
        fn default() -> Self {
            Self {
                server_port: 8080,
                database_url: "sqlite://app.db".to_string(),
                log_level: LogLevel::Info,
                max_connections: 100,
                enable_metrics: false,
            }
        }
    }
    
    // Convention: Helper function for environment variable parsing
    fn get_env_var<T>(key: &str) -> Option<T>
    where
        T: FromStr,
    {
        env::var(key).ok()?.parse().ok()
    }
}

// Convention: Organize tests in a tests module
#[cfg(test)]
mod tests {
    use super::*;
    use user_management::*;
    
    // Convention: Descriptive test names that explain what is being tested
    #[test]
    fn test_user_creation_with_valid_data() {
        let mut service = UserService::new();
        let result = service.create_user(
            "alice".to_string(),
            "alice@example.com".to_string(),
        );
        
        assert!(result.is_ok());
        let user = result.unwrap();
        assert_eq!(user.username, "alice");
        assert_eq!(user.email, "alice@example.com");
        assert_eq!(user.id, 1);
    }
    
    #[test]
    fn test_user_creation_fails_with_invalid_email() {
        let mut service = UserService::new();
        let result = service.create_user(
            "alice".to_string(),
            "invalid-email".to_string(),
        );
        
        assert!(result.is_err());
        match result.unwrap_err() {
            UserError::InvalidEmail(email) => assert_eq!(email, "invalid-email"),
            _ => panic!("Expected InvalidEmail error"),
        }
    }
    
    #[test]
    fn test_duplicate_username_rejection() {
        let mut service = UserService::new();
        
        // Create first user
        service.create_user("alice".to_string(), "alice1@example.com".to_string())
            .expect("First user creation should succeed");
        
        // Try to create second user with same username
        let result = service.create_user("alice".to_string(), "alice2@example.com".to_string());
        
        assert!(result.is_err());
        assert!(matches!(result.unwrap_err(), UserError::DuplicateUsername(_)));
    }
    
    #[test]
    fn test_user_retrieval() {
        let mut service = UserService::new();
        let created_user = service.create_user(
            "bob".to_string(),
            "bob@example.com".to_string(),
        ).unwrap();
        
        let retrieved_user = service.get_user(created_user.id).unwrap();
        assert_eq!(retrieved_user, &created_user);
    }
    
    #[test]
    fn test_nonexistent_user_retrieval() {
        let service = UserService::new();
        let result = service.get_user(999);
        
        assert!(result.is_err());
        assert!(matches!(result.unwrap_err(), UserError::NotFound(999)));
    }
    
    // Convention: Use helper functions for test setup
    fn create_test_users(service: &mut UserService) -> Vec<User> {
        vec![
            service.create_user("alice".to_string(), "alice@example.com".to_string()).unwrap(),
            service.create_user("bob".to_string(), "bob@example.com".to_string()).unwrap(),
            service.create_user("charlie".to_string(), "charlie@example.com".to_string()).unwrap(),
        ]
    }
    
    #[test]
    fn test_list_all_users() {
        let mut service = UserService::new();
        let created_users = create_test_users(&mut service);
        
        let all_users = service.list_all_users();
        assert_eq!(all_users.len(), 3);
        
        // Verify all created users are in the list
        for user in &created_users {
            assert!(all_users.contains(&user));
        }
    }
}

// Convention: Main function demonstrating typical usage
fn main() -> Result<(), Box<dyn Error>> {
    // Load configuration
    let config = config::AppConfig::from_env();
    println!("Application Configuration:");
    println!("  Port: {}", config.server_port);
    println!("  Database: {}", config.database_url);
    println!("  Log Level: {:?}", config.log_level);
    println!("  Max Connections: {}", config.max_connections);
    println!("  Metrics Enabled: {}", config.enable_metrics);
    
    // Create user service and demonstrate usage
    let mut user_service = user_management::UserService::new();
    
    // Create some users
    println!("\\nCreating users...");
    let user1 = user_service.create_user(
        "alice".to_string(),
        "alice@example.com".to_string(),
    )?;
    println!("Created: {}", user1);
    
    let user2 = user_service.create_user(
        "bob".to_string(),
        "bob@example.com".to_string(),
    )?;
    println!("Created: {}", user2);
    
    // Try to create a user with duplicate username
    match user_service.create_user("alice".to_string(), "alice2@example.com".to_string()) {
        Ok(_) => println!("Unexpected success"),
        Err(e) => println!("Expected error: {}", e),
    }
    
    // List all users
    println!("\\nAll users:");
    for user in user_service.list_all_users() {
        println!("  {}", user);
    }
    
    println!("\\nCoding Standards Applied:");
    println!("- Consistent naming conventions");
    println!("- Comprehensive documentation");
    println!("- Proper error handling");
    println!("- Modular organization");
    println!("- Thorough testing");
    println!("- Clear function signatures");
    
    Ok(())
}
```

Consistent coding standards improve team productivity and code quality.  
Establish conventions for naming, organization, documentation, error  
handling, and testing, then enforce them consistently across the project.  

