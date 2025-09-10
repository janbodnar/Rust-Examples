# Rust struct


## Simple

A struct is a custom data type that groups related data together.  
Here's how to define and use a basic struct with named fields.  

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

This example demonstrates the basic syntax for defining a struct with two  
String fields. We create instances by specifying values for each field  
using struct literal syntax. The dot notation (.) is used to access  
individual fields of the struct instances.  

## Field init shorthand

When variable names match struct field names, you can use shorthand  
syntax to initialize struct fields more concisely.  

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

The field init shorthand syntax allows you to omit the field name when  
the variable name matches the field name. Instead of writing  
`Rectangle { width: width, height: height }`, you can simply write  
`Rectangle { width, height }`. This reduces repetition and makes the  
code more readable when variable names align with struct fields.  

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

The `#[derive(Debug)]` attribute automatically implements the Debug trait  
for the struct, enabling the use of `{:?}` format specifier in println!  
macros. This provides a default debug representation that shows the  
struct name and all field values, which is very useful for debugging  
and development purposes.  

## Custom Display implementation

You can implement the Display trait to control how your struct appears  
when printed with `{}` format specifier instead of `{:?}`.  

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

By implementing the Display trait, you define custom formatting for your  
struct when using `{}` in print statements. The `fmt` method receives a  
formatter and returns a Result. The `write!` macro is used to format  
the output string. This gives you full control over how your struct  
appears to users, creating a more polished and readable output format.  

## Struct update syntax

The struct update syntax allows you to create a new struct instance  
based on an existing one, specifying only the fields you want to change.  

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

The `..` syntax copies the remaining fields from another struct instance.  
In this example, `user2` gets a new `username` but inherits the  
`occupation` from `user1.clone()`. This is particularly useful for  
structs with many fields where you only want to modify a few. The  
`Clone` trait is derived to enable the `clone()` method call.  

## Filtering a struct 

This example demonstrates how to work with collections of structs,  
filtering them based on specific field values.  

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

This example shows how to use iterator methods with structs. We create  
a vector of User structs, then use `iter()` to iterate over references,  
`filter()` to select only users with occupation "teacher", and  
`collect()` to gather the results. The closure `|user|` takes a  
reference to each User and checks the occupation field. This pattern  
is common when working with collections of structured data.  

## Destructuring

Destructuring allows you to extract individual fields from a struct  
into separate variables, providing convenient access to struct data.  

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

Destructuring uses pattern matching to extract struct fields into  
individual variables. The syntax `let User { name: u_name, ... }`  
assigns the `name` field to a new variable `u_name`. This technique  
is useful when you need to work with struct fields as separate  
variables, and it moves ownership of the fields to the new variables.  

## Serialize/deserialize

Using the serde crate, you can convert structs to and from various  
data formats like JSON, providing seamless data interchange capabilities.  

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

The `#[derive(Serialize, Deserialize)]` attributes from the serde crate  
automatically implement serialization and deserialization for the  
struct. The `serde_json::to_string()` function converts the struct to  
JSON format, while `serde_json::from_str()` converts JSON back to the  
struct. Error handling with pattern matching ensures robust operation  
when dealing with potentially invalid data.  

## Tuple structs

Tuple structs are structs with unnamed fields, accessed by position  
rather than name. They're useful for simple data grouping.  

```rust
// Tuple struct definition
struct Color(u8, u8, u8);

fn main() {
    // Create an instance of the tuple struct
    let red = Color(255, 0, 0);

    // Access fields by position
    println!("Red: ({}, {}, {})", red.0, red.1, red.2);

    // You can use destructuring, too
    let Color(r, g, b) = red;
    println!("Destructured: r = {}, g = {}, b = {}", r, g, b);
}
```

Tuple structs combine the benefits of structs with the simplicity of  
tuples. Fields are accessed using dot notation with numeric indices  
(`.0`, `.1`, `.2`). They're particularly useful when the meaning of  
each position is clear from context, like RGB color values. You can  
also destructure tuple structs using pattern matching, extracting  
values into named variables for better readability.  

## Default traits

The Default trait provides a convenient way to create instances with  
predefined default values for all fields.  

```rust
#[derive(Debug)]
struct Config {
    host: String,
    port: u16,
}

fn main() {
    // Use the default() method provided by the Default trait
    let mut config = Config::default();
    println!("Default config: {:?}", config);

    // You can override some fields while keeping others default
    config.host = "127.0.0.1".to_string();
    config.port = 8081;
    println!("Updated config: {:?}", config);
}

impl Default for Config {
    fn default() -> Self {
        Config {
            host: "localhost".to_string(),
            port: 8080,
        }
    }
}
```

By implementing the Default trait, you define sensible default values  
for your struct. The `default()` method returns a new instance with  
all fields set to their default values. This is particularly useful  
for configuration structs or when you want to provide fallback values.  
You can create a default instance and then modify only the fields you  
need to change, making your code more flexible and maintainable.  

