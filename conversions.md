# Conversions

Rust provides powerful and safe mechanisms for converting between different  
types. The type system ensures that conversions are explicit and safe,  
preventing many common programming errors. Rust offers several approaches  
for type conversion including the `as` keyword for simple casting, trait-based  
conversions with `From` and `Into`, and parsing methods for string  
conversions.  

Key conversion mechanisms in Rust:  
- **`as` keyword**: Direct casting between compatible types  
- **`From` and `Into` traits**: Safe, trait-based conversions  
- **`parse()` method**: Converting strings to other types  
- **`TryFrom` and `TryInto`**: Fallible conversions that can fail  
- **`format!` and `toString()`**: Converting types to strings  

Understanding these conversion patterns is essential for writing robust  
Rust code that handles data transformation safely and efficiently.  

## Basic integer conversions

Simple conversions between integer types using the `as` keyword for direct  
casting and type methods for safe conversions.  

```rust
fn main() {
    // Direct casting with 'as' keyword
    let a: i32 = 42;
    let b = a as i64;  // Safe: i32 to i64 (widening)
    let c = a as u32;  // Safe: positive i32 to u32
    
    // Using From trait (safe conversions)
    let d: i64 = i64::from(a);  // Safe: i32 to i64
    let e: u32 = a.try_into().unwrap();  // Fallible: i32 to u32
    
    // Narrowing conversions (potentially unsafe)
    let large: i64 = 300;
    let small = large as i8;  // May truncate! (300 -> 44)
    
    println!("Original: {}, as i64: {}, as u32: {}", a, b, c);
    println!("From trait: {}, try_into: {}", d, e);
    println!("Narrowing: {} -> {} (truncated)", large, small);
}
```

The `as` keyword performs direct casting between compatible numeric types.  
Widening conversions (smaller to larger) are always safe, while narrowing  
conversions may truncate data. The `From` trait provides safe conversions  
where the target type can represent all values of the source type.  
`TryInto` is used for fallible conversions that might fail.  

## String to number parsing

Convert string representations to numeric types using `parse()` method  
with proper error handling for invalid input.  

```rust
fn main() {
    // Basic string parsing
    let num_str = "42";
    let num: i32 = num_str.parse().unwrap();
    println!("Parsed: {}", num);
    
    // With explicit type annotation
    let float_str = "3.14159";
    let pi = float_str.parse::<f64>().unwrap();
    println!("Pi: {:.2}", pi);
    
    // Safe parsing with error handling
    let inputs = ["123", "45.67", "not_a_number", "-89"];
    
    for input in inputs {
        match input.parse::<i32>() {
            Ok(n) => println!("'{}' parsed as: {}", input, n),
            Err(e) => println!("Failed to parse '{}': {}", input, e),
        }
    }
    
    // Using Result with if let
    if let Ok(value) = "999".parse::<u32>() {
        println!("Successfully parsed: {}", value);
    }
}
```

The `parse()` method attempts to convert a string slice to the specified  
type, returning a `Result<T, ParseError>`. The turbofish syntax `::<T>`  
explicitly specifies the target type. Always handle parsing errors as  
invalid strings will cause the parse to fail, not panic the program.  

## Number to string formatting

Convert numeric types to string representations using various formatting  
methods and format specifiers.  

```rust
fn main() {
    let integer = 42;
    let float = 3.14159;
    let large_num = 1_234_567_890u64;
    
    // Basic toString conversion
    let int_str = integer.to_string();
    let float_str = float.to_string();
    
    // Using format! macro for advanced formatting
    let formatted = format!("Integer: {}, Float: {:.2}", integer, float);
    let hex = format!("Hex: 0x{:X}", integer);
    let binary = format!("Binary: 0b{:b}", integer);
    let padded = format!("Padded: {:08}", integer);
    
    // Scientific notation and precision
    let scientific = format!("Scientific: {:.2e}", float);
    let percentage = format!("Percentage: {:.1%}", 0.75);
    
    // Number formatting with separators (using external crate pattern)
    let with_commas = format!("{:,}", large_num);  // Note: requires num-format crate
    
    println!("{}", int_str);
    println!("{}", float_str);
    println!("{}", formatted);
    println!("{}", hex);
    println!("{}", binary);
    println!("{}", padded);
    println!("{}", scientific);
    println!("{}", percentage);
    
    // Direct string conversion methods
    let methods = vec![
        format!("Display: {}", integer),
        format!("Debug: {:?}", integer),
        format!("Lower hex: {:x}", integer),
        format!("Upper hex: {:X}", integer),
        format!("Octal: {:o}", integer),
    ];
    
    for method in methods {
        println!("{}", method);
    }
}
```

