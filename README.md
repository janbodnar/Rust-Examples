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

## 

We cannot directly print the value of `x` and its mutable reference `*y` at  
the same time within the same scope due to the borrow checker's restrictions.    

```rust
fn main() {
    let mut x = 10;
    println!("x has: {x}");

    {
        let y = &mut x;
        *y = 20;

        println!("Value through y: {}", *y);
    }

    println!("x has {x}");
}
```

Another option: 

```rust
fn main() {
    let mut x = 10;
    println!("x has: {x}");

    let y = &mut x;
    *y = 20;

    println!("Value through y: {}", *y); 
    println!("x has {x}");
}
```

This does not compile: 

```rust
fn main() {
    let mut x = 10;
    println!("x has: {x}");

    let y = &mut x;
    *y = 20;

    println!("x has {x}");
    println!("Value through y: {}", *y); 
}
```

The second `println!` macro tries to borrow `x` immutably again, but `x` is already   
borrowed mutably by `y`, which is not allowed in Rust because Rust enforces a rule   
that you can't have mutable and immutable references to the same value in the same scope.  

