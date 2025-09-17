# Introduction to Rust

Welcome to Rust, a modern systems programming language that emphasizes  
safety, speed, and concurrency. This chapter introduces you to Rust's  
history, core principles, and gets you started with your first programs.  

## Brief History

Rust was originally developed by Mozilla Research, with Graydon Hoare  
starting the project in 2010. The first stable release (Rust 1.0) was  
announced in May 2015. Mozilla created Rust to address the challenges  
of systems programming while maintaining memory safety without garbage  
collection.  

## Core Goals

Rust was designed with three primary goals in mind:  

**Safety**: Rust prevents common programming errors like null pointer  
dereferences, buffer overflows, and memory leaks through its ownership  
system and type checking at compile time.  

**Performance**: Rust aims to be as fast as C and C++ with zero-cost  
abstractions, meaning high-level features don't sacrifice runtime  
performance.  

**Concurrency**: Rust makes it easier to write safe concurrent code  
by preventing data races and other concurrency bugs at compile time.  

## Common Use Cases

Rust is particularly well-suited for:  

**Systems Programming**: Operating systems, embedded systems, and  
low-level system components where performance and control are critical.  

**Web Backends**: High-performance web servers and APIs that need to  
handle many concurrent connections efficiently.  

**Command Line Tools**: Fast, reliable CLI applications that benefit  
from Rust's performance and cross-platform capabilities.  

**Embedded Development**: Programming microcontrollers and IoT devices  
where memory usage and performance matter.  

## Installing Rust

The recommended way to install Rust is through rustup, the official  
Rust toolchain installer.  

### Using rustup

Visit [rustup.rs](https://rustup.rs/) and follow the instructions for  
your operating system, or run this command in your terminal:  

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

On Windows, download and run the rustup installer from the website.  

### Verifying Installation

After installation, restart your terminal and verify Rust is installed:  

```bash
rustc --version
```

You should see output similar to:  
```
rustc 1.89.0 (29483883e 2025-08-04)
```

### Setting Up a Project

Rust provides Cargo, a build tool and package manager. Create a new  
project with:  

```bash
cargo new hello_rust
cd hello_rust
```

This creates a new directory with a basic Rust project structure:  
```
hello_rust/
├── Cargo.toml
└── src/
    └── main.rs
```

## Your First Rust Program

Let's examine the Hello World program that Cargo generated.  

### Hello World

```rust
fn main() {
    println!("Hello, world!");
}
```

This simple program demonstrates Rust's basic syntax. The `main` function  
is the entry point of every Rust program. The `println!` macro prints  
text to the console with a newline.  

### Compiling and Running

To compile and run your program:  

```bash
cargo run
```

You should see:  
```
   Compiling hello_rust v0.1.0 (/path/to/hello_rust)
    Finished dev [unoptimized + debuginfo] target(s) in 1.23s
     Running `target/debug/hello_rust`
Hello, world!
```

Cargo automatically compiles your code and runs the resulting executable.  

## A Second Example

Let's explore a slightly more complex example with variables and a  
function.  

### Variables and Functions

```rust
fn main() {
    let name = "Rustacean";
    let age = 25;
    
    greet_user(name, age);
}

fn greet_user(name: &str, age: i32) {
    println!("Hello, {}! You are {} years old.", name, age);
}
```

This example introduces several concepts:  

- `let` statements create variables (`name` and `age`)  
- Function definitions with parameters and types  
- String literals and integer literals  
- Function calls with arguments  

Replace the contents of `src/main.rs` with this code and run `cargo run`  
again. You should see:  
```
Hello, Rustacean! You are 25 years old.
```

The `&str` type represents a string slice, and `i32` is a 32-bit signed  
integer. Rust requires explicit type annotations for function parameters,  
making the code's intent clear and catching errors early.  

## What's Next

Congratulations! You've successfully installed Rust and written your  
first programs. In the following chapters, you'll explore:  

- Variables, data types, and basic operations  
- Control flow with conditionals and loops  
- Ownership and borrowing - Rust's unique memory management  
- Structs, enums, and pattern matching  
- Error handling and testing  

## Encouragement to Experiment

Don't hesitate to modify the examples and experiment with the code.  
Rust's compiler provides helpful error messages that will guide you  
when you make mistakes. Try changing variable values, adding new  
functions, or exploring what happens when you introduce errors.  

The Rust community is welcoming and helpful. When you're ready to  
explore further, consider visiting:  

- The official Rust documentation at [doc.rust-lang.org](https://doc.rust-lang.org/)  
- The Rust community forum at [users.rust-lang.org](https://users.rust-lang.org/)  

Happy coding with Rust!  