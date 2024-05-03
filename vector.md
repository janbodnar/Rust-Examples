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

## Vec::from

The `Vec::from` creates a vector from an array. The elements are accessed via  
index.  

```rust
fn main() {
    let words = Vec::from(["rock", "pub", "cloud", "book", "chair", "glass"]);

    let n = words.len();

    let w1 = words[0];
    println!("{w1}");

    let w2 = words[n-1];
    println!("{w2}");
}
```
