# Ownership and Borrowing

Rust's ownership system is a unique approach to memory management that  
ensures memory safety without garbage collection. It prevents common  
programming errors like null pointer dereferences, buffer overflows,  
and memory leaks at compile time.  

The ownership system is built on three key concepts:  
- **Ownership**: Each value has a single owner responsible for cleanup  
- **Borrowing**: Temporary access to values without taking ownership  
- **Lifetimes**: Ensure references are valid for as long as needed  

This system allows Rust to provide memory safety guarantees while  
maintaining zero-cost abstractions and predictable performance.  

## Basic Ownership - Move Semantics

This example demonstrates the fundamental concept of ownership transfer  
in Rust. When a value is assigned to a new variable, ownership moves.  

```rust
fn main() {
    let s1 = String::from("Hello");
    let s2 = s1; // Ownership moves from s1 to s2
    
    println!("{}", s2); // This works
    // println!("{}", s1); // This would cause a compile error
    
    let x = 42;
    let y = x; // Copy semantics for primitive types
    
    println!("{}", x); // This works because i32 implements Copy
    println!("{}", y); // Both are valid
}
```

Non-Copy types like String transfer ownership when assigned or passed  
to functions. Copy types like integers are copied instead of moved,  
allowing continued use of the original variable.  

## Copy vs Move Types

This example shows the difference between Copy and Move types and how  
they behave differently in ownership scenarios.  

```rust
fn main() {
    // Copy types: primitives that implement Copy trait
    let a = 10;
    let b = a;
    println!("a: {}, b: {}", a, b); // Both valid
    
    let x = true;
    let y = x;
    println!("x: {}, y: {}", x, y); // Both valid
    
    // Move types: heap-allocated or complex types
    let s1 = String::from("hello");
    let s2 = s1; // Move occurs
    // println!("{}", s1); // Error: s1 is no longer valid
    println!("{}", s2);
    
    let v1 = vec![1, 2, 3];
    let v2 = v1; // Move occurs
    // println!("{:?}", v1); // Error: v1 is no longer valid
    println!("{:?}", v2);
}
```

Copy types include most primitives (integers, floats, booleans, chars)  
and types composed entirely of Copy types. Move types include String,  
Vec, HashMap, and most user-defined types.  

## Clone for Deep Copying

The clone method creates a deep copy of data, allowing you to have  
multiple owned copies of the same data when necessary.  

```rust
fn main() {
    let s1 = String::from("Hello, World!");
    let s2 = s1.clone(); // Explicit deep copy
    
    println!("s1: {}", s1); // Both are valid
    println!("s2: {}", s2); // s1 wasn't moved
    
    let v1 = vec![1, 2, 3, 4, 5];
    let v2 = v1.clone(); // Deep copy of vector
    
    println!("v1: {:?}", v1);
    println!("v2: {:?}", v2);
    
    // Clone is explicit and potentially expensive
    let original = vec![String::from("one"), String::from("two")];
    let cloned = original.clone(); // Clones the vector and all strings
    
    println!("Original: {:?}", original);
    println!("Cloned: {:?}", cloned);
}
```

Clone creates independent copies with separate memory allocations.  
Use clone sparingly as it can be expensive for large data structures.  
Prefer borrowing when you don't need to own the data.  

## Immutable References (Borrowing)

Immutable references allow you to read data without taking ownership.  
Multiple immutable references can exist simultaneously.  

```rust
fn main() {
    let s = String::from("Hello, Rust!");
    
    let r1 = &s; // First immutable reference
    let r2 = &s; // Second immutable reference
    let r3 = &s; // Third immutable reference
    
    println!("r1: {}", r1);
    println!("r2: {}", r2);
    println!("r3: {}", r3);
    println!("original: {}", s); // Original is still accessible
    
    // References don't take ownership
    let len = calculate_length(&s);
    println!("Length of '{}' is {}", s, len);
}

fn calculate_length(s: &String) -> usize {
    s.len() // Can read but not modify
}
```

Immutable references provide read-only access to data. The original  
owner retains ownership and can continue using the value after the  
references go out of scope.  

## Mutable References

Mutable references allow modification of borrowed data. Only one mutable  
reference can exist at a time to prevent data races.  

```rust
fn main() {
    let mut s = String::from("Hello");
    
    {
        let r1 = &mut s;
        r1.push_str(", World!");
        println!("r1: {}", r1);
    } // r1 goes out of scope here
    
    let r2 = &mut s;
    r2.push('!');
    println!("r2: {}", r2);
    
    // Can have mutable reference after previous one is done
    modify_string(&mut s);
    println!("Final: {}", s);
}

fn modify_string(s: &mut String) {
    s.push_str(" - Modified");
}
```

Mutable references enforce exclusive access to prevent data races.  
You cannot have mutable and immutable references active simultaneously  
for the same data.  

## Reference Rules Demonstration

This example shows Rust's reference rules in action, demonstrating  
what's allowed and what would cause compilation errors.  

```rust
fn main() {
    let mut s = String::from("Hello");
    
    // Valid: Multiple immutable references
    let r1 = &s;
    let r2 = &s;
    println!("r1: {}, r2: {}", r1, r2);
    
    // r1 and r2 are no longer used after this point
    
    // Valid: Single mutable reference after immutable ones are done
    let r3 = &mut s;
    r3.push_str(", Rust!");
    println!("r3: {}", r3);
    
    // r3 is no longer used after this point
    
    // Valid: New immutable reference after mutable one is done
    let r4 = &s;
    println!("r4: {}", r4);
    
    // These would cause errors:
    // let r5 = &s;
    // let r6 = &mut s; // Error: cannot borrow as mutable and immutable
}
```

References must follow these rules: any number of immutable references  
OR exactly one mutable reference, but not both simultaneously. This  
prevents data races at compile time.  

## Function Parameter Ownership

Functions can take ownership, borrow immutably, or borrow mutably  
depending on what they need to do with the data.  

```rust
fn main() {
    let s1 = String::from("Hello");
    let mut s2 = String::from("World");
    let s3 = String::from("Rust");
    
    // Function takes ownership
    take_ownership(s1);
    // s1 is no longer valid here
    
    // Function borrows immutably
    let length = calculate_length(&s3);
    println!("Length of '{}' is {}", s3, length); // s3 still valid
    
    // Function borrows mutably
    modify_string(&mut s2);
    println!("Modified: {}", s2); // s2 still valid and modified
    
    // Function returns ownership
    let s4 = create_string();
    println!("Created: {}", s4);
}

fn take_ownership(s: String) {
    println!("Taking ownership of: {}", s);
} // s goes out of scope and is dropped

fn calculate_length(s: &String) -> usize {
    s.len()
} // s goes out of scope but nothing is dropped

fn modify_string(s: &mut String) {
    s.push_str(" modified");
}

fn create_string() -> String {
    String::from("New string")
} // Ownership is transferred to caller
```

Choose the appropriate parameter type based on what the function needs:  
take ownership for consuming the value, borrow for reading, or borrow  
mutably for modification while preserving ownership.  

## Return Values and Ownership

Functions can return ownership of values, enabling transfer of data  
from function scope to the calling scope.  

```rust
fn main() {
    let s1 = create_and_return();
    println!("Returned: {}", s1);
    
    let s2 = String::from("Input");
    let s3 = take_and_return(s2);
    // s2 is no longer valid, but s3 owns the returned value
    println!("Processed: {}", s3);
    
    let s4 = String::from("Original");
    let (s5, length) = calculate_length_and_return(s4);
    // s4 moved in, s5 owns the returned string
    println!("String: {}, Length: {}", s5, length);
}

fn create_and_return() -> String {
    let s = String::from("Created in function");
    s // Return ownership to caller
}

fn take_and_return(mut s: String) -> String {
    s.push_str(" - processed");
    s // Return modified string
}

fn calculate_length_and_return(s: String) -> (String, usize) {
    let length = s.len();
    (s, length) // Return both string and length
}
```

