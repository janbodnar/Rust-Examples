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
