# Rust strings 

## Simple example 

```rust
fn main() {
    let mut word = String::from("falcon");

    let s1 = &word;
    let s2 = &word;

    println!("{s1} {s2} = {}", s1.to_owned() + " " + s2);

    let s3 = &mut word;

    s3.push('s');
    println!("{s3}");
}
```
The line `println!("{s1} {s2} = {}", s1.to_owned() + " " + s2);` cannot go after  
`let s3 = &mut word;`  b/c there cannot be both mutable and immutable references in Rust.  

## Parse string

```rust
fn main() {
    if let Ok(n) = "123".parse::<i32>() {
        println!("{}", n);
    }

    let parsed: Result<i32, _> = "123".parse();
    if let Ok(n) = parsed {
        println!("{}", n);
    }
}
```


## String interpolation

```rust
fn main() {
    // Basic variable interpolation
    let name = "John Doe";
    let occupation = "gardener";
    println!("My name is {name}, I am a {occupation}.");
    
    // Positional arguments
    println!("My name is {0}, I am a {1}.", name, occupation);
    
    // Named arguments
    println!("My name is {fullname}, I am a {job}.", fullname = name, job = occupation);
    
    // Formatting numbers with precision
    let price = 12.3456;
    println!("The price is {:.2}", price);
    
    // Formatting with different bases
    let number = 42;
    println!("Binary: {:b}, Octal: {:o}, Hexadecimal: {:x}", number, number, number);
    
    // Debug formatting
    let numbers = vec![1, 2, 3];
    println!("Debug output: {:?}", numbers);
    
    // Pretty print for complex data structures
    let map = std::collections::HashMap::from([
        ("sk", "Slovakia"),
        ("cz", "Czechia"),
        ("pl", "Poland"),
        ("hu", "Hungary"),
    ]);
    println!("Pretty print: {:#?}", map);
}
```

## The format! macro

```rust
use std::collections::HashMap;

fn main() {
    // The format! macro is used to create formatted strings in Rust.
    // It works similarly to the println! macro but returns a String
    // instead of printing to the console.

    let name = "John Doe";
    let occupation = "gardener";
    let number = 42;
    let price = 12.34;
    let numbers = vec![1, 2, 3, 4, 5];
    let map = HashMap::from([
        ("sk", "Slovakia"),
        ("cz", "Czechia"),
        ("pl", "Poland"),
        ("hu", "Hungary"),
    ]);

    // Detailed format! macro examples
    println!("\n=== format! macro examples ===");

    // Basic usage of format!
    let formatted_string = format!("My name is {name}, I am a {occupation}.");
    println!("{}", formatted_string);

    // Format with positional arguments
    let formatted_string = format!("My name is {0}, I am a {1}.", name, occupation);
    println!("{}", formatted_string);

    // Format with named arguments
    let formatted_string = format!(
        "My name is {fullname}, I am a {job}.",
        fullname = name,
        job = occupation
    );
    println!("{}", formatted_string);

    // Format with number formatting
    let formatted_string = format!("The price is {:.2}", price);
    println!("{}", formatted_string);

    // Format with different bases
    let formatted_string = format!(
        "Binary: {:b}, Octal: {:o}, Hexadecimal: {:x}",
        number, number, number
    );
    println!("{}", formatted_string);

    // Format with padding and alignment
    let text = "Hello";
    let formatted_string = format!("Left aligned: {:<10}!", text);
    println!("{}", formatted_string);

    let formatted_string = format!("Right aligned: {:>10}!", text);
    println!("{}", formatted_string);

    let formatted_string = format!("Center aligned: {:^10}!", text);
    println!("{}", formatted_string);

    // Format with padding character
    let formatted_string = format!("Padded with zeros: {:0>5}!", 7);
    println!("{}", formatted_string);

    // Format with debug and pretty print
    let formatted_string = format!("Debug output: {:?}", numbers);
    println!("{}", formatted_string);

    let formatted_string = format!("Pretty print: {:#?}", map);
    println!("{}", formatted_string);

    // Complex formatting example
    let score = 95.789;
    let player = "Alice";
    let level = 3;
    let formatted_string = format!(
        "Player: {:<10} | Score: {:>8.1} | Level: {:0>2} | Status: {:?}",
        player,
        score,
        level,
        if score > 90.0 { "Excellent" } else { "Good" }
    );
    println!("{}", formatted_string);
}
```