Returning values transfers ownership to the caller. You can return  
tuples to transfer multiple values or return the original value after  
processing to avoid unnecessary moves.  

## Ownership with Structs

Struct fields can own data or hold references, affecting how the  
struct behaves in ownership scenarios.  

```rust
struct Person {
    name: String,    // Owned data
    age: u32,       // Copy type
}

struct PersonRef<'a> {
    name: &'a str,  // Borrowed data with lifetime
    age: u32,
}

fn main() {
    // Struct with owned data
    let person1 = Person {
        name: String::from("Alice"),
        age: 30,
    };
    
    let person2 = person1; // Whole struct moves
    // person1 is no longer valid
    println!("Person2: {} is {} years old", person2.name, person2.age);
    
    // Struct with references
    let name = "Bob";
    let person_ref = PersonRef {
        name: &name,
        age: 25,
    };
    
    println!("PersonRef: {} is {} years old", person_ref.name, person_ref.age);
    // Both name and person_ref are still valid
    
    // Partial moves
    let person3 = Person {
        name: String::from("Charlie"),
        age: 35,
    };
    
    let name_moved = person3.name; // Only name field moves
    let age_copied = person3.age;  // Age is copied (Copy type)
    // person3 is partially moved and no longer fully valid
    
    println!("Name: {}, Age: {}", name_moved, age_copied);
}
```

When a struct contains owned data, moving the struct moves all owned  
fields. Structs with references need lifetime annotations to ensure  
the borrowed data lives long enough.  

## Ownership in Collections

Collections like Vec and HashMap have specific ownership behaviors  
when adding, removing, and iterating over elements.  

```rust
fn main() {
    let mut vec = Vec::new();
    
    // Adding owned values
    vec.push(String::from("First"));
    vec.push(String::from("Second"));
    vec.push(String::from("Third"));
    
    // Accessing without moving
    println!("First element: {}", &vec[0]);
    println!("Length: {}", vec.len());
    
    // Moving out of collection
    let first = vec.remove(0); // Moves the first element out
    println!("Removed: {}", first);
    println!("Remaining: {:?}", vec);
    
    // Iterating by reference (borrowing)
    for item in &vec {
        println!("Item: {}", item);
    }
    println!("Vec still valid: {:?}", vec);
    
    // Iterating by value (consuming)
    for item in vec {
        println!("Consuming: {}", item);
    }
    // vec is no longer valid after consuming iteration
    
    // Working with indices
    let mut numbers = vec![1, 2, 3, 4, 5];
    for i in 0..numbers.len() {
        numbers[i] *= 2; // Modify in place
    }
    println!("Doubled: {:?}", numbers);
}
```

Collections own their elements. Use references for non-consuming access,  
and be aware that some operations like `remove` or consuming iteration  
will transfer ownership of elements out of the collection.  

## Iterator Patterns and Ownership

Different iterator methods have different ownership implications.  
Understanding these patterns is crucial for effective Rust programming.  

```rust
fn main() {
    let words = vec![
        String::from("hello"),
        String::from("world"),
        String::from("rust"),
    ];
    
    // iter() - borrows each element
    let lengths: Vec<usize> = words.iter()
        .map(|s| s.len())
        .collect();
    println!("Lengths: {:?}", lengths);
    println!("Original words still valid: {:?}", words);
    
    // into_iter() - takes ownership of each element
    let uppercase: Vec<String> = words.into_iter()
        .map(|s| s.to_uppercase())
        .collect();
    // words is no longer valid here
    println!("Uppercase: {:?}", uppercase);
    
    // iter_mut() - mutable borrow of each element
    let mut numbers = vec![1, 2, 3, 4, 5];
    numbers.iter_mut().for_each(|x| *x *= 2);
    println!("Modified numbers: {:?}", numbers);
    
    // Chain iterators while managing ownership
    let vec1 = vec![1, 2, 3];
    let vec2 = vec![4, 5, 6];
    
    let combined: Vec<i32> = vec1.iter()
        .chain(vec2.iter())
        .cloned() // Convert &i32 to i32
        .collect();
    
    println!("Combined: {:?}", combined);
    println!("vec1 still valid: {:?}", vec1);
    println!("vec2 still valid: {:?}", vec2);
}
```

Choose the right iterator method: `iter()` for borrowing, `into_iter()`  
for consuming, and `iter_mut()` for mutable borrowing. Use `cloned()`  
or `copied()` when you need owned values from borrowed iterators.  

## Closures and Ownership

Closures can capture variables from their environment by reference,  
mutable reference, or by value, affecting ownership semantics.  

```rust
fn main() {
    let x = 10;
    let y = String::from("Hello");
    
    // Closure captures by reference (default)
    let closure1 = || {
        println!("x: {}, y: {}", x, y);
    };
    
    closure1();
    println!("x and y still accessible: {}, {}", x, y);
    
    // Closure captures by value with 'move'
    let z = String::from("World");
    let closure2 = move || {
        println!("z: {}", z); // z is moved into closure
    };
    
    closure2();
    // println!("{}", z); // Error: z was moved
    
    // Mutable capture
    let mut count = 0;
    let mut increment = || {
        count += 1;
        println!("Count: {}", count);
    };
    
    increment();
    increment();
    println!("Final count: {}", count);
    
    // Function that takes closure
    let numbers = vec![1, 2, 3, 4, 5];
    let doubled = apply_to_each(numbers, |x| x * 2);
    println!("Doubled: {:?}", doubled);
}

fn apply_to_each<F>(vec: Vec<i32>, f: F) -> Vec<i32>
where
    F: Fn(i32) -> i32,
{
    vec.into_iter().map(f).collect()
}
```

Closures automatically capture variables by reference when possible.  
Use the `move` keyword to force capture by value. The closure type  
depends on what it captures and how it's used.  

## Pattern Matching and Ownership

Pattern matching with `match` can move, borrow, or copy values  
depending on how patterns are written.  

```rust
enum Message {
    Text(String),
    Number(i32),
    Quit,
}

fn main() {
    let msg1 = Message::Text(String::from("Hello"));
    
    // Pattern matching moves by default
    match msg1 {
        Message::Text(s) => println!("Text: {}", s), // s owns the String
        Message::Number(n) => println!("Number: {}", n),
        Message::Quit => println!("Quit"),
    }
    // msg1 is no longer valid
    
    let msg2 = Message::Text(String::from("World"));
    
    // Pattern matching with references
    match &msg2 {
        Message::Text(s) => println!("Text ref: {}", s), // s is &String
        Message::Number(n) => println!("Number: {}", n),
        Message::Quit => println!("Quit"),
    }
    println!("msg2 still valid"); // msg2 is still accessible
    
    let msg3 = Message::Number(42);
    
    // Using ref in patterns
    match msg3 {
        Message::Text(ref s) => println!("Text: {}", s),
        Message::Number(n) => println!("Number: {}", n), // Copy type
        Message::Quit => println!("Quit"),
    }
    // msg3 is partially moved (Number variant doesn't move)
    
    // Destructuring with ownership
    let point = (String::from("x"), String::from("y"));
    let (x_coord, y_coord) = point; // Both strings move
    println!("Coordinates: {}, {}", x_coord, y_coord);
    // point is no longer valid
}
```

Pattern matching follows ownership rules: destructuring moves values  
unless you use references (`&`) or the `ref` keyword. Be mindful of  
what gets moved in complex pattern matches.  

## Scope and Lifetime Basics

Understanding scopes and when values are dropped is fundamental to  
working with Rust's ownership system effectively.  

