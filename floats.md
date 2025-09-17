# Floats

Rust provides two primary floating-point types: `f32` (32-bit) and `f64`  
(64-bit). These types follow the IEEE 754 standard for floating-point  
arithmetic. The `f64` type is the default for floating-point literals  
and provides higher precision, while `f32` uses less memory and can be  
faster on some systems.  

Floating-point numbers can represent real numbers with fractional parts,  
scientific notation, and special values like infinity and NaN (Not a  
Number). Understanding floating-point behavior is crucial for numerical  
computing and avoiding precision-related bugs in calculations.  

## Basic float types

Declaration and initialization of floating-point variables with explicit  
type annotations and default type inference.  

```rust
fn main() {
    // Default type inference (f64)
    let default_float = 3.14159;
    let scientific = 1.23e-4;
    
    // Explicit f32 type
    let float32: f32 = 2.718;
    let float32_suffix = 2.718_f32;
    
    // Explicit f64 type
    let float64: f64 = 3.14159265359;
    let float64_suffix = 3.14159265359_f64;
    
    // Zero values
    let zero_f32 = 0.0_f32;
    let zero_f64 = 0.0_f64;
    
    println!("Default (f64): {}", default_float);
    println!("Scientific: {}", scientific);
    println!("f32: {}", float32);
    println!("f32 with suffix: {}", float32_suffix);
    println!("f64: {}", float64);
    println!("f64 with suffix: {}", float64_suffix);
    println!("Zero f32: {}, Zero f64: {}", zero_f32, zero_f64);
}
```

Rust infers `f64` as the default floating-point type when no explicit  
type is specified. Type suffixes (`_f32`, `_f64`) provide an alternative  
to explicit type annotations. Both integer and fractional parts can use  
underscores for readability in large numbers.  

## Scientific notation and special values

Floating-point literals support scientific notation and special IEEE 754  
values including infinity and NaN.  

```rust
fn main() {
    // Scientific notation
    let small = 1.23e-10;      // 1.23 × 10^-10
    let large = 4.56e8;        // 4.56 × 10^8
    let negative_exp = 7.89e-5; // 7.89 × 10^-5
    let positive_exp = 2.34e3;  // 2.34 × 10^3
    
    // Special values
    let infinity = f64::INFINITY;
    let neg_infinity = f64::NEG_INFINITY;
    let nan = f64::NAN;
    
    // Constants
    let pi = std::f64::consts::PI;
    let e = std::f64::consts::E;
    let ln_2 = std::f64::consts::LN_2;
    
    println!("Scientific notation:");
    println!("Small: {}", small);
    println!("Large: {}", large);
    println!("Negative exp: {}", negative_exp);
    println!("Positive exp: {}", positive_exp);
    
    println!("\nSpecial values:");
    println!("Infinity: {}", infinity);
    println!("Negative infinity: {}", neg_infinity);
    println!("NaN: {}", nan);
    
    println!("\nMath constants:");
    println!("π: {}", pi);
    println!("e: {}", e);
    println!("ln(2): {}", ln_2);
}
```

Scientific notation uses `e` or `E` followed by the exponent. Rust  
provides constants for infinity, negative infinity, and NaN through  
the `f32` and `f64` types. Mathematical constants are available in  
`std::f32::consts` and `std::f64::consts` modules.  

## Arithmetic operations and precision

Basic arithmetic operations with floating-point numbers and demonstration  
of precision differences between f32 and f64.  

```rust
fn main() {
    let a = 10.5_f64;
    let b = 3.2_f64;
    
    // Basic arithmetic
    let sum = a + b;
    let difference = a - b;
    let product = a * b;
    let quotient = a / b;
    let remainder = a % b;
    
    println!("a = {}, b = {}", a, b);
    println!("Sum: {}", sum);
    println!("Difference: {}", difference);
    println!("Product: {}", product);
    println!("Quotient: {}", quotient);
    println!("Remainder: {}", remainder);
    
    // Precision comparison between f32 and f64
    let precise_f64 = 0.1_f64 + 0.2_f64;
    let precise_f32 = 0.1_f32 + 0.2_f32;
    
    println!("\nPrecision comparison:");
    println!("f64: 0.1 + 0.2 = {:.17}", precise_f64);
    println!("f32: 0.1 + 0.2 = {:.17}", precise_f32);
    println!("Expected: 0.3");
    
    // Demonstrating precision loss
    let large = 16777216.0_f32;
    let incremented = large + 1.0;
    println!("\nPrecision loss:");
    println!("f32: {} + 1.0 = {}", large, incremented);
    println!("Lost precision: {}", incremented == large);
}
```

Floating-point arithmetic follows standard mathematical operations but  
with limited precision. The classic example of `0.1 + 0.2` demonstrates  
how binary floating-point cannot exactly represent some decimal fractions.  
F32 has about 7 decimal digits of precision while f64 has about 15-17.  

## Comparison and ordering

Floating-point comparison requires special handling due to NaN values  
and precision limitations.  

