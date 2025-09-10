# Rust struct


## Simple

Simple example

```rust
struct User {
    username: String,
    occupation: String,
}

fn main() {
    let user1 = User {
        username: String::from("John Doe"),
        occupation: String::from("gardener"),
    };

    let user2 = User {
        username: String::from("Roger Roe"),
        occupation: String::from("driver"),
    };

    println!("User 1: {} is a {}", user1.username, user1.occupation);
    println!("User 2: {} is a {}", user2.username, user2.occupation);
}
```

## Deriving debug

The `#[derive(Debug)]` attribute allows the usage of `:?` operator.  

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: i32,
}

fn main() {
    let p = Person {
        name: "John".to_owned(),
        age: 42,
    };

    println!("{}", p.name);
    println!("{age}", age = p.age);
    
    println!("{p:?}");
}
```

## Custom Display implementation

```rust
use std::fmt::{Display, Formatter, Result};

struct Person {
    name: String,
    age: i32,
}

impl Display for Person {
    fn fmt(&self, f: &mut Formatter<'_>) -> Result {
        write!(f, "Person {{ name: {}, age: {} }}", self.name, self.age)
    }
}

fn main() {
    let p = Person {
        name: "John Doe".to_owned(),
        age: 34,
    };

    println!("{p}");
}
```

## Struct update syntax

```rust
#[derive(Clone)]
struct User {
    username: String,
    occupation: String,
}

fn main() {
    let user1 = User {
        username: String::from("John Doe"),
        occupation: String::from("gardener"),
    };

    let user2 = User {
        username: String::from("Jane Smith"),
        ..user1.clone()
    };


    println!("User 1: {} is a {}", user1.username, user1.occupation);
    println!("User 2: {} is a {}", user2.username, user2.occupation);
}
```
