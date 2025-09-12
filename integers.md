# Integers



```rust
fn main() {
    println!("{:<8} {:>40} {:>40}", "Type", "Min", "Max");
    println!("{:-<100}", "");

    // Signed integers
    println!("{:<8} {:>40} {:>40}", "i8", i8::MIN, i8::MAX);
    println!("{:<8} {:>40} {:>40}", "i16", i16::MIN, i16::MAX);
    println!("{:<8} {:>40} {:>40}", "i32", i32::MIN, i32::MAX);
    println!("{:<8} {:>40} {:>40}", "i64", i64::MIN, i64::MAX);
    println!("{:<8} {:>40} {:>40}", "i128", i128::MIN, i128::MAX);
    println!("{:<8} {:>40} {:>40}", "isize", isize::MIN, isize::MAX);

    println!("{:-<100}", "");

    // Unsigned integers
    println!("{:<8} {:>40} {:>40}", "u8", u8::MIN, u8::MAX);
    println!("{:<8} {:>40} {:>40}", "u16", u16::MIN, u16::MAX);
    println!("{:<8} {:>40} {:>40}", "u32", u32::MIN, u32::MAX);
    println!("{:<8} {:>40} {:>40}", "u64", u64::MIN, u64::MAX);
    println!("{:<8} {:>40} {:>40}", "u128", u128::MIN, u128::MAX);
    println!("{:<8} {:>40} {:>40}", "usize", usize::MIN, usize::MAX);
}
```

##  Rust Integer Types: Bounds & Overflow Methods

Table of all **primitive integer types in Rust**, showing their **bit width**, **value range**,  
and **overflow-safe methods** for arithmetic operations:

| Type    | Bits | Min Value              | Max Value              | Wrapping | Checked | Saturating | Overflowing |
|---------|------|------------------------|------------------------|----------|---------|------------|-------------|
| `i8`    | 8    | -128                   | 127                    | ✔︎        | ✔︎       | ✔︎          | ✔︎           |
| `u8`    | 8    | 0                      | 255                    | ✔︎        | ✔︎       | ✔︎          | ✔︎           |
| `i16`   | 16   | -32,768                | 32,767                 | ✔︎        | ✔︎       | ✔︎          | ✔︎           |
| `u16`   | 16   | 0                      | 65,535                 | ✔︎        | ✔︎       | ✔︎          | ✔︎           |
| `i32`   | 32   | -2,147,483,648         | 2,147,483,647          | ✔︎        | ✔︎       | ✔︎          | ✔︎           |
| `u32`   | 32   | 0                      | 4,294,967,295          | ✔︎        | ✔︎       | ✔︎          | ✔︎           |
| `i64`   | 64   | -9,223,372,036,854,775,808 | 9,223,372,036,854,775,807 | ✔︎        | ✔︎       | ✔︎          | ✔︎           |
| `u64`   | 64   | 0                      | 18,446,744,073,709,551,615 | ✔︎        | ✔︎       | ✔︎          | ✔︎           |
| `i128`  | 128  | -2⁽¹²⁷⁾                | 2⁽¹²⁷⁾−1               | ✔︎        | ✔︎       | ✔︎          | ✔︎           |
| `u128`  | 128  | 0                      | 2⁽¹²⁸⁾−1               | ✔︎        | ✔︎       | ✔︎          | ✔︎           |
| `isize`| Arch | varies (e.g. −2³¹ to 2³¹−1 on 32-bit) | varies (e.g. 2³²−1 on 32-bit) | ✔︎        | ✔︎       | ✔︎          | ✔︎           |
| `usize`| Arch | 0                      | varies (e.g. 2³²−1 on 32-bit) | ✔︎        | ✔︎       | ✔︎          | ✔︎           |

---

## Overflow-Safe Methods (on all integer types):

- `.wrapping_add(x)` → wraps around on overflow  
- `.checked_add(x)` → returns `None` on overflow  
- `.saturating_add(x)` → clamps to max/min  
- `.overflowing_add(x)` → returns `(value, overflowed: bool)`

These methods exist for `add`, `sub`, `mul`, `div`, and `neg` where applicable.


```rust
let mut y = i8::MAX; // y = 127
println!("{y}");

y += 1;              // Attempt to increment beyond i8::MAX
println!("{y}");
```

### Behavior depends on build mode:

| Mode        | Behavior on overflow       | Result of `y += 1` |
|-------------|----------------------------|---------------------|
| **Debug**   | **Panics at runtime**      | Program crashes     |
| **Release** | **Wraps around silently**  | `y` becomes `-128`  |

- In **debug mode** (default when using `cargo run`), Rust inserts runtime checks
  and will **panic** if you exceed bounds.
- In **release mode** (`cargo run --release`), Rust skips these checks for performance
  and uses **two’s complement wrapping**.

### Idiomatic ways to handle overflow:

Rust offers explicit methods to control overflow behavior:

```rust
let x: i8 = 127;

// Wrapping (like release mode)
let wrap = x.wrapping_add(1); // -128

// Checked (returns Option)
let checked = x.checked_add(1); // None

// Saturating (clamps to max)
let saturate = x.saturating_add(1); // 127

// Overflowing (returns tuple)
let (val, overflowed) = x.overflowing_add(1); // (-128, true)
```


Rust’s strict overflow handling in debug mode is part of its safety guarantees — catching  
bugs early and making overflow **explicit**, not silent.




