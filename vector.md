# Vector 

A vector is a contiguous growable collection of data.  

```rust
fn main() {
    let mut words = Vec::<&str>::new();
    words.push("sky");
    words.push("road");
    words.push("cloud");

    words.iter().for_each(|word| println!("{word}"));
}
```
