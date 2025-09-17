# Common Rust Pitfalls

Rust's ownership system and strict compiler can prevent many bugs at  
compile time, but newcomers often encounter common pitfalls that can  
lead to frustration or incorrect code. Understanding these pitfalls  
helps you write better, more idiomatic Rust code and avoid common  
mistakes that can impact performance, safety, or maintainability.  

This guide covers 20 common pitfalls organized by category:  
- **Ownership and borrowing mistakes**: Moving when borrowing is better  
- **Error handling antipatterns**: Ignoring or mishandling errors  
- **Performance pitfalls**: Unnecessary allocations and copies  
- **Type system misuse**: Fighting instead of leveraging Rust's types  
- **Memory safety issues**: Unsafe code and lifetime problems  

Each pitfall is demonstrated with a problematic example followed by  
the correct approach, explaining why the solution works better.  

## Using moved values

Attempting to use a value after it has been moved is one of the most  
common beginner mistakes in Rust.  

```rust
fn main() {
    // Bad: Trying to use moved value
    let original = String::from("Hello, World!");
    let moved = original;  // ownership moves here
    
    // println!("{}", original);  // Error! original is no longer valid
    println!("{}", moved);  // This works
    
    // Solution 1: Clone when you need both values
    let original = String::from("Hello, World!");
    let cloned = original.clone();  // explicit copy
    println!("Original: {}", original);  // Both work
    println!("Cloned: {}", cloned);
    
    // Solution 2: Borrow instead of moving
    let data = String::from("Hello, World!");
    let borrowed = &data;  // borrow, don't move
    println!("Data: {}", data);      // Both work
    println!("Borrowed: {}", borrowed);
    
    // Solution 3: Use references in function calls
    let message = String::from("Important data");
    process_message(&message);  // pass by reference
    println!("Still have: {}", message);  // Still accessible
}

fn process_message(msg: &str) {  // take reference, not ownership
    println!("Processing: {}", msg);
}
```

Prefer borrowing over moving when you don't need to transfer ownership.  
Use cloning sparingly as it can be expensive for large data structures.  

## Mixing mutable and immutable references

Rust prevents data races by disallowing simultaneous mutable and  
immutable references to the same data.  

```rust
fn main() {
    let mut data = vec![1, 2, 3, 4, 5];
    
    // Bad: Mixing reference types in same scope
    let immutable_ref = &data;
    // let mutable_ref = &mut data;  // Error! Can't have both
    // data.push(6);  // Error! Can't mutate while borrowed immutably
    
    println!("Immutable: {:?}", immutable_ref);
    
    // Solution 1: Use scopes to separate references
    {
        let immutable_ref = &data;
        println!("Reading: {:?}", immutable_ref);
    }  // immutable reference goes out of scope
    
    {
        let mutable_ref = &mut data;
        mutable_ref.push(6);
        println!("After mutation: {:?}", mutable_ref);
    }  // mutable reference goes out of scope
    
    // Solution 2: Use references sequentially
    let immutable_ref = &data;
    println!("Data: {:?}", immutable_ref);
    // immutable_ref is no longer used after this point
    
    data.push(7);  // Now we can mutate
    println!("Final: {:?}", data);
}
```

Rust's borrow checker ensures memory safety by preventing data races.  
Use scopes or sequential access to manage reference lifetimes properly.  

## Returning references to local variables

Attempting to return references to local variables that will be  
deallocated when the function returns is a common lifetime error.  

```rust
// Bad: Returning reference to local variable
// fn return_local_reference() -> &str {
//     let local_string = String::from("local");
//     &local_string  // Error! local_string will be dropped
// }

// Solution 1: Return owned values
fn return_owned_string() -> String {
    let local_string = String::from("local");
    local_string  // Transfer ownership to caller
}

// Solution 2: Accept and return references with same lifetime
fn process_and_return<'a>(input: &'a str) -> &'a str {
    input  // Return reference with same lifetime as input
}

// Solution 3: Use static string literals
fn return_static_string() -> &'static str {
    "static string"  // Lives for entire program duration
}

fn main() {
    // Using the correct approaches
    let owned = return_owned_string();
    println!("Owned: {}", owned);
    
    let input = "test input";
    let processed = process_and_return(input);
    println!("Processed: {}", processed);
    
    let static_str = return_static_string();
    println!("Static: {}", static_str);
}
```

Return owned values when creating new data, use lifetime parameters  
when working with borrowed data, or use static references for  
constant data that lives for the entire program duration.  

## Modifying collections while iterating

Attempting to modify a collection while iterating over it can cause  
iterator invalidation and compilation errors.  

```rust
fn main() {
    let mut numbers = vec![1, 2, 3, 4, 5];
    
    // Bad: Modifying collection during iteration
    // for num in &numbers {
    //     if *num % 2 == 0 {
    //         numbers.push(*num * 2);  // Error! Can't modify while borrowed
    //     }
    // }
    
    // Solution 1: Collect indices first, then modify
    let even_indices: Vec<usize> = numbers
        .iter()
        .enumerate()
        .filter(|(_, &value)| value % 2 == 0)
        .map(|(index, _)| index)
        .collect();
    
    for &index in even_indices.iter().rev() {  // reverse to avoid index shifts
        let value = numbers[index];
        numbers.insert(index + 1, value * 2);
    }
    println!("After insertion: {:?}", numbers);
    
    // Solution 2: Create new collection
    let mut numbers = vec![1, 2, 3, 4, 5];
    let mut result = Vec::new();
    
    for num in numbers {
        result.push(num);
        if num % 2 == 0 {
            result.push(num * 2);  // Add doubled even numbers
        }
    }
    println!("New collection: {:?}", result);
    
    // Solution 3: Use drain with filter for removal
    let mut numbers = vec![1, 2, 3, 4, 5, 6, 7, 8];
    numbers.retain(|&x| x % 2 != 0);  // Keep only odd numbers
    println!("After retain: {:?}", numbers);
}
```

