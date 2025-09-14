# Rust-Examples

## Underscores in numbers

Underscore characters can be used in number values for readability.  

```rust
fn main() {
    let x = 212_125_874;
    println!("{x}");

    let y = 0b1000_1101;
    println!("{y}");
}
```

## Multiple assignment

```rust
fn main() {
    let (a, b, c) = (1, 2, 3);
    println!("{a} {b} {c}");
}
```

## Join array of strings

```rust
fn main() {
    let words = ["an", "old", "falcon"];

    let msg = words.join(" ");
    println!("{msg}");
}
```

## Array indexing

```rust
fn main() {
    let a = [[1, 2, 3], [4, 5, 6], [7, 8, 9]];

    println!("{:?}", a[0]);
    println!("{:?}", a[2]);
    println!("{}", a[0][0]);
    println!("{}", a[2][1]);
}
```

## Ranges

A range of integers with step.  

```rust
fn main() {
    for val in (1..10).step_by(3) {
        println!("{val}");
    }
}
```

Letters.  

```rust
fn main() {
    let letters = 'a'..='z';

    for e in letters {
        println!("{e}");
    }
}
```

## Safe cube function

```rust
fn main() {
    let small_val: i64 = 1_000_000;
    if let Some(result) = safe_cube(small_val) {
        println!("Cube of {} is {}", small_val, result);
    }

    // This value will overflow
    let large_val: i64 = 3_000_000_000_000_000_000;
    if let Some(result) = safe_cube(large_val) {
        println!("Cube of {} is {}", large_val, result);
    } else {
        println!("Cubing {} would overflow!", large_val);
    }
}

// The function signature now correctly indicates that the operation can fail.
fn safe_cube(val: i64) -> Option<i128> {
    // checked_pow returns Some(result) on success and None on overflow.
    (val as i128).checked_pow(3)
}
```


## Run external program

Running notepad.

```rust
use std::process::Command;

fn main() {
    let mut cmd = Command::new("notepad.exe");
    cmd.spawn().expect("failed to run program");
}
```

The ping command. 

```rust
// Import necessary modules for executing system commands
use std::process::Command;

// Main function that pings a website three times using the system's ping command
fn main() {
    for seq in 1..=3 {
        // Execute the ping command with the specified arguments
        let output = Command::new("ping")
            .arg("-n") // Number of echo requests to send
            .arg("1")  // Send only one packet
            .arg("8.8.8.8") // Target IP address (Google's public DNS)
            .output() // Capture the command's output
            .expect("Failed to execute ping");

        // Check the command's exit status to determine success
        if output.status.success() {
            println!("Reply from 8.8.8.8: seq={}", seq);
        } else {
            println!("Request timed out (seq={})", seq);
        }

        // Wait for a second before the next ping
        std::thread::sleep(std::time::Duration::from_secs(1));
    }
}
```

## Split a string

```rust
fn main() {
    let res = "John Doe gardener".split(' ').collect::<Vec<&str>>();
    res.iter().for_each(|e| println!("{e}"));
}
```

## Regex 

```rust
use regex::Regex;

fn main() {
    let re = Regex::new(r"\s+").unwrap();
    let data = "an\t\told  falcon\nin the\r\nsky";
    let parts: Vec<_> = re.split(data).collect();

    assert_eq!(parts, vec!["an", "old", "falcon", "in", "the", "sky"]);

    println!("{parts:?}");
}
```


## Numeric notations

```rust
fn main() {
fn main() {
    let x = 212;
    println!("{x}");

    let x = 0x2fe;
    println!("{x}");

    let x = 0o242;
    println!("{x}");

    let x = 0b01101110;
    println!("{x}");

    let big = 1_000_000; // same as 1000000
    let hex = 0xFF_FF_FF; // grouped hex
    let bin = 0b1010_1010; // grouped binary

    println!("{big}, {hex}, {bin}");
}
```

## Number literals

