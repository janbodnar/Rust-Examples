# Rust-Examples


## Read env variable

```rust
use std::env;

fn main() {
    let key = "HOME";
    match env::var(key) {
        Ok(val) => println!("{}: {:?}", key, val),
        Err(e) => println!("failed to read {}: {}", key, e),
    }
}
```

## Calculate sum & handle return value

```rust
use std::vec;

fn do_sum(vals: &[i32]) -> Result<i32, String> {
    let sum = vals.iter().sum();
    Ok(sum)
}

fn main() {
    let vals = vec![1, 2, 3, 4, 5, 6, 7];
    match do_sum(&vals) {
        Ok(sum) => println!("Sum: {sum}"),
        Err(err) => eprintln!("Error: {err}"),
    }
}
```