Collect indices or create new collections instead of modifying during  
iteration. Use methods like `retain()` or `drain_filter()` for  
safe modification patterns.  

## Unnecessary cloning and allocations

Excessive cloning can hurt performance, especially with large data  
structures. Prefer borrowing when ownership transfer isn't needed.  

```rust
use std::collections::HashMap;

// Bad: Unnecessary cloning
fn process_data_bad(data: &HashMap<String, i32>) -> Vec<String> {
    let mut results = Vec::new();
    
    for (key, value) in data {
        let key_copy = key.clone();  // Unnecessary clone
        let formatted = format!("{}: {}", key_copy, value);
        results.push(formatted);
    }
    
    results
}

// Good: Use references efficiently
fn process_data_good(data: &HashMap<String, i32>) -> Vec<String> {
    data.iter()
        .map(|(key, value)| format!("{}: {}", key, value))  // Use references directly
        .collect()
}

// Bad: Cloning in loops
fn find_max_bad(strings: &[String]) -> Option<String> {
    let mut max_len = 0;
    let mut result = None;
    
    for s in strings {
        if s.len() > max_len {
            max_len = s.len();
            result = Some(s.clone());  // Unnecessary clone
        }
    }
    
    result
}

// Good: Return reference or clone only at the end
fn find_max_good(strings: &[String]) -> Option<&String> {
    strings.iter()
        .max_by_key(|s| s.len())  // Returns reference to longest string
}

fn main() {
    let mut data = HashMap::new();
    data.insert("apple".to_string(), 5);
    data.insert("banana".to_string(), 12);
    data.insert("cherry".to_string(), 8);
    
    let formatted = process_data_good(&data);
    println!("Formatted: {:?}", formatted);
    
    let strings = vec![
        "short".to_string(),
        "medium length".to_string(),
        "very long string indeed".to_string(),
    ];
    
    if let Some(longest) = find_max_good(&strings) {
        println!("Longest: {}", longest);
    }
}
```

Clone only when necessary. Use references and iterator methods to  
avoid unnecessary allocations. Profile your code to identify  
performance bottlenecks caused by excessive cloning.  

## Ignoring error handling

Using `unwrap()` or `expect()` excessively can cause panics in  
production code. Proper error handling improves robustness.  

```rust
use std::fs;
use std::io;

// Bad: Using unwrap everywhere
fn read_config_bad(filename: &str) -> String {
    let content = fs::read_to_string(filename).unwrap();  // Can panic!
    let lines: Vec<&str> = content.lines().collect();
    lines[0].to_string()  // Can panic if file is empty!
}

// Good: Proper error handling
fn read_config_good(filename: &str) -> Result<String, Box<dyn std::error::Error>> {
    let content = fs::read_to_string(filename)?;  // Propagate error
    let first_line = content
        .lines()
        .next()
        .ok_or("File is empty")?;  // Handle empty file case
    Ok(first_line.to_string())
}

// Bad: Ignoring Result types
fn process_files_bad() {
    let files = ["config.txt", "data.txt", "log.txt"];
    
    for file in &files {
        fs::read_to_string(file);  // Warning: unused Result
    }
}

// Good: Handle all possible outcomes
fn process_files_good() -> Result<(), io::Error> {
    let files = ["config.txt", "data.txt", "log.txt"];
    
    for file in &files {
        match fs::read_to_string(file) {
            Ok(content) => println!("Read {} bytes from {}", content.len(), file),
            Err(e) => eprintln!("Failed to read {}: {}", file, e),
        }
    }
    
    Ok(())
}

fn main() {
    // Demonstrate proper error handling
    match read_config_good("config.txt") {
        Ok(config) => println!("Config: {}", config),
        Err(e) => eprintln!("Error reading config: {}", e),
    }
    
    if let Err(e) = process_files_good() {
        eprintln!("Error processing files: {}", e);
    }
}
```

Use `Result` and `Option` types properly. Avoid `unwrap()` in  
library code. Use `?` operator for error propagation and pattern  
matching for explicit error handling.  

## Using String when &str suffices

Unnecessary String allocations when string slices would work can  
impact performance, especially in function parameters.  

```rust
// Bad: Taking ownership when borrowing is sufficient
fn greet_bad(name: String) -> String {
    format!("Hello, {}!", name)  // Forces caller to give up ownership
}

fn print_info_bad(title: String, content: String) {
    println!("{}: {}", title, content);  // Takes ownership unnecessarily
}

// Good: Use string slices for parameters
fn greet_good(name: &str) -> String {
    format!("Hello, {}!", name)  // Can work with both &str and &String
}

fn print_info_good(title: &str, content: &str) {
    println!("{}: {}", title, content);  // Works with borrowed data
}

// Bad: Converting to String unnecessarily
fn process_line_bad(line: &str) -> String {
    let owned = line.to_string();  // Unnecessary allocation
    if owned.starts_with('#') {
        String::new()  // Return empty string
    } else {
        owned.trim().to_string()  // Another allocation
    }
}

// Good: Work with string slices when possible
fn process_line_good(line: &str) -> Option<&str> {
    if line.starts_with('#') {
        None  // Use Option instead of empty string
    } else {
        Some(line.trim())  // Return borrowed slice
    }
}

fn main() {
    let name = "Alice";
    let title = "Info";
    let content = "Some important data";
    
    // These work with string slices
    let greeting = greet_good(name);
    println!("{}", greeting);
    
    print_info_good(title, content);
    
    let lines = ["# Comment", "  actual data  ", "# Another comment"];
    
    for line in &lines {
        if let Some(processed) = process_line_good(line) {
            println!("Processed: '{}'", processed);
        }
    }
}
```

