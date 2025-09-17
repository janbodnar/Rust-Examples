# Closures

Closures in Rust are anonymous functions that can capture variables from  
their enclosing scope. They provide a concise way to define inline  
functions and are commonly used with iterators and functional programming  
patterns. Closures can capture variables by reference, mutable reference,  
or by value.  

## Basic closure syntax

A simple closure that takes no parameters and returns a value.  

```rust
fn main() {
    let simple_closure = || {
        println!("Hello from closure!");
        42
    };

    let result = simple_closure();
    println!("Result: {}", result);
}
```

The closure `|| { ... }` takes no parameters (empty `||`) and returns the  
value `42`. Closures can be called like regular functions using `()`.  

## Closure with parameters

Closures can accept parameters just like regular functions.  

```rust
fn main() {
    let add = |x, y| x + y;
    let multiply = |a: i32, b: i32| -> i32 { a * b };

    println!("3 + 5 = {}", add(3, 5));
    println!("4 * 6 = {}", multiply(4, 6));
}
```

The first closure `|x, y| x + y` uses type inference for parameters and  
return type. The second closure `|a: i32, b: i32| -> i32 { a * b }` has  
explicit type annotations for both parameters and return type.  

## Capturing variables from environment

Closures can capture variables from their surrounding scope.  

```rust
fn main() {
    let x = 10;
    let y = 20;

    let capture_closure = |z| {
        println!("Captured x: {}, y: {}", x, y);
        x + y + z
    };

    let result = capture_closure(5);
    println!("Result: {}", result);
}
```

The closure captures variables `x` and `y` from the surrounding scope by  
reference. These captured variables can be used inside the closure body  
along with the closure's own parameters.  

## Move closures

The `move` keyword forces the closure to take ownership of captured  
variables.  

```rust
fn main() {
    let message = String::from("Hello, World!");
    
    let move_closure = move || {
        println!("Moved message: {}", message);
        message.len()
    };

    let length = move_closure();
    println!("Message length: {}", length);
    
    // println!("{}", message); // This would cause a compile error
}
```

The `move` closure takes ownership of `message`, so the original variable  
cannot be used after the closure is created. This is useful when passing  
closures to other threads or when you need the closure to own its data.  

## Closure as function parameter

Closures can be passed as parameters to functions using trait bounds.  

```rust
fn apply_operation<F>(x: i32, y: i32, operation: F) -> i32
where
    F: Fn(i32, i32) -> i32,
{
    operation(x, y)
}

fn main() {
    let add = |a, b| a + b;
    let multiply = |a, b| a * b;

    println!("5 + 3 = {}", apply_operation(5, 3, add));
    println!("5 * 3 = {}", apply_operation(5, 3, multiply));
    println!("7 - 2 = {}", apply_operation(7, 2, |x, y| x - y));
}
```

The function `apply_operation` accepts any closure that implements the  
`Fn(i32, i32) -> i32` trait. This allows passing different operations  
as closures, including inline closures defined at the call site.  

## Closure with explicit return type

When closures have complex logic, you can specify the return type  
explicitly.  

```rust
fn main() {
    let calculate = |x: f64| -> Result<f64, String> {
        if x < 0.0 {
            Err(String::from("Cannot calculate square root of negative number"))
        } else {
            Ok(x.sqrt())
        }
    };

    match calculate(16.0) {
        Ok(result) => println!("Square root: {}", result),
        Err(error) => println!("Error: {}", error),
    }

    match calculate(-4.0) {
        Ok(result) => println!("Square root: {}", result),
        Err(error) => println!("Error: {}", error),
    }
}
```

The closure returns a `Result<f64, String>` type, allowing for error  
handling. The explicit return type annotation `-> Result<f64, String>`  
makes the closure's intent clear and helps with type checking.  

## Closures with iterator methods

Closures are commonly used with iterator methods like `map`, `filter`,  
and `fold`.  

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    let doubled: Vec<i32> = numbers
        .iter()
        .map(|x| x * 2)
        .collect();

    let evens: Vec<&i32> = numbers
        .iter()
        .filter(|&x| x % 2 == 0)
        .collect();

    let sum = numbers
        .iter()
        .fold(0, |acc, x| acc + x);

    println!("Original: {:?}", numbers);
    println!("Doubled: {:?}", doubled);
    println!("Evens: {:?}", evens);
    println!("Sum: {}", sum);
}
```

The closures `|x| x * 2`, `|&x| x % 2 == 0`, and `|acc, x| acc + x` are  
used with iterator methods to transform, filter, and reduce the data.  
Iterator methods provide a functional programming approach to data  
processing.  

## Storing closures in variables

Closures can be stored in variables and called multiple times.  

```rust
fn main() {
    let mut counter = 0;

    let mut increment = || {
        counter += 1;
        counter
    };

    println!("Count: {}", increment());
    println!("Count: {}", increment());
    println!("Count: {}", increment());

    let formatter = |name: &str, age: u32| {
        format!("Name: {}, Age: {}", name, age)
    };

    println!("{}", formatter("Alice", 30));
    println!("{}", formatter("Bob", 25));
}
```

The closure `increment` captures `counter` mutably and can modify it each  
time it's called. The `formatter` closure can be reused with different  
parameters. Note that `increment` requires `mut` because it mutates the  
captured variable.  

## Fn, FnMut, and FnOnce traits

Rust has three closure traits that define how closures can be called.  

```rust
fn call_fn<F>(closure: F) 
where
    F: Fn() -> i32,
{
    println!("Fn result: {}", closure());
    println!("Fn result: {}", closure()); // Can call multiple times
}

