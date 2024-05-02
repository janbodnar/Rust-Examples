# Rust-Examples

## Tuples 

```rust
fn main() {
    let c: (i32, i32) = (0, 1);

    println!("{}", c.0);
    println!("{}", c.1);

    let (mut x, mut y) = (10, 11);

    println!("{x}");
    println!("{y}");

    println!("--------------");

    (x, y) = (y, x);

    println!("{x}");
    println!("{y}");
}
```

```rust
fn main() {

    let nums = (1, 2, 3, 4, 5);
    let (x, y, z, _, _) = nums;

    println!("{x} {y} {z}");
}
```


## Calculate a sum of integers

```rust
fn main() {

    let vals = (1..=10).collect::<Vec<i32>>();
    println!("{:?}", vals);

    let sum: i32 = vals.iter().sum();
    println!("{sum}");

    let mut sum2 = 0;

    for val in vals.iter() {
        sum2 += val;
    }

    println!("{sum2}");

    let sum3 = vals.iter().fold(0, |acc, x| acc + x);
    println!("{sum3}");
}
```


## Print string n times 

```rust
fn main() {
    
    for _ in 1..=5 {
        println!("falcon");
    }

    let mut i = 0;
    while i < 5 {
        println!("hawk");
        i += 1;
    }

    "owl\n".repeat(5).lines().for_each(|word| println!("{word}"));

    std::iter::repeat("eagle").take(5).for_each(|word| println!("{word}"));
}
```

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