Rust provides multiple ways to convert numbers to strings. The `to_string()`  
method works for any type implementing the `ToString` trait. The `format!`  
macro offers extensive formatting options including precision control,  
padding, number bases, and scientific notation. Format specifiers like  
`{:08}` add zero padding, `{:.2}` controls decimal places.  

## Between integer types safely

Demonstrate safe conversions between different integer types using checked  
methods and TryFrom trait to prevent overflow errors.  

```rust
use std::convert::TryFrom;

fn main() {
    let value: i32 = 1000;
    
    // Safe widening conversions (always succeed)
    let as_i64: i64 = value.into();  // i32 -> i64
    let as_i128: i128 = i128::from(value);  // i32 -> i128
    
    println!("Safe widening: {} -> {} -> {}", value, as_i64, as_i128);
    
    // Potentially unsafe narrowing conversions
    let large_value: i64 = 300;
    
    // Using TryFrom for safe narrowing
    match i8::try_from(large_value) {
        Ok(small) => println!("Conversion succeeded: {} -> {}", large_value, small),
        Err(e) => println!("Conversion failed: {}", e),
    }
    
    // Checking bounds manually
    if large_value >= i8::MIN as i64 && large_value <= i8::MAX as i64 {
        let safe_small = large_value as i8;
        println!("Manual bounds check: {} -> {}", large_value, safe_small);
    } else {
        println!("Value {} out of i8 range", large_value);
    }
    
    // Using checked conversion methods
    let test_values = [100i64, 200, 300, -200];
    
    for val in test_values {
        if let Ok(as_i8) = i8::try_from(val) {
            println!("✓ {} fits in i8: {}", val, as_i8);
        } else {
            println!("✗ {} does not fit in i8", val);
        }
    }
    
    // Unsigned to signed conversions
    let unsigned: u32 = 4_000_000_000;  // Close to u32::MAX
    match i32::try_from(unsigned) {
        Ok(signed) => println!("u32 -> i32: {} -> {}", unsigned, signed),
        Err(_) => println!("u32 value {} too large for i32", unsigned),
    }
}
```

Safe integer conversions use `TryFrom` and `TryInto` traits that return  
`Result` types, allowing graceful handling of overflow conditions.  
Widening conversions (smaller to larger types) are always safe and can  
use `From`/`Into`. Narrowing conversions should use checked methods to  
prevent silent data loss or unexpected behavior.  

## Float to integer conversions

Convert floating-point numbers to integers with different rounding  
strategies and overflow handling.  

```rust
fn main() {
    let floats = [3.14, 2.7, -1.5, 999.99, f64::INFINITY, f64::NAN];
    
    for f in floats {
        println!("\nConverting: {}", f);
        
        // Direct casting (truncates toward zero)
        if f.is_finite() {
            let as_int = f as i32;
            println!("  as i32 (truncate): {}", as_int);
        } else {
            println!("  as i32: Cannot convert non-finite value");
        }
        
        // Different rounding strategies
        if f.is_finite() {
            let floor_val = f.floor() as i32;
            let ceil_val = f.ceil() as i32;
            let round_val = f.round() as i32;
            
            println!("  floor: {}", floor_val);
            println!("  ceil: {}", ceil_val);
            println!("  round: {}", round_val);
            
            // Safe conversion with bounds checking
            if f >= i32::MIN as f64 && f <= i32::MAX as f64 {
                println!("  ✓ Within i32 bounds");
            } else {
                println!("  ✗ Outside i32 bounds");
            }
        }
        
        // Fractional and integer parts
        if f.is_finite() {
            let (fract, integer) = (f.fract(), f.trunc());
            println!("  fractional: {}, integer: {}", fract, integer);
        }
    }
    
    // Demonstrating precision loss
    let precise = 16777217.0f32;  // Beyond f32 precision for integers
    let back_to_int = precise as i32;
    println!("\nPrecision demo: {} -> {}", precise, back_to_int);
    
    // Safe float to int with range checking
    fn safe_float_to_int(f: f64) -> Option<i32> {
        if f.is_finite() && f >= i32::MIN as f64 && f <= i32::MAX as f64 {
            Some(f as i32)
        } else {
            None
        }
    }
    
    let test_floats = [42.8, 1e10, -1e10, f64::NAN];
    for f in test_floats {
        match safe_float_to_int(f) {
            Some(i) => println!("Safe conversion: {} -> {}", f, i),
            None => println!("Cannot safely convert: {}", f),
        }
    }
}
```

