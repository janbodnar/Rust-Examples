# Rust Macros

Macros in Rust are powerful metaprogramming tools that generate code at  
compile time. They enable writing code that writes other code, reducing  
boilerplate and creating domain-specific languages. Rust offers two main  
types: declarative macros (macro_rules!) and procedural macros.  

Declarative macros use pattern matching on the structure of Rust code,  
while procedural macros operate on token streams and can transform code  
more freely. Macros are expanded before compilation, enabling zero-cost  
abstractions and compile-time code generation.  

## Basic println! usage

The `println!` macro is one of the most commonly used macros in Rust,  
demonstrating basic macro functionality with variable arguments.  

```rust
fn main() {
    // Basic usage
    println!("Hello, world!");
    
    // With variables
    let name = "Alice";
    let age = 30;
    println!("My name is {} and I am {} years old", name, age);
    
    // Named parameters
    println!("Hello {name}, you are {age} years old", name = name, age = age);
    
    // Debug printing
    let numbers = vec![1, 2, 3, 4, 5];
    println!("Numbers: {:?}", numbers);
    
    // Pretty printing
    println!("Numbers: {:#?}", numbers);
}
```

The `println!` macro accepts a format string and optional arguments,  
performing compile-time validation of format specifiers. It supports  
positional arguments, named parameters, and various formatting options  
like debug (`{:?}`) and pretty-print (`{:#?}`) for complex data structures.  

## Simple declarative macro

Creating a basic declarative macro using `macro_rules!` to generate  
repetitive code patterns.  

```rust
macro_rules! say_hello {
    () => {
        println!("Hello from a macro!");
    };
}

macro_rules! greet {
    ($name:expr) => {
        println!("Hello, {}!", $name);
    };
}

macro_rules! greet_multiple {
    ($($name:expr),*) => {
        $(
            println!("Hello, {}!", $name);
        )*
    };
}

fn main() {
    // Basic macro call
    say_hello!();
    
    // Macro with single parameter
    greet!("Alice");
    greet!("Bob");
    
    // Macro with multiple parameters
    greet_multiple!("Charlie", "David", "Eve");
    
    // Using expressions
    let name = "Frank";
    greet!(name);
    greet!(format!("Mr. {}", "Smith"));
}
```

Declarative macros use pattern matching with `$variable:type` syntax.  
The `expr` fragment specifier matches expressions, while `*` indicates  
zero or more repetitions. The `$()*` pattern repeats the enclosed code  
for each matched input, enabling flexible argument handling.  

## Macro with different patterns

Advanced pattern matching in macros to handle multiple input formats  
and provide flexible APIs.  

```rust
macro_rules! calculate {
    // Addition
    (add $a:expr, $b:expr) => {
        $a + $b
    };
    
    // Subtraction
    (sub $a:expr, $b:expr) => {
        $a - $b
    };
    
    // Multiplication
    (mul $a:expr, $b:expr) => {
        $a * $b
    };
    
    // Division
    (div $a:expr, $b:expr) => {
        $a / $b
    };
    
    // Square
    (square $a:expr) => {
        $a * $a
    };
    
    // Sum of multiple values
    (sum $($x:expr),+) => {
        0 $(+ $x)*
    };
}

fn main() {
    println!("5 + 3 = {}", calculate!(add 5, 3));
    println!("10 - 4 = {}", calculate!(sub 10, 4));
    println!("6 * 7 = {}", calculate!(mul 6, 7));
    println!("15 / 3 = {}", calculate!(div 15, 3));
    
    println!("Square of 8 = {}", calculate!(square 8));
    
    println!("Sum = {}", calculate!(sum 1, 2, 3, 4, 5));
    
    // Works with variables too
    let x = 10;
    let y = 20;
    println!("x + y = {}", calculate!(add x, y));
}
```

Pattern matching allows macros to handle different syntax forms. Each  
pattern branch matches specific input structures and generates appropriate  
code. The `+` repetition indicator requires at least one match, while  
literal tokens like `square` and operators can be matched directly.  

## Creating a vec-like macro

Building a custom collection macro that mimics the behavior of `vec!`  
for creating vectors with initial values.  

```rust
macro_rules! make_vec {
    // Empty vector
    () => {
        Vec::new()
    };
    
    // Single value repeated n times
    ($val:expr; $count:expr) => {
        vec![$val; $count]
    };
    
    // List of values
    ($($val:expr),* $(,)?) => {
        {
            let mut v = Vec::new();
            $(
                v.push($val);
            )*
            v
        }
    };
}

macro_rules! make_hashmap {
    ($($key:expr => $val:expr),* $(,)?) => {
        {
            let mut map = std::collections::HashMap::new();
            $(
                map.insert($key, $val);
            )*
            map
        }
    };
}

fn main() {
    // Empty vector
    let empty: Vec<i32> = make_vec!();
    println!("Empty vector: {:?}", empty);
    
    // Repeated values
    let repeated = make_vec![0; 5];
    println!("Repeated: {:?}", repeated);
    
    // List of values
    let numbers = make_vec![1, 2, 3, 4, 5];
    println!("Numbers: {:?}", numbers);
    
    // Trailing comma support
    let fruits = make_vec!["apple", "banana", "cherry",];
    println!("Fruits: {:?}", fruits);
    
    // HashMap creation
    let map = make_hashmap! {
        "name" => "Alice",
        "age" => "30",
        "city" => "New York",
    };
    println!("Map: {:?}", map);
}
```

The optional trailing comma pattern `$(,)?` makes macros more user-friendly.  
Block expressions `{}` create new scopes for temporary variables. Multiple  
patterns handle different use cases while maintaining a consistent interface  
similar to standard library macros.  

## Debugging and inspection macros

Utility macros for debugging that print variable names along with their  
values, making development and troubleshooting easier.  

```rust
macro_rules! debug_print {
    ($val:expr) => {
        println!("{} = {:?}", stringify!($val), $val);
    };
    
    ($($val:expr),+) => {
        $(
            println!("{} = {:?}", stringify!($val), $val);
        )+
    };
}

macro_rules! function_name {
    () => {{
        fn f() {}
        fn type_name_of<T>(_: T) -> &'static str {
            std::any::type_name::<T>()
        }
        let name = type_name_of(f);
        &name[..name.len() - 3]
    }};
}

macro_rules! here {
    () => {
        println!("Reached line {} in {}", line!(), file!());
    };
    ($msg:expr) => {
        println!("{} at line {} in {}", $msg, line!(), file!());
    };
}

fn main() {
    let x = 42;
    let y = "hello";
    let z = vec![1, 2, 3];
    
    // Debug single value
    debug_print!(x);
    
    // Debug multiple values
    debug_print!(x, y, z);
    
    // Location tracking
    here!();
    here!("Processing data");
    
    // Function name (simplified example)
    println!("Current function: {}", function_name!());
    
    let result = calculate_something();
    debug_print!(result);
}

fn calculate_something() -> i32 {
    here!("Inside calculate_something");
    42 * 2
}
```

The `stringify!` macro converts code to string literals, preserving the  
original variable names for debugging output. Built-in macros like `line!`  
and `file!` provide compile-time location information. These debugging  
macros are invaluable during development and testing phases.  

## Conditional compilation macros

Macros that enable different code generation based on compile-time  
conditions, useful for platform-specific code and feature flags.  

```rust
macro_rules! platform_code {
    (windows => $windows_code:block, unix => $unix_code:block) => {
        #[cfg(target_os = "windows")]
        $windows_code
        
        #[cfg(target_family = "unix")]
        $unix_code
    };
}

macro_rules! debug_only {
    ($code:block) => {
        #[cfg(debug_assertions)]
        $code
    };
}

macro_rules! feature_gate {
    ($feature:literal => $code:block) => {
        #[cfg(feature = $feature)]
        $code
    };
}

fn main() {
    platform_code! {
        windows => {
            println!("Running on Windows");
            println!("Using Windows-specific APIs");
        },
        unix => {
            println!("Running on Unix-like system");
            println!("Using POSIX APIs");
        }
    }
    
    debug_only! {
        {
            println!("This only runs in debug builds");
            let debug_info = "Extra debugging information";
            println!("Debug info: {}", debug_info);
        }
    }
    
    feature_gate! {
        "experimental" => {
            println!("Experimental feature enabled");
        }
    }
    
    #[cfg(target_arch = "x86_64")]
    println!("Running on x86_64 architecture");
    
    println!("Main function completed");
}
```

