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

## Random values 

Shuffling a vector and taking a random sample.  

```rust
use rand::prelude::IteratorRandom;
use rand::seq::SliceRandom;
use rand::thread_rng;

fn main() {
    let mut rng = thread_rng();
    let mut words = vec!["sky", "cup", "word", "cloud"];

    words.shuffle(&mut rng);
    println!("{:?}", words);

    let sample = words.iter().choose_multiple(&mut rng, 2);
    println!("{:?}", sample);
}
```


## Mutable and immutable references in one scope

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
Here `x` is first mutably borrowed by `y`, and after the mutable borrow is used, `x` is then   
immutably borrowed by the last `println!` macro, which is now allowed. This change satisfies   
Rust's borrowing rules.  


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

