# Pattern matching




```rust
fn main() {
    let grades = vec!["A", "B", "C", "D", "E", "F", "FX"];
  
    for grade in grades {
        match grade {
            "A" | "B" | "C" | "D" | "E" | "F" => println!("passed"),
            "FX" => println!("failed"),
            _ => println!("invalid"),
        }
    }
}
```


```rust
use rand::Rng;

fn main() {
    let mut rng = rand::rng();
    let r = rng.random_range(-5..=5);

    match r {
        x if x > 0 => println!("The number {} is positive", x),
        0 => println!("The number is zero"),
        _ => println!("The number {} is negative", r),
    }
}
```