Conditional compilation macros combine macro generation with `cfg`  
attributes to produce different code for different targets. This enables  
platform-specific implementations, debug-only code, and optional features  
while maintaining a single codebase.  

## Error handling macros

Macros that simplify error handling patterns and provide convenient  
shortcuts for common error scenarios.  

```rust
macro_rules! try_or_return {
    ($expr:expr, $err:expr) => {
        match $expr {
            Ok(val) => val,
            Err(_) => return Err($err.into()),
        }
    };
}

macro_rules! ensure {
    ($condition:expr, $error:expr) => {
        if !$condition {
            return Err($error.into());
        }
    };
}

macro_rules! bail {
    ($error:expr) => {
        return Err($error.into());
    };
}

macro_rules! unwrap_or_bail {
    ($expr:expr, $error:expr) => {
        match $expr {
            Some(val) => val,
            None => bail!($error),
        }
    };
}

#[derive(Debug)]
enum MyError {
    InvalidInput(String),
    OutOfRange,
    NotFound,
}

impl std::fmt::Display for MyError {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self {
            MyError::InvalidInput(msg) => write!(f, "Invalid input: {}", msg),
            MyError::OutOfRange => write!(f, "Value out of range"),
            MyError::NotFound => write!(f, "Item not found"),
        }
    }
}

impl std::error::Error for MyError {}

fn validate_number(input: &str) -> Result<i32, MyError> {
    ensure!(!input.is_empty(), MyError::InvalidInput("Empty string".to_string()));
    
    let number = try_or_return!(
        input.parse::<i32>(),
        MyError::InvalidInput(format!("Invalid number: {}", input))
    );
    
    ensure!(number >= 0, MyError::OutOfRange);
    ensure!(number <= 100, MyError::OutOfRange);
    
    Ok(number)
}

fn find_item(items: &[&str], target: &str) -> Result<usize, MyError> {
    let index = unwrap_or_bail!(
        items.iter().position(|&item| item == target),
        MyError::NotFound
    );
    
    Ok(index)
}

fn main() {
    let test_cases = vec!["", "abc", "-5", "150", "42"];
    
    for input in test_cases {
        match validate_number(input) {
            Ok(num) => println!("Valid number: {}", num),
            Err(e) => println!("Error with '{}': {}", input, e),
        }
    }
    
    let items = vec!["apple", "banana", "cherry"];
    match find_item(&items, "banana") {
        Ok(index) => println!("Found banana at index {}", index),
        Err(e) => println!("Error: {}", e),
    }
    
    match find_item(&items, "grape") {
        Ok(index) => println!("Found grape at index {}", index),
        Err(e) => println!("Error: {}", e),
    }
}
```

Error handling macros encapsulate common patterns like early returns,  
condition checking, and error conversion. They make error-heavy code  
more readable and reduce boilerplate while maintaining type safety  
and proper error propagation.  

## Struct generation macros

Macros that automatically generate struct definitions with common  
functionality like builders, defaults, and serialization helpers.  

```rust
macro_rules! define_struct {
    (
        $name:ident {
            $($field:ident: $type:ty),* $(,)?
        }
    ) => {
        #[derive(Debug, Clone, PartialEq)]
        struct $name {
            $(pub $field: $type,)*
        }
        
        impl $name {
            pub fn new($($field: $type),*) -> Self {
                Self {
                    $($field,)*
                }
            }
        }
    };
}

macro_rules! builder_struct {
    (
        $name:ident {
            $($field:ident: $type:ty = $default:expr),* $(,)?
        }
    ) => {
        #[derive(Debug, Clone)]
        struct $name {
            $(pub $field: $type,)*
        }
        
        impl $name {
            pub fn new() -> Self {
                Self {
                    $($field: $default,)*
                }
            }
            
            $(
                pub fn $field(mut self, value: $type) -> Self {
                    self.$field = value;
                    self
                }
            )*
            
            pub fn build(self) -> Self {
                self
            }
        }
        
        impl Default for $name {
            fn default() -> Self {
                Self::new()
            }
        }
    };
}

// Define structs using macros
define_struct! {
    Person {
        name: String,
        age: u32,
        email: String,
    }
}

builder_struct! {
    Config {
        host: String = "localhost".to_string(),
        port: u16 = 8080,
        debug: bool = false,
        timeout: u64 = 30,
    }
}

fn main() {
    // Using the generated struct
    let person = Person::new(
        "Alice Johnson".to_string(),
        28,
        "alice@example.com".to_string(),
    );
    
    println!("Person: {:?}", person);
    
    // Using the builder pattern
    let config = Config::new()
        .host("example.com".to_string())
        .port(9000)
        .debug(true)
        .build();
    
    println!("Config: {:?}", config);
    
    // Using default values
    let default_config = Config::default();
    println!("Default config: {:?}", default_config);
    
    // Partial configuration
    let partial_config = Config::new()
        .port(3000)
        .debug(true)
        .build();
    
    println!("Partial config: {:?}", partial_config);
}
```

Struct generation macros reduce boilerplate code by automatically creating  
common implementations. The builder pattern macro generates setter methods  
for fluent APIs, while maintaining type safety and providing sensible  
defaults for configuration objects.  

## Testing and assertion macros

Custom macros for testing that provide enhanced assertion capabilities  
and test case generation.  

```rust
macro_rules! assert_near {
    ($a:expr, $b:expr, $epsilon:expr) => {
        assert!(
            ($a - $b).abs() < $epsilon,
            "Assertion failed: {} ≈ {} (within {}), actual difference: {}",
            $a, $b, $epsilon, ($a - $b).abs()
        );
    };
}

macro_rules! test_case {
    ($name:ident: $input:expr => $expected:expr) => {
        #[test]
        fn $name() {
            assert_eq!($input, $expected);
        }
    };
    
    ($name:ident: $func:ident($($arg:expr),*) => $expected:expr) => {
        #[test]
        fn $name() {
            assert_eq!($func($($arg),*), $expected);
        }
    };
}

macro_rules! test_cases {
    ($func:ident: [$(($($arg:expr),*) => $expected:expr),* $(,)?]) => {
        $(
            {
                let result = $func($($arg),*);
                assert_eq!(result, $expected, 
                    "Test failed for {}({:?}): expected {:?}, got {:?}",
                    stringify!($func), ($($arg,)*), $expected, result);
            }
        )*
    };
}

macro_rules! benchmark_simple {
    ($name:expr, $code:block) => {
        {
            let start = std::time::Instant::now();
            let result = $code;
            let duration = start.elapsed();
            println!("{}: {:?}", $name, duration);
            result
        }
    };
}

fn add(a: i32, b: i32) -> i32 {
    a + b
}

fn multiply(a: i32, b: i32) -> i32 {
    a * b
}

fn fibonacci(n: u32) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}

// Generate test cases using macros
test_case!(test_add_positive: add(2, 3) => 5);
test_case!(test_add_negative: add(-1, 1) => 0);
test_case!(test_multiply_basic: multiply(4, 5) => 20);

fn main() {
    // Floating point comparison
    let result = 0.1 + 0.2;
    assert_near!(result, 0.3, 1e-10);
    
    // Batch testing
    test_cases!(add: [
        (1, 2) => 3,
        (0, 0) => 0,
        (-1, 1) => 0,
        (10, -5) => 5,
    ]);
    
    test_cases!(multiply: [
        (2, 3) => 6,
        (0, 5) => 0,
        (-2, 4) => -8,
    ]);
    
    // Simple benchmarking
    let fib_result = benchmark_simple!("Fibonacci(20)", {
        fibonacci(20)
    });
    println!("Fibonacci(20) = {}", fib_result);
    
    let sum_result = benchmark_simple!("Sum 1-1000", {
        (1..=1000).sum::<i32>()
    });
    println!("Sum 1-1000 = {}", sum_result);
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_floating_point_precision() {
        assert_near!(3.14159, 22.0/7.0, 0.01);
    }
}
```

