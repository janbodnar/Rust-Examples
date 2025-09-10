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