```rust
fn main() {
    {
        let s1 = String::from("Inner scope");
        println!("s1: {}", s1);
    } // s1 goes out of scope and is dropped here
    
    let s2 = String::from("Outer scope");
    
    {
        let s3 = &s2; // Borrow s2
        println!("s3 (borrowed): {}", s3);
        
        let s4 = String::from("Another inner");
        let s5 = s4; // Move s4 to s5
        println!("s5: {}", s5);
        // s4 is no longer valid, s5 will be dropped at end of scope
    } // s3, s5 go out of scope; s3 was a reference so only s5 is dropped
    
    println!("s2 still valid: {}", s2); // s2 is still accessible
    
    // Demonstrating RAII (Resource Acquisition Is Initialization)
    let _file_content = {
        let temp_string = String::from("Temporary data");
        temp_string.clone() // Return a clone, original is dropped
    }; // temp_string is dropped here, but clone is returned
    
    // Conditional scopes
    let condition = true;
    let result = if condition {
        String::from("True branch")
    } else {
        String::from("False branch")
    };
    println!("Result: {}", result);
} // s2, _file_content, result are dropped here
```

Values are automatically dropped when they go out of scope, following  
RAII principles. Understanding scope rules helps predict when memory  
is freed and when references become invalid.  

## Common Ownership Patterns

These patterns demonstrate idiomatic ways to handle ownership in  
common programming scenarios.  

```rust
fn main() {
    // Builder pattern with ownership transfer
    let config = ConfigBuilder::new()
        .set_name("MyApp".to_string())
        .set_version("1.0".to_string())
        .build();
    
    println!("Config: {} v{}", config.name, config.version);
    
    // Processing data with temporary ownership
    let data = vec![1, 2, 3, 4, 5];
    let processed = process_and_return(data);
    println!("Processed: {:?}", processed);
    
    // Conditional ownership transfer
    let mut items = vec![String::from("apple"), String::from("banana")];
    let should_take = true;
    
    if should_take {
        let taken = items.remove(0);
        println!("Took: {}", taken);
    }
    println!("Remaining items: {:?}", items);
    
    // Working with optional ownership
    let maybe_string = Some(String::from("Maybe"));
    let extracted = maybe_string.unwrap_or_else(|| String::from("Default"));
    println!("Extracted: {}", extracted);
}

struct Config {
    name: String,
    version: String,
}

struct ConfigBuilder {
    name: Option<String>,
    version: Option<String>,
}

impl ConfigBuilder {
    fn new() -> Self {
        ConfigBuilder {
            name: None,
            version: None,
        }
    }
    
    fn set_name(mut self, name: String) -> Self {
        self.name = Some(name);
        self
    }
    
    fn set_version(mut self, version: String) -> Self {
        self.version = Some(version);
        self
    }
    
    fn build(self) -> Config {
        Config {
            name: self.name.unwrap_or_else(|| "Unknown".to_string()),
            version: self.version.unwrap_or_else(|| "0.0".to_string()),
        }
    }
}

fn process_and_return(mut data: Vec<i32>) -> Vec<i32> {
    data.iter_mut().for_each(|x| *x *= 2);
    data // Return the modified vector
}
```

These patterns show how to design APIs that work well with Rust's  
ownership system, including builder patterns, data transformation  
pipelines, and conditional ownership transfers.  

## Error Handling with Ownership

Error handling in Rust interacts with ownership, especially when  
dealing with Result types and error propagation.  

```rust
use std::fs::File;
use std::io::{self, Read};

fn main() {
    // Result with owned values
    let result = read_file_contents("example.txt".to_string());
    match result {
        Ok(contents) => println!("File contents: {}", contents),
        Err(e) => println!("Error: {}", e),
    }
    
    // Chaining operations with Results
    let processed = "42"
        .parse::<i32>()
        .map(|n| n * 2)
        .map_err(|_| "Parse error".to_string());
    
    println!("Processed result: {:?}", processed);
    
    // Working with borrowed data in Results
    let data = String::from("123");
    let borrowed_result = parse_number(&data);
    match borrowed_result {
        Ok(num) => println!("Parsed: {}", num),
        Err(msg) => println!("Error: {}", msg),
    }
    println!("Original data still available: {}", data);
    
    // Error propagation with ownership
    let computation_result = perform_computation();
    match computation_result {
        Ok(result) => println!("Computation result: {}", result),
        Err(e) => println!("Computation failed: {}", e),
    }
}

fn read_file_contents(filename: String) -> Result<String, io::Error> {
    let mut file = File::open(filename)?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    Ok(contents)
}

fn parse_number(s: &str) -> Result<i32, &'static str> {
    s.parse().map_err(|_| "Invalid number format")
}

fn perform_computation() -> Result<String, String> {
    let intermediate = calculate_intermediate()
        .map_err(|e| format!("Intermediate calculation failed: {}", e))?;
    
    let final_result = format!("Result: {}", intermediate);
    Ok(final_result)
}

fn calculate_intermediate() -> Result<i32, &'static str> {
    // Simulate a computation that might fail
    Ok(42)
}
```

Result types can contain owned or borrowed data. Use appropriate  
error types for your use case: owned strings for dynamic errors,  
static strings for simple cases, or custom error types for complex  
scenarios.  

## Reference Counting with Rc

Rc (Reference Counted) allows multiple ownership of the same data  
through reference counting, useful for shared ownership scenarios.  

```rust
use std::rc::Rc;

struct Node {
    value: i32,
    children: Vec<Rc<Node>>,
}

fn main() {
    // Creating shared ownership with Rc
    let data = Rc::new(String::from("Shared data"));
    let data_ref1 = Rc::clone(&data);
    let data_ref2 = Rc::clone(&data);
    
    println!("Reference count: {}", Rc::strong_count(&data));
    println!("data: {}", data);
    println!("data_ref1: {}", data_ref1);
    println!("data_ref2: {}", data_ref2);
    
    {
        let data_ref3 = Rc::clone(&data);
        println!("Reference count in inner scope: {}", Rc::strong_count(&data));
        // data_ref3 is dropped here
    }
    
    println!("Reference count after inner scope: {}", Rc::strong_count(&data));
    
    // Tree structure with shared nodes
    let leaf1 = Rc::new(Node {
        value: 1,
        children: vec![],
    });
    
    let leaf2 = Rc::new(Node {
        value: 2,
        children: vec![],
    });
    
    let parent = Rc::new(Node {
        value: 0,
        children: vec![Rc::clone(&leaf1), Rc::clone(&leaf2)],
    });
    
    println!("leaf1 reference count: {}", Rc::strong_count(&leaf1));
    println!("parent value: {}", parent.value);
    
    // Shared computation results
    let expensive_computation = Rc::new(compute_expensive_value());
    let result1 = use_computation(&expensive_computation);
    let result2 = use_computation(&expensive_computation);
    
    println!("Results: {}, {}", result1, result2);
}

fn compute_expensive_value() -> i32 {
    println!("Computing expensive value...");
    42 // Simulate expensive computation
}

fn use_computation(value: &Rc<i32>) -> i32 {
    **value * 2
}
```

Rc provides shared ownership through reference counting. It's useful  
for scenarios where you need multiple owners of the same data, like  
tree structures or shared caches. Note that Rc is not thread-safe.  

## Interior Mutability with RefCell

RefCell allows mutable borrowing of immutable data through runtime  
borrow checking, enabling mutation in otherwise immutable contexts.  