Float to integer conversion truncates the fractional part toward zero.  
Use `floor()`, `ceil()`, or `round()` for different rounding behaviors.  
Always check for finite values and range bounds when converting floats  
to prevent undefined behavior. Floating-point numbers have limited  
precision for large integers, which can cause data loss.  

## Character and byte conversions

Convert between characters, bytes, and Unicode representations with  
proper handling of encoding and decoding.  

```rust
fn main() {
    // Character to byte conversions
    let ch = 'A';
    let byte = ch as u8;  // ASCII characters only
    println!("Character '{}' as byte: {}", ch, byte);
    
    // Byte to character (ASCII range)
    let ascii_byte = 65u8;
    let ascii_char = ascii_byte as char;
    println!("Byte {} as character: '{}'", ascii_byte, ascii_char);
    
    // Unicode character handling
    let unicode_chars = ['A', 'ñ', '🦀', '中'];
    
    for ch in unicode_chars {
        let code_point = ch as u32;
        let utf8_bytes: Vec<u8> = ch.to_string().bytes().collect();
        let utf8_len = ch.len_utf8();
        
        println!("\nCharacter: '{}'", ch);
        println!("  Unicode code point: U+{:04X} ({})", code_point, code_point);
        println!("  UTF-8 bytes: {:?} ({} bytes)", utf8_bytes, utf8_len);
        println!("  Is ASCII: {}", ch.is_ascii());
    }
    
    // String to bytes and back
    let text = "Hello 🌍";
    let bytes: Vec<u8> = text.bytes().collect();
    let back_to_string = String::from_utf8(bytes.clone()).unwrap();
    
    println!("\nString: '{}'", text);
    println!("As bytes: {:?}", bytes);
    println!("Back to string: '{}'", back_to_string);
    
    // Handling invalid UTF-8
    let invalid_bytes = vec![0xFF, 0xFE, 0xFD];
    match String::from_utf8(invalid_bytes.clone()) {
        Ok(s) => println!("Valid UTF-8: {}", s),
        Err(e) => println!("Invalid UTF-8: {}", e),
    }
    
    // Safe conversion with replacement
    let lossy = String::from_utf8_lossy(&invalid_bytes);
    println!("Lossy conversion: '{}'", lossy);
    
    // Character iterator from bytes
    let byte_string = b"Hello";  // &[u8]
    for &byte in byte_string {
        if byte.is_ascii() {
            println!("ASCII byte {}: '{}'", byte, byte as char);
        }
    }
    
    // Unicode escape sequences
    let escaped = "Unicode: \u{1F980} \u{4E2D}";  // 🦀 中
    println!("Escaped: {}", escaped);
}
```

Character conversions must handle Unicode properly. ASCII characters  
(0-127) can be safely cast to/from bytes. For Unicode characters, use  
`to_string().bytes()` to get UTF-8 encoding. The `from_utf8()` function  
can fail with invalid byte sequences, so use `from_utf8_lossy()` for  
lossy conversion with replacement characters when needed.  

## Collection conversions

Transform between different collection types like vectors, arrays,  
and iterators with efficient conversion methods.  