Testing macros simplify test creation and provide domain-specific  
assertions. The `test_case!` macro generates individual test functions,  
while `test_cases!` enables batch testing with multiple inputs. Custom  
assertions like `assert_near!` handle floating-point comparisons properly.  

## Functional programming macros

Macros that enable functional programming patterns and higher-order  
function-like behavior in Rust.  

```rust
macro_rules! compose {
    ($f:expr, $g:expr) => {
        |x| $f($g(x))
    };
    
    ($f:expr, $g:expr, $($rest:expr),+) => {
        compose!($f, compose!($g, $($rest),+))
    };
}

macro_rules! partial {
    ($func:expr, $($bound:expr),+) => {
        move |$($unbound),*| $func($($bound,)+ $($unbound),*)
    };
}

macro_rules! curry {
    ($func:ident, $arg:expr) => {
        move |next_arg| $func($arg, next_arg)
    };
}

macro_rules! pipe {
    ($value:expr, $func:expr) => {
        $func($value)
    };
    
    ($value:expr, $func:expr, $($rest:expr),+) => {
        pipe!($func($value), $($rest),+)
    };
}

macro_rules! lazy {
    ($expr:expr) => {
        || $expr
    };
}

fn add_numbers(a: i32, b: i32) -> i32 {
    a + b
}

fn multiply_by_two(x: i32) -> i32 {
    x * 2
}

fn add_one(x: i32) -> i32 {
    x + 1
}

fn square(x: i32) -> i32 {
    x * x
}

fn main() {
    // Function composition
    let add_one_then_double = compose!(multiply_by_two, add_one);
    println!("add_one_then_double(5) = {}", add_one_then_double(5));
    
    let complex_composition = compose!(square, multiply_by_two, add_one);
    println!("complex_composition(3) = {}", complex_composition(3));
    
    // Partial application
    let add_five = partial!(add_numbers, 5);
    println!("add_five(10) = {}", add_five(10));
    
    // Currying
    let add_ten = curry!(add_numbers, 10);
    println!("add_ten(7) = {}", add_ten(7));
    
    // Pipeline operations
    let result = pipe!(5, add_one, multiply_by_two, square);
    println!("Pipeline result: {}", result);
    
    // Lazy evaluation
    let expensive_calculation = lazy!({
        println!("Performing expensive calculation...");
        42 * 42
    });
    
    println!("Before calling lazy computation");
    let result = expensive_calculation();
    println!("Lazy result: {}", result);
    
    // Chaining operations
    let numbers = vec![1, 2, 3, 4, 5];
    let processed: Vec<i32> = numbers
        .into_iter()
        .map(add_one)
        .map(multiply_by_two)
        .collect();
    
    println!("Processed numbers: {:?}", processed);
}
```

Functional programming macros enable composition, currying, and pipeline  
operations. These patterns promote immutability and pure functions,  
making code more predictable and easier to reason about. The `pipe!`  
macro creates readable data transformation chains.  

## Repetition and iteration macros

Macros that generate repetitive code patterns and iteration constructs  
for reducing boilerplate in common scenarios.  

```rust
macro_rules! repeat {
    ($code:block, $count:expr) => {
        for _ in 0..$count {
            $code
        }
    };
}

macro_rules! times {
    ($n:expr => $code:block) => {
        for i in 0..$n {
            let index = i;
            $code
        }
    };
}

macro_rules! generate_array {
    ($expr:expr; $count:expr) => {
        {
            let mut array = Vec::new();
            for i in 0..$count {
                array.push($expr);
            }
            array
        }
    };
}

macro_rules! foreach {
    ($item:ident in $collection:expr => $code:block) => {
        for $item in $collection {
            $code
        }
    };
}

macro_rules! while_let {
    ($pattern:pat = $expr:expr => $code:block) => {
        while let $pattern = $expr {
            $code
        }
    };
}

macro_rules! unroll {
    ($($code:stmt),*) => {
        $($code)*
    };
}

fn main() {
    // Simple repetition
    println!("=== Simple Repetition ===");
    repeat!({
        println!("Hello from repeat macro!");
    }, 3);
    
    // Indexed repetition
    println!("\n=== Indexed Repetition ===");
    times!(5 => {
        println!("Iteration {}", index);
    });
    
    // Generate array with expression
    println!("\n=== Generated Arrays ===");
    let squares = generate_array!(i * i; 5);
    println!("Squares: {:?}", squares);
    
    let random_numbers = generate_array!(rand::random::<u8>() % 100; 5);
    println!("Random numbers: {:?}", random_numbers);
    
    // Foreach iteration
    println!("\n=== Foreach Iteration ===");
    let numbers = vec![1, 2, 3, 4, 5];
    foreach!(num in &numbers => {
        println!("Processing number: {}", num);
    });
    
    // While let pattern
    println!("\n=== While Let Pattern ===");
    let mut iter = numbers.iter();
    while_let!(Some(value) = iter.next() => {
        println!("Iterator value: {}", value);
        if *value == 3 {
            break;
        }
    });
    
    // Code unrolling
    println!("\n=== Code Unrolling ===");
    unroll! {
        println!("First statement"),
        println!("Second statement"),
        println!("Third statement")
    }
    
    // Complex iteration with multiple patterns
    println!("\n=== Complex Iteration ===");
    let data = vec![
        ("Alice", 25),
        ("Bob", 30),
        ("Charlie", 35),
    ];
    
    foreach!(person in &data => {
        let (name, age) = person;
        if *age > 28 {
            println!("{} is {} years old (older than 28)", name, age);
        } else {
            println!("{} is {} years old", name, age);
        }
    });
}
```

Repetition macros provide convenient ways to generate repetitive code  
and iteration patterns. They reduce boilerplate while maintaining  
readability and can generate different code patterns based on the  
input parameters. The indexed iteration is particularly useful for  
array initialization and batch operations.  

## State machine macros

Macros that generate state machine implementations with automatic  
transition validation and state management.  

```rust
macro_rules! state_machine {
    (
        $name:ident {
            states: [$($state:ident),* $(,)?],
            events: [$($event:ident),* $(,)?],
            transitions: [
                $(($from:ident, $ev:ident) => $to:ident),* $(,)?
            ],
            initial: $initial:ident
        }
    ) => {
        #[derive(Debug, Clone, Copy, PartialEq)]
        enum $name {
            $($state,)*
        }
        
        #[derive(Debug, Clone, Copy, PartialEq)]
        enum Event {
            $($event,)*
        }
        
        impl $name {
            fn new() -> Self {
                $name::$initial
            }
            
            fn transition(self, event: Event) -> Result<Self, String> {
                match (self, event) {
                    $(
                        ($name::$from, Event::$ev) => Ok($name::$to),
                    )*
                    (state, event) => Err(format!(
                        "Invalid transition from {:?} with event {:?}",
                        state, event
                    )),
                }
            }
            
            fn can_transition(self, event: Event) -> bool {
                self.transition(event).is_ok()
            }
        }
    };
}

macro_rules! define_transitions {
    ($machine:ty, $($from:ident --$event:ident--> $to:ident);* $(;)?) => {
        impl $machine {
            fn describe_transitions() {
                println!("Valid transitions:");
                $(
                    println!("  {:?} --{:?}--> {:?}", 
                             <$machine>::$from, 
                             Event::$event, 
                             <$machine>::$to);
                )*
            }
        }
    };
}

// Generate a simple traffic light state machine
state_machine! {
    TrafficLight {
        states: [Red, Yellow, Green],
        events: [Timer, Emergency],
        transitions: [
            (Red, Timer) => Green,
            (Green, Timer) => Yellow,
            (Yellow, Timer) => Red,
            (Red, Emergency) => Yellow,
            (Green, Emergency) => Yellow,
            (Yellow, Emergency) => Red
        ],
        initial: Red
    }
}

// Generate a more complex order processing state machine
state_machine! {
    OrderState {
        states: [Pending, Processing, Shipped, Delivered, Cancelled],
        events: [Process, Ship, Deliver, Cancel, Return],
        transitions: [
            (Pending, Process) => Processing,
            (Pending, Cancel) => Cancelled,
            (Processing, Ship) => Shipped,
            (Processing, Cancel) => Cancelled,
            (Shipped, Deliver) => Delivered,
            (Shipped, Return) => Processing,
            (Delivered, Return) => Processing
        ],
        initial: Pending
    }
}

fn main() {
    // Traffic light example
    println!("=== Traffic Light State Machine ===");
    let mut light = TrafficLight::new();
    println!("Initial state: {:?}", light);
    
    let events = vec![Event::Timer, Event::Timer, Event::Emergency, Event::Timer];
    
    for event in events {
        match light.transition(event) {
            Ok(new_state) => {
                light = new_state;
                println!("Event {:?} -> New state: {:?}", event, light);
            }
            Err(error) => {
                println!("Error: {}", error);
            }
        }
    }
    
    // Order processing example
    println!("\n=== Order Processing State Machine ===");
    let mut order = OrderState::new();
    println!("Initial order state: {:?}", order);
    
    let order_events = vec![
        Event::Process,
        Event::Ship,
        Event::Return,
        Event::Ship,
        Event::Deliver,
    ];
    
    for event in order_events {
        if order.can_transition(event) {
            order = order.transition(event).unwrap();
            println!("Order event {:?} -> New state: {:?}", event, order);
        } else {
            println!("Cannot apply event {:?} to state {:?}", event, order);
        }
    }
    
    // Test invalid transitions
    println!("\n=== Testing Invalid Transitions ===");
    let light = TrafficLight::Red;
    match light.transition(Event::Timer) {
        Ok(new_state) => println!("Valid: Red + Timer -> {:?}", new_state),
        Err(e) => println!("Invalid: {}", e),
    }
    
    // This would be invalid - no such transition defined
    let invalid_result = light.transition(Event::Emergency);
    println!("Emergency from Red: {:?}", invalid_result);
}
```