```rust
use std::cell::RefCell;
use std::rc::Rc;

struct Database {
    records: RefCell<Vec<String>>,
}

impl Database {
    fn new() -> Self {
        Database {
            records: RefCell::new(Vec::new()),
        }
    }
    
    fn add_record(&self, record: String) {
        self.records.borrow_mut().push(record);
    }
    
    fn get_records(&self) -> Vec<String> {
        self.records.borrow().clone()
    }
    
    fn record_count(&self) -> usize {
        self.records.borrow().len()
    }
}

fn main() {
    // Interior mutability with RefCell
    let data = RefCell::new(vec![1, 2, 3]);
    
    {
        let mut borrowed = data.borrow_mut();
        borrowed.push(4);
        borrowed.push(5);
    } // Mutable borrow ends here
    
    println!("Data: {:?}", data.borrow());
    
    // Shared mutable state with Rc + RefCell
    let shared_data = Rc::new(RefCell::new(vec![String::from("initial")]));
    
    let data_ref1 = Rc::clone(&shared_data);
    let data_ref2 = Rc::clone(&shared_data);
    
    // Modify through first reference
    data_ref1.borrow_mut().push(String::from("from ref1"));
    
    // Modify through second reference
    data_ref2.borrow_mut().push(String::from("from ref2"));
    
    println!("Shared data: {:?}", shared_data.borrow());
    
    // Database example
    let db = Database::new();
    db.add_record("User 1".to_string());
    db.add_record("User 2".to_string());
    
    println!("Records: {:?}", db.get_records());
    println!("Count: {}", db.record_count());
    
    // Demonstrating runtime borrow checking
    let cell = RefCell::new(42);
    let borrowed1 = cell.borrow();
    let borrowed2 = cell.borrow(); // Multiple immutable borrows OK
    
    println!("borrowed1: {}, borrowed2: {}", *borrowed1, *borrowed2);
    drop(borrowed1);
    drop(borrowed2);
    
    let mut borrowed_mut = cell.borrow_mut(); // Mutable borrow after immutable borrows end
    *borrowed_mut = 100;
    drop(borrowed_mut);
    
    println!("Final value: {}", cell.borrow());
}
```

RefCell enables interior mutability by moving borrow checking to  
runtime. Use it when you need to mutate data through shared  
references. Combine with Rc for shared mutable state.  

## Thread Safety with Arc and Mutex

Arc (Atomically Reference Counted) and Mutex provide thread-safe  
shared ownership and mutation across multiple threads.  

```rust
use std::sync::{Arc, Mutex};
use std::thread;
use std::time::Duration;

fn main() {
    // Shared immutable data across threads
    let data = Arc::new(vec![1, 2, 3, 4, 5]);
    let mut handles = vec![];
    
    for i in 0..3 {
        let data_clone = Arc::clone(&data);
        let handle = thread::spawn(move || {
            println!("Thread {}: {:?}", i, data_clone);
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    // Shared mutable data with Mutex
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    
    for i in 0..10 {
        let counter_clone = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter_clone.lock().unwrap();
            *num += 1;
            println!("Thread {} incremented counter", i);
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Final counter value: {}", *counter.lock().unwrap());
    
    // Producer-consumer pattern
    let buffer = Arc::new(Mutex::new(Vec::new()));
    let buffer_producer = Arc::clone(&buffer);
    let buffer_consumer = Arc::clone(&buffer);
    
    let producer = thread::spawn(move || {
        for i in 0..5 {
            {
                let mut buf = buffer_producer.lock().unwrap();
                buf.push(i);
                println!("Produced: {}", i);
            }
            thread::sleep(Duration::from_millis(100));
        }
    });
    
    let consumer = thread::spawn(move || {
        loop {
            let mut buf = buffer_consumer.lock().unwrap();
            if let Some(item) = buf.pop() {
                println!("Consumed: {}", item);
                drop(buf); // Explicitly drop lock
                thread::sleep(Duration::from_millis(150));
            } else {
                drop(buf);
                thread::sleep(Duration::from_millis(50));
            }
        }
    });
    
    producer.join().unwrap();
    thread::sleep(Duration::from_millis(1000)); // Let consumer finish
}
```

Arc provides thread-safe reference counting, while Mutex provides  
thread-safe mutable access. This combination enables safe sharing  
of mutable data across threads, preventing data races.  

## Advanced Lifetime Scenarios

Lifetimes ensure that references remain valid for as long as they're  
used, preventing dangling pointer errors.  

```rust
fn main() {
    // Basic lifetime relationships
    let string1 = String::from("Hello");
    let string2 = String::from("World!");
    
    let result = longest(&string1, &string2);
    println!("Longest string: {}", result);
    
    // Lifetime with structs
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find '.'");
    let excerpt = Excerpt {
        part: first_sentence,
    };
    println!("Excerpt: {}", excerpt.part);
    
    // Multiple lifetime parameters
    let x = "short";
    let y = "a much longer string";
    let announcement = "Breaking news!";
    
    let result = announce_and_return_longest(x, y, announcement);
    println!("Result: {}", result);
    
    // Lifetime elision in practice
    let numbers = vec![1, 2, 3, 4, 5];
    let first_two = get_first_two(&numbers);
    println!("First two: {:?}", first_two);
    
    // Static lifetime
    let static_str: &'static str = "This string lives for the entire program";
    println!("Static: {}", static_str);
    
    // Lifetime bounds with generics
    let wrapper = Wrapper { value: &string1 };
    wrapper.print_value();
}

// Function with explicit lifetime annotation
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

// Struct with lifetime parameter
struct Excerpt<'a> {
    part: &'a str,
}

// Multiple lifetime parameters
fn announce_and_return_longest<'a, 'b>(
    x: &'a str,
    y: &'a str,
    announcement: &'b str,
) -> &'a str {
    println!("Attention! {}", announcement);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

// Lifetime elision (compiler infers lifetimes)
fn get_first_two(slice: &[i32]) -> &[i32] {
    &slice[..2.min(slice.len())]
}

// Generic struct with lifetime bound
struct Wrapper<'a, T: 'a> {
    value: &'a T,
}

impl<'a, T: std::fmt::Display + 'a> Wrapper<'a, T> {
    fn print_value(&self) {
        println!("Wrapped value: {}", self.value);
    }
}
```

Lifetimes ensure memory safety by preventing use of invalid references.  
The compiler often infers lifetimes, but explicit annotations are  
needed when relationships are ambiguous.  

## Ownership Best Practices

These examples demonstrate idiomatic patterns and best practices for  
working effectively with Rust's ownership system.  

```rust
use std::collections::HashMap;

fn main() {
    // Prefer borrowing over cloning
    let data = vec![1, 2, 3, 4, 5];
    process_data(&data); // Borrow instead of moving
    println!("Data still available: {:?}", data);
    
    // Use string slices for parameters when possible
    let owned_string = String::from("Hello, Rust!");
    print_message(&owned_string); // Can accept both String and &str
    print_message("Direct string literal");
    
    // Return owned data from functions when creating new values
    let generated = generate_data(10);
    println!("Generated: {:?}", generated);
    
    // Use Option and Result for error handling
    let maybe_value = safe_divide(10.0, 2.0);
    match maybe_value {
        Some(result) => println!("Division result: {}", result),
        None => println!("Division by zero!"),
    }
    
    // Efficient string building
    let words = vec!["Hello", "beautiful", "world"];
    let sentence = build_sentence(&words);
    println!("Sentence: {}", sentence);
    
    // Working with hashmaps efficiently
    let mut scores = HashMap::new();
    add_score(&mut scores, "Alice".to_string(), 95);
    add_score(&mut scores, "Bob".to_string(), 87);
    
    if let Some(score) = get_score(&scores, "Alice") {
        println!("Alice's score: {}", score);
    }
    
    // Chaining operations efficiently
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    let even_squares: Vec<i32> = numbers
        .iter()
        .filter(|&&x| x % 2 == 0)
        .map(|&x| x * x)
        .collect();
    
    println!("Even squares: {:?}", even_squares);
    println!("Original numbers still available: {:?}", numbers);
}

fn process_data(data: &[i32]) {
    let sum: i32 = data.iter().sum();
    println!("Sum: {}", sum);
}

fn print_message(message: &str) {
    println!("Message: {}", message);
}

fn generate_data(size: usize) -> Vec<i32> {
    (1..=size as i32).collect()
}

fn safe_divide(a: f64, b: f64) -> Option<f64> {
    if b != 0.0 {
        Some(a / b)
    } else {
        None
    }
}

fn build_sentence(words: &[&str]) -> String {
    words.join(" ")
}

fn add_score(scores: &mut HashMap<String, i32>, name: String, score: i32) {
    scores.insert(name, score);
}

fn get_score(scores: &HashMap<String, i32>, name: &str) -> Option<&i32> {
    scores.get(name)
}
```

