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