```rust
// This program demonstrates different number literals in Rust
// and shows how to work with random numbers using the rand crate

use rand::Rng;

fn main() {
    // Integer literals in different bases
    println!("=== Integer Literals ===");
    
    // Decimal (base 10) - the default number format
    let decimal: i32 = 212;
    println!("Decimal: {}", decimal);
    
    // Hexadecimal (base 16) - prefixed with 0x
    let hex: i32 = 0x2fe;  // 2*16² + 15*16¹ + 14*16⁰ = 512 + 240 + 14 = 766
    println!("Hexadecimal (0x2fe): {}", hex);
    
    // Octal (base 8) - prefixed with 0o
    let octal: i32 = 0o242;  // 2*8² + 4*8¹ + 2*8⁰ = 128 + 32 + 2 = 162
    println!("Octal (0o242): {}", octal);
    
    // Binary (base 2) - prefixed with 0b
    let binary: i32 = 0b01101110;  // 64 + 32 + 8 + 4 + 2 = 110
    println!("Binary (0b01101110): {}", binary);
    
    // Type suffixes for explicit typing
    println!("\n=== Type Suffixes ===");
    let million = 1_000_000;          // Default integer type (i32)
    let million_u64 = 1_000_000u64;   // Explicit u64 type
    let hundred_f32 = 100.0_f32;      // Explicit f32 type
    
    println!("Default integer: {}", million);
    println!("Explicit u64: {}", million_u64);
    println!("Explicit f32: {}", hundred_f32);
    
    // Floating-point literals
    println!("\n=== Floating-Point Literals ===");
    let float_default = 3.14;         // Default float type (f64)
    let float_f32 = 3.14_f32;         // Explicit f32 type
    let float_scientific = 1.2e3;     // Scientific notation (1.2 * 10³ = 1200)
    let float_negative_exp = 5.6e-2;  // Negative exponent (5.6 * 10⁻² = 0.056)
    
    println!("Default float (f64): {}", float_default);
    println!("Explicit f32: {}", float_f32);
    println!("Scientific notation (1.2e3): {}", float_scientific);
    println!("Negative exponent (5.6e-2): {}", float_negative_exp);
    
    // Working with random numbers using the rand crate
    println!("\n=== Random Numbers ===");
    let mut rng = rand::rng();
    
    // Generate random integers in different ranges
    let random_u8: u8 = rng.random();
    let random_range: u32 = rng.random_range(1..=100);
    let random_float: f64 = rng.random();
    
    println!("Random u8: {}", random_u8);
    println!("Random number between 1-100: {}", random_range);
    println!("Random float between 0.0-1.0: {:.4}", random_float);
    
    // Converting between number types
    println!("\n=== Type Conversions ===");
    let small_num: u8 = 100;
    let big_num: u32 = small_num as u32;  // Safe conversion from smaller to larger type
    
    let float_num: f64 = 3.14159;
    let int_from_float: u32 = float_num as u32;  // Truncates the decimal part
    
    println!("u8 {} converted to u32: {}", small_num, big_num);
    println!("f64 {} converted to u32: {}", float_num, int_from_float);
    
    // Mathematical operations with different number types
    println!("\n=== Mathematical Operations ===");
    let a: i32 = 42;
    let b: f64 = 3.14;
    
    // To perform operations between different types, we need to convert them
    let result: f64 = (a as f64) + b;
    println!("{} (i32) + {} (f64) = {:.2}", a, b, result);
    
    // Bitwise operations with binary numbers
    println!("\n=== Bitwise Operations ===");
    let x: u8 = 0b1010;  // 10 in decimal
    let y: u8 = 0b1100;  // 12 in decimal
    
    println!("x = {:04b} ({})", x, x);
    println!("y = {:04b} ({})", y, y);
    println!("x & y = {:04b} ({})", x & y, x & y);  // AND
    println!("x | y = {:04b} ({})", x | y, x | y);  // OR
    println!("x ^ y = {:04b} ({})", x ^ y, x ^ y);  // XOR
    println!("!x = {:04b} ({})", !x, !x);           // NOT (with padding for display)
    
    // Practical example: converting between units
    println!("\n=== Practical Example: Unit Conversion ===");
    let temperature_fahrenheit = 100.0_f32;
    let temperature_celsius = (temperature_fahrenheit - 32.0) * 5.0 / 9.0;
    println!("{}°F is {:.2}°C", temperature_fahrenheit, temperature_celsius);
    
    let distance_miles: f64 = 10.0;
    let distance_kilometers = distance_miles * 1.60934;
    println!("{} miles is {:.2} kilometers", distance_miles, distance_kilometers);
}
```


## Upper and lower bounds

```rust
fn main() {
    let x: i32 = 32;

    println!("{x}");

    println!("{}", i32::MAX);
    println!("{}", i32::MIN);
}
```

## Length of string

The length of a string can be calculated with a `str::len` function or a `len` member function.   

```rust
fn main() {
    let word = "falcon";

    let n1 = word.len();
    let n2 = str::len(word);
    println!("{n1} {n2}");
}
```

## Tuples 