State machine macros automatically generate type-safe state transitions  
with compile-time validation of allowed state changes. This prevents  
invalid state transitions and makes state management explicit and  
trackable. The generated code includes transition validation and  
error handling for robust state management.  

## Domain-specific language macros

Macros that create embedded domain-specific languages (DSLs) for  
expressing complex logic in a more natural syntax.  

```rust
macro_rules! sql {
    (SELECT $($field:ident),+ FROM $table:ident WHERE $condition:expr) => {
        {
            let fields = vec![$(stringify!($field)),+];
            let table = stringify!($table);
            let query = format!(
                "SELECT {} FROM {} WHERE {}",
                fields.join(", "),
                table,
                $condition
            );
            query
        }
    };
    
    (INSERT INTO $table:ident ($($field:ident),+) VALUES ($($value:expr),+)) => {
        {
            let fields = vec![$(stringify!($field)),+];
            let values = vec![$(format!("{:?}", $value)),+];
            let table = stringify!($table);
            format!(
                "INSERT INTO {} ({}) VALUES ({})",
                table,
                fields.join(", "),
                values.join(", ")
            )
        }
    };
}

macro_rules! html {
    ($tag:ident { $($attr:ident: $value:expr),* } [ $($child:tt)* ]) => {
        {
            let mut attributes = String::new();
            $(
                attributes.push_str(&format!(" {}=\"{}\"", stringify!($attr), $value));
            )*
            
            let children = html_children!($($child)*);
            format!("<{}{}>{}</{}>", 
                   stringify!($tag), 
                   attributes, 
                   children, 
                   stringify!($tag))
        }
    };
    
    ($tag:ident [ $($child:tt)* ]) => {
        {
            let children = html_children!($($child)*);
            format!("<{}>{}</{}>", 
                   stringify!($tag), 
                   children, 
                   stringify!($tag))
        }
    };
    
    ($text:literal) => {
        $text.to_string()
    };
}

macro_rules! html_children {
    () => { String::new() };
    ($first:tt $($rest:tt)*) => {
        html!($first) + &html_children!($($rest)*)
    };
}

macro_rules! route {
    (GET $path:literal => $handler:expr) => {
        Route::new("GET", $path, $handler)
    };
    
    (POST $path:literal => $handler:expr) => {
        Route::new("POST", $path, $handler)
    };
    
    (PUT $path:literal => $handler:expr) => {
        Route::new("PUT", $path, $handler)
    };
    
    (DELETE $path:literal => $handler:expr) => {
        Route::new("DELETE", $path, $handler)
    };
}

struct Route {
    method: &'static str,
    path: &'static str,
    handler: fn() -> String,
}

impl Route {
    fn new(method: &'static str, path: &'static str, handler: fn() -> String) -> Self {
        Self { method, path, handler }
    }
}

fn get_users() -> String {
    "List of users".to_string()
}

fn create_user() -> String {
    "User created".to_string()
}

fn get_user_by_id() -> String {
    "User details".to_string()
}

fn main() {
    // SQL DSL examples
    println!("=== SQL DSL ===");
    
    let select_query = sql!(
        SELECT name, email, age 
        FROM users 
        WHERE "age > 21"
    );
    println!("Select query: {}", select_query);
    
    let insert_query = sql!(
        INSERT INTO users (name, email, age) 
        VALUES ("John Doe", "john@example.com", 30)
    );
    println!("Insert query: {}", insert_query);
    
    // HTML DSL examples
    println!("\n=== HTML DSL ===");
    
    let simple_html = html!(div ["Hello, World!"]);
    println!("Simple HTML: {}", simple_html);
    
    let complex_html = html!(div { class: "container", id: "main" } [
        html!(h1 ["Welcome"])
        html!(p { class: "description" } ["This is a paragraph"])
    ]);
    println!("Complex HTML: {}", complex_html);
    
    // Route DSL examples
    println!("\n=== Route DSL ===");
    
    let routes = vec![
        route!(GET "/users" => get_users),
        route!(POST "/users" => create_user),
        route!(GET "/users/:id" => get_user_by_id),
    ];
    
    for route in &routes {
        println!("{} {} -> {}", 
                 route.method, 
                 route.path, 
                 (route.handler)());
    }
    
    // Configuration DSL
    println!("\n=== Configuration Example ===");
    
    let config_queries = vec![
        sql!(SELECT host, port FROM config WHERE "environment = 'production'"),
        sql!(SELECT database_url FROM config WHERE "service = 'api'"),
    ];
    
    for query in config_queries {
        println!("Config query: {}", query);
    }
}
```

Domain-specific language macros enable creating mini-languages within  
Rust that express domain logic more naturally. These DSLs can generate  
appropriate Rust code while providing intuitive syntax for specific  
problem domains like SQL queries, HTML generation, or routing.  

## Async and concurrency macros

Macros that simplify asynchronous programming patterns and concurrent  
code generation in Rust.  