Use `&str` for function parameters when you don't need ownership.  
This makes functions more flexible and avoids unnecessary allocations.  
Convert to `String` only when you need owned data.  

## Fighting the borrow checker with Rc<RefCell<T>>

Overusing `Rc<RefCell<T>>` when simpler ownership patterns would work  
adds runtime overhead and complexity.  

```rust
use std::rc::Rc;
use std::cell::RefCell;

// Bad: Overusing Rc<RefCell<T>> for simple cases
struct BadCounter {
    value: Rc<RefCell<i32>>,
}

impl BadCounter {
    fn new() -> Self {
        BadCounter {
            value: Rc::new(RefCell::new(0)),
        }
    }
    
    fn increment(&self) {
        *self.value.borrow_mut() += 1;  // Runtime borrow checking
    }
    
    fn get(&self) -> i32 {
        *self.value.borrow()  // Runtime borrow checking
    }
}

// Good: Simple ownership for simple cases
struct GoodCounter {
    value: i32,
}

impl GoodCounter {
    fn new() -> Self {
        GoodCounter { value: 0 }
    }
    
    fn increment(&mut self) {  // Clear mutable access
        self.value += 1;
    }
    
    fn get(&self) -> i32 {
        self.value
    }
}

// Rc<RefCell<T>> is appropriate for shared ownership cases
#[derive(Debug)]
struct Node {
    value: i32,
    children: Vec<Rc<RefCell<Node>>>,
}

impl Node {
    fn new(value: i32) -> Rc<RefCell<Self>> {
        Rc::new(RefCell::new(Node {
            value,
            children: Vec::new(),
        }))
    }
    
    fn add_child(parent: &Rc<RefCell<Node>>, child: Rc<RefCell<Node>>) {
        parent.borrow_mut().children.push(child);
    }
}

fn main() {
    // Simple counter doesn't need Rc<RefCell<T>>
    let mut counter = GoodCounter::new();
    counter.increment();
    println!("Counter: {}", counter.get());
    
    // Tree structure benefits from Rc<RefCell<T>>
    let root = Node::new(1);
    let child1 = Node::new(2);
    let child2 = Node::new(3);
    
    Node::add_child(&root, child1);
    Node::add_child(&root, child2);
    
    println!("Root children: {}", root.borrow().children.len());
}
```

Use `Rc<RefCell<T>>` only when you truly need shared ownership with  
interior mutability. For simple cases, prefer clear ownership  
patterns with mutable references.  

## Premature optimization with unsafe code

Using unsafe code for micro-optimizations without measuring performance  
can introduce bugs while providing minimal benefits.  

```rust
// Bad: Unnecessary unsafe optimization
fn sum_unsafe_bad(numbers: &[i32]) -> i32 {
    unsafe {
        let mut sum = 0;
        let ptr = numbers.as_ptr();
        for i in 0..numbers.len() {
            sum += *ptr.add(i);  // Unsafe pointer arithmetic
        }
        sum
    }
}

// Good: Safe, idiomatic code that's likely just as fast
fn sum_safe_good(numbers: &[i32]) -> i32 {
    numbers.iter().sum()  // Compiler optimizes this well
}

// Bad: Manual memory management when Vec would work
fn create_buffer_unsafe_bad(size: usize) -> Vec<u8> {
    unsafe {
        let layout = std::alloc::Layout::array::<u8>(size).unwrap();
        let ptr = std::alloc::alloc(layout);
        if ptr.is_null() {
            panic!("Allocation failed");
        }
        
        // Convert to Vec (dangerous!)
        Vec::from_raw_parts(ptr, size, size)
    }
}

// Good: Use standard library allocations
fn create_buffer_safe_good(size: usize) -> Vec<u8> {
    vec![0; size]  // Safe, clear, and efficient
}

// Appropriate use of unsafe: FFI or proven performance bottlenecks
fn copy_memory_fast(src: &[u8], dst: &mut [u8]) {
    assert_eq!(src.len(), dst.len());
    
    unsafe {
        std::ptr::copy_nonoverlapping(src.as_ptr(), dst.as_mut_ptr(), src.len());
    }
    // This is appropriate when you've measured that it's faster than safe alternatives
}

fn main() {
    let numbers = vec![1, 2, 3, 4, 5];
    
    println!("Sum: {}", sum_safe_good(&numbers));
    
    let buffer = create_buffer_safe_good(1000);
    println!("Buffer size: {}", buffer.len());
    
    // Demonstrate fast copy
    let src = [1, 2, 3, 4, 5];
    let mut dst = [0; 5];
    copy_memory_fast(&src, &mut dst);
    println!("Copied: {:?}", dst);
}
```

Avoid unsafe code for premature optimization. Use safe Rust first,  
profile your code, and only add unsafe optimizations when you've  
proven they're necessary and correct.  

## Overcomplicating generic constraints

Adding unnecessary trait bounds to generic parameters makes code  
harder to understand and maintain.  