Follow these patterns: prefer borrowing over moving, use string slices  
for parameters, return owned data when creating new values, and  
leverage iterators for efficient data processing while maintaining  
ownership clarity.  

## Common Ownership Pitfalls

Understanding common mistakes helps avoid compilation errors and  
write more idiomatic Rust code.  

```rust
fn main() {
    // Pitfall 1: Trying to use moved values
    let s1 = String::from("Hello");
    let s2 = s1; // s1 is moved
    // println!("{}", s1); // Error! s1 is no longer valid
    println!("{}", s2); // This works
    
    // Solution: Clone if you need both
    let s3 = String::from("Hello");
    let s4 = s3.clone(); // Explicit clone
    println!("s3: {}, s4: {}", s3, s4); // Both work
    
    // Pitfall 2: Mixing mutable and immutable references
    let mut data = vec![1, 2, 3];
    let immutable_ref = &data;
    // data.push(4); // Error! Cannot mutate while borrowed immutably
    println!("Immutable ref: {:?}", immutable_ref);
    data.push(4); // This works after immutable reference is done
    
    // Pitfall 3: Returning references to local data
    // let bad_ref = return_local_reference(); // This would not compile
    let good_string = return_owned_string(); // This works
    println!("Good string: {}", good_string);
    
    // Pitfall 4: Iterator invalidation
    let mut numbers = vec![1, 2, 3, 4, 5];
    // for num in &numbers {
    //     if *num == 3 {
    //         numbers.push(6); // Error! Cannot modify while iterating
    //     }
    // }
    
    // Solution: Collect indices first or use different approach
    let indices_to_remove: Vec<usize> = numbers
        .iter()
        .enumerate()
        .filter(|(_, &value)| value % 2 == 0)
        .map(|(index, _)| index)
        .collect();
    
    for &index in indices_to_remove.iter().rev() {
        numbers.remove(index);
    }
    println!("After removal: {:?}", numbers);
    
    // Pitfall 5: Partial moves from structs
    let person = Person {
        name: String::from("Alice"),
        age: 30,
    };
    
    let name = person.name; // Partial move
    // println!("{:?}", person); // Error! person is partially moved
    println!("Name: {}, Age: {}", name, person.age); // age is still accessible
    
    // Solution: Use references or destructuring
    let person2 = Person {
        name: String::from("Bob"),
        age: 25,
    };
    
    let Person { name: person_name, age } = person2; // Destructure completely
    println!("Destructured - Name: {}, Age: {}", person_name, age);
}

#[derive(Debug)]
struct Person {
    name: String,
    age: u32,
}

// This function would not compile
// fn return_local_reference() -> &String {
//     let s = String::from("local");
//     &s // Error! Returning reference to local variable
// }

fn return_owned_string() -> String {
    String::from("owned") // Return owned value
}
```

Avoid these common pitfalls: using moved values, mixing reference  
types, returning local references, modifying during iteration, and  
partial moves. Understanding these helps write correct Rust code  
from the start.  

## Performance Considerations

Understanding the performance implications of different ownership  
patterns helps write efficient Rust code.  

```rust
use std::time::Instant;

fn main() {
    // Benchmark: Clone vs Borrow
    let data = vec![1; 1_000_000];
    
    let start = Instant::now();
    for _ in 0..100 {
        process_by_clone(data.clone()); // Expensive clone
    }
    let clone_time = start.elapsed();
    
    let start = Instant::now();
    for _ in 0..100 {
        process_by_borrow(&data); // Cheap borrow
    }
    let borrow_time = start.elapsed();
    
    println!("Clone time: {:?}", clone_time);
    println!("Borrow time: {:?}", borrow_time);
    
    // String operations: &str vs String
    let static_str = "Hello, World!";
    let owned_string = String::from("Hello, World!");
    
    let start = Instant::now();
    for _ in 0..1_000_000 {
        process_str_slice(static_str); // Very fast
    }
    let str_time = start.elapsed();
    
    let start = Instant::now();
    for _ in 0..1_000_000 {
        process_string(&owned_string); // Fast borrow
    }
    let string_borrow_time = start.elapsed();
    
    let start = Instant::now();
    for _ in 0..100_000 {
        process_string_owned(owned_string.clone()); // Expensive clone
    }
    let string_clone_time = start.elapsed();
    
    println!("&str processing: {:?}", str_time);
    println!("String borrow: {:?}", string_borrow_time);
    println!("String clone: {:?}", string_clone_time);
    
    // Efficient collection operations
    let numbers: Vec<i32> = (1..=1000).collect();
    
    // Inefficient: Multiple allocations
    let start = Instant::now();
    let mut result = Vec::new();
    for &num in &numbers {
        if num % 2 == 0 {
            result.push(num * num);
        }
    }
    let inefficient_time = start.elapsed();
    
    // Efficient: Iterator chain with size hint
    let start = Instant::now();
    let result2: Vec<i32> = numbers
        .iter()
        .filter(|&&x| x % 2 == 0)
        .map(|&x| x * x)
        .collect();
    let efficient_time = start.elapsed();
    
    println!("Inefficient loop: {:?}", inefficient_time);
    println!("Efficient iterator: {:?}", efficient_time);
    
    // Memory usage patterns
    demonstrate_memory_efficiency();
}

fn process_by_clone(data: Vec<i32>) -> usize {
    data.len()
}

fn process_by_borrow(data: &[i32]) -> usize {
    data.len()
}

fn process_str_slice(s: &str) -> usize {
    s.len()
}

fn process_string(s: &str) -> usize {
    s.len()
}

fn process_string_owned(s: String) -> usize {
    s.len()
}

fn demonstrate_memory_efficiency() {
    // Avoid unnecessary allocations
    let data = "hello,world,rust,programming";
    
    // Good: No allocations, just borrowing
    let parts: Vec<&str> = data.split(',').collect();
    println!("Parts count: {}", parts.len());
    
    // Less efficient: Creates owned strings
    let owned_parts: Vec<String> = data
        .split(',')
        .map(|s| s.to_string())
        .collect();
    println!("Owned parts count: {}", owned_parts.len());
    
    // Reuse allocations when possible
    let mut buffer = String::new();
    for i in 0..5 {
        buffer.clear(); // Reuse existing allocation
        buffer.push_str(&format!("Item {}", i));
        println!("Buffer: {}", buffer);
    }
}
```

Prefer borrowing over cloning for performance. Use string slices when  
possible, leverage iterator chains for efficiency, and reuse  
allocations when appropriate. Profile your code to identify  
performance bottlenecks related to ownership.  

## Real-World Usage Patterns

These examples show how ownership and borrowing work in practical,  
real-world scenarios you'll encounter in Rust applications.  

