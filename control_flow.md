# Control flow


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