```rust
macro_rules! spawn_tasks {
    ($($task:expr),+ $(,)?) => {
        {
            let mut handles = Vec::new();
            $(
                handles.push(std::thread::spawn($task));
            )+
            handles
        }
    };
}

macro_rules! parallel_map {
    ($collection:expr, $transform:expr) => {
        {
            let handles: Vec<_> = $collection
                .into_iter()
                .map(|item| {
                    std::thread::spawn(move || $transform(item))
                })
                .collect();
            
            handles
                .into_iter()
                .map(|handle| handle.join().unwrap())
                .collect::<Vec<_>>()
        }
    };
}

macro_rules! timeout {
    ($duration:expr, $operation:expr) => {
        {
            use std::time::Duration;
            use std::sync::mpsc;
            
            let (tx, rx) = mpsc::channel();
            let handle = std::thread::spawn(move || {
                let result = $operation;
                let _ = tx.send(result);
            });
            
            match rx.recv_timeout($duration) {
                Ok(result) => Some(result),
                Err(_) => {
                    // Note: handle will be orphaned, but this is for demonstration
                    None
                }
            }
        }
    };
}

macro_rules! retry {
    ($max_attempts:expr, $operation:expr) => {
        {
            let mut last_error = None;
            for attempt in 1..=$max_attempts {
                match $operation {
                    Ok(result) => return Ok(result),
                    Err(e) => {
                        last_error = Some(e);
                        if attempt < $max_attempts {
                            println!("Attempt {} failed, retrying...", attempt);
                            std::thread::sleep(std::time::Duration::from_millis(100 * attempt));
                        }
                    }
                }
            }
            Err(last_error.unwrap())
        }
    };
}

macro_rules! actor {
    ($name:ident {
        state: $state_type:ty = $initial_state:expr,
        messages: {
            $($msg_name:ident($($param:ident: $param_type:ty),*) => $handler:expr),* $(,)?
        }
    }) => {
        struct $name {
            state: $state_type,
        }
        
        #[derive(Debug)]
        enum Message {
            $(
                $msg_name($($param_type,)*),
            )*
        }
        
        impl $name {
            fn new() -> Self {
                Self {
                    state: $initial_state,
                }
            }
            
            fn handle_message(&mut self, message: Message) {
                match message {
                    $(
                        Message::$msg_name($($param,)*) => {
                            $handler(self, $($param,)*)
                        }
                    )*
                }
            }
        }
    };
}

fn expensive_computation(n: u64) -> u64 {
    // Simulate expensive work
    std::thread::sleep(std::time::Duration::from_millis(100));
    n * n
}

fn unreliable_operation() -> Result<String, String> {
    use std::collections::hash_map::DefaultHasher;
    use std::hash::{Hash, Hasher};
    
    let mut hasher = DefaultHasher::new();
    std::thread::current().id().hash(&mut hasher);
    let random = hasher.finish() % 3;
    
    if random == 0 {
        Ok("Success!".to_string())
    } else {
        Err("Random failure".to_string())
    }
}

// Define an actor using the macro
actor! {
    Counter {
        state: i32 = 0,
        messages: {
            Increment() => |actor, | {
                actor.state += 1;
                println!("Counter incremented to: {}", actor.state);
            },
            Decrement() => |actor, | {
                actor.state -= 1;
                println!("Counter decremented to: {}", actor.state);
            },
            Add(value: i32) => |actor, value| {
                actor.state += value;
                println!("Added {} to counter, new value: {}", value, actor.state);
            },
            Get() => |actor, | {
                println!("Current counter value: {}", actor.state);
            }
        }
    }
}

fn main() {
    println!("=== Parallel Task Spawning ===");
    
    let tasks = spawn_tasks![
        || {
            println!("Task 1 running");
            expensive_computation(5)
        },
        || {
            println!("Task 2 running");
            expensive_computation(10)
        },
        || {
            println!("Task 3 running");
            expensive_computation(15)
        }
    ];
    
    let results: Vec<u64> = tasks
        .into_iter()
        .map(|handle| handle.join().unwrap())
        .collect();
    
    println!("Task results: {:?}", results);
    
    println!("\n=== Parallel Map ===");
    
    let numbers = vec![1, 2, 3, 4, 5];
    let squared = parallel_map!(numbers, |x| {
        println!("Processing {}", x);
        x * x
    });
    
    println!("Squared numbers: {:?}", squared);
    
    println!("\n=== Timeout Operations ===");
    
    let quick_result = timeout!(
        std::time::Duration::from_millis(200),
        || {
            std::thread::sleep(std::time::Duration::from_millis(50));
            "Quick operation completed"
        }
    );
    println!("Quick operation: {:?}", quick_result);
    
    let slow_result = timeout!(
        std::time::Duration::from_millis(50),
        || {
            std::thread::sleep(std::time::Duration::from_millis(200));
            "Slow operation completed"
        }
    );
    println!("Slow operation: {:?}", slow_result);
    
    println!("\n=== Retry Operations ===");
    
    let retry_result = retry!(3, unreliable_operation());
    println!("Retry result: {:?}", retry_result);
    
    println!("\n=== Actor Pattern ===");
    
    let mut counter = Counter::new();
    
    let messages = vec![
        Message::Increment(),
        Message::Add(5),
        Message::Decrement(),
        Message::Get(),
    ];
    
    for message in messages {
        counter.handle_message(message);
    }
}
```

Async and concurrency macros simplify parallel programming by generating  
thread management code and providing high-level abstractions for common  
patterns. The actor macro demonstrates creating message-passing systems  
with type-safe message handling and encapsulated state management.  

## Memory and resource management macros

Macros that help with memory management, resource allocation, and  
automatic cleanup patterns in Rust.  

```rust
macro_rules! scoped {
    ($setup:expr => $cleanup:expr, $body:block) => {
        {
            let _resource = $setup;
            let result = $body;
            $cleanup;
            result
        }
    };
}

macro_rules! defer {
    ($cleanup:expr) => {
        let _guard = defer_guard(|| $cleanup);
    };
}

struct DeferGuard<F: FnOnce()> {
    cleanup: Option<F>,
}

fn defer_guard<F: FnOnce()>(cleanup: F) -> DeferGuard<F> {
    DeferGuard {
        cleanup: Some(cleanup),
    }
}

impl<F: FnOnce()> Drop for DeferGuard<F> {
    fn drop(&mut self) {
        if let Some(cleanup) = self.cleanup.take() {
            cleanup();
        }
    }
}

macro_rules! with_capacity {
    (vec: $capacity:expr) => {
        Vec::with_capacity($capacity)
    };
    
    (string: $capacity:expr) => {
        String::with_capacity($capacity)
    };
    
    (hashmap: $capacity:expr) => {
        std::collections::HashMap::with_capacity($capacity)
    };
}

macro_rules! pool {
    ($name:ident, $type:ty, $size:expr, $constructor:expr) => {
        struct $name {
            items: std::sync::Mutex<Vec<$type>>,
        }
        
        impl $name {
            fn new() -> Self {
                let mut items = Vec::with_capacity($size);
                for _ in 0..$size {
                    items.push($constructor);
                }
                Self {
                    items: std::sync::Mutex::new(items),
                }
            }
            
            fn get(&self) -> Option<$type> {
                self.items.lock().unwrap().pop()
            }
            
            fn put(&self, item: $type) {
                let mut items = self.items.lock().unwrap();
                if items.len() < $size {
                    items.push(item);
                }
            }
        }
    };
}

macro_rules! arena {
    ($name:ident for $type:ty) => {
        struct $name {
            items: Vec<$type>,
        }
        
        impl $name {
            fn new() -> Self {
                Self {
                    items: Vec::new(),
                }
            }
            
            fn allocate(&mut self, item: $type) -> &$type {
                self.items.push(item);
                self.items.last().unwrap()
            }
            
            fn allocate_mut(&mut self, item: $type) -> &mut $type {
                self.items.push(item);
                self.items.last_mut().unwrap()
            }
            
            fn clear(&mut self) {
                self.items.clear();
            }
            
            fn len(&self) -> usize {
                self.items.len()
            }
        }
    };
}

macro_rules! lazy_static {
    ($name:ident: $type:ty = $init:expr) => {
        use std::sync::Once;
        static mut $name: Option<$type> = None;
        static INIT: Once = Once::new();
        
        fn $name() -> &'static $type {
            unsafe {
                INIT.call_once(|| {
                    $name = Some($init);
                });
                $name.as_ref().unwrap()
            }
        }
    };
}

// Create object pools
pool!(StringPool, String, 10, String::new());
pool!(VecPool, Vec<i32>, 5, Vec::new());

// Create arenas
arena!(StringArena for String);
arena!(NumberArena for i32);

fn simulate_file_operation() -> std::io::Result<String> {
    println!("Opening file...");
    // Simulate file operation
    Ok("File content".to_string())
}

fn main() {
    println!("=== Scoped Resource Management ===");
    
    let result = scoped!(
        {
            println!("Acquiring resource...");
            "resource_handle"
        } => {
            println!("Cleaning up resource...");
        },
        {
            println!("Using resource...");
            "operation_result"
        }
    );
    
    println!("Scoped result: {}", result);
    
    println!("\n=== Defer Pattern ===");
    
    {
        defer!(println!("This will be called when scope exits"));
        println!("Doing some work...");
        defer!(println!("This will be called second"));
        println!("More work...");
    } // Deferred actions execute here in reverse order
    
    println!("\n=== Pre-allocated Collections ===");
    
    let mut large_vec: Vec<i32> = with_capacity!(vec: 1000);
    let mut large_string: String = with_capacity!(string: 500);
    let mut large_map: std::collections::HashMap<String, i32> = with_capacity!(hashmap: 100);
    
    // Fill collections without reallocation
    for i in 0..100 {
        large_vec.push(i);
        large_string.push_str(&format!("item{} ", i));
        large_map.insert(format!("key{}", i), i);
    }
    
    println!("Vec length: {}, capacity: {}", large_vec.len(), large_vec.capacity());
    println!("String length: {}, capacity: {}", large_string.len(), large_string.capacity());
    println!("HashMap length: {}", large_map.len());
    
    println!("\n=== Object Pooling ===");
    
    let string_pool = StringPool::new();
    
    // Get strings from pool
    let mut borrowed_strings = Vec::new();
    for i in 0..5 {
        if let Some(mut s) = string_pool.get() {
            s.clear();
            s.push_str(&format!("Pooled string {}", i));
            borrowed_strings.push(s);
        }
    }
    
    println!("Borrowed {} strings from pool", borrowed_strings.len());
    
    // Return strings to pool
    for s in borrowed_strings {
        string_pool.put(s);
    }
    
    println!("\n=== Arena Allocation ===");
    
    let mut arena = StringArena::new();
    
    // Allocate strings in arena
    let str1 = arena.allocate("First string".to_string());
    let str2 = arena.allocate("Second string".to_string());
    let str3 = arena.allocate("Third string".to_string());
    
    println!("Arena contains {} items", arena.len());
    println!("Allocated strings: {}, {}, {}", str1, str2, str3);
    
    // All allocations are cleaned up when arena is dropped
    arena.clear();
    println!("Arena cleared, now contains {} items", arena.len());
    
    println!("\n=== File Operation with Defer ===");
    
    match simulate_file_operation() {
        Ok(content) => {
            defer!(println!("File operation completed"));
            println!("File content: {}", content);
        }
        Err(e) => {
            defer!(println!("File operation failed"));
            println!("Error: {}", e);
        }
    }
}
```