```rust
use std::fmt::Debug;

// Bad: Unnecessary trait bounds
fn print_value_bad<T: Debug + Clone + PartialEq + Send + Sync>(value: T) {
    println!("{:?}", value);  // Only uses Debug trait
}

// Good: Only constrain what you actually use
fn print_value_good<T: Debug>(value: T) {
    println!("{:?}", value);
}

// Bad: Complex bounds when simple ones would work
fn process_items_bad<T, F, R>(
    items: Vec<T>,
    processor: F,
) -> Vec<R>
where
    T: Clone + Debug + Send + Sync + 'static,  // Overly restrictive
    F: Fn(T) -> R + Send + Sync + 'static,
    R: Debug + Clone + Send + Sync,
{
    items.into_iter().map(processor).collect()
}

// Good: Minimal necessary bounds
fn process_items_good<T, F, R>(items: Vec<T>, processor: F) -> Vec<R>
where
    F: Fn(T) -> R,  // Only what's needed for the map operation
{
    items.into_iter().map(processor).collect()
}

// Appropriate use of bounds when you need them
fn sort_and_debug<T: Ord + Debug>(mut items: Vec<T>) -> Vec<T> {
    items.sort();  // Requires Ord
    for item in &items {
        println!("{:?}", item);  // Requires Debug
    }
    items
}

fn main() {
    let numbers = vec![3, 1, 4, 1, 5];
    
    // Works with minimal constraints
    let doubled = process_items_good(numbers.clone(), |x| x * 2);
    println!("Doubled: {:?}", doubled);
    
    let sorted = sort_and_debug(numbers);
    println!("Sorted: {:?}", sorted);
}
```

Add trait bounds only when you actually use them. Start with minimal  
constraints and add more only as needed. This makes your code more  
flexible and easier to use.  

## Inefficient iterator usage

Creating unnecessary intermediate collections or not using iterator  
methods idiomatically can hurt performance.  

```rust
// Bad: Collecting intermediate results unnecessarily
fn process_numbers_bad(numbers: Vec<i32>) -> Vec<String> {
    let positive: Vec<i32> = numbers
        .into_iter()
        .filter(|&x| x > 0)
        .collect();  // Unnecessary intermediate collection
    
    let doubled: Vec<i32> = positive
        .into_iter()
        .map(|x| x * 2)
        .collect();  // Another unnecessary collection
    
    doubled
        .into_iter()
        .map(|x| x.to_string())
        .collect()
}

// Good: Chain operations without intermediate collections
fn process_numbers_good(numbers: Vec<i32>) -> Vec<String> {
    numbers
        .into_iter()
        .filter(|&x| x > 0)
        .map(|x| x * 2)
        .map(|x| x.to_string())
        .collect()  // Only collect the final result
}

// Bad: Using manual loops instead of iterator methods
fn find_first_even_bad(numbers: &[i32]) -> Option<i32> {
    for &num in numbers {
        if num % 2 == 0 {
            return Some(num);
        }
    }
    None
}

// Good: Use iterator methods
fn find_first_even_good(numbers: &[i32]) -> Option<i32> {
    numbers.iter().copied().find(|&x| x % 2 == 0)
}

// Bad: Not using iterator adapters efficiently
fn group_by_parity_bad(numbers: Vec<i32>) -> (Vec<i32>, Vec<i32>) {
    let mut evens = Vec::new();
    let mut odds = Vec::new();
    
    for num in numbers {
        if num % 2 == 0 {
            evens.push(num);
        } else {
            odds.push(num);
        }
    }
    
    (evens, odds)
}

// Good: Use partition for better performance
fn group_by_parity_good(numbers: Vec<i32>) -> (Vec<i32>, Vec<i32>) {
    numbers.into_iter().partition(|&x| x % 2 == 0)
}

fn main() {
    let numbers = vec![-2, -1, 0, 1, 2, 3, 4, 5];
    
    let processed = process_numbers_good(numbers.clone());
    println!("Processed: {:?}", processed);
    
    if let Some(first_even) = find_first_even_good(&numbers) {
        println!("First even: {}", first_even);
    }
    
    let (evens, odds) = group_by_parity_good(numbers);
    println!("Evens: {:?}, Odds: {:?}", evens, odds);
}
```

Chain iterator operations without intermediate collections. Use  
iterator methods like `find`, `partition`, and `fold` instead of  
manual loops. This often results in both cleaner and faster code.  

## Misunderstanding Copy vs Clone

Confusion about when types implement Copy versus requiring explicit  
cloning can lead to unexpected moves or unnecessary allocations.  

```rust
// Copy types: primitives, references, and types that derive Copy
#[derive(Debug, Copy, Clone)]
struct Point {
    x: i32,
    y: i32,
}

// Non-Copy type: contains heap-allocated data
#[derive(Debug, Clone)]
struct Person {
    name: String,  // String is not Copy
    age: u32,
}

fn main() {
    // Copy types are implicitly copied
    let x = 42;
    let y = x;  // x is copied, not moved
    println!("x: {}, y: {}", x, y);  // Both are valid
    
    let point1 = Point { x: 10, y: 20 };
    let point2 = point1;  // Copied automatically
    println!("point1: {:?}, point2: {:?}", point1, point2);  // Both valid
    
    // Non-Copy types are moved
    let person1 = Person {
        name: "Alice".to_string(),
        age: 30,
    };
    let person2 = person1;  // person1 is moved
    // println!("{:?}", person1);  // Error! person1 is no longer valid
    println!("{:?}", person2);  // This works
    
    // Explicit cloning when you need both values
    let person3 = Person {
        name: "Bob".to_string(),
        age: 25,
    };
    let person4 = person3.clone();  // Explicit clone
    println!("person3: {:?}", person3);  // Both work
    println!("person4: {:?}", person4);
    
    // Working with vectors: understand the difference
    let numbers = vec![1, 2, 3];  // Vec is not Copy
    let copied_ref = &numbers;     // Borrow, don't move
    let cloned_vec = numbers.clone();  // Explicit clone
    
    println!("Original: {:?}", numbers);     // Still valid
    println!("Reference: {:?}", copied_ref);
    println!("Cloned: {:?}", cloned_vec);
}

// Function that works with Copy types
fn double_copy<T: Copy + std::ops::Add<Output = T>>(value: T) -> T {
    value + value  // value is copied, so we can use it
}

// Function that works with Clone types
fn double_clone<T: Clone + std::ops::Add<Output = T>>(value: T) -> T {
    let cloned = value.clone();
    value + cloned
}
```