```rust
use std::collections::HashMap;
use std::fs::File;
use std::io::{BufRead, BufReader, Result};

// Configuration management
#[derive(Debug, Clone)]
struct AppConfig {
    database_url: String,
    api_key: String,
    max_connections: u32,
}

impl AppConfig {
    fn new(database_url: String, api_key: String) -> Self {
        AppConfig {
            database_url,
            api_key,
            max_connections: 10,
        }
    }
    
    fn with_max_connections(mut self, max_connections: u32) -> Self {
        self.max_connections = max_connections;
        self
    }
}

// Data processing pipeline
struct DataProcessor {
    filters: Vec<Box<dyn Fn(&str) -> bool>>,
    transformers: Vec<Box<dyn Fn(&str) -> String>>,
}

impl DataProcessor {
    fn new() -> Self {
        DataProcessor {
            filters: Vec::new(),
            transformers: Vec::new(),
        }
    }
    
    fn add_filter<F>(mut self, filter: F) -> Self
    where
        F: Fn(&str) -> bool + 'static,
    {
        self.filters.push(Box::new(filter));
        self
    }
    
    fn add_transformer<F>(mut self, transformer: F) -> Self
    where
        F: Fn(&str) -> String + 'static,
    {
        self.transformers.push(Box::new(transformer));
        self
    }
    
    fn process(&self, input: &str) -> Option<String> {
        // Apply filters
        for filter in &self.filters {
            if !filter(input) {
                return None;
            }
        }
        
        // Apply transformations
        let mut result = input.to_string();
        for transformer in &self.transformers {
            result = transformer(&result);
        }
        
        Some(result)
    }
}

// Cache implementation
struct Cache<K, V> {
    data: HashMap<K, V>,
    max_size: usize,
}

impl<K, V> Cache<K, V>
where
    K: std::hash::Hash + Eq + Clone,
    V: Clone,
{
    fn new(max_size: usize) -> Self {
        Cache {
            data: HashMap::new(),
            max_size,
        }
    }
    
    fn get(&self, key: &K) -> Option<&V> {
        self.data.get(key)
    }
    
    fn insert(&mut self, key: K, value: V) -> Option<V> {
        if self.data.len() >= self.max_size {
            // Simple eviction: remove a random entry
            if let Some(old_key) = self.data.keys().next().cloned() {
                self.data.remove(&old_key);
            }
        }
        self.data.insert(key, value)
    }
}

fn main() -> Result<()> {
    // Configuration pattern
    let config = AppConfig::new(
        "postgresql://localhost/mydb".to_string(),
        "secret_key_123".to_string(),
    ).with_max_connections(20);
    
    println!("Config: {:?}", config);
    
    // Data processing pipeline
    let processor = DataProcessor::new()
        .add_filter(|s| !s.is_empty())
        .add_filter(|s| s.len() > 3)
        .add_transformer(|s| s.to_uppercase())
        .add_transformer(|s| format!("PROCESSED: {}", s));
    
    let test_data = vec!["hello", "hi", "", "world", "rust"];
    for data in test_data {
        if let Some(processed) = processor.process(data) {
            println!("'{}' -> '{}'", data, processed);
        } else {
            println!("'{}' was filtered out", data);
        }
    }
    
    // Cache usage
    let mut cache = Cache::new(3);
    cache.insert("user:1".to_string(), "Alice".to_string());
    cache.insert("user:2".to_string(), "Bob".to_string());
    cache.insert("user:3".to_string(), "Charlie".to_string());
    
    if let Some(user) = cache.get(&"user:1".to_string()) {
        println!("Found user: {}", user);
    }
    
    // File processing with proper resource management
    if let Ok(lines) = read_lines_efficiently("sample.txt") {
        for line in lines.take(5) { // Only process first 5 lines
            println!("Line: {}", line);
        }
    }
    
    // Working with borrowed data in complex scenarios
    let data = vec![
        ("apple", 10),
        ("banana", 5),
        ("cherry", 15),
        ("date", 8),
    ];
    
    let expensive_items = find_expensive_items(&data, 9);
    println!("Expensive items: {:?}", expensive_items);
    
    // Resource cleanup demonstration
    {
        let _resource = ManagedResource::new("database_connection");
        // Resource is automatically cleaned up when it goes out of scope
    }
    println!("Resource has been cleaned up");
    
    Ok(())
}

fn read_lines_efficiently(filename: &str) -> Result<impl Iterator<Item = String>> {
    let file = File::open(filename)?;
    let reader = BufReader::new(file);
    Ok(reader.lines().map(|line| line.unwrap_or_else(|_| String::new())))
}

fn find_expensive_items(items: &[(&str, i32)], threshold: i32) -> Vec<&str> {
    items
        .iter()
        .filter(|(_, price)| *price > threshold)
        .map(|(name, _)| *name)
        .collect()
}

// RAII pattern for resource management
struct ManagedResource {
    name: String,
}

impl ManagedResource {
    fn new(name: &str) -> Self {
        println!("Acquiring resource: {}", name);
        ManagedResource {
            name: name.to_string(),
        }
    }
}

impl Drop for ManagedResource {
    fn drop(&mut self) {
        println!("Releasing resource: {}", self.name);
    }
}
```

These patterns demonstrate practical ownership usage: configuration  
builders, data processing pipelines, caching systems, file processing,  
and resource management. They show how Rust's ownership system  
enables safe, efficient, and maintainable code.

## Ownership with Enums and Option

Enums and Option types have special ownership characteristics that  
are important to understand for effective Rust programming.  

```rust
#[derive(Debug)]
enum Vehicle {
    Car { brand: String, model: String },
    Bike { brand: String },
    Truck { capacity: u32 },
}

fn main() {
    // Creating enum variants with owned data
    let car = Vehicle::Car {
        brand: String::from("Toyota"),
        model: String::from("Camry"),
    };
    
    // Moving enum values
    let my_vehicle = car; // car is moved
    // println!("{:?}", car); // Error: car is no longer valid
    println!("My vehicle: {:?}", my_vehicle);
    
    // Working with Option<T>
    let maybe_name: Option<String> = Some(String::from("Alice"));
    
    // Pattern matching moves the inner value
    match maybe_name {
        Some(name) => {
            println!("Name: {}", name);
            // name owns the String now
        }
        None => println!("No name provided"),
    }
    // maybe_name is no longer valid after the match
    
    // Using as_ref() to avoid moving
    let maybe_data = Some(String::from("Important data"));
    match maybe_data.as_ref() {
        Some(data) => println!("Data: {}", data), // data is &String
        None => println!("No data"),
    }
    println!("Original data still available: {:?}", maybe_data);
    
    // take() method moves the value out of Option
    let mut optional_value = Some(String::from("Take me"));
    let taken = optional_value.take(); // Replaces with None
    println!("Taken: {:?}", taken);
    println!("Optional after take: {:?}", optional_value);
    
    // Cloning enum variants when needed
    let bike = Vehicle::Bike {
        brand: String::from("Trek"),
    };
    
    let bike_copy = match &bike {
        Vehicle::Bike { brand } => Vehicle::Bike { brand: brand.clone() },
        _ => panic!("Not a bike"),
    };
    
    println!("Original: {:?}", bike);
    println!("Copy: {:?}", bike_copy);
}
```

Enums move their data when pattern matched unless you use references.  
Option's methods like `as_ref()`, `take()`, and `map()` provide  
different ways to work with ownership of the contained value.  

## Ownership in Generic Functions

Generic functions must respect ownership rules while working with  
different types that may or may not implement Copy.  

```rust
use std::fmt::Display;

fn main() {
    // Generic function that takes ownership
    let numbers = vec![1, 2, 3];
    let result = process_container(numbers);
    // numbers is no longer valid
    println!("Processed: {:?}", result);
    
    // Generic function that borrows
    let strings = vec![String::from("hello"), String::from("world")];
    let length = calculate_total_length(&strings);
    println!("Total length: {}, strings still valid: {:?}", length, strings);
    
    // Generic function with Clone bound
    let original_data = vec![10, 20, 30];
    let doubled = duplicate_and_double(&original_data);
    println!("Original: {:?}, Doubled: {:?}", original_data, doubled);
    
    // Generic function with different ownership patterns
    let value1 = 42;  // Copy type
    let value2 = String::from("Hello");  // Move type
    
    print_and_return(value1); // value1 is copied
    println!("value1 still available: {}", value1);
    
    let returned_string = print_and_return(value2);
    // value2 is moved, but we get it back
    println!("Returned string: {}", returned_string);
    
    // Working with trait objects and ownership
    let items: Vec<Box<dyn Display>> = vec![
        Box::new(42),
        Box::new(String::from("Hello")),
        Box::new(3.14),
    ];
    
    display_all(&items);
}

// Generic function that takes ownership
fn process_container<T>(mut container: Vec<T>) -> Vec<T>
where
    T: Clone,
{
    container.reverse();
    container
}

// Generic function that borrows
fn calculate_total_length<T>(items: &[T]) -> usize
where
    T: AsRef<str>,
{
    items.iter().map(|item| item.as_ref().len()).sum()
}

// Generic function with Clone bound
fn duplicate_and_double<T>(items: &[T]) -> Vec<T>
where
    T: Clone + std::ops::Add<Output = T>,
{
    items.iter().map(|item| item.clone() + item.clone()).collect()
}

// Generic function that takes and returns ownership
fn print_and_return<T: Display>(value: T) -> T {
    println!("Value: {}", value);
    value // Return ownership to caller
}

// Working with trait objects
fn display_all(items: &[Box<dyn Display>]) {
    for item in items {
        println!("Item: {}", item);
    }
}
```

