# Rust struct


## Simple

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
