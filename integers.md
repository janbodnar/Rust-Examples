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
