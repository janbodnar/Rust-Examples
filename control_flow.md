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

## while loop

The `while` statement is a control flow statement that allows code to be executed  
repeatedly based on a given boolean condition. The `while` keyword executes the statements  
inside the block enclosed by the curly brackets. The statements are executed each time the  
expression is evaluated to true.  

```rust
fn main() {
    let vals = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    let mut i = 0;
    let mut sum = 0;
    let n = vals.len();

    while i < n {
        sum += vals[i];
        i += 1;
    }

    println!("The sum is: {sum}");
}
```

## for loop


Calculating a sum of range of integers.  

```rust
fn main() {

    let mut sum = 0;
    for e in 1..=10 {
        sum += e;
    }

    println!("{sum}");
}
```

Doing a task n times.  

```rust
fn main() {
    for _ in 1..=5 {
        println!("falcon");
    }
}
```

Iterating over an array.  

```rust
fn main() {
    
    let words = ["sky", "oak", "wine", "bottle", "cop", "cloud"];

    for e in words {
        println!("{e}");
    }
}
```

Going over a vector and modifying each element.  

```rust
fn main() {
    let mut vals: Vec<i32> = vec![1, 2, 3, 4, 5];

    for num in &mut vals {
        *num *= 2;
    }

    println!("{vals:?}");
}
```

Iterating over a string.  

```rust
fn main() {
    let msg = "an old falcon";
    for e in msg.chars() {
        println!("{e}");
    }
}
```

## loop keyword

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

## The match expression

Pattern matching is a powerful control flow construct that allows us to  
compare a value against a series of patterns and then execute code based  
on which pattern matches.

```rust
fn main() {
    let grades = ["A", "B", "C", "D", "E", "F", "FX"];

    for grade in grades {
        match grade {
            "A" | "B" | "C" | "D" | "E" | "F" => println!("passed"),
            "FX" => println!("failed"),
            _ => println!("unknown")
        }
    }
}
```