```rust
use std::cmp::Ordering;

fn main() {
    let a = 3.14;
    let b = 3.14;
    let c = 2.71;
    let nan = f64::NAN;
    
    // Basic comparisons
    println!("a == b: {}", a == b);
    println!("a > c: {}", a > c);
    println!("a < c: {}", a < c);
    
    // NaN comparisons (always false)
    println!("\nNaN comparisons:");
    println!("nan == nan: {}", nan == nan);
    println!("nan > a: {}", nan > a);
    println!("nan < a: {}", nan < a);
    
    // Safe comparison with epsilon
    fn approx_equal(x: f64, y: f64, epsilon: f64) -> bool {
        (x - y).abs() < epsilon
    }
    
    let x = 0.1 + 0.2;
    let y = 0.3;
    println!("\nEpsilon comparison:");
    println!("Direct: {} == {} = {}", x, y, x == y);
    println!("Approx: {} ≈ {} = {}", x, y, approx_equal(x, y, 1e-10));
    
    // Partial ordering
    let values = [1.0, 2.5, f64::NAN, 3.14, f64::INFINITY];
    for &val in &values {
        match val.partial_cmp(&2.0) {
            Some(Ordering::Less) => println!("{} < 2.0", val),
            Some(Ordering::Equal) => println!("{} == 2.0", val),
            Some(Ordering::Greater) => println!("{} > 2.0", val),
            None => println!("{} is not comparable to 2.0", val),
        }
    }
}
```

Direct equality comparison of floating-point numbers can be unreliable  
due to precision errors. Use epsilon-based comparison for approximate  
equality. NaN values are not equal to anything, including themselves,  
and comparisons with NaN always return false.  

## Mathematical functions

Standard mathematical functions available for floating-point calculations  
including trigonometry, logarithms, and power functions.  

```rust
fn main() {
    let x = 2.0;
    let angle = std::f64::consts::PI / 4.0; // 45 degrees in radians
    
    // Power and root functions
    println!("Power and roots:");
    println!("{}.sqrt() = {}", x, x.sqrt());
    println!("{}.cbrt() = {}", x, x.cbrt());
    println!("{}.powf(3.0) = {}", x, x.powf(3.0));
    println!("{}.powi(4) = {}", x, x.powi(4));
    println!("2.0.powf(0.5) = {}", 2.0_f64.powf(0.5));
    
    // Logarithmic functions
    println!("\nLogarithms:");
    println!("{}.ln() = {}", x, x.ln());
    println!("{}.log10() = {}", x, x.log10());
    println!("{}.log2() = {}", x, x.log2());
    println!("{}.log({}) = {}", 100.0, x, 100.0_f64.log(x));
    
    // Trigonometric functions
    println!("\nTrigonometry (angle = π/4):");
    println!("sin({}) = {}", angle, angle.sin());
    println!("cos({}) = {}", angle, angle.cos());
    println!("tan({}) = {}", angle, angle.tan());
    
    // Inverse trigonometric functions
    let value = 0.5;
    println!("\nInverse trigonometry:");
    println!("asin({}) = {}", value, value.asin());
    println!("acos({}) = {}", value, value.acos());
    println!("atan({}) = {}", value, value.atan());
    println!("atan2({}, {}) = {}", value, 1.0, value.atan2(1.0));
    
    // Hyperbolic functions
    println!("\nHyperbolic:");
    println!("sinh({}) = {}", x, x.sinh());
    println!("cosh({}) = {}", x, x.cosh());
    println!("tanh({}) = {}", x, x.tanh());
}
```

Rust provides a comprehensive set of mathematical functions through  
methods on floating-point types. Trigonometric functions work with  
radians, not degrees. The `powf()` method accepts floating-point  
exponents while `powi()` is optimized for integer exponents.  

## Rounding and truncation methods

Various methods for rounding floating-point numbers to integers or  
controlling decimal precision.  

```rust
fn main() {
    let values = [3.2, 3.7, -2.3, -2.8, 3.5, -3.5];
    
    println!("{:<8} {:<8} {:<8} {:<8} {:<8} {:<8}", 
             "Value", "floor", "ceil", "round", "trunc", "fract");
    println!("{:-<50}", "");
    
    for &val in &values {
        println!("{:<8.1} {:<8} {:<8} {:<8} {:<8} {:<8.1}", 
                 val, 
                 val.floor() as i32,
                 val.ceil() as i32,
                 val.round() as i32,
                 val.trunc() as i32,
                 val.fract());
    }
    
    // Rounding to specific decimal places
    let pi = std::f64::consts::PI;
    println!("\nRounding π to decimal places:");
    for decimals in 0..=5 {
        let factor = 10.0_f64.powi(decimals);
        let rounded = (pi * factor).round() / factor;
        println!("{} decimal places: {:.precision$}", 
                 decimals, rounded, precision = decimals as usize);
    }
    
    // Custom rounding function
    fn round_to_places(value: f64, places: u32) -> f64 {
        let factor = 10.0_f64.powi(places as i32);
        (value * factor).round() / factor
    }
    
    let test_value = 123.456789;
    println!("\nCustom rounding of {}:", test_value);
    for places in 0..=4 {
        println!("{} places: {}", places, round_to_places(test_value, places));
    }
}
```