```rust
fn main() {
    // Array to Vec and back
    let array = [1, 2, 3, 4, 5];
    let vec_from_array: Vec<i32> = array.to_vec();
    let vec_from_iter: Vec<i32> = array.iter().cloned().collect();
    
    println!("Array: {:?}", array);
    println!("Vec from array: {:?}", vec_from_array);
    println!("Vec from iter: {:?}", vec_from_iter);
    
    // Vec to array (fixed size)
    let vec = vec![10, 20, 30];
    let array_from_vec: [i32; 3] = vec.try_into().unwrap();
    println!("Array from vec: {:?}", array_from_vec);
    
    // Vec to slice
    let slice: &[i32] = &vec;
    println!("Slice from vec: {:?}", slice);
    
    // String collections
    let words = vec!["hello", "world", "rust"];
    let sentence: String = words.join(" ");
    let back_to_words: Vec<&str> = sentence.split_whitespace().collect();
    
    println!("Words: {:?}", words);
    println!("Sentence: '{}'", sentence);
    println!("Back to words: {:?}", back_to_words);
    
    // Converting element types
    let str_numbers = vec!["1", "2", "3", "4"];
    let numbers: Result<Vec<i32>, _> = str_numbers
        .iter()
        .map(|s| s.parse::<i32>())
        .collect();
    
    match numbers {
        Ok(nums) => println!("Parsed numbers: {:?}", nums),
        Err(e) => println!("Parse error: {}", e),
    }
    
    // Flattening nested collections
    let nested = vec![vec![1, 2], vec![3, 4, 5], vec![6]];
    let flattened: Vec<i32> = nested.into_iter().flatten().collect();
    println!("Nested: [[1,2], [3,4,5], [6]]");
    println!("Flattened: {:?}", flattened);
    
    // Iterator conversions
    let range_vec: Vec<i32> = (1..=5).collect();
    let doubled: Vec<i32> = range_vec
        .iter()
        .map(|x| x * 2)
        .collect();
    
    println!("Range as vec: {:?}", range_vec);
    println!("Doubled: {:?}", doubled);
    
    // Converting between collection types
    use std::collections::{HashMap, HashSet};
    
    let vec_pairs = vec![(1, "one"), (2, "two"), (3, "three")];
    let hash_map: HashMap<i32, &str> = vec_pairs.iter().cloned().collect();
    let keys: HashSet<i32> = hash_map.keys().cloned().collect();
    
    println!("HashMap: {:?}", hash_map);
    println!("Keys as HashSet: {:?}", keys);
}
```

Collection conversions often use the `collect()` method to transform  
iterators into concrete collection types. The `try_into()` method works  
for fixed-size conversions but requires matching lengths. Use `map()`  
and `filter()` to transform elements during conversion. The `flatten()`  
method is useful for converting nested collections to flat structures.  

## Option and Result conversions

Handle optional values and error cases with conversion methods between  
Option, Result, and regular values.  

```rust
fn main() {
    // Option conversions
    let some_value = Some(42);
    let none_value: Option<i32> = None;
    
    // Option to Result
    let result_from_some = some_value.ok_or("No value present");
    let result_from_none = none_value.ok_or("No value present");
    
    println!("Some as Result: {:?}", result_from_some);
    println!("None as Result: {:?}", result_from_none);
    
    // Option unwrapping with defaults
    let default_value = none_value.unwrap_or(0);
    let computed_default = none_value.unwrap_or_else(|| 42 * 2);
    
    println!("With default: {}", default_value);
    println!("With computed default: {}", computed_default);
    
    // Result conversions
    let success: Result<i32, &str> = Ok(100);
    let failure: Result<i32, &str> = Err("Something went wrong");
    
    // Result to Option
    let option_from_ok = success.ok();
    let option_from_err = failure.ok();
    
    println!("Ok as Option: {:?}", option_from_ok);
    println!("Err as Option: {:?}", option_from_err);
    
    // Mapping over Option and Result
    let mapped_option = some_value.map(|x| x * 2);
    let mapped_result = success.map(|x| x.to_string());
    
    println!("Mapped Option: {:?}", mapped_option);
    println!("Mapped Result: {:?}", mapped_result);
    
    // Chaining operations
    let chained = Some("42")
        .map(|s| s.parse::<i32>())  // Option<Result<i32, ParseIntError>>
        .transpose()               // Result<Option<i32>, ParseIntError>
        .unwrap_or(None);
    
    println!("Chained conversion: {:?}", chained);
    
    // Collecting Results
    let str_nums = vec!["1", "2", "not_a_number", "4"];
    let results: Vec<Result<i32, _>> = str_nums
        .iter()
        .map(|s| s.parse::<i32>())
        .collect();
    
    println!("Parse results: {:?}", results);
    
    // Partition successes and failures
    let (successes, failures): (Vec<_>, Vec<_>) = results
        .into_iter()
        .partition(Result::is_ok);
    
    let success_values: Vec<i32> = successes
        .into_iter()
        .map(Result::unwrap)
        .collect();
    
    println!("Successes: {:?}", success_values);
    println!("Failure count: {}", failures.len());
    
    // Early return pattern simulation
    fn parse_and_double(s: &str) -> Result<i32, std::num::ParseIntError> {
        let num = s.parse::<i32>()?;  // Early return on error
        Ok(num * 2)
    }
    
    let test_inputs = ["5", "abc", "10"];
    for input in test_inputs {
        match parse_and_double(input) {
            Ok(result) => println!("'{}' -> {}", input, result),
            Err(e) => println!("Error parsing '{}': {}", input, e),
        }
    }
}
```

