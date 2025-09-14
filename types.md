# Types


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