The `floor()` method rounds down to the nearest integer, `ceil()` rounds  
up, `round()` rounds to the nearest integer (ties round away from zero),  
and `trunc()` removes the fractional part. The `fract()` method returns  
only the fractional component.  

## Float to integer conversions

Converting floating-point numbers to integers with different strategies  
and overflow handling.  

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
    }
    
    // Safe conversion function
    fn safe_float_to_int(f: f64) -> Option<i32> {
        if f.is_finite() && f >= i32::MIN as f64 && f <= i32::MAX as f64 {
            Some(f as i32)
        } else {
            None
        }
    }
    
    let test_values = [42.8, 1e10, -1e10, f64::NAN];
    println!("\nSafe conversions:");
    for f in test_values {
        match safe_float_to_int(f) {
            Some(i) => println!("{} -> {}", f, i),
            None => println!("{} -> Cannot convert safely", f),
        }
    }
}
```

Direct casting with `as` truncates the fractional part toward zero.  
Always check for finite values and range bounds when converting to  
prevent undefined behavior. Use rounding methods before conversion  
for different rounding behaviors.  

## String parsing and formatting

Converting between strings and floating-point numbers with error handling  
and various formatting options.  

```rust
fn main() {
    // Parsing strings to floats
    let float_strings = ["3.14", "2.718e2", "infinity", "NaN", "invalid"];
    
    println!("Parsing strings to f64:");
    for s in float_strings {
        match s.parse::<f64>() {
            Ok(f) => println!("'{}' -> {}", s, f),
            Err(e) => println!("'{}' -> Error: {}", s, e),
        }
    }
    
    // Safe parsing with unwrap_or
    let safe_parse = |s: &str| s.parse::<f64>().unwrap_or(0.0);
    println!("\nSafe parsing with default:");
    for s in float_strings {
        println!("'{}' -> {}", s, safe_parse(s));
    }
    
    // Formatting floats to strings
    let pi = std::f64::consts::PI;
    let large = 1234567.89;
    
    println!("\nFormatting options:");
    println!("Default: {}", pi);
    println!("2 decimals: {:.2}", pi);
    println!("6 decimals: {:.6}", pi);
    println!("Scientific: {:e}", pi);
    println!("Scientific uppercase: {:E}", pi);
    println!("Debug: {:?}", pi);
    
    println!("\nLarge number formatting:");
    println!("Default: {}", large);
    println!("Scientific: {:e}", large);
    println!("With width: {:12.2}", large);
    println!("Left aligned: {:<12.2}", large);
    println!("Right aligned: {:>12.2}", large);
    println!("Zero padded: {:012.2}", large);
    
    // Converting to string
    let as_string = pi.to_string();
    let formatted = format!("{:.3}", pi);
    println!("\nString conversion:");
    println!("to_string(): {}", as_string);
    println!("format!: {}", formatted);
}
```

The `parse()` method converts strings to floating-point numbers and  
returns a `Result` for error handling. Rust supports parsing scientific  
notation, infinity, and NaN. Various formatting specifiers control  
decimal places, scientific notation, and alignment.  

## IEEE 754 special values

Working with special floating-point values including NaN, infinity,  
and testing for various float properties.  

```rust
fn main() {
    // Creating special values
    let positive_inf = f64::INFINITY;
    let negative_inf = f64::NEG_INFINITY;
    let nan = f64::NAN;
    let zero = 0.0;
    let negative_zero = -0.0;
    
    let values = [1.0, positive_inf, negative_inf, nan, zero, negative_zero];
    
    println!("{:<15} {:<8} {:<8} {:<8} {:<8} {:<8} {:<8}", 
             "Value", "finite", "infinite", "nan", "normal", "sign", "subnormal");
    println!("{:-<75}", "");
    
    for &val in &values {
        println!("{:<15} {:<8} {:<8} {:<8} {:<8} {:<8} {:<8}", 
                 format!("{:?}", val),
                 val.is_finite(),
                 val.is_infinite(),
                 val.is_nan(),
                 val.is_normal(),
                 val.is_sign_positive(),
                 val.is_subnormal());
    }
    
    // Arithmetic with special values
    println!("\nArithmetic with special values:");
    println!("1.0 + ∞ = {}", 1.0 + positive_inf);
    println!("∞ + ∞ = {}", positive_inf + positive_inf);
    println!("∞ - ∞ = {}", positive_inf - positive_inf);
    println!("∞ / ∞ = {}", positive_inf / positive_inf);
    println!("1.0 / 0.0 = {}", 1.0 / 0.0);
    println!("0.0 / 0.0 = {}", 0.0 / 0.0);
    
    // NaN propagation
    println!("\nNaN propagation:");
    println!("NaN + 1.0 = {}", nan + 1.0);
    println!("NaN * 0.0 = {}", nan * 0.0);
    println!("NaN.sqrt() = {}", nan.sqrt());
    
    // Testing for specific values
    let test_val = 0.0 / 0.0;
    if test_val.is_nan() {
        println!("\nDetected NaN value");
    }
    
    // Handling signed zeros
    println!("\nSigned zeros:");
    println!("0.0 == -0.0: {}", 0.0 == -0.0);
    println!("0.0.is_sign_positive(): {}", 0.0_f64.is_sign_positive());
    println!("(-0.0).is_sign_positive(): {}", (-0.0_f64).is_sign_positive());
}
```

IEEE 754 defines special values that represent mathematical concepts  
like infinity and undefined results (NaN). These values propagate  
through calculations and require special handling. Always use `is_nan()`,  
`is_infinite()`, and `is_finite()` to test for special values.  

## Precision and epsilon comparisons

Handling floating-point precision issues with epsilon-based comparisons  
and relative error calculations.  

```rust
fn main() {
    // Machine epsilon for different types
    println!("Machine epsilon:");
    println!("f32::EPSILON = {}", f32::EPSILON);
    println!("f64::EPSILON = {}", f64::EPSILON);
    
    // Precision problems
    let a = 0.1 + 0.2;
    let b = 0.3;
    println!("\nPrecision issue:");
    println!("0.1 + 0.2 = {:.17}", a);
    println!("0.3 = {:.17}", b);
    println!("Equal? {}", a == b);
    println!("Difference: {:.2e}", (a - b).abs());
    
    // Absolute epsilon comparison
    fn abs_equal(x: f64, y: f64, epsilon: f64) -> bool {
        (x - y).abs() < epsilon
    }
    
    // Relative epsilon comparison
    fn rel_equal(x: f64, y: f64, epsilon: f64) -> bool {
        if x == y { return true; }
        let diff = (x - y).abs();
        let largest = x.abs().max(y.abs());
        diff <= epsilon * largest
    }
    
    // Combined comparison (handles values near zero)
    fn approx_equal(x: f64, y: f64, abs_eps: f64, rel_eps: f64) -> bool {
        let diff = (x - y).abs();
        if diff <= abs_eps {
            return true;
        }
        let largest = x.abs().max(y.abs());
        diff <= rel_eps * largest
    }
    
    let test_pairs = [
        (0.1 + 0.2, 0.3),
        (1000000.1, 1000000.2),
        (1e-10, 2e-10),
        (0.0, 1e-15),
    ];
    
    println!("\nComparison methods:");
    println!("{:<20} {:<15} {:<15} {:<15} {:<15}", 
             "Values", "Direct ==", "Abs epsilon", "Rel epsilon", "Combined");
    println!("{:-<85}", "");
    
    for &(x, y) in &test_pairs {
        println!("{:<20} {:<15} {:<15} {:<15} {:<15}", 
                 format!("({:.2e}, {:.2e})", x, y),
                 x == y,
                 abs_equal(x, y, 1e-9),
                 rel_equal(x, y, 1e-9),
                 approx_equal(x, y, 1e-12, 1e-9));
    }
    
    // ULP (Units in the Last Place) comparison
    fn ulp_equal(x: f64, y: f64, max_ulps: u64) -> bool {
        if x == y { return true; }
        if x.is_nan() || y.is_nan() { return false; }
        
        let x_bits = x.to_bits();
        let y_bits = y.to_bits();
        
        // Handle different signs
        if (x_bits >> 63) != (y_bits >> 63) {
            return x == 0.0 && y == 0.0;
        }
        
        x_bits.abs_diff(y_bits) <= max_ulps
    }
    
    println!("\nULP comparison (max 2 ULPs):");
    for &(x, y) in &test_pairs {
        println!("({:.2e}, {:.2e}): {}", x, y, ulp_equal(x, y, 2));
    }
}
```

Floating-point numbers have limited precision, making direct equality  
comparison unreliable. Use absolute epsilon for values near zero and  
relative epsilon for larger values. ULP comparison measures the  
difference in representable floating-point values.  

## Range operations and clamping

Constraining floating-point values within specified ranges and performing  
boundary checks.  

```rust
fn main() {
    // Basic clamping
    let value = 7.5;
    let min = 2.0;
    let max = 5.0;
    
    let clamped = value.clamp(min, max);
    println!("Clamp {} to [{}, {}]: {}", value, min, max, clamped);
    
    // Manual clamping function
    fn manual_clamp(value: f64, min: f64, max: f64) -> f64 {
        if value < min { min }
        else if value > max { max }
        else { value }
    }
    
    let test_values = [-1.0, 0.5, 3.0, 8.0, 15.0];
    println!("\nClamping values to [0.0, 10.0]:");
    for &val in &test_values {
        println!("{:5.1} -> {:5.1} (clamp) | {:5.1} (manual)", 
                 val, val.clamp(0.0, 10.0), manual_clamp(val, 0.0, 10.0));
    }
    
    // Range checking
    fn in_range(value: f64, min: f64, max: f64) -> bool {
        value >= min && value <= max
    }
    
    fn normalize_to_range(value: f64, old_min: f64, old_max: f64, 
                         new_min: f64, new_max: f64) -> f64 {
        let old_range = old_max - old_min;
        let new_range = new_max - new_min;
        (value - old_min) * new_range / old_range + new_min
    }
    
    println!("\nRange operations:");
    let temp_celsius = 25.0;
    println!("Temperature {}°C in range [0, 40]? {}", 
             temp_celsius, in_range(temp_celsius, 0.0, 40.0));
    
    // Normalize temperature from Celsius [0, 100] to Fahrenheit [32, 212]
    let temp_f = normalize_to_range(temp_celsius, 0.0, 100.0, 32.0, 212.0);
    println!("{}°C normalized to Fahrenheit scale: {}°F", temp_celsius, temp_f);
    
    // Wrapping values in a range (modulo-like operation)
    fn wrap_range(value: f64, min: f64, max: f64) -> f64 {
        let range = max - min;
        if range <= 0.0 { return min; }
        
        let wrapped = (value - min) % range;
        if wrapped < 0.0 { wrapped + range + min } else { wrapped + min }
    }
    
    println!("\nWrapping angles to [0°, 360°):");
    let angles = [-45.0, 90.0, 270.0, 450.0, 720.0];
    for &angle in &angles {
        println!("{}° -> {}°", angle, wrap_range(angle, 0.0, 360.0));
    }
    
    // Linear interpolation (lerp)
    fn lerp(start: f64, end: f64, t: f64) -> f64 {
        start + t * (end - start)
    }
    
    println!("\nLinear interpolation between 10.0 and 50.0:");
    for i in 0..=10 {
        let t = i as f64 / 10.0;
        println!("t={:.1}: {:.1}", t, lerp(10.0, 50.0, t));
    }
}
```

The `clamp()` method constrains values to a specified range. Range  
normalization maps values from one range to another linearly. Wrapping  
operations are useful for cyclic values like angles. Linear interpolation  
provides smooth transitions between two values.  

## Random float generation

Generating random floating-point numbers using the rand crate with  
various distributions and ranges.  

```rust
// Note: This example requires adding `rand = "0.8"` to Cargo.toml
use rand::{Rng, thread_rng};
use rand::distributions::{Uniform, Standard};