Understand which types implement Copy (primitives, references) versus  
Clone (custom types with heap data). Use Copy for cheap duplication,  
Clone for explicit expensive duplication.  

## Incorrect pattern matching with references

Misunderstanding how pattern matching works with references and owned  
values can lead to unnecessary cloning or compilation errors.  

```rust
#[derive(Debug)]
enum Message {
    Text(String),
    Number(i32),
    Quit,
}

fn main() {
    let messages = vec![
        Message::Text("Hello".to_string()),
        Message::Number(42),
        Message::Quit,
    ];
    
    // Bad: Cloning unnecessarily in match
    for message in &messages {
        match message {
            Message::Text(text) => {
                // This moves the String out of the reference, which doesn't work
                // let owned_text = text;  // Error!
                let owned_text = text.clone();  // Unnecessary clone
                println!("Text: {}", owned_text);
            }
            Message::Number(num) => {
                println!("Number: {}", num);  // Numbers are Copy, so this works
            }
            Message::Quit => println!("Quit message"),
        }
    }
    
    // Good: Pattern matching with references
    for message in &messages {
        match message {
            Message::Text(text) => {
                println!("Text: {}", text);  // Use the reference directly
            }
            Message::Number(num) => {
                println!("Number: {}", num);
            }
            Message::Quit => println!("Quit message"),
        }
    }
    
    // Alternative: Use ref in patterns when you need references
    for message in &messages {
        match message {
            Message::Text(ref text) => {  // Explicit ref
                println!("Text length: {}", text.len());
            }
            Message::Number(ref num) => {
                println!("Number: {}", num);
            }
            Message::Quit => println!("Quit message"),
        }
    }
    
    // When you actually need to move out of the collection
    for message in messages {  // Consume the vector
        match message {
            Message::Text(text) => {
                println!("Owned text: {}", text);  // Now we own the String
            }
            Message::Number(num) => {
                println!("Number: {}", num);
            }
            Message::Quit => println!("Quit message"),
        }
    }
    // messages is no longer available here
}

// Function demonstrating proper pattern matching
fn process_option(opt: &Option<String>) -> usize {
    match opt {
        Some(s) => s.len(),  // s is &String here
        None => 0,
    }
}

// Alternative with ref pattern
fn process_option_ref(opt: Option<&String>) -> usize {
    match opt {
        Some(ref s) => s.len(),  // s is &String here too
        None => 0,
    }
}
```

When pattern matching on references, the matched values are also  
references. Use patterns like `ref` when you need to extract  
references, or work directly with the borrowed values.  

## Blocking async functions

Mixing blocking operations with async code can deadlock the async  
runtime or severely hurt performance.  

```rust
use std::time::Duration;
use tokio::time::sleep;

// Bad: Blocking operations in async context
async fn bad_async_function() {
    println!("Starting async work...");
    
    // This blocks the entire async runtime!
    std::thread::sleep(Duration::from_secs(2));  // BAD!
    
    println!("Async work done");
}

// Good: Use async alternatives
async fn good_async_function() {
    println!("Starting async work...");
    
    // This yields control to other tasks
    sleep(Duration::from_secs(2)).await;  // GOOD!
    
    println!("Async work done");
}

// Bad: Synchronous file I/O in async context
async fn read_file_badly() -> Result<String, std::io::Error> {
    // This blocks the runtime!
    std::fs::read_to_string("config.txt")  // BAD!
}

// Good: Use async file I/O
async fn read_file_well() -> Result<String, tokio::io::Error> {
    // This doesn't block other tasks
    tokio::fs::read_to_string("config.txt").await  // GOOD!
}

// When you must use blocking code, use spawn_blocking
async fn use_blocking_code() -> Result<String, Box<dyn std::error::Error>> {
    let result = tokio::task::spawn_blocking(|| {
        // Expensive CPU work or blocking I/O here
        std::thread::sleep(Duration::from_millis(100));
        "Blocking work result".to_string()
    }).await?;
    
    Ok(result)
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    println!("Starting async examples...");
    
    // Run multiple async tasks concurrently
    let task1 = good_async_function();
    let task2 = good_async_function();
    let task3 = use_blocking_code();
    
    // This would block everything:
    // bad_async_function().await;
    
    // These run concurrently:
    tokio::join!(task1, task2, task3);
    
    Ok(())
}
```

Use async alternatives to blocking operations in async contexts.  
When you must use blocking code, wrap it in `spawn_blocking` to  
prevent blocking the async runtime.  

## Dangling pointers with raw pointers

Improper use of raw pointers can lead to use-after-free bugs and  
undefined behavior, especially when converting from references.  