Generic functions must be designed carefully to work with both Copy  
and Move types. Use trait bounds like Clone when you need to duplicate  
values, and consider ownership transfer patterns in your API design.  

## Ownership with Async and Futures

Async programming in Rust has unique ownership requirements due to  
the need to move data across await points.  

```rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll};
use std::time::Duration;

// Simulated async runtime for demonstration
struct SimpleExecutor;

impl SimpleExecutor {
    fn block_on<F: Future>(future: F) -> F::Output {
        // Simplified executor - in real code, use tokio or async-std
        std::thread::sleep(Duration::from_millis(100));
        // This is a mock - real executors are much more complex
        unreachable!("This is for demonstration only")
    }
}

fn main() {
    // Note: This example shows concepts but won't run without a real async runtime
    
    // Async functions and ownership
    let data = String::from("Async data");
    
    // This would move data into the async block
    let future = async move {
        process_async_data(data).await
    };
    
    // data is no longer accessible here due to 'move'
    
    // Shared ownership in async contexts
    use std::sync::Arc;
    
    let shared_data = Arc::new(String::from("Shared async data"));
    let shared_data_clone = Arc::clone(&shared_data);
    
    let future1 = async move {
        println!("Future 1: {}", shared_data_clone);
        simulate_async_work().await;
        shared_data_clone.len()
    };
    
    let shared_data_clone2 = Arc::clone(&shared_data);
    let future2 = async move {
        println!("Future 2: {}", shared_data_clone2);
        simulate_async_work().await;
        shared_data_clone2.chars().count()
    };
    
    // Both futures can run concurrently
    println!("Original data still accessible: {}", shared_data);
    
    // Working with references in async
    demonstrate_async_borrowing();
}

async fn process_async_data(data: String) -> usize {
    simulate_async_work().await;
    data.len()
}

async fn simulate_async_work() {
    // In real async code, this would be an actual async operation
    // like reading from a file, making a network request, etc.
}

// Demonstrating borrowing challenges in async
fn demonstrate_async_borrowing() {
    let data = vec![1, 2, 3, 4, 5];
    
    // This won't work in real async code because references
    // can't cross await points safely:
    /*
    let future = async {
        let borrowed = &data; // Borrow data
        some_async_operation().await; // Await point
        println!("{:?}", borrowed); // Use after await - potential issue
    };
    */
    
    // Solution: Move or clone the data
    let data_clone = data.clone();
    let future = async move {
        let local_data = data_clone; // Own the data
        simulate_async_work().await;
        println!("Processed data: {:?}", local_data);
    };
    
    // Original data is still available
    println!("Original data: {:?}", data);
}

// Custom future demonstrating ownership
struct CustomFuture {
    data: Option<String>,
}

impl CustomFuture {
    fn new(data: String) -> Self {
        CustomFuture { data: Some(data) }
    }
}

impl Future for CustomFuture {
    type Output = String;
    
    fn poll(mut self: Pin<&mut Self>, _cx: &mut Context<'_>) -> Poll<Self::Output> {
        if let Some(data) = self.data.take() {
            Poll::Ready(format!("Processed: {}", data))
        } else {
            Poll::Pending
        }
    }
}
```

Async Rust requires careful ownership management. Use `move` closures  
to transfer ownership into async blocks, Arc for shared ownership  
across futures, and be aware that references can't cross await points.  

## Ownership with Smart Pointers

Smart pointers provide additional ownership patterns beyond basic  
references, enabling more complex memory management scenarios.  

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;
use std::sync::Arc;

fn main() {
    // Box: Heap allocation with single ownership
    let boxed_value = Box::new(String::from("Boxed data"));
    let moved_box = boxed_value;
    // boxed_value is no longer valid
    println!("Moved box: {}", moved_box);
    
    // Dereferencing Box
    let boxed_number = Box::new(42);
    let value = *boxed_number; // Dereference and copy
    println!("Boxed number: {}, copied value: {}", boxed_number, value);
    
    // Rc: Reference counting for shared ownership
    let rc_data = Rc::new(vec![1, 2, 3, 4, 5]);
    let rc_clone1 = Rc::clone(&rc_data);
    let rc_clone2 = Rc::clone(&rc_data);
    
    println!("RC count: {}", Rc::strong_count(&rc_data));
    println!("Data via rc_data: {:?}", rc_data);
    println!("Data via rc_clone1: {:?}", rc_clone1);
    
    // Weak references to break cycles
    let strong_ref = Rc::new(String::from("Strong reference"));
    let weak_ref: Weak<String> = Rc::downgrade(&strong_ref);
    
    // Try to upgrade weak reference
    if let Some(upgraded) = weak_ref.upgrade() {
        println!("Weak reference is still valid: {}", upgraded);
    }
    
    drop(strong_ref); // Drop the strong reference
    
    // Now weak reference can't be upgraded
    if weak_ref.upgrade().is_none() {
        println!("Weak reference is no longer valid");
    }
    
    // Rc + RefCell for shared mutable data
    let shared_mutable = Rc::new(RefCell::new(vec![1, 2, 3]));
    let clone1 = Rc::clone(&shared_mutable);
    let clone2 = Rc::clone(&shared_mutable);
    
    // Modify through different clones
    clone1.borrow_mut().push(4);
    clone2.borrow_mut().push(5);
    
    println!("Shared mutable data: {:?}", shared_mutable.borrow());
    
    // Arc for thread-safe reference counting
    let arc_data = Arc::new(String::from("Thread-safe data"));
    let arc_clone = Arc::clone(&arc_data);
    
    let handle = std::thread::spawn(move || {
        println!("From thread: {}", arc_clone);
    });
    
    handle.join().unwrap();
    println!("From main: {}", arc_data);
    
    // Custom smart pointer demonstration
    let custom_box = CustomBox::new(String::from("Custom smart pointer"));
    println!("Custom box value: {}", *custom_box);
}

// Custom smart pointer implementation
struct CustomBox<T> {
    value: T,
}

impl<T> CustomBox<T> {
    fn new(value: T) -> Self {
        CustomBox { value }
    }
}

impl<T> std::ops::Deref for CustomBox<T> {
    type Target = T;
    
    fn deref(&self) -> &Self::Target {
        &self.value
    }
}

impl<T> Drop for CustomBox<T> {
    fn drop(&mut self) {
        println!("Dropping CustomBox");
    }
}
```

Smart pointers extend Rust's ownership model: Box for heap allocation,  
Rc for shared ownership, Weak for cycle breaking, and Arc for  
thread-safe sharing. They enable complex ownership patterns while  
maintaining memory safety.  

## Ownership Patterns in Data Structures

Different data structures have unique ownership characteristics that  
affect how you design and use them in Rust.  

```rust
use std::collections::{HashMap, BTreeMap, LinkedList, VecDeque};
use std::rc::Rc;