Memory and resource management macros provide patterns for efficient  
memory usage and automatic cleanup. The `scoped!` macro ensures resources  
are properly cleaned up, while `defer!` provides Go-like defer semantics.  
Object pools and arenas optimize allocation patterns for performance-critical  
code.  

## Serialization and data transformation macros

Macros that generate serialization code and data transformation logic  
for converting between different data formats.  

```rust
macro_rules! json_object {
    () => {
        std::collections::HashMap::new()
    };
    
    ($($key:expr => $value:expr),+ $(,)?) => {
        {
            let mut object = std::collections::HashMap::new();
            $(
                object.insert($key.to_string(), json_value!($value));
            )+
            object
        }
    };
}

macro_rules! json_value {
    (null) => { JsonValue::Null };
    ($value:expr) => {
        match &$value {
            v if std::any::type_name::<std::string::String>().contains("String") => 
                JsonValue::String($value.to_string()),
            v if std::mem::size_of_val(v) == std::mem::size_of::<i32>() => 
                JsonValue::Number($value as f64),
            v if std::mem::size_of_val(v) == std::mem::size_of::<f64>() => 
                JsonValue::Number($value),
            v if std::mem::size_of_val(v) == std::mem::size_of::<bool>() => 
                JsonValue::Boolean($value),
            _ => JsonValue::String(format!("{:?}", $value)),
        }
    };
}

macro_rules! to_csv {
    (headers: [$($header:expr),+], rows: [$([$($value:expr),+]),+]) => {
        {
            let mut csv = String::new();
            
            // Add headers
            csv.push_str(&vec![$($header),+].join(","));
            csv.push('\n');
            
            // Add rows
            $(
                csv.push_str(&vec![$($value.to_string()),+].join(","));
                csv.push('\n');
            )+
            
            csv
        }
    };
}

macro_rules! struct_to_map {
    ($struct_instance:expr, $($field:ident),+) => {
        {
            let mut map = std::collections::HashMap::new();
            $(
                map.insert(stringify!($field).to_string(), format!("{:?}", $struct_instance.$field));
            )+
            map
        }
    };
}

macro_rules! deserialize_from_map {
    ($map:expr, $struct_name:ident { $($field:ident: $field_type:ty),+ }) => {
        {
            $struct_name {
                $(
                    $field: $map.get(stringify!($field))
                        .and_then(|v| v.parse::<$field_type>().ok())
                        .unwrap_or_default(),
                )+
            }
        }
    };
}

#[derive(Debug, Clone)]
enum JsonValue {
    Null,
    Boolean(bool),
    Number(f64),
    String(String),
    Array(Vec<JsonValue>),
    Object(std::collections::HashMap<String, JsonValue>),
}

impl std::fmt::Display for JsonValue {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self {
            JsonValue::Null => write!(f, "null"),
            JsonValue::Boolean(b) => write!(f, "{}", b),
            JsonValue::Number(n) => write!(f, "{}", n),
            JsonValue::String(s) => write!(f, "\"{}\"", s),
            JsonValue::Array(arr) => {
                let items: Vec<String> = arr.iter().map(|v| v.to_string()).collect();
                write!(f, "[{}]", items.join(", "))
            }
            JsonValue::Object(obj) => {
                let items: Vec<String> = obj.iter()
                    .map(|(k, v)| format!("\"{}\": {}", k, v))
                    .collect();
                write!(f, "{{{}}}", items.join(", "))
            }
        }
    }
}

#[derive(Debug, Default)]
struct Person {
    name: String,
    age: u32,
    email: String,
}

fn main() {
    println!("=== JSON Object Creation ===");
    
    let json_obj = json_object! {
        "name" => "Alice Johnson",
        "age" => 30,
        "active" => true,
        "balance" => 1250.75,
        "metadata" => null
    };
    
    let json_value = JsonValue::Object(json_obj);
    println!("JSON Object: {}", json_value);
    
    println!("\n=== CSV Generation ===");
    
    let csv_data = to_csv! {
        headers: ["Name", "Age", "City", "Salary"],
        rows: [
            ["Alice", "30", "New York", "75000"],
            ["Bob", "25", "San Francisco", "85000"],
            ["Charlie", "35", "Chicago", "70000"],
            ["Diana", "28", "Austin", "80000"]
        ]
    };
    
    println!("CSV Data:\n{}", csv_data);
    
    println!("=== Struct to Map Conversion ===");
    
    let person = Person {
        name: "John Doe".to_string(),
        age: 42,
        email: "john@example.com".to_string(),
    };
    
    let person_map = struct_to_map!(person, name, age, email);
    println!("Person as map: {:?}", person_map);
    
    println!("\n=== Map to Struct Deserialization ===");
    
    let mut data_map = std::collections::HashMap::new();
    data_map.insert("name".to_string(), "Jane Smith".to_string());
    data_map.insert("age".to_string(), "35".to_string());
    data_map.insert("email".to_string(), "jane@example.com".to_string());
    
    let deserialized_person = deserialize_from_map!(data_map, Person {
        name: String,
        age: u32,
        email: String
    });
    
    println!("Deserialized person: {:?}", deserialized_person);
    
    println!("\n=== Complex Data Transformation ===");
    
    // Create nested JSON structure
    let user_data = json_object! {
        "user_id" => 12345,
        "profile" => json_object! {
            "name" => "Alice",
            "preferences" => json_object! {
                "theme" => "dark",
                "notifications" => true
            }
        },
        "last_login" => "2024-01-15T10:30:00Z"
    };
    
    let nested_json = JsonValue::Object(user_data);
    println!("Nested JSON: {}", nested_json);
    
    // Transform to CSV format
    let user_csv = to_csv! {
        headers: ["User ID", "Name", "Theme", "Notifications", "Last Login"],
        rows: [
            ["12345", "Alice", "dark", "true", "2024-01-15T10:30:00Z"]
        ]
    };
    
    println!("\nTransformed to CSV:\n{}", user_csv);
}
```

Serialization macros automate the conversion between different data  
formats, reducing manual coding and ensuring consistency. The JSON  
macros provide a convenient syntax for creating structured data,  
while CSV macros generate tabular output from structured inputs.  

