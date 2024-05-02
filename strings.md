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
The line `println!("{s1} {s2} = {}", s1.to_owned() + " " + s2);` cannot go after  
`let s3 = &mut word;`  b/c there cannot be both mutable and immutable references in Rust.  
