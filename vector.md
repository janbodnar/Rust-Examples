# Vector 

A vector is a contiguous growable collection of data.  

## for_each function

The `for_each` function is used for declarative iteration.  

```rust
fn main() {
    let mut words = Vec::<&str>::new();
    words.push("sky");
    words.push("road");
    words.push("cloud");

    words.iter().for_each(|word| println!("{word}"));
}
```

## for loop

The `for/in` loop is used for classic, procedural iteration.  

```rust
fn main() {
    let words = vec!["rock", "pub", "cloud", "book", "chair", "glass"];

    println!("the collection has {} elements", words.len());

    for word in &words {
        println!("{word} has {} latin characters", word.len());
    }
}
```