fn call_fn_mut<F>(mut closure: F) 
where
    F: FnMut() -> i32,
{
    println!("FnMut result: {}", closure());
    println!("FnMut result: {}", closure()); // Can call multiple times
}

fn call_fn_once<F>(closure: F) 
where
    F: FnOnce() -> i32,
{
    println!("FnOnce result: {}", closure()); // Can only call once
}

fn main() {
    let x = 10;
    let immutable_closure = || x + 5;
    call_fn(immutable_closure);

    let mut y = 10;
    let mutable_closure = || {
        y += 1;
        y
    };
    call_fn_mut(mutable_closure);

    let owned_string = String::from("Hello");
    let consuming_closure = || {
        owned_string.len() // Moves owned_string
    };
    call_fn_once(consuming_closure);
}
```

`Fn` closures can be called multiple times and borrow immutably. `FnMut`  
closures can be called multiple times and borrow mutably. `FnOnce`  
closures can only be called once and may take ownership of captured  
variables.  

## Closure returning another closure

Closures can return other closures, creating higher-order functions.  

```rust
fn main() {
    let create_adder = |x: i32| {
        move |y: i32| x + y
    };

    let add_five = create_adder(5);
    let add_ten = create_adder(10);

    println!("5 + 3 = {}", add_five(3));
    println!("10 + 7 = {}", add_ten(7));

    let create_multiplier = |factor: f64| {
        move |value: f64| value * factor
    };

    let double = create_multiplier(2.0);
    let triple = create_multiplier(3.0);

    println!("2.5 * 2 = {}", double(2.5));
    println!("2.5 * 3 = {}", triple(2.5));
}
```

The outer closure returns an inner closure that captures the outer  
closure's parameter. The `move` keyword ensures the inner closure owns  
the captured value. This pattern is useful for creating specialized  
functions.  

## Different capture modes

Closures can capture variables in different ways: by reference, by  
mutable reference, or by value.  

```rust
fn main() {
    let mut count = 0;
    let message = String::from("Hello");

    // Capture by immutable reference
    let read_closure = || {
        println!("Count: {}, Message: {}", count, message);
    };

    // Capture by mutable reference
    let mut modify_closure = || {
        count += 1;
        println!("Updated count: {}", count);
    };

    // Capture by value (move)
    let name = String::from("Alice");
    let move_closure = move || {
        println!("Moved name: {}", name);
        name.len()
    };

    read_closure();
    modify_closure();
    modify_closure();
    
    let length = move_closure();
    println!("Name length: {}", length);
    
    // name is no longer accessible here
    println!("Final count: {}", count);
}
```

Rust automatically chooses the most restrictive capture mode that allows  
the closure to work. Use `move` to force ownership transfer, which is  
required when passing closures to other threads.  

## Closures with generic parameters

You can create closures that work with generic types using function  
wrappers.  

```rust
fn create_comparator<T>() -> impl Fn(&T, &T) -> bool 
where
    T: PartialOrd,
{
    |a, b| a > b
}

fn find_max<T, F>(slice: &[T], compare: F) -> Option<&T>
where
    F: Fn(&T, &T) -> bool,
{
    slice.iter().reduce(|a, b| if compare(a, b) { a } else { b })
}