fn main() {
    let mut rng = thread_rng();
    
    // Basic random float generation [0.0, 1.0)
    println!("Random floats in [0.0, 1.0):");
    for _ in 0..5 {
        let random_f64: f64 = rng.gen();
        let random_f32: f32 = rng.gen();
        println!("f64: {:.6}, f32: {:.6}", random_f64, random_f32);
    }
    
    // Random floats in specific range
    println!("\nRandom floats in specific ranges:");
    for _ in 0..5 {
        let range_01 = rng.gen_range(0.0..1.0);
        let range_custom = rng.gen_range(-10.0..=10.0);
        println!("[0.0, 1.0): {:.3}, [-10.0, 10.0]: {:.3}", 
                 range_01, range_custom);
    }
    
    // Using distributions
    let uniform_dist = Uniform::new(5.0, 15.0);
    println!("\nUniform distribution [5.0, 15.0):");
    for _ in 0..5 {
        let value: f64 = rng.sample(uniform_dist);
        println!("{:.3}", value);
    }
    
    // Simulating different distributions
    fn normal_approx(mean: f64, std_dev: f64, rng: &mut impl Rng) -> f64 {
        // Box-Muller approximation
        let u1: f64 = rng.gen();
        let u2: f64 = rng.gen();
        let z = (-2.0 * u1.ln()).sqrt() * (2.0 * std::f64::consts::PI * u2).cos();
        mean + std_dev * z
    }
    
    println!("\nApproximate normal distribution (mean=0, std=1):");
    for _ in 0..5 {
        let normal = normal_approx(0.0, 1.0, &mut rng);
        println!("{:.3}", normal);
    }
    
    // Random boolean with probability
    fn random_bool_with_prob(probability: f64, rng: &mut impl Rng) -> bool {
        rng.gen::<f64>() < probability
    }
    
    println!("\nRandom boolean with 30% true probability:");
    let mut true_count = 0;
    let trials = 10;
    for i in 0..trials {
        let result = random_bool_with_prob(0.3, &mut rng);
        if result { true_count += 1; }
        println!("Trial {}: {}", i + 1, result);
    }
    println!("True ratio: {}/{} = {:.1}%", 
             true_count, trials, (true_count as f64 / trials as f64) * 100.0);
    
    // Random point in circle
    fn random_point_in_circle(radius: f64, rng: &mut impl Rng) -> (f64, f64) {
        loop {
            let x = rng.gen_range(-radius..=radius);
            let y = rng.gen_range(-radius..=radius);
            if x * x + y * y <= radius * radius {
                return (x, y);
            }
        }
    }
    
    println!("\nRandom points in unit circle:");
    for _ in 0..3 {
        let (x, y) = random_point_in_circle(1.0, &mut rng);
        let distance = (x * x + y * y).sqrt();
        println!("({:.3}, {:.3}), distance: {:.3}", x, y, distance);
    }
}
```

Random floating-point generation uses the `rand` crate. The `gen()`  
method produces values in [0.0, 1.0) while `gen_range()` allows custom  
ranges. Uniform distributions ensure equal probability across the range.  
The Box-Muller transform approximates normal distributions.  

## Performance considerations

Comparing performance characteristics between f32 and f64 types and  
optimization techniques for floating-point calculations.  

```rust
use std::time::Instant;