## Validation and constraint macros

Macros that generate validation logic and enforce constraints on  
data structures and function parameters.  

```rust
macro_rules! validate {
    ($value:expr, $constraint:expr, $error:expr) => {
        if !$constraint($value) {
            return Err($error.to_string());
        }
    };
    
    ($value:expr, range $min:expr..$max:expr) => {
        if $value < $min || $value >= $max {
            return Err(format!("Value {} is out of range {}..{}", $value, $min, $max));
        }
    };
    
    ($value:expr, min_length $len:expr) => {
        if $value.len() < $len {
            return Err(format!("Length {} is less than minimum {}", $value.len(), $len));
        }
    };
    
    ($value:expr, max_length $len:expr) => {
        if $value.len() > $len {
            return Err(format!("Length {} exceeds maximum {}", $value.len(), $len));
        }
    };
    
    ($value:expr, not_empty) => {
        if $value.is_empty() {
            return Err("Value cannot be empty".to_string());
        }
    };
}

macro_rules! constrained_struct {
    (
        $name:ident {
            $(
                $field:ident: $type:ty = $constraint:tt
            ),+ $(,)?
        }
    ) => {
        #[derive(Debug, Clone)]
        struct $name {
            $(pub $field: $type,)+
        }
        
        impl $name {
            fn new($($field: $type),+) -> Result<Self, String> {
                $(
                    validate!($field, $constraint);
                )+
                
                Ok(Self {
                    $($field,)+
                })
            }
            
            $(
                paste::paste! {
                    fn [<set_ $field>](&mut self, value: $type) -> Result<(), String> {
                        validate!(value, $constraint);
                        self.$field = value;
                        Ok(())
                    }
                }
            )+
        }
    };
}

macro_rules! assert_valid {
    ($struct:expr, $($field:ident),+) => {
        $(
            if let Err(e) = $struct.validate_field(stringify!($field), &$struct.$field) {
                panic!("Validation failed for field {}: {}", stringify!($field), e);
            }
        )+
    };
}

macro_rules! business_rules {
    (
        $name:ident for $type:ty {
            $(
                $rule_name:ident: $condition:expr => $error:expr
            ),+ $(,)?
        }
    ) => {
        struct $name;
        
        impl $name {
            $(
                fn $rule_name(value: &$type) -> Result<(), String> {
                    if !($condition)(value) {
                        return Err($error.to_string());
                    }
                    Ok(())
                }
            )+
            
            fn validate_all(value: &$type) -> Result<(), Vec<String>> {
                let mut errors = Vec::new();
                
                $(
                    if let Err(e) = Self::$rule_name(value) {
                        errors.push(e);
                    }
                )+
                
                if errors.is_empty() {
                    Ok(())
                } else {
                    Err(errors)
                }
            }
        }
    };
}

macro_rules! typed_id {
    ($name:ident, $inner:ty) => {
        #[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
        struct $name($inner);
        
        impl $name {
            fn new(value: $inner) -> Result<Self, String> {
                if value == Default::default() {
                    return Err(format!("{} cannot be default value", stringify!($name)));
                }
                Ok($name(value))
            }
            
            fn value(&self) -> $inner {
                self.0
            }
        }
        
        impl std::fmt::Display for $name {
            fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
                write!(f, "{}({})", stringify!($name), self.0)
            }
        }
    };
}

// Create constrained struct with validation
constrained_struct! {
    User {
        name: String = not_empty,
        age: u32 = range 0..150,
        email: String = min_length 5,
    }
}

// Define business rules
business_rules! {
    UserValidation for User {
        valid_email: |user: &User| user.email.contains('@') => "Email must contain @",
        adult_age: |user: &User| user.age >= 18 => "User must be at least 18 years old",
        reasonable_name: |user: &User| user.name.len() <= 50 => "Name must be 50 characters or less"
    }
}

// Create typed IDs
typed_id!(UserId, u64);
typed_id!(ProductId, u32);
typed_id!(OrderId, String);

fn main() {
    println!("=== Constrained Struct Validation ===");
    
    // Valid user
    match User::new("Alice Johnson".to_string(), 25, "alice@example.com".to_string()) {
        Ok(user) => {
            println!("Valid user created: {:?}", user);
            
            // Test business rules
            match UserValidation::validate_all(&user) {
                Ok(()) => println!("All business rules passed"),
                Err(errors) => println!("Business rule violations: {:?}", errors),
            }
        }
        Err(e) => println!("User creation failed: {}", e),
    }
    
    // Invalid user - empty name
    match User::new("".to_string(), 25, "alice@example.com".to_string()) {
        Ok(_) => println!("Unexpected success"),
        Err(e) => println!("Expected validation error: {}", e),
    }
    
    // Invalid user - age out of range
    match User::new("Bob".to_string(), 200, "bob@example.com".to_string()) {
        Ok(_) => println!("Unexpected success"),
        Err(e) => println!("Expected validation error: {}", e),
    }
    
    // Valid user but failing business rules
    if let Ok(young_user) = User::new("Charlie".to_string(), 16, "charlie_at_school".to_string()) {
        match UserValidation::validate_all(&young_user) {
            Ok(()) => println!("Unexpected business rule success"),
            Err(errors) => println!("Expected business rule errors: {:?}", errors),
        }
    }
    
    println!("\n=== Typed ID Validation ===");
    
    // Valid typed IDs
    match UserId::new(12345) {
        Ok(id) => println!("Valid user ID: {}", id),
        Err(e) => println!("Error: {}", e),
    }
    
    match ProductId::new(98765) {
        Ok(id) => println!("Valid product ID: {}", id),
        Err(e) => println!("Error: {}", e),
    }
    
    // Invalid typed ID (default value)
    match UserId::new(0) {
        Ok(_) => println!("Unexpected success"),
        Err(e) => println!("Expected error: {}", e),
    }
    
    // String-based typed ID
    match OrderId::new("ORD-2024-001".to_string()) {
        Ok(id) => println!("Valid order ID: {}", id),
        Err(e) => println!("Error: {}", e),
    }
    
    match OrderId::new("".to_string()) {
        Ok(_) => println!("Unexpected success"),
        Err(e) => println!("Expected error: {}", e),
    }
    
    println!("\n=== Field Mutation Validation ===");
    
    if let Ok(mut user) = User::new("Dave".to_string(), 30, "dave@example.com".to_string()) {
        println!("Original user: {:?}", user);
        
        // Valid field update
        match user.set_age(35) {
            Ok(()) => println!("Age updated successfully"),
            Err(e) => println!("Age update failed: {}", e),
        }
        
        // Invalid field update
        match user.set_age(200) {
            Ok(()) => println!("Unexpected success"),
            Err(e) => println!("Expected age validation error: {}", e),
        }
        
        println!("Final user: {:?}", user);
    }
}
```

Validation macros enforce data integrity by generating constraint checking  
code automatically. Constrained structs ensure invariants are maintained  
throughout the object's lifetime, while business rule macros separate  
domain logic from data structure definitions. Typed IDs prevent mixing  
different identifier types at compile time.  

## Code generation and metaprogramming

Advanced macros that generate complex code structures and enable  
sophisticated metaprogramming patterns.  