fn main() {
    let numbers = vec![3, 1, 4, 1, 5, 9, 2, 6];
    let strings = vec!["apple", "banana", "cherry", "date"];

    let num_comparator = create_comparator::<i32>();
    let str_comparator = |a: &&str, b: &&str| a.len() > b.len();

    if let Some(max_num) = find_max(&numbers, |a, b| a > b) {
        println!("Max number: {}", max_num);
    }

    if let Some(longest) = find_max(&strings, str_comparator) {
        println!("Longest string: {}", longest);
    }
}
```

Generic closures enable reusable comparison and processing logic that  
works with different types. The `impl Fn` return type allows returning  
closures from functions while maintaining type safety.  

## Environment modification example

Closures can modify their environment and maintain state between calls.  

```rust
fn main() {
    let mut scores = vec![85, 92, 78, 96, 88];
    let mut total_adjustments = 0;

    let mut grade_adjuster = |adjustment: i32| {
        for score in &mut scores {
            *score += adjustment;
            total_adjustments += adjustment.abs();
        }
        println!("Applied adjustment: {}", adjustment);
    };

    println!("Original scores: {:?}", scores);
    
    grade_adjuster(5);  // Boost all scores by 5
    println!("After +5: {:?}", scores);
    
    grade_adjuster(-2); // Reduce all scores by 2
    println!("After -2: {:?}", scores);
    
    println!("Total adjustments made: {}", total_adjustments);

    // Calculate statistics with closure
    let calculate_stats = || {
        let sum: i32 = scores.iter().sum();
        let avg = sum as f64 / scores.len() as f64;
        let max = *scores.iter().max().unwrap();
        let min = *scores.iter().min().unwrap();
        
        (avg, max, min)
    };

    let (average, maximum, minimum) = calculate_stats();
    println!("Stats - Avg: {:.1}, Max: {}, Min: {}", average, maximum, minimum);
}
```

The `grade_adjuster` closure captures both `scores` and  
`total_adjustments` mutably, allowing it to modify the environment.  
The `calculate_stats` closure demonstrates immutable borrowing for  
read-only operations.  

## Practical filtering and processing

Real-world example of using closures for data processing and filtering.  

```rust
#[derive(Debug)]
struct Product {
    name: String,
    price: f64,
    category: String,
    in_stock: bool,
}

fn main() {
    let products = vec![
        Product { name: "Laptop".to_string(), price: 999.99, category: "Electronics".to_string(), in_stock: true },
        Product { name: "Book".to_string(), price: 19.99, category: "Education".to_string(), in_stock: true },
        Product { name: "Phone".to_string(), price: 699.99, category: "Electronics".to_string(), in_stock: false },
        Product { name: "Desk".to_string(), price: 299.99, category: "Furniture".to_string(), in_stock: true },
    ];

    // Filter products with closures
    let available_electronics: Vec<&Product> = products
        .iter()
        .filter(|product| product.category == "Electronics" && product.in_stock)
        .collect();

    // Transform data with closures
    let discounted_prices: Vec<f64> = products
        .iter()
        .filter(|product| product.in_stock)
        .map(|product| product.price * 0.9) // 10% discount
        .collect();

    // Complex processing with closures
    let category_summary = |category: &str| {
        let (count, total_value) = products
            .iter()
            .filter(|p| p.category == category && p.in_stock)
            .fold((0, 0.0), |(count, sum), product| {
                (count + 1, sum + product.price)
            });
        
        if count > 0 {
            println!("{}: {} items, total value: ${:.2}", category, count, total_value);
        }
    };

    println!("Available Electronics: {:?}", available_electronics);
    println!("Discounted prices: {:?}", discounted_prices);
    
    category_summary("Electronics");
    category_summary("Education");
    category_summary("Furniture");
}
```

This example demonstrates practical use of closures for filtering,  
mapping, and reducing data collections. Closures provide a clean,  
functional approach to data processing with minimal boilerplate code.  

## Advanced closure composition

Combining multiple closures to create complex data processing pipelines.  

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    // Create reusable closures
    let is_even = |x: &i32| *x % 2 == 0;
    let square = |x: i32| x * x;
    let add_ten = |x: i32| x + 10;

    // Compose closures in a processing pipeline
    let result: Vec<i32> = numbers
        .iter()
        .filter(is_even)                    // Keep only even numbers
        .map(|&x| square(x))                // Square each number
        .map(add_ten)                       // Add 10 to each result
        .filter(|&x| x < 50)                // Keep results less than 50
        .collect();

    println!("Original: {:?}", numbers);
    println!("Processed: {:?}", result);

    // Function that creates closure compositions
    let create_processor = |threshold: i32| {
        move |numbers: &[i32]| -> Vec<i32> {
            numbers
                .iter()
                .copied()
                .filter(|&x| x > threshold)
                .map(|x| x * 2)
                .collect()
        }
    };

    let process_above_five = create_processor(5);
    let doubled_high = process_above_five(&numbers);
    println!("Numbers > 5, doubled: {:?}", doubled_high);

    // Closure that returns a closure for custom filtering
    let create_range_filter = |min: i32, max: i32| {
        move |x: &i32| *x >= min && *x <= max
    };

    let in_range_3_to_7 = create_range_filter(3, 7);
    let filtered: Vec<&i32> = numbers
        .iter()
        .filter(in_range_3_to_7)
        .collect();

    println!("Numbers 3-7: {:?}", filtered);
}
```

Closure composition allows building complex data processing pipelines  
from simple, reusable components. This functional programming approach  
leads to more readable and maintainable code.  