```rust
use std::ptr;

// Bad: Creating dangling pointers
fn create_dangling_pointer() -> *const i32 {
    let local_value = 42;
    &local_value as *const i32  // BAD! local_value will be dropped
}

// Good: Only use raw pointers with valid lifetimes
fn use_valid_pointer(value: &i32) -> *const i32 {
    value as *const i32  // Pointer is valid as long as reference is valid
}

// Bad: Dereferencing potentially invalid pointers
unsafe fn bad_pointer_usage() {
    let ptr = create_dangling_pointer();
    // let value = *ptr;  // UNDEFINED BEHAVIOR!
    println!("This is dangerous: {:p}", ptr);
}

// Good: Safe pointer usage with proper lifetime management
fn safe_pointer_usage() {
    let value = 42;
    let ptr = use_valid_pointer(&value);
    
    unsafe {
        // Safe because value is still alive
        let dereferenced = *ptr;
        println!("Safe dereference: {}", dereferenced);
    }
    // ptr becomes invalid after this point when value is dropped
}

// Bad: Manual memory management without proper cleanup
fn bad_allocation() {
    unsafe {
        let layout = std::alloc::Layout::new::<i32>();
        let ptr = std::alloc::alloc(layout) as *mut i32;
        
        if !ptr.is_null() {
            ptr.write(42);
            // MEMORY LEAK! Forgot to deallocate
        }
    }
}

// Good: Use Vec or Box for heap allocation
fn good_allocation() {
    let heap_value = Box::new(42);
    println!("Heap value: {}", heap_value);
    // Automatically deallocated when Box is dropped
}

// When raw pointers are necessary (FFI, performance), be very careful
fn careful_raw_pointer_usage() {
    let mut vec = vec![1, 2, 3, 4, 5];
    
    unsafe {
        let ptr = vec.as_mut_ptr();
        let len = vec.len();
        
        // This is safe because vec is still alive and we respect bounds
        for i in 0..len {
            let element = ptr.add(i);
            *element *= 2;
        }
    }
    
    println!("Modified vec: {:?}", vec);
}

fn main() {
    safe_pointer_usage();
    good_allocation();
    careful_raw_pointer_usage();
    
    // Don't call bad_pointer_usage() - it's undefined behavior!
}
```

Avoid raw pointers unless absolutely necessary. When you must use  
them, ensure the pointed-to memory remains valid. Prefer Box, Vec,  
and other smart pointers for heap allocation.  

## Incorrect use of Mutex and Arc

Misusing synchronization primitives can lead to deadlocks, poor  
performance, or unnecessary complexity in concurrent code.  

```rust
use std::sync::{Arc, Mutex};
use std::thread;
use std::time::Duration;

// Bad: Holding locks too long
fn bad_lock_usage() {
    let data = Arc::new(Mutex::new(vec![1, 2, 3]));
    let data_clone = Arc::clone(&data);
    
    thread::spawn(move || {
        let mut guard = data_clone.lock().unwrap();
        
        // BAD: Holding lock during expensive operation
        thread::sleep(Duration::from_millis(100));  // Simulating work
        guard.push(4);
        
        // BAD: Holding lock during I/O
        println!("Modified data: {:?}", *guard);
        // Lock held until end of scope
    });
}

// Good: Minimize lock scope
fn good_lock_usage() {
    let data = Arc::new(Mutex::new(vec![1, 2, 3]));
    let data_clone = Arc::clone(&data);
    
    thread::spawn(move || {
        // Do expensive work outside the lock
        thread::sleep(Duration::from_millis(100));
        
        let result = {
            let mut guard = data_clone.lock().unwrap();
            guard.push(4);
            guard.clone()  // Copy data we need
        };  // Lock released here
        
        // I/O outside the lock
        println!("Modified data: {:?}", result);
    });
}

// Bad: Potential deadlock with multiple locks
fn deadlock_example() {
    let data1 = Arc::new(Mutex::new(1));
    let data2 = Arc::new(Mutex::new(2));
    
    let data1_clone = Arc::clone(&data1);
    let data2_clone = Arc::clone(&data2);
    
    let handle1 = thread::spawn(move || {
        let _guard1 = data1_clone.lock().unwrap();
        thread::sleep(Duration::from_millis(10));
        let _guard2 = data2_clone.lock().unwrap();  // Potential deadlock
    });
    
    let handle2 = thread::spawn(move || {
        let _guard2 = data2.lock().unwrap();
        thread::sleep(Duration::from_millis(10));
        let _guard1 = data1.lock().unwrap();  // Potential deadlock
    });
    
    // This might deadlock!
    // handle1.join().unwrap();
    // handle2.join().unwrap();
}

// Good: Consistent lock ordering to prevent deadlocks
fn prevent_deadlock() {
    let data1 = Arc::new(Mutex::new(1));
    let data2 = Arc::new(Mutex::new(2));
    
    let data1_clone = Arc::clone(&data1);
    let data2_clone = Arc::clone(&data2);
    
    let handle1 = thread::spawn(move || {
        // Always acquire locks in the same order
        let _guard1 = data1_clone.lock().unwrap();
        let _guard2 = data2_clone.lock().unwrap();
        // Do work with both locks
    });
    
    let handle2 = thread::spawn(move || {
        // Same order here
        let _guard1 = data1.lock().unwrap();
        let _guard2 = data2.lock().unwrap();
        // Do work with both locks
    });
    
    handle1.join().unwrap();
    handle2.join().unwrap();
}

// Bad: Using Arc<Mutex<T>> when single ownership would work
fn overuse_arc_mutex() {
    let data = Arc::new(Mutex::new(vec![1, 2, 3]));
    
    // If only one thread needs the data, this is overkill
    let result = data.lock().unwrap().iter().sum::<i32>();
    println!("Sum: {}", result);
}

// Good: Use simple ownership when possible
fn simple_ownership() {
    let data = vec![1, 2, 3];
    let result = data.iter().sum::<i32>();
    println!("Sum: {}", result);
}

fn main() {
    good_lock_usage();
    prevent_deadlock();
    simple_ownership();
    
    // Don't run deadlock_example() - it might hang!
    
    // Give threads time to complete
    thread::sleep(Duration::from_millis(200));
}
```