Option and Result types provide safe ways to handle potentially missing  
or erroneous values. Use `ok_or()` to convert Option to Result with a  
custom error. The `transpose()` method swaps Option and Result nesting.  
The `?` operator provides clean early returns for Result types in functions  
that return compatible Result types.  

## Custom type conversions

Implement From and Into traits for custom types to enable idiomatic  
conversions between your own types and standard types.  

```rust
#[derive(Debug)]
struct Temperature {
    celsius: f64,
}

impl Temperature {
    fn new(celsius: f64) -> Self {
        Temperature { celsius }
    }
    
    fn to_fahrenheit(&self) -> f64 {
        self.celsius * 9.0 / 5.0 + 32.0
    }
    
    fn to_kelvin(&self) -> f64 {
        self.celsius + 273.15
    }
}

// Implement From trait for conversion from f64
impl From<f64> for Temperature {
    fn from(celsius: f64) -> Self {
        Temperature::new(celsius)
    }
}

// Implement Into trait for conversion to f64 (Celsius)
impl Into<f64> for Temperature {
    fn into(self) -> f64 {
        self.celsius
    }
}

// Custom conversion with validation
#[derive(Debug)]
struct PositiveNumber(u32);

impl TryFrom<i32> for PositiveNumber {
    type Error = &'static str;
    
    fn try_from(value: i32) -> Result<Self, Self::Error> {
        if value >= 0 {
            Ok(PositiveNumber(value as u32))
        } else {
            Err("Number must be positive")
        }
    }
}

// String conversion traits
#[derive(Debug)]
struct Person {
    name: String,
    age: u8,
}

impl From<(&str, u8)> for Person {
    fn from((name, age): (&str, u8)) -> Self {
        Person {
            name: name.to_string(),
            age,
        }
    }
}

impl std::fmt::Display for Person {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f, "{} (age {})", self.name, self.age)
    }
}

fn main() {
    // Using custom From/Into implementations
    let temp1 = Temperature::from(25.0);
    let temp2: Temperature = 30.0.into();  // Using Into
    
    println!("Temperature 1: {:?} ({}°F, {}K)", 
             temp1, temp1.to_fahrenheit(), temp1.to_kelvin());
    println!("Temperature 2: {:?}", temp2);
    
    // Converting back to f64
    let celsius_value: f64 = temp1.into();
    println!("Back to f64: {}°C", celsius_value);
    
    // TryFrom with validation
    let valid_numbers = [5i32, 0, -3, 42];
    
    for num in valid_numbers {
        match PositiveNumber::try_from(num) {
            Ok(pos_num) => println!("✓ {} -> {:?}", num, pos_num),
            Err(e) => println!("✗ {}: {}", num, e),
        }
    }
    
    // Complex type conversion
    let person1 = Person::from(("Alice", 30));
    let person2: Person = ("Bob", 25).into();
    
    println!("Person 1: {}", person1);
    println!("Person 2: {}", person2);
    
    // Chaining conversions
    let chain_result: Result<PositiveNumber, _> = "42"
        .parse::<i32>()
        .map_err(|_| "Parse error")
        .and_then(|i| PositiveNumber::try_from(i));
    
    println!("Chained conversion: {:?}", chain_result);
    
    // Collection of custom types
    let temp_data = vec![20.0, 25.5, 30.0, -5.0];
    let temperatures: Vec<Temperature> = temp_data
        .into_iter()
        .map(Temperature::from)
        .collect();
    
    for (i, temp) in temperatures.iter().enumerate() {
        println!("Temperature {}: {:?}", i + 1, temp);
    }
}
```

Custom conversions make your types integrate seamlessly with Rust's  
ecosystem. Implement `From` for infallible conversions and `TryFrom`  
for conversions that might fail. The `Into` trait is automatically  
implemented when you implement `From`. Use these traits to create  
clean, idiomatic APIs that feel natural to Rust developers.  

## Reference conversions and borrowing

Convert between owned values, references, and smart pointers while  
understanding borrowing rules and lifetime requirements.  

