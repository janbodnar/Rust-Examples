# Rust strings 


# Simple example

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