fn main() {
    // Vector ownership patterns
    let mut vec = vec![String::from("first"), String::from("second")];
    
    // Moving elements out of vector
    let first = vec.remove(0); // Moves the element
    println!("Removed: {}, remaining: {:?}", first, vec);
    
    // Swapping elements (no allocation)
    vec.push(String::from("third"));
    vec.swap(0, 1);
    println!("After swap: {:?}", vec);
    
    // HashMap with owned keys and values
    let mut map: HashMap<String, Vec<i32>> = HashMap::new();
    map.insert(String::from("numbers"), vec![1, 2, 3]);
    map.insert(String::from("more_numbers"), vec![4, 5, 6]);
    
    // Taking ownership of a value
    if let Some(numbers) = map.remove("numbers") {
        println!("Taken numbers: {:?}", numbers);
    }
    
    // Borrowing values from HashMap
    if let Some(numbers) = map.get("more_numbers") {
        println!("Borrowed numbers: {:?}", numbers);
    }
    
    // LinkedList with ownership transfer
    let mut list = LinkedList::new();
    list.push_back(String::from("item1"));
    list.push_back(String::from("item2"));
    list.push_back(String::from("item3"));
    
    // Pop moves ownership
    while let Some(item) = list.pop_front() {
        println!("Popped: {}", item);
    }
    
    // VecDeque for efficient front/back operations
    let mut deque = VecDeque::new();
    deque.push_back(1);
    deque.push_front(0);
    deque.push_back(2);
    
    println!("Deque: {:?}", deque);
    
    // BTreeMap with ordered iteration
    let mut btree: BTreeMap<i32, String> = BTreeMap::new();
    btree.insert(3, String::from("three"));
    btree.insert(1, String::from("one"));
    btree.insert(2, String::from("two"));
    
    // Iterate over owned values
    for (key, value) in btree {
        println!("{}: {}", key, value);
    }
    // btree is no longer valid after consuming iteration
    
    // Tree-like structure with shared ownership
    let root = create_tree();
    traverse_tree(&root);
}

// Node for tree structure
#[derive(Debug)]
struct TreeNode {
    value: i32,
    children: Vec<Rc<TreeNode>>,
}

impl TreeNode {
    fn new(value: i32) -> Self {
        TreeNode {
            value,
            children: Vec::new(),
        }
    }
    
    fn add_child(mut self, child: Rc<TreeNode>) -> Self {
        self.children.push(child);
        self
    }
}

fn create_tree() -> Rc<TreeNode> {
    let leaf1 = Rc::new(TreeNode::new(1));
    let leaf2 = Rc::new(TreeNode::new(2));
    let leaf3 = Rc::new(TreeNode::new(3));
    
    let branch1 = Rc::new(
        TreeNode::new(10)
            .add_child(Rc::clone(&leaf1))
            .add_child(Rc::clone(&leaf2))
    );
    
    let branch2 = Rc::new(
        TreeNode::new(20)
            .add_child(Rc::clone(&leaf3))
    );
    
    Rc::new(
        TreeNode::new(0)
            .add_child(branch1)
            .add_child(branch2)
    )
}

fn traverse_tree(node: &TreeNode) {
    println!("Node value: {}", node.value);
    for child in &node.children {
        traverse_tree(child);
    }
}
```

Different data structures have unique ownership implications: vectors  
support efficient element removal, hashmaps transfer ownership on  
insert/remove, and tree structures often require shared ownership  
patterns with Rc.  

## Zero-Cost Abstractions and Ownership

Rust's ownership system enables zero-cost abstractions where high-level  
code compiles to efficient low-level operations.  

```rust
use std::marker::PhantomData;

fn main() {
    // Iterator chains compile to simple loops
    let numbers: Vec<i32> = (1..1000)
        .filter(|&x| x % 2 == 0)
        .map(|x| x * x)
        .take(10)
        .collect();
    
    println!("Processed numbers: {:?}", numbers);
    
    // Zero-cost wrapper types
    let user_id = UserId::new(12345);
    let product_id = ProductId::new(67890);
    
    process_user(user_id);
    process_product(product_id);
    
    // Type-safe builders with zero runtime cost
    let config = ConfigBuilder::new()
        .with_host("localhost".to_string())
        .with_port(8080)
        .with_ssl(true)
        .build();
    
    println!("Config: {:?}", config);
    
    // Compile-time state machines
    let connection = Connection::new();
    let authenticated = connection.authenticate("user", "pass");
    let _result = authenticated.send_data("Hello, World!");
    
    // Phantom types for compile-time guarantees
    let raw_data = UnsafeData::new(vec![1, 2, 3, 4, 5]);
    let validated = validate_data(raw_data);
    let processed = process_validated_data(validated);
    println!("Processed: {:?}", processed);
}

// Zero-cost wrapper types
#[derive(Debug, Clone, Copy)]
struct UserId(u32);

#[derive(Debug, Clone, Copy)]
struct ProductId(u32);

impl UserId {
    fn new(id: u32) -> Self {
        UserId(id)
    }
}

impl ProductId {
    fn new(id: u32) -> Self {
        ProductId(id)
    }
}

fn process_user(user_id: UserId) {
    println!("Processing user: {:?}", user_id);
}

fn process_product(product_id: ProductId) {
    println!("Processing product: {:?}", product_id);
}

// Builder pattern with zero-cost state transitions
#[derive(Debug)]
struct Config {
    host: String,
    port: u16,
    ssl: bool,
}

struct ConfigBuilder {
    host: Option<String>,
    port: Option<u16>,
    ssl: bool,
}

impl ConfigBuilder {
    fn new() -> Self {
        ConfigBuilder {
            host: None,
            port: None,
            ssl: false,
        }
    }
    
    fn with_host(mut self, host: String) -> Self {
        self.host = Some(host);
        self
    }
    
    fn with_port(mut self, port: u16) -> Self {
        self.port = Some(port);
        self
    }
    
    fn with_ssl(mut self, ssl: bool) -> Self {
        self.ssl = ssl;
        self
    }
    
    fn build(self) -> Config {
        Config {
            host: self.host.unwrap_or_else(|| "localhost".to_string()),
            port: self.port.unwrap_or(80),
            ssl: self.ssl,
        }
    }
}

// Compile-time state machine
struct Disconnected;
struct Authenticated;

struct Connection<State = Disconnected> {
    state: PhantomData<State>,
}

impl Connection<Disconnected> {
    fn new() -> Self {
        Connection {
            state: PhantomData,
        }
    }
    
    fn authenticate(self, _user: &str, _pass: &str) -> Connection<Authenticated> {
        Connection {
            state: PhantomData,
        }
    }
}

impl Connection<Authenticated> {
    fn send_data(&self, data: &str) -> Result<(), &'static str> {
        println!("Sending: {}", data);
        Ok(())
    }
}

// Phantom types for validation states
struct Unvalidated;
struct Validated;

struct UnsafeData<State = Unvalidated> {
    data: Vec<i32>,
    state: PhantomData<State>,
}

impl UnsafeData<Unvalidated> {
    fn new(data: Vec<i32>) -> Self {
        UnsafeData {
            data,
            state: PhantomData,
        }
    }
}

impl UnsafeData<Validated> {
    fn data(&self) -> &[i32] {
        &self.data
    }
}

fn validate_data(unsafe_data: UnsafeData<Unvalidated>) -> UnsafeData<Validated> {
    // Perform validation logic here
    UnsafeData {
        data: unsafe_data.data,
        state: PhantomData,
    }
}

fn process_validated_data(validated: UnsafeData<Validated>) -> Vec<i32> {
    validated.data().iter().map(|&x| x * 2).collect()
}
```

Rust's ownership system enables zero-cost abstractions where complex  
type safety and state management compile away to efficient code.  
Use phantom types and type-level programming for compile-time  
guarantees without runtime overhead.