fn main() {
    // Memory usage comparison
    println!("Memory usage:");
    println!("f32: {} bytes", std::mem::size_of::<f32>());
    println!("f64: {} bytes", std::mem::size_of::<f64>());
    println!("Array of 1000 f32: {} bytes", std::mem::size_of::<[f32; 1000]>());
    println!("Array of 1000 f64: {} bytes", std::mem::size_of::<[f64; 1000]>());
    
    // Performance comparison
    const SIZE: usize = 1_000_000;
    
    // f32 operations
    let mut data_f32 = vec![1.0_f32; SIZE];
    let start = Instant::now();
    for i in 0..SIZE {
        data_f32[i] = (data_f32[i] * 2.0).sqrt().sin();
    }
    let duration_f32 = start.elapsed();
    
    // f64 operations
    let mut data_f64 = vec![1.0_f64; SIZE];
    let start = Instant::now();
    for i in 0..SIZE {
        data_f64[i] = (data_f64[i] * 2.0).sqrt().sin();
    }
    let duration_f64 = start.elapsed();
    
    println!("\nPerformance comparison ({} operations):", SIZE);
    println!("f32: {:?}", duration_f32);
    println!("f64: {:?}", duration_f64);
    println!("Ratio: {:.2}x", duration_f64.as_nanos() as f64 / duration_f32.as_nanos() as f64);
    
    // Precision vs performance trade-offs
    fn fast_inverse_sqrt_f32(x: f32) -> f32 {
        // Fast inverse square root approximation
        let i = x.to_bits();
        let i = 0x5f3759df - (i >> 1);
        let y = f32::from_bits(i);
        y * (1.5 - 0.5 * x * y * y)
    }
    
    let test_value = 4.0_f32;
    let accurate = 1.0 / test_value.sqrt();
    let fast = fast_inverse_sqrt_f32(test_value);
    
    println!("\nPrecision vs speed for 1/sqrt(4):");
    println!("Accurate: {:.10}", accurate);
    println!("Fast approx: {:.10}", fast);
    println!("Error: {:.2e}", (accurate - fast).abs());
    
    // Vectorization-friendly operations
    fn sum_of_squares_naive(data: &[f64]) -> f64 {
        let mut sum = 0.0;
        for &x in data {
            sum += x * x;
        }
        sum
    }
    
    fn sum_of_squares_iter(data: &[f64]) -> f64 {
        data.iter().map(|&x| x * x).sum()
    }
    
    let test_data: Vec<f64> = (0..10000).map(|i| i as f64 * 0.001).collect();
    
    let start = Instant::now();
    let result1 = sum_of_squares_naive(&test_data);
    let time1 = start.elapsed();
    
    let start = Instant::now();
    let result2 = sum_of_squares_iter(&test_data);
    let time2 = start.elapsed();
    
    println!("\nVectorization comparison:");
    println!("Naive loop: {:.6} in {:?}", result1, time1);
    println!("Iterator: {:.6} in {:?}", result2, time2);
    
    // Cache-friendly access patterns
    println!("\nCache considerations:");
    println!("L1 cache line: ~64 bytes");
    println!("f32 per cache line: {}", 64 / std::mem::size_of::<f32>());
    println!("f64 per cache line: {}", 64 / std::mem::size_of::<f64>());
    
    // Demonstrating denormal number performance impact
    let normal = 1.0e-30_f64;
    let denormal = 1.0e-320_f64;
    
    println!("\nDenormal number detection:");
    println!("Normal ({:.2e}): is_normal() = {}", normal, normal.is_normal());
    println!("Denormal ({:.2e}): is_normal() = {}", denormal, denormal.is_normal());
    println!("Note: Denormal numbers can be much slower to process");
}
```

F32 uses half the memory of f64 and may be faster on some architectures.  
However, f64 provides significantly higher precision. Consider your  
specific needs for accuracy versus performance. Denormal numbers near  
zero can cause performance penalties on some processors.  

## Practical calculations

Real-world examples demonstrating floating-point arithmetic in practical  
applications like unit conversions and mathematical calculations.  

```rust
fn main() {
    // Temperature conversions
    fn celsius_to_fahrenheit(c: f64) -> f64 {
        c * 9.0 / 5.0 + 32.0
    }
    
    fn fahrenheit_to_celsius(f: f64) -> f64 {
        (f - 32.0) * 5.0 / 9.0
    }
    
    fn celsius_to_kelvin(c: f64) -> f64 {
        c + 273.15
    }
    
    println!("Temperature conversions:");
    let temp_c = 25.0;
    println!("{}°C = {:.1}°F = {:.2}K", 
             temp_c, 
             celsius_to_fahrenheit(temp_c), 
             celsius_to_kelvin(temp_c));
    
    // Distance and speed conversions
    fn miles_to_kilometers(miles: f64) -> f64 {
        miles * 1.60934
    }
    
    fn mph_to_kmh(mph: f64) -> f64 {
        mph * 1.60934
    }
    
    fn feet_to_meters(feet: f64) -> f64 {
        feet * 0.3048
    }
    
    println!("\nDistance conversions:");
    let distance_miles = 100.0;
    let speed_mph = 60.0;
    let height_feet = 6.0;
    
    println!("{} miles = {:.2} km", distance_miles, miles_to_kilometers(distance_miles));
    println!("{} mph = {:.1} km/h", speed_mph, mph_to_kmh(speed_mph));
    println!("{} feet = {:.2} meters", height_feet, feet_to_meters(height_feet));
    
    // Financial calculations
    fn compound_interest(principal: f64, rate: f64, time: f64, compounds_per_year: f64) -> f64 {
        principal * (1.0 + rate / compounds_per_year).powf(compounds_per_year * time)
    }
    
    fn monthly_payment(principal: f64, annual_rate: f64, years: f64) -> f64 {
        let monthly_rate = annual_rate / 12.0;
        let num_payments = years * 12.0;
        principal * (monthly_rate * (1.0 + monthly_rate).powf(num_payments)) / 
                   ((1.0 + monthly_rate).powf(num_payments) - 1.0)
    }
    
    println!("\nFinancial calculations:");
    let principal = 10000.0;
    let annual_rate = 0.05; // 5%
    let years = 10.0;
    
    let final_amount = compound_interest(principal, annual_rate, years, 12.0);
    println!("${:.2} at {:.1}% for {:.0} years = ${:.2}", 
             principal, annual_rate * 100.0, years, final_amount);
    
    let loan_amount = 200000.0;
    let loan_rate = 0.04; // 4%
    let loan_years = 30.0;
    let payment = monthly_payment(loan_amount, loan_rate, loan_years);
    println!("${:.0} loan at {:.1}% for {:.0} years = ${:.2}/month", 
             loan_amount, loan_rate * 100.0, loan_years, payment);
    
    // Physics calculations
    fn kinetic_energy(mass: f64, velocity: f64) -> f64 {
        0.5 * mass * velocity * velocity
    }
    
    fn gravitational_force(m1: f64, m2: f64, distance: f64) -> f64 {
        const G: f64 = 6.67430e-11; // Gravitational constant
        G * m1 * m2 / (distance * distance)
    }
    
    fn projectile_range(velocity: f64, angle_degrees: f64) -> f64 {
        const G: f64 = 9.81; // Gravity
        let angle_rad = angle_degrees.to_radians();
        velocity * velocity * (2.0 * angle_rad).sin() / G
    }
    
    println!("\nPhysics calculations:");
    let mass = 70.0; // kg
    let velocity = 10.0; // m/s
    println!("Kinetic energy: {:.1} J", kinetic_energy(mass, velocity));
    
    let earth_mass = 5.972e24; // kg
    let moon_mass = 7.342e22; // kg
    let distance = 3.844e8; // meters
    let force = gravitational_force(earth_mass, moon_mass, distance);
    println!("Earth-Moon gravitational force: {:.2e} N", force);
    
    let launch_velocity = 50.0; // m/s
    let launch_angle = 45.0; // degrees
    let range = projectile_range(launch_velocity, launch_angle);
    println!("Projectile range: {:.1} m", range);
    
    // Statistical calculations
    fn mean(data: &[f64]) -> f64 {
        data.iter().sum::<f64>() / data.len() as f64
    }
    
    fn standard_deviation(data: &[f64]) -> f64 {
        let mean_val = mean(data);
        let variance = data.iter()
            .map(|x| (x - mean_val).powi(2))
            .sum::<f64>() / data.len() as f64;
        variance.sqrt()
    }
    
    let test_scores = [85.5, 92.0, 78.5, 88.0, 95.5, 82.0, 90.0, 86.5];
    println!("\nStatistical analysis of test scores:");
    println!("Mean: {:.2}", mean(&test_scores));
    println!("Standard deviation: {:.2}", standard_deviation(&test_scores));
    println!("Scores: {:?}", test_scores);
}
```

Practical floating-point calculations require careful consideration of  
precision and rounding errors. Always validate results and consider  
using appropriate data types for financial calculations where precision  
is critical. Physical calculations often involve very large or very  
small numbers requiring scientific notation.  

## Advanced float operations

Advanced floating-point operations including bit manipulation, next  
representable values, and IEEE 754 compliance features.  

```rust
fn main() {
    let value = 3.14159_f64;
    
    // Bit representation
    println!("Bit manipulation:");
    println!("Value: {}", value);
    println!("Bits: 0x{:016X}", value.to_bits());
    println!("From bits: {}", f64::from_bits(value.to_bits()));
    
    // Sign manipulation
    println!("\nSign operations:");
    println!("abs({}): {}", -5.5, (-5.5_f64).abs());
    println!("signum({}): {}", -5.5, (-5.5_f64).signum());
    println!("copysign({}, {}): {}", 3.0, -2.0, (3.0_f64).copysign(-2.0));
    
    // Next representable values
    let x = 1.0_f64;
    println!("\nNext representable values for {}:", x);
    
    // Manual implementation of next_up/next_down
    fn next_up(x: f64) -> f64 {
        if x.is_nan() || x == f64::INFINITY {
            return x;
        }
        if x == f64::NEG_INFINITY {
            return f64::MIN;
        }
        
        let bits = x.to_bits();
        if x >= 0.0 {
            f64::from_bits(bits + 1)
        } else {
            f64::from_bits(bits - 1)
        }
    }
    
    fn next_down(x: f64) -> f64 {
        if x.is_nan() || x == f64::NEG_INFINITY {
            return x;
        }
        if x == f64::INFINITY {
            return f64::MAX;
        }
        
        let bits = x.to_bits();
        if x > 0.0 {
            f64::from_bits(bits - 1)
        } else {
            f64::from_bits(bits + 1)
        }
    }
    
    let next_higher = next_up(x);
    let next_lower = next_down(x);
    
    println!("Next up: {:.17}", next_higher);
    println!("Next down: {:.17}", next_lower);
    println!("ULP size: {:.2e}", next_higher - x);
    
    // Precision analysis
    println!("\nPrecision analysis:");
    let values = [1.0, 1000.0, 1e6, 1e12];
    for &val in &values {
        let next = next_up(val);
        let ulp = next - val;
        println!("Value: {:.2e}, ULP: {:.2e}, Relative: {:.2e}", 
                 val, ulp, ulp / val);
    }
    
    // Decomposition into parts
    println!("\nFloat decomposition:");
    let test_val = 123.456_f64;
    println!("Value: {}", test_val);
    println!("Integer part: {}", test_val.trunc());
    println!("Fractional part: {}", test_val.fract());
    
    // Extracting exponent and mantissa (manual)
    fn decompose_float(x: f64) -> (bool, i32, u64) {
        let bits = x.to_bits();
        let sign = (bits >> 63) != 0;
        let exponent = ((bits >> 52) & 0x7FF) as i32 - 1023;
        let mantissa = bits & 0xFFFFFFFFFFFFF;
        (sign, exponent, mantissa)
    }
    
    let (sign, exp, mantissa) = decompose_float(test_val);
    println!("Sign: {}, Exponent: {}, Mantissa: 0x{:013X}", sign, exp, mantissa);
    
    // Error accumulation demonstration
    println!("\nError accumulation:");
    let mut sum1 = 0.0_f64;
    let mut sum2 = 0.0_f32;
    
    for i in 1..=1000 {
        sum1 += 0.1;
        sum2 += 0.1_f32;
    }
    
    println!("1000 * 0.1 (f64): {:.15}", sum1);
    println!("1000 * 0.1 (f32): {:.15}", sum2);
    println!("Expected: 100.0");
    println!("f64 error: {:.2e}", (sum1 - 100.0).abs());
    println!("f32 error: {:.2e}", (sum2 as f64 - 100.0).abs());
    
    // Kahan summation for improved accuracy
    fn kahan_sum(values: &[f64]) -> f64 {
        let mut sum = 0.0;
        let mut c = 0.0; // Compensation for lost low-order bits
        
        for &value in values {
            let y = value - c;     // Compensated value
            let t = sum + y;       // New sum
            c = (t - sum) - y;     // New compensation
            sum = t;
        }
        sum
    }
    
    let values: Vec<f64> = (0..1000).map(|_| 0.1).collect();
    let simple_sum: f64 = values.iter().sum();
    let kahan_sum_result = kahan_sum(&values);
    
    println!("\nKahan summation:");
    println!("Simple sum: {:.15}", simple_sum);
    println!("Kahan sum: {:.15}", kahan_sum_result);
    println!("Improvement: {:.2e}", (simple_sum - 100.0).abs() / (kahan_sum_result - 100.0).abs());
}
```

Advanced floating-point operations provide fine-grained control over  
IEEE 754 behavior. Bit manipulation allows direct access to the float  
representation. ULP (Units in the Last Place) distance measures the  
precision between adjacent floating-point values. Kahan summation  
reduces accumulation errors in repeated addition operations.