```rust
macro_rules! enum_dispatch {
    (
        $trait_name:ident for $enum_name:ident {
            $(
                $variant:ident($type:ty)
            ),+ $(,)?
        }
    ) => {
        enum $enum_name {
            $(
                $variant($type),
            )+
        }
        
        impl $trait_name for $enum_name {
            fn process(&self) -> String {
                match self {
                    $(
                        $enum_name::$variant(inner) => inner.process(),
                    )+
                }
            }
        }
    };
}

macro_rules! generate_api {
    (
        $api_name:ident {
            $(
                $method:ident($($param:ident: $param_type:ty),*) -> $return_type:ty
            ),+ $(,)?
        }
    ) => {
        trait $api_name {
            $(
                fn $method(&self, $($param: $param_type),*) -> $return_type;
            )+
        }
        
        paste::paste! {
            struct [<Mock $api_name>] {
                $(
                    [<$method _calls>]: std::cell::RefCell<Vec<($($param_type,)*)>>,
                    [<$method _return>]: std::cell::RefCell<Option<$return_type>>,
                )+
            }
            
            impl [<Mock $api_name>] {
                fn new() -> Self {
                    Self {
                        $(
                            [<$method _calls>]: std::cell::RefCell::new(Vec::new()),
                            [<$method _return>]: std::cell::RefCell::new(None),
                        )+
                    }
                }
                
                $(
                    fn [<expect_ $method>](&self, return_value: $return_type) {
                        *self.[<$method _return>].borrow_mut() = Some(return_value);
                    }
                    
                    fn [<verify_ $method _called_with>](&self, $($param: $param_type),*) -> bool {
                        self.[<$method _calls>].borrow().contains(&($($param,)*))
                    }
                )+
            }
            
            impl $api_name for [<Mock $api_name>] {
                $(
                    fn $method(&self, $($param: $param_type),*) -> $return_type {
                        self.[<$method _calls>].borrow_mut().push(($($param,)*));
                        self.[<$method _return>].borrow_mut()
                            .take()
                            .unwrap_or_else(|| panic!("No return value set for {}", stringify!($method)))
                    }
                )+
            }
        }
    };
}

macro_rules! builder_pattern {
    (
        $name:ident {
            required: { $($req_field:ident: $req_type:ty),* $(,)? },
            optional: { $($opt_field:ident: $opt_type:ty = $default:expr),* $(,)? }
        }
    ) => {
        #[derive(Debug, Clone)]
        struct $name {
            $($req_field: $req_type,)*
            $($opt_field: $opt_type,)*
        }
        
        paste::paste! {
            struct [<$name Builder>] {
                $($req_field: Option<$req_type>,)*
                $($opt_field: $opt_type,)*
            }
            
            impl [<$name Builder>] {
                fn new() -> Self {
                    Self {
                        $($req_field: None,)*
                        $($opt_field: $default,)*
                    }
                }
                
                $(
                    fn $req_field(mut self, value: $req_type) -> Self {
                        self.$req_field = Some(value);
                        self
                    }
                )*
                
                $(
                    fn $opt_field(mut self, value: $opt_type) -> Self {
                        self.$opt_field = value;
                        self
                    }
                )*
                
                fn build(self) -> Result<$name, String> {
                    $(
                        let $req_field = self.$req_field
                            .ok_or_else(|| format!("Required field '{}' not set", stringify!($req_field)))?;
                    )*
                    
                    Ok($name {
                        $($req_field,)*
                        $($opt_field: self.$opt_field,)*
                    })
                }
            }
            
            impl $name {
                fn builder() -> [<$name Builder>] {
                    [<$name Builder>]::new()
                }
            }
        }
    };
}

macro_rules! derive_operations {
    ($struct_name:ident { $($field:ident),+ $(,)? }) => {
        impl $struct_name {
            fn sum(&self) -> f64 {
                0.0 $(+ self.$field as f64)+
            }
            
            fn product(&self) -> f64 {
                1.0 $(* self.$field as f64)*
            }
            
            fn max(&self) -> f64 {
                vec![$(self.$field as f64),+]
                    .into_iter()
                    .fold(f64::NEG_INFINITY, f64::max)
            }
            
            fn min(&self) -> f64 {
                vec![$(self.$field as f64),+]
                    .into_iter()
                    .fold(f64::INFINITY, f64::min)
            }
            
            fn average(&self) -> f64 {
                self.sum() / ${count($field)}
            }
        }
    };
}

// Define processor trait and implementations
trait Processor {
    fn process(&self) -> String;
}

struct TextProcessor {
    content: String,
}

impl Processor for TextProcessor {
    fn process(&self) -> String {
        format!("Processed text: {}", self.content)
    }
}

struct NumberProcessor {
    value: i32,
}

impl Processor for NumberProcessor {
    fn process(&self) -> String {
        format!("Processed number: {}", self.value * 2)
    }
}

struct ListProcessor {
    items: Vec<String>,
}

impl Processor for ListProcessor {
    fn process(&self) -> String {
        format!("Processed list: [{}]", self.items.join(", "))
    }
}

// Generate enum dispatch
enum_dispatch! {
    Processor for ProcessorEnum {
        Text(TextProcessor),
        Number(NumberProcessor),
        List(ListProcessor),
    }
}

// Generate API and mock
generate_api! {
    UserService {
        get_user(id: u64) -> String,
        create_user(name: String, email: String) -> u64,
        delete_user(id: u64) -> bool,
    }
}

// Generate builder pattern
builder_pattern! {
    DatabaseConfig {
        required: {
            host: String,
            port: u16,
            database: String,
        },
        optional: {
            username: String = "default".to_string(),
            password: String = "".to_string(),
            timeout: u32 = 30,
            pool_size: u32 = 10,
        }
    }
}

// Numeric operations struct
#[derive(Debug)]
struct Statistics {
    a: i32,
    b: i32,
    c: i32,
    d: i32,
}

derive_operations! {
    Statistics { a, b, c, d }
}

fn main() {
    println!("=== Enum Dispatch ===");
    
    let processors = vec![
        ProcessorEnum::Text(TextProcessor {
            content: "Hello, World!".to_string(),
        }),
        ProcessorEnum::Number(NumberProcessor { value: 42 }),
        ProcessorEnum::List(ListProcessor {
            items: vec!["apple".to_string(), "banana".to_string(), "cherry".to_string()],
        }),
    ];
    
    for processor in processors {
        println!("{}", processor.process());
    }
    
    println!("\n=== API Mock Testing ===");
    
    let mock_service = MockUserService::new();
    
    // Set up expectations
    mock_service.expect_get_user("User: Alice".to_string());
    mock_service.expect_create_user(12345);
    mock_service.expect_delete_user(true);
    
    // Use the mock
    let user = mock_service.get_user(1);
    println!("Got user: {}", user);
    
    let new_id = mock_service.create_user("Alice".to_string(), "alice@example.com".to_string());
    println!("Created user with ID: {}", new_id);
    
    let deleted = mock_service.delete_user(1);
    println!("User deleted: {}", deleted);
    
    // Verify calls
    println!("Get user called with 1: {}", 
             mock_service.verify_get_user_called_with(1));
    println!("Create user called with Alice: {}", 
             mock_service.verify_create_user_called_with("Alice".to_string(), "alice@example.com".to_string()));
    
    println!("\n=== Builder Pattern ===");
    
    // Build with all required fields
    let config = DatabaseConfig::builder()
        .host("localhost".to_string())
        .port(5432)
        .database("myapp".to_string())
        .username("admin".to_string())
        .timeout(60)
        .build();
    
    match config {
        Ok(cfg) => println!("Database config: {:?}", cfg),
        Err(e) => println!("Config error: {}", e),
    }
    
    // Try to build without required field
    let incomplete_config = DatabaseConfig::builder()
        .host("localhost".to_string())
        .port(5432)
        // Missing database field
        .build();
    
    match incomplete_config {
        Ok(_) => println!("Unexpected success"),
        Err(e) => println!("Expected error: {}", e),
    }
    
    println!("\n=== Derived Operations ===");
    
    let stats = Statistics { a: 10, b: 20, c: 30, d: 40 };
    println!("Statistics: {:?}", stats);
    println!("Sum: {}", stats.sum());
    println!("Product: {}", stats.product());
    println!("Max: {}", stats.max());
    println!("Min: {}", stats.min());
    println!("Average: {}", stats.average());
}
```

Code generation macros enable sophisticated metaprogramming by creating  
entire code structures automatically. The enum dispatch pattern provides  
type-safe polymorphism, while API generation creates both traits and  
mock implementations for testing. Builder patterns and derived operations  
demonstrate how macros can reduce repetitive implementations while  
maintaining type safety and performance.  

These 25 macro examples showcase the full spectrum of Rust's macro  
capabilities, from simple text substitution to complex code generation.  
Macros enable zero-cost abstractions, domain-specific languages, and  
powerful metaprogramming while maintaining Rust's safety guarantees.  
Understanding these patterns enables writing more expressive and  
maintainable Rust code with reduced boilerplate and increased reusability.  