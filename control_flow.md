# Control flow

## if/else if/else 

```rust
use rand::Rng;

fn main() {
    let mut rng = rand::thread_rng();
    let n = rng.gen_range(-5..5);

    if n < 0 {
        println!("negative value");
    } else if n == 0 {
        println!("zero");
    } else {
        println!("positive value");
    }
}
```

## if expression

The `if` keyword can be used as an expression which returns a value.  

```rust
use rand::Rng;

fn main() {

    let mut rng = rand::thread_rng();
    let n = rng.gen_range(-5..5);

    let msg = if n < 0 {
        "negative value"
    } else if n == 0 {
        "zero"
    } else {
        "positive value"
    };

    println!("{}", msg);
}
```

## The loop keyword

The `loop` keyword creates an endless loop. To terminate the loop, we use  
the `break` statement.

```rust
use rand::{thread_rng, Rng};

fn main() {
    
    let mut rnd = thread_rng();

    loop {
        let r = rnd.gen_range(1..=30);

        print!("{r} ");

        if r == 22 {
            break;
        }
    }
}
```