```rust
fn main() {
    // Basic reference conversions
    let owned_string = String::from("Hello, Rust!");
    let string_ref: &str = &owned_string;  // &String to &str
    let slice_ref: &str = &owned_string[0..5];  // Slice reference
    
    println!("Owned: {}", owned_string);
    println!("Reference: {}", string_ref);
    println!("Slice: {}", slice_ref);
    
    // Vector reference conversions
    let vec = vec![1, 2, 3, 4, 5];
    let vec_ref: &Vec<i32> = &vec;
    let slice_from_vec: &[i32] = &vec;  // &Vec<T> to &[T]
    let slice_range: &[i32] = &vec[1..4];
    
    println!("Vector: {:?}", vec);
    println!("Vec ref: {:?}", vec_ref);
    println!("Slice from vec: {:?}", slice_from_vec);
    println!("Slice range: {:?}", slice_range);
    
    // Mutable reference conversions
    let mut mutable_vec = vec![10, 20, 30];
    {
        let mutable_ref: &mut Vec<i32> = &mut mutable_vec;
        mutable_ref.push(40);
        
        let mutable_slice: &mut [i32] = mutable_ref;
        mutable_slice[0] = 15;
    }  // Mutable borrow ends here
    
    println!("After mutation: {:?}", mutable_vec);
    
    // Converting to owned values
    let borrowed_str = "Hello";
    let owned_from_str = borrowed_str.to_string();
    let owned_from_string = string_ref.to_owned();
    
    println!("Borrowed: {}", borrowed_str);
    println!("Owned from &str: {}", owned_from_str);
    println!("Owned from String ref: {}", owned_from_string);
    
    // Box and smart pointer conversions
    let boxed_value = Box::new(42);
    let value_ref: &i32 = &boxed_value;  // Box<T> to &T
    let owned_value = *boxed_value;      // Box<T> to T (move)
    
    println!("Boxed value reference: {}", value_ref);
    println!("Owned value: {}", owned_value);
    
    // Rc (Reference Counted) conversions
    use std::rc::Rc;
    
    let rc_value = Rc::new(String::from("Shared"));
    let rc_clone = Rc::clone(&rc_value);  // Increment reference count
    let rc_ref: &String = &rc_value;      // Rc<T> to &T
    
    println!("RC value: {}", rc_value);
    println!("RC clone: {}", rc_clone);
    println!("RC reference: {}", rc_ref);
    println!("Reference count: {}", Rc::strong_count(&rc_value));
    
    // Converting function parameters (borrowing patterns)
    fn takes_string_ref(s: &str) {
        println!("Function received: {}", s);
    }
    
    fn takes_slice(slice: &[i32]) {
        println!("Function received slice: {:?}", slice);
    }
    
    // These all work due to deref coercion
    takes_string_ref(&owned_string);     // &String -> &str
    takes_string_ref(string_ref);        // &str -> &str
    takes_string_ref("literal");         // &str -> &str
    
    takes_slice(&vec);                   // &Vec<i32> -> &[i32]
    takes_slice(slice_from_vec);         // &[i32] -> &[i32]
    takes_slice(&[1, 2, 3]);            // &[i32] -> &[i32]
    
    // Lifetime and reference conversion patterns
    fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
        if x.len() > y.len() { x } else { y }
    }
    
    let str1 = "Hello";
    let str2 = "World!";
    let longer = longest(str1, str2);
    println!("Longer string: {}", longer);
    
    // Converting between different string types
    let string_types_demo = || {
        let literal: &'static str = "literal";
        let string: String = String::from("owned");
        let cow_borrowed = std::borrow::Cow::from(literal);
        let cow_owned = std::borrow::Cow::from(string.clone());
        
        println!("Literal: {}", literal);
        println!("String: {}", string);
        println!("Cow borrowed: {}", cow_borrowed);
        println!("Cow owned: {}", cow_owned);
        
        // Converting Cow to owned
        let owned_from_cow = cow_borrowed.into_owned();
        println!("Owned from Cow: {}", owned_from_cow);
    };
    
    string_types_demo();
    
    // Reference counting and weak references
    use std::rc::Weak;
    
    let strong_rc = Rc::new(100);
    let weak_ref: Weak<i32> = Rc::downgrade(&strong_rc);
    
    // Try to upgrade weak reference
    if let Some(upgraded) = weak_ref.upgrade() {
        println!("Weak reference upgraded: {}", upgraded);
    }
    
    drop(strong_rc);  // Drop the strong reference
    
    // Now weak reference cannot be upgraded
    if weak_ref.upgrade().is_none() {
        println!("Weak reference is now invalid");
    }
}
```

Reference conversions are fundamental to Rust's ownership system.  
Deref coercion automatically converts `&String` to `&str` and `&Vec<T>`  
to `&[T]` when needed. Smart pointers like `Box<T>` and `Rc<T>` can be  
dereferenced to access the contained value. Understanding these  
conversions is crucial for writing efficient Rust code that minimizes  
unnecessary cloning while respecting borrowing rules.  
