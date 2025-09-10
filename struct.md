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

## Field init shorthand

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let width = 50;
    let height = 30;

    // Field init shorthand: variables with same name as struct fields
    let rect = Rectangle { width, height };

    println!("Rectangle: {} x {}", rect.width, rect.height);
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

## Filtering a struct 

```rust
#[derive(Debug)]
struct User {
    username: String,
    occupation: String,
}

fn main() {

    let users = vec![
        User {
            username: String::from("Alice"),
            occupation: String::from("engineer"),
        },
        User {
            username: String::from("Bob"),
            occupation: String::from("designer"),
        },
        User {
            username: String::from("Charlie"),
            occupation: String::from("teacher"),
        },
        User {
            username: String::from("Diana"),
            occupation: String::from("doctor"),
        },
        User {
            username: String::from("Eve"),
            occupation: String::from("teacher"),
        },
    ];

    let teachers: Vec<&User> = users
        .iter()
        .filter(|user| user.occupation == "teacher")
        .collect();

    println!("Teachers found:");

    for teacher in &teachers {
        println!("Teacher: {}", teacher.username);
    }

    println!("Total number of teachers: {}", teachers.len());
    println!("Total number of users: {}", users.len());
    println!("users: {:?}", users);
}
```

## Destructuring

```rust
#[derive(Debug)]
struct User {
    name: String,
    email: String,
    age: u8,
    occupation: String,
}

fn main() {
    let user = User {
        name: String::from("John Doe"),
        email: String::from("john.doe@example.com"),
        age: 30,
        occupation: String::from("gardener"),
    };

    println!("{:#?}", user);

    let User {
        name: u_name,
        email: u_email,
        age: u_age,
        occupation: u_occupation,
    } = user;

    println!("Name: {}", u_name);
    println!("Email: {}", u_email);
    println!("Age: {}", u_age);
    println!("Occupation: {}", u_occupation);
}
```

## Serialize/deserialize

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Book {
    title: String,
    author: String,
    pages: u32,
}

fn main() {
    // Create an instance of Book
    let book = Book {
        title: "Rust Programming".to_string(),
        author: "Paul Smith".to_string(),
        pages: 320,
    };

    // Serialize the struct to a JSON string with pattern matching
    let json = match serde_json::to_string(&book) {
        Ok(json_str) => json_str,
        Err(e) => {
            eprintln!("Serialization error: {}", e);
            return; // Exit early on error
        }
    };
    println!("Serialized Book: {}", json);

    // Deserialize the JSON string back to a struct with pattern matching
    let deserialized: Book = match serde_json::from_str(&json) {
        Ok(book) => book,
        Err(e) => {
            eprintln!("Deserialization error: {}", e);
            return; // Exit early on error
        }
    };
    println!("Deserialized Book: {:?}", deserialized);
}
```