```rust
fn main() {
    let c: (i32, i32) = (0, 1);

    println!("{}", c.0);
    println!("{}", c.1);

    let (mut x, mut y) = (10, 11);

    println!("{x}");
    println!("{y}");

    println!("--------------");

    (x, y) = (y, x);

    println!("{x}");
    println!("{y}");
}
```

```rust
fn main() {

    let nums = (1, 2, 3, 4, 5);
    let (x, y, z, _, _) = nums;

    println!("{x} {y} {z}");
}
```


## Calculate a sum of integers

```rust
fn main() {

    let vals = (1..=10).collect::<Vec<i32>>();
    println!("{:?}", vals);

    let sum: i32 = vals.iter().sum();
    println!("{sum}");

    let mut sum2 = 0;

    for val in vals.iter() {
        sum2 += val;
    }

    println!("{sum2}");

    let sum3 = vals.iter().fold(0, |acc, x| acc + x);
    println!("{sum3}");
}
```


## Print string n times 

```rust
macro_rules! repeat_print {
    ($word:expr, $count:expr) => {
        for _ in 0..$count {
            println!("{}", $word);
        }
    };
}

fn main() {
    for _ in 1..=5 {
        println!("falcon");
    }

    let word = "book";
    (0..5).for_each(|_| println!("{}", word));

    let mut i = 0;
    while i < 5 {
        println!("hawk");
        i += 1;
    }

    "owl\n"
        .repeat(5)
        .lines()
        .for_each(|word| println!("{word}"));

    std::iter::repeat("eagle")
        .take(5)
        .for_each(|word| println!("{word}"));

    repeat_print!("rock", 5);

    let mut count = 0;
    loop {
        if count >= 5 {
            break;
        }
        println!("pigeon");
        count += 1;
    }
}
```

## Read env variable

```rust
use std::env;

fn main() {
    let key = "HOME";
    match env::var(key) {
        Ok(val) => println!("{}: {:?}", key, val),
        Err(e) => println!("failed to read {}: {}", key, e),
    }
}
```

## Calculate sum & handle return value

```rust
use std::vec;

fn do_sum(vals: &[i32]) -> Result<i32, String> {
    let sum = vals.iter().sum();
    Ok(sum)
}

fn main() {
    let vals = vec![1, 2, 3, 4, 5, 6, 7];
    match do_sum(&vals) {
        Ok(sum) => println!("Sum: {sum}"),
        Err(err) => eprintln!("Error: {err}"),
    }
}
```

## Random values 

```rust
use rand::Rng;

fn main() {
    let mut rnd = rand::rng();

    let mut random_vals = vec![];

    for _ in 1..=20 {
        let n = rnd.random_range(1..=100);

        random_vals.push(n);
    }

    println!("{:?}", random_vals);

    // pick random word from array
    let words = ["small", "tall", "lumen", "rock"];


    let random_word = words[rnd.random_range(0..words.len())];
    println!("Random word: {}", random_word);
    
}
```

Shuffling a vector and taking a random sample.  

```rust
use rand::prelude::IteratorRandom;
use rand::seq::SliceRandom;
use rand::rng;

fn main() {
    let mut rng = rng();
    let mut words = vec!["sky", "cup", "word", "cloud"];

    words.shuffle(&mut rng);
    println!("{:?}", words);

    let sample = words.iter().choose_multiple(&mut rng, 2);
    println!("{:?}", sample);
}
```


## Mutable and immutable references in one scope

We cannot directly print the value of `x` and its mutable reference `*y` at  
the same time within the same scope due to the borrow checker's restrictions.    

```rust
fn main() {
    let mut x = 10;
    println!("x has: {x}");

    {
        let y = &mut x;
        *y = 20;

        println!("Value through y: {}", *y);
    }

    println!("x has {x}");
}
```

Another option: 

```rust
fn main() {
    let mut x = 10;
    println!("x has: {x}");

    let y = &mut x;
    *y = 20;

    println!("Value through y: {}", *y); 
    println!("x has {x}");
}
```
Here `x` is first mutably borrowed by `y`, and after the mutable borrow is used, `x` is then   
immutably borrowed by the last `println!` macro, which is now allowed. This change satisfies   
Rust's borrowing rules.  


This does not compile: 

```rust
fn main() {
    let mut x = 10;
    println!("x has: {x}");

    let y = &mut x;
    *y = 20;

    println!("x has {x}");
    println!("Value through y: {}", *y); 
}
```

The second `println!` macro tries to borrow `x` immutably again, but `x` is already   
borrowed mutably by `y`, which is not allowed in Rust because Rust enforces a rule   
that you can't have mutable and immutable references to the same value in the same scope.  