Minimize lock scope, use consistent lock ordering to prevent  
deadlocks, and prefer simpler ownership patterns when you don't  
actually need shared mutable state across threads.  

## Misunderstanding Option and Result chaining

Not leveraging the full power of Option and Result combinators can  
lead to verbose error handling and nested match statements.  

```rust
use std::num::ParseIntError;

// Bad: Nested match statements
fn parse_and_double_bad(s: &str) -> Result<i32, ParseIntError> {
    match s.parse::<i32>() {
        Ok(num) => {
            match num.checked_mul(2) {
                Some(doubled) => Ok(doubled),
                None => Err(s.parse::<i32>().unwrap_err()),  // This is wrong!
            }
        }
        Err(e) => Err(e),
    }
}

// Good: Use combinator methods
fn parse_and_double_good(s: &str) -> Result<Option<i32>, ParseIntError> {
    s.parse::<i32>()
        .map(|num| num.checked_mul(2))  // Result<Option<i32>, ParseIntError>
}

// Bad: Manual Option handling
fn get_first_number_bad(strings: &[String]) -> Option<i32> {
    for s in strings {
        match s.parse::<i32>() {
            Ok(num) => return Some(num),
            Err(_) => continue,
        }
    }
    None
}

// Good: Use iterator methods
fn get_first_number_good(strings: &[String]) -> Option<i32> {
    strings
        .iter()
        .find_map(|s| s.parse().ok())  // Parse and keep first success
}

// Bad: Not using ? operator
fn process_config_bad(filename: &str) -> Result<i32, Box<dyn std::error::Error>> {
    match std::fs::read_to_string(filename) {
        Ok(content) => {
            match content.trim().parse::<i32>() {
                Ok(num) => {
                    if num > 0 {
                        Ok(num * 2)
                    } else {
                        Err("Number must be positive".into())
                    }
                }
                Err(e) => Err(e.into()),
            }
        }
        Err(e) => Err(e.into()),
    }
}

// Good: Use ? operator and combinators
fn process_config_good(filename: &str) -> Result<i32, Box<dyn std::error::Error>> {
    let content = std::fs::read_to_string(filename)?;
    let num: i32 = content.trim().parse()?;
    
    if num > 0 {
        Ok(num * 2)
    } else {
        Err("Number must be positive".into())
    }
}

// Demonstrating useful combinators
fn advanced_option_usage() {
    let numbers = vec!["1", "not_a_number", "3", "4"];
    
    // Collect all successful parses
    let parsed: Vec<i32> = numbers
        .iter()
        .filter_map(|s| s.parse().ok())
        .collect();
    println!("Parsed numbers: {:?}", parsed);
    
    // Get first successful parse with default
    let first_or_default = numbers
        .iter()
        .find_map(|s| s.parse::<i32>().ok())
        .unwrap_or(0);
    println!("First or default: {}", first_or_default);
    
    // Transform and filter in one go
    let doubled_evens: Vec<i32> = numbers
        .iter()
        .filter_map(|s| s.parse::<i32>().ok())
        .filter(|&n| n % 2 == 0)
        .map(|n| n * 2)
        .collect();
    println!("Doubled evens: {:?}", doubled_evens);
}

fn main() {
    // Test the improved functions
    match parse_and_double_good("42") {
        Ok(Some(result)) => println!("Doubled: {}", result),
        Ok(None) => println!("Overflow occurred"),
        Err(e) => println!("Parse error: {}", e),
    }
    
    let strings = vec!["abc".to_string(), "42".to_string(), "def".to_string()];
    if let Some(first_num) = get_first_number_good(&strings) {
        println!("First number: {}", first_num);
    }
    
    advanced_option_usage();
}
```

Use combinator methods like `map`, `and_then`, `filter_map`, and the  
`?` operator to write more concise and readable error handling code.  
Chain operations instead of nesting match statements.  

## Inefficient string concatenation

Building strings inefficiently, especially in loops, can lead to  
quadratic time complexity and poor performance.  

```rust
// Bad: String concatenation with + in loops
fn build_string_bad(words: &[&str]) -> String {
    let mut result = String::new();
    
    for word in words {
        result = result + word + " ";  // Creates new String each time!
    }
    
    result
}

// Good: Use String::push_str for efficient concatenation
fn build_string_good(words: &[&str]) -> String {
    let mut result = String::new();
    
    for word in words {
        result.push_str(word);
        result.push(' ');
    }
    
    result
}

// Better: Use Vec::join when appropriate
fn build_string_better(words: &[&str]) -> String {
    words.join(" ")
}

// Bad: Creating multiple intermediate strings
fn format_data_bad(items: &[i32]) -> String {
    let mut result = String::new();
    
    for item in items {
        let formatted = format!("Item: {}", item);  // Allocation
        let with_newline = formatted + "\n";        // Another allocation
        result = result + &with_newline;            // Yet another allocation
    }
    
    result
}

// Good: Use format! or write! directly
fn format_data_good(items: &[i32]) -> String {
    let mut result = String::new();
    
    for item in items {
        use std::fmt::Write;
        writeln!(&mut result, "Item: {}", item).unwrap();
    }
    
    result
}

// Best: Use iterator methods when possible
fn format_data_best(items: &[i32]) -> String {
    items
        .iter()
        .map(|item| format!("Item: {}", item))
        .collect::<Vec<_>>()
        .join("\n")
}

// Bad: Not pre-allocating when size is known
fn create_repeated_string_bad(word: &str, count: usize) -> String {
    let mut result = String::new();
    
    for _ in 0..count {
        result.push_str(word);  // May reallocate multiple times
    }
    
    result
}

// Good: Pre-allocate capacity when known
fn create_repeated_string_good(word: &str, count: usize) -> String {
    let mut result = String::with_capacity(word.len() * count);
    
    for _ in 0..count {
        result.push_str(word);  // No reallocations needed
    }
    
    result
}

// Alternative: Use repeat method
fn create_repeated_string_best(word: &str, count: usize) -> String {
    word.repeat(count)
}

fn main() {
    let words = ["hello", "world", "from", "rust"];
    
    println!("Joined: '{}'", build_string_better(&words));
    
    let numbers = [1, 2, 3, 4, 5];
    println!("Formatted:\n{}", format_data_best(&numbers));
    
    let repeated = create_repeated_string_best("Hi! ", 5);
    println!("Repeated: '{}'", repeated);
    
    // Demonstrate performance difference
    let large_words: Vec<&str> = (0..1000).map(|_| "word").collect();
    
    let start = std::time::Instant::now();
    let _result1 = build_string_bad(&large_words);
    let bad_time = start.elapsed();
    
    let start = std::time::Instant::now();
    let _result2 = build_string_good(&large_words);
    let good_time = start.elapsed();
    
    println!("Bad method: {:?}", bad_time);
    println!("Good method: {:?}", good_time);
}
```

Use `String::push_str()` instead of `+` for concatenation, pre-allocate  
capacity when known, and prefer methods like `join()` or iterator  
collect for building strings from multiple parts.  

## Memory leaks with reference cycles

Creating reference cycles with `Rc<RefCell<T>>` can prevent memory  
from being deallocated, leading to memory leaks.  

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

// Bad: Creating reference cycles that prevent cleanup
#[derive(Debug)]
struct BadNode {
    value: i32,
    parent: Option<Rc<RefCell<BadNode>>>,
    children: Vec<Rc<RefCell<BadNode>>>,
}

fn create_cycle_bad() {
    let parent = Rc::new(RefCell::new(BadNode {
        value: 1,
        parent: None,
        children: Vec::new(),
    }));
    
    let child = Rc::new(RefCell::new(BadNode {
        value: 2,
        parent: Some(Rc::clone(&parent)),  // Strong reference to parent
        children: Vec::new(),
    }));
    
    parent.borrow_mut().children.push(Rc::clone(&child));  // Strong reference to child
    
    // Reference cycle: parent -> child -> parent
    // Both will never be deallocated!
}

// Good: Use Weak references to break cycles
#[derive(Debug)]
struct GoodNode {
    value: i32,
    parent: Option<Weak<RefCell<GoodNode>>>,  // Weak reference up
    children: Vec<Rc<RefCell<GoodNode>>>,     // Strong references down
}

impl GoodNode {
    fn new(value: i32) -> Rc<RefCell<Self>> {
        Rc::new(RefCell::new(GoodNode {
            value,
            parent: None,
            children: Vec::new(),
        }))
    }
    
    fn add_child(parent: &Rc<RefCell<GoodNode>>, child: Rc<RefCell<GoodNode>>) {
        child.borrow_mut().parent = Some(Rc::downgrade(parent));
        parent.borrow_mut().children.push(child);
    }
    
    fn get_parent(&self) -> Option<Rc<RefCell<GoodNode>>> {
        self.parent.as_ref()?.upgrade()  // Convert Weak to Rc if still alive
    }
}

fn create_tree_good() {
    let root = GoodNode::new(1);
    let child1 = GoodNode::new(2);
    let child2 = GoodNode::new(3);
    
    GoodNode::add_child(&root, child1);
    GoodNode::add_child(&root, child2);
    
    // No reference cycle: parent holds strong refs to children,
    // children hold weak refs to parent
    
    println!("Root children: {}", root.borrow().children.len());
    
    // When root goes out of scope, everything gets cleaned up properly
}

// Alternative: Use indices instead of references
#[derive(Debug)]
struct IndexedNode {
    value: i32,
    parent: Option<usize>,
    children: Vec<usize>,
}

struct Tree {
    nodes: Vec<IndexedNode>,
}

impl Tree {
    fn new() -> Self {
        Tree { nodes: Vec::new() }
    }
    
    fn add_node(&mut self, value: i32, parent: Option<usize>) -> usize {
        let node_id = self.nodes.len();
        
        let node = IndexedNode {
            value,
            parent,
            children: Vec::new(),
        };
        
        if let Some(parent_id) = parent {
            if parent_id < self.nodes.len() {
                self.nodes[parent_id].children.push(node_id);
            }
        }
        
        self.nodes.push(node);
        node_id
    }
    
    fn get_node(&self, id: usize) -> Option<&IndexedNode> {
        self.nodes.get(id)
    }
}

fn create_indexed_tree() {
    let mut tree = Tree::new();
    
    let root = tree.add_node(1, None);
    let child1 = tree.add_node(2, Some(root));
    let child2 = tree.add_node(3, Some(root));
    
    println!("Tree has {} nodes", tree.nodes.len());
    
    if let Some(root_node) = tree.get_node(root) {
        println!("Root has {} children", root_node.children.len());
    }
    
    // No reference counting at all - memory is managed by Vec
}

fn main() {
    // This demonstrates proper cleanup
    create_tree_good();
    
    // This shows an alternative approach
    create_indexed_tree();
    
    // Don't call create_cycle_bad() - it leaks memory!
    
    println!("Memory management demonstrations complete");
}
```

Use `Weak` references to break reference cycles, or consider  
alternative designs like index-based relationships that avoid  
reference counting altogether.  