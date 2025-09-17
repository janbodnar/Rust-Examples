# Arrays

## Basic array creation

Creating arrays with explicit values and type inference.  

```rust
fn main() {
    // Array with explicit values
    let numbers = [1, 2, 3, 4, 5];
    let fruits = ["apple", "banana", "orange"];
    
    // Type is inferred from the values
    println!("Numbers: {:?}", numbers);
    println!("Fruits: {:?}", fruits);
    
    // Array length
    println!("Numbers has {} elements", numbers.len());
    println!("Fruits has {} elements", fruits.len());
}
```

Arrays are fixed-size collections where all elements must be the same  
type. Rust infers the type from the initial values, and the size is  
determined at compile time.  

## Array type annotations

Explicitly specifying array types with element type and length.  

```rust
fn main() {
    // Explicit type annotations: [type; length]
    let integers: [i32; 4] = [10, 20, 30, 40];
    let floats: [f64; 3] = [1.1, 2.2, 3.3];
    let booleans: [bool; 2] = [true, false];
    
    // Mixed numeric types require explicit conversion
    let mixed: [f32; 3] = [1.0, 2.5, 3.7];
    
    println!("Integers: {:?}", integers);
    println!("Floats: {:?}", floats);
    println!("Booleans: {:?}", booleans);
    println!("Mixed: {:?}", mixed);
}
```

The type signature [T; N] specifies the element type T and the fixed  
length N. All elements must be the same type, and the length cannot  
change after creation.  

## Array initialization with repeated values

Creating arrays with the same value repeated multiple times.  

```rust
fn main() {
    // Initialize array with repeated values: [value; count]
    let zeros: [i32; 5] = [0; 5];
    let ones: [u8; 10] = [1; 10];
    let default_strings: [&str; 3] = ["default"; 3];
    
    println!("Zeros: {:?}", zeros);
    println!("Ones: {:?}", ones);
    println!("Default strings: {:?}", default_strings);
    
    // Useful for initializing large arrays
    let buffer: [u8; 1024] = [0; 1024];
    println!("Buffer size: {} bytes", buffer.len());
}
```

The [value; count] syntax creates an array by copying the same value  
count times. This is efficient for primitive types and types that  
implement the Copy trait.  

## Array indexing and access

Accessing array elements using indices and handling bounds.  

```rust
fn main() {
    let colors = ["red", "green", "blue", "yellow", "purple"];
    
    // Zero-based indexing
    println!("First color: {}", colors[0]);
    println!("Third color: {}", colors[2]);
    println!("Last color: {}", colors[colors.len() - 1]);
    
    // Access from the end
    println!("Second to last: {}", colors[colors.len() - 2]);
    
    // This would panic if uncommented (index out of bounds)
    // println!("Invalid: {}", colors[10]);
    
    // Array length
    println!("Array length: {}", colors.len());
}
```

Array indexing is zero-based and bounds-checked at runtime. Accessing  
an out-of-bounds index causes a panic. Use array.len() to get the  
number of elements.  

## Safe array access with get()

Using the get() method for safe array access that returns Option.  

```rust
fn main() {
    let languages = ["Rust", "Python", "JavaScript", "Go"];
    
    // Safe access with get() - returns Option<&T>
    match languages.get(1) {
        Some(lang) => println!("Language at index 1: {}", lang),
        None => println!("No language at index 1"),
    }
    
    // Safe access for out-of-bounds index
    match languages.get(10) {
        Some(lang) => println!("Language at index 10: {}", lang),
        None => println!("No language at index 10"),
    }
    
    // Using unwrap_or for default values
    let lang = languages.get(5).unwrap_or(&"Unknown");
    println!("Language or default: {}", lang);
    
    // Using map to transform if present
    let upper = languages.get(0).map(|s| s.to_uppercase());
    println!("First language uppercase: {:?}", upper);
}
```

The get() method returns Option<&T> instead of panicking on  
out-of-bounds access. This allows safe handling of potentially invalid  
indices using Option methods.  

## Array iteration

Different ways to iterate over array elements.  

```rust
fn main() {
    let scores = [85, 92, 78, 96, 88];
    
    // Iterate by value (copies elements)
    println!("Scores:");
    for score in scores {
        println!("Score: {}", score);
    }
    
    // Iterate by reference (borrowing)
    println!("\nScores with references:");
    for score in &scores {
        println!("Score ref: {}", score);
    }
    
    // Iterate with indices using enumerate
    println!("\nScores with indices:");
    for (index, score) in scores.iter().enumerate() {
        println!("Index {}: {}", index, score);
    }
    
    // Using iter() method
    println!("\nUsing iter():");
    scores.iter().for_each(|score| println!("Score: {}", score));
}
```

Arrays can be iterated by value (moving/copying elements), by reference  
(borrowing), or with indices using enumerate(). The iter() method  
provides additional iterator functionality.  

## Array slicing

Creating slices from arrays using range syntax.  

```rust
fn main() {
    let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    
    // Slice syntax: &array[start..end] (end is exclusive)
    let slice1 = &numbers[2..5];        // Elements 3, 4, 5
    let slice2 = &numbers[..3];         // Elements 1, 2, 3
    let slice3 = &numbers[7..];         // Elements 8, 9, 10
    let slice4 = &numbers[..];          // All elements
    
    println!("Original: {:?}", numbers);
    println!("Slice [2..5]: {:?}", slice1);
    println!("Slice [..3]: {:?}", slice2);
    println!("Slice [7..]: {:?}", slice3);
    println!("Slice [..]: {:?}", slice4);
    
    // Inclusive range with ..=
    let slice5 = &numbers[1..=4];       // Elements 2, 3, 4, 5
    println!("Slice [1..=4]: {:?}", slice5);
}
```

Slicing creates a view into a portion of the array without copying  
data. Use [start..end] for exclusive end, [start..=end] for inclusive  
end, [..end] from beginning, and [start..] to end.  

## Multi-dimensional arrays

Working with arrays of arrays (matrices and nested structures).  

```rust
fn main() {
    // 2D array (matrix): [[type; inner_length]; outer_length]
    let matrix: [[i32; 3]; 2] = [
        [1, 2, 3],
        [4, 5, 6]
    ];
    
    // 3D array
    let cube: [[[i32; 2]; 2]; 2] = [
        [[1, 2], [3, 4]],
        [[5, 6], [7, 8]]
    ];
    
    // Accessing elements
    println!("Matrix[0][1]: {}", matrix[0][1]);      // 2
    println!("Cube[1][0][1]: {}", cube[1][0][1]);    // 6
    
    // Iterating over 2D array
    println!("Matrix:");
    for (i, row) in matrix.iter().enumerate() {
        for (j, value) in row.iter().enumerate() {
            println!("matrix[{}][{}] = {}", i, j, value);
        }
    }
    
    // Pretty printing
    println!("\nMatrix display:");
    for row in matrix {
        println!("{:?}", row);
    }
}
```

Multi-dimensional arrays are arrays containing other arrays. The type  
signature specifies dimensions from innermost to outermost. Access  
elements using multiple index operations.  

## Array methods and operations

Using built-in array methods for common operations.  

```rust
fn main() {
    let numbers = [3, 1, 4, 1, 5, 9, 2, 6];
    let words = ["hello", "world", "rust", "programming"];
    
    // Length and size information
    println!("Length: {}", numbers.len());
    println!("Is empty: {}", numbers.is_empty());
    println!("Size in bytes: {}", std::mem::size_of_val(&numbers));
    
    // First and last elements
    println!("First: {:?}", numbers.first());
    println!("Last: {:?}", numbers.last());
    
    // Contains check
    println!("Contains 5: {}", numbers.contains(&5));
    println!("Contains 10: {}", numbers.contains(&10));
    
    // Binary search (requires sorted array)
    let sorted = [1, 2, 3, 4, 5, 6, 7, 8, 9];
    println!("Binary search for 5: {:?}", sorted.binary_search(&5));
    println!("Binary search for 10: {:?}", sorted.binary_search(&10));
    
    // Starts with and ends with for slices
    println!("Starts with [3, 1]: {}", numbers.starts_with(&[3, 1]));
    println!("Ends with [2, 6]: {}", numbers.ends_with(&[2, 6]));
}
```

Arrays provide methods for inspection (len, first, last), searching  
(contains, binary_search), and pattern matching (starts_with,  
ends_with). Some methods return Option for safe access.  

## Array comparison and equality

Comparing arrays for equality and ordering.  

```rust
fn main() {
    let array1 = [1, 2, 3];
    let array2 = [1, 2, 3];
    let array3 = [1, 2, 4];
    let array4 = [1, 2];
    
    // Equality comparison
    println!("array1 == array2: {}", array1 == array2);  // true
    println!("array1 == array3: {}", array1 == array3);  // false
    println!("array1 == array4: {}", array1 == array4);  // false (different lengths)
    
    // Inequality
    println!("array1 != array3: {}", array1 != array3);  // true
    
    // Lexicographic ordering
    println!("array1 < array3: {}", array1 < array3);    // true
    println!("array3 > array1: {}", array3 > array1);    // true
    
    // String arrays
    let words1 = ["apple", "banana"];
    let words2 = ["apple", "cherry"];
    println!("words1 < words2: {}", words1 < words2);    // true
    
    // Arrays must have same length and type for comparison
    let floats1 = [1.0, 2.0, 3.0];
    let floats2 = [1.0, 2.0, 3.1];
    println!("floats1 < floats2: {}", floats1 < floats2); // true
}
```

Arrays of the same type and length can be compared for equality and  
ordering. Comparison is lexicographic (element by element). Arrays  
must be the same type and length to be comparable.  

## Array conversion to Vec

Converting arrays to vectors and other collection types.  

```rust
fn main() {
    let array = [1, 2, 3, 4, 5];
    
    // Convert to Vec using to_vec() (clones elements)
    let vec1: Vec<i32> = array.to_vec();
    println!("Array to Vec: {:?}", vec1);
    
    // Convert using Vec::from()
    let vec2: Vec<i32> = Vec::from(array);
    println!("Vec::from array: {:?}", vec2);
    
    // Convert using into()
    let vec3: Vec<i32> = array.into();
    println!("Array into Vec: {:?}", vec3);
    
    // Convert via iterator and collect
    let vec4: Vec<i32> = array.iter().cloned().collect();
    println!("Iter collect: {:?}", vec4);
    
    // Convert with transformation
    let vec5: Vec<String> = array.iter()
        .map(|&x| format!("Item {}", x))
        .collect();
    println!("Transformed: {:?}", vec5);
    
    // Original array still exists (to_vec, from, iter clone)
    println!("Original array: {:?}", array);
}
```

Arrays can be converted to Vec using to_vec(), Vec::from(), into(), or  
iterator methods. Most conversions clone the data, leaving the original  
array intact. Vec provides dynamic sizing unlike fixed arrays.  

## Mutable arrays

Working with mutable arrays and modifying elements.  

```rust
fn main() {
    // Mutable array
    let mut numbers = [1, 2, 3, 4, 5];
    println!("Original: {:?}", numbers);
    
    // Modify individual elements
    numbers[0] = 10;
    numbers[4] = 50;
    println!("After modification: {:?}", numbers);
    
    // Modify multiple elements
    numbers[1..4].copy_from_slice(&[20, 30, 40]);
    println!("After slice copy: {:?}", numbers);
    
    // Fill array with a value
    numbers.fill(99);
    println!("After fill: {:?}", numbers);
    
    // Fill with closure result
    let mut counters = [0; 5];
    let mut counter = 10;
    counters.fill_with(|| { counter += 5; counter });
    println!("Counter filled: {:?}", counters);
    
    // Swap elements
    let mut letters = ['a', 'b', 'c', 'd'];
    letters.swap(0, 3);
    println!("After swap: {:?}", letters);
    
    // Reverse array in place
    letters.reverse();
    println!("After reverse: {:?}", letters);
}
```

Mutable arrays allow modification of elements but not changing the  
array size. Use fill() for setting all elements, swap() for exchanging  
elements, and reverse() for in-place reversal.  

## Array pattern matching

Using pattern matching to destructure and match arrays.  

```rust
fn main() {
    let coords = [10, 20];
    let rgb = [255, 128, 0];
    let values = [1, 2, 3, 4, 5];
    
    // Destructure arrays
    let [x, y] = coords;
    println!("Coordinates: x={}, y={}", x, y);
    
    let [r, g, b] = rgb;
    println!("RGB: red={}, green={}, blue={}", r, g, b);
    
    // Pattern matching with match
    match coords {
        [0, 0] => println!("Origin point"),
        [x, 0] => println!("On X-axis at {}", x),
        [0, y] => println!("On Y-axis at {}", y),
        [x, y] => println!("Point at ({}, {})", x, y),
    }
    
    // Partial matching with rest pattern
    match values {
        [first, .., last] => {
            println!("First: {}, Last: {}", first, last);
        }
    }
    
    // Match specific patterns
    match values {
        [1, 2, rest @ ..] => {
            println!("Starts with [1, 2], rest: {:?}", rest);
        }
        _ => println!("Different pattern"),
    }
    
    // Guard conditions
    match rgb {
        [r, g, b] if r > 200 => println!("High red component"),
        [r, g, b] if g > 200 => println!("High green component"),
        _ => println!("Normal color"),
    }
}
```

Array pattern matching allows destructuring arrays into variables and  
matching specific array patterns. Use [..] for rest patterns and  
guards for conditional matching.  

## Array sorting and searching

Sorting arrays and searching for elements.  

```rust
fn main() {
    let mut numbers = [64, 34, 25, 12, 22, 11, 90];
    let mut names = ["Charlie", "Alice", "Bob", "David"];
    
    println!("Original numbers: {:?}", numbers);
    println!("Original names: {:?}", names);
    
    // Sort in ascending order
    numbers.sort();
    names.sort();
    println!("Sorted numbers: {:?}", numbers);
    println!("Sorted names: {:?}", names);
    
    // Sort in descending order
    let mut desc_numbers = [64, 34, 25, 12, 22, 11, 90];
    desc_numbers.sort_by(|a, b| b.cmp(a));
    println!("Descending numbers: {:?}", desc_numbers);
    
    // Sort with custom comparator
    let mut words = ["elephant", "cat", "dog", "butterfly"];
    words.sort_by_key(|word| word.len());
    println!("Sorted by length: {:?}", words);
    
    // Binary search (requires sorted array)
    let position = numbers.binary_search(&25);
    println!("Position of 25: {:?}", position);
    
    // Linear search
    let found = numbers.iter().position(|&x| x == 34);
    println!("Position of 34: {:?}", found);
    
    // Check if array is sorted
    println!("Is numbers sorted: {}", numbers.windows(2).all(|w| w[0] <= w[1]));
}
```

Arrays can be sorted using sort(), sort_by(), and sort_by_key().  
Binary search works on sorted arrays, while linear search works on any  
array. Use position() to find element indices.  

## Array aggregation operations

Computing sums, products, and other aggregate values from arrays.  

```rust
fn main() {
    let numbers = [1, 2, 3, 4, 5];
    let prices = [19.99, 24.50, 15.75, 32.00];
    let temperatures = [-5.0, 0.0, 3.2, 8.1, 12.5];
    
    // Sum using iter().sum()
    let sum: i32 = numbers.iter().sum();
    println!("Sum of numbers: {}", sum);
    
    // Product using iter().product()
    let product: i32 = numbers.iter().product();
    println!("Product of numbers: {}", product);
    
    // Sum with fold
    let sum_fold = numbers.iter().fold(0, |acc, &x| acc + x);
    println!("Sum with fold: {}", sum_fold);
    
    // Find minimum and maximum
    let min = numbers.iter().min();
    let max = numbers.iter().max();
    println!("Min: {:?}, Max: {:?}", min, max);
    
    // Average calculation
    let total: f64 = prices.iter().sum();
    let average = total / prices.len() as f64;
    println!("Average price: ${:.2}", average);
    
    // Count elements meeting condition
    let positive_temps = temperatures.iter()
        .filter(|&&temp| temp > 0.0)
        .count();
    println!("Positive temperatures: {}", positive_temps);
    
    // All and any predicates
    let all_positive = numbers.iter().all(|&x| x > 0);
    let any_negative = temperatures.iter().any(|&temp| temp < 0.0);
    println!("All positive: {}, Any negative: {}", all_positive, any_negative);
}
```

Use iterator methods for aggregation: sum() and product() for  
arithmetic, min() and max() for extremes, and fold() for custom  
reductions. Filter and count for conditional aggregation.  

## Array copying and cloning

Different ways to copy and clone arrays.  

```rust
fn main() {
    let original = [1, 2, 3, 4, 5];
    
    // Arrays implement Copy trait for Copy types
    let copy1 = original;  // This is a copy, not a move
    let copy2 = original;  // original is still valid
    
    println!("Original: {:?}", original);
    println!("Copy1: {:?}", copy1);
    println!("Copy2: {:?}", copy2);
    
    // Explicit clone (same as copy for arrays of Copy types)
    let cloned = original.clone();
    println!("Cloned: {:?}", cloned);
    
    // Copy array to new array
    let mut target = [0; 5];
    target.copy_from_slice(&original);
    println!("Target after copy_from_slice: {:?}", target);
    
    // Partial copying
    let mut partial = [0; 3];
    partial.copy_from_slice(&original[1..4]);
    println!("Partial copy: {:?}", partial);
    
    // Arrays of non-Copy types require clone
    let strings = ["hello".to_string(), "world".to_string()];
    let cloned_strings = strings.clone();
    println!("Original strings: {:?}", strings);
    println!("Cloned strings: {:?}", cloned_strings);
    
    // Array assignment with Copy types creates a new array
    let mut mutable = original;
    mutable[0] = 99;
    println!("Original unchanged: {:?}", original);
    println!("Mutable changed: {:?}", mutable);
}
```

Arrays of Copy types (like integers) are automatically copied on  
assignment. For non-Copy types, use clone(). Use copy_from_slice() to  
copy data between arrays of the same size.  

## Array performance and memory layout

Understanding array performance characteristics and memory usage.  

```rust
fn main() {
    use std::mem;
    
    // Arrays are stack-allocated with contiguous memory
    let small_array = [1u8; 4];
    let large_array = [0i32; 1000];
    
    // Memory size calculations
    println!("Size of u8: {} bytes", mem::size_of::<u8>());
    println!("Size of [u8; 4]: {} bytes", mem::size_of_val(&small_array));
    println!("Size of i32: {} bytes", mem::size_of::<i32>());
    println!("Size of [i32; 1000]: {} bytes", mem::size_of_val(&large_array));
    
    // Arrays have zero runtime overhead for bounds checking in release mode
    let data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    
    // This is optimized to direct memory access
    let sum = data[0] + data[1] + data[2];
    println!("Sum of first three: {}", sum);
    
    // Memory alignment
    println!("Alignment of [i32; 10]: {}", mem::align_of::<[i32; 10]>());
    println!("Alignment of [u8; 10]: {}", mem::align_of::<[u8; 10]>());
    
    // Cache-friendly iteration
    let mut total = 0;
    for &value in data.iter() {
        total += value;
    }
    println!("Total: {}", total);
    
    // Arrays vs Vectors performance comparison
    let array_time = std::time::Instant::now();
    let mut array_sum = 0;
    for _ in 0..1000 {
        for &item in data.iter() {
            array_sum += item;
        }
    }
    let array_duration = array_time.elapsed();
    
    let vec_data: Vec<i32> = data.to_vec();
    let vec_time = std::time::Instant::now();
    let mut vec_sum = 0;
    for _ in 0..1000 {
        for &item in vec_data.iter() {
            vec_sum += item;
        }
    }
    let vec_duration = vec_time.elapsed();
    
    println!("Array sum: {}, time: {:?}", array_sum, array_duration);
    println!("Vec sum: {}, time: {:?}", vec_sum, vec_duration);
}
```

Arrays are stack-allocated with contiguous memory layout, providing  
excellent cache performance. They have zero-cost abstractions and  
compile-time known sizes. Bounds checking is optimized away in release  
builds for direct indexing.  

## Advanced array operations

Complex array manipulations and functional programming patterns.  

```rust
fn main() {
    let matrix = [
        [1, 2, 3],
        [4, 5, 6],
        [7, 8, 9]
    ];
    
    // Matrix transpose (for square matrices)
    let mut transposed = [[0; 3]; 3];
    for i in 0..3 {
        for j in 0..3 {
            transposed[j][i] = matrix[i][j];
        }
    }
    
    println!("Original matrix:");
    for row in matrix {
        println!("{:?}", row);
    }
    
    println!("Transposed matrix:");
    for row in transposed {
        println!("{:?}", row);
    }
    
    // Functional operations on arrays
    let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    
    // Chain operations
    let result: Vec<i32> = numbers
        .iter()
        .filter(|&&x| x % 2 == 0)          // Even numbers
        .map(|&x| x * x)                   // Square them
        .filter(|&x| x > 10)               // Greater than 10
        .collect();
    
    println!("Filtered and squared evens > 10: {:?}", result);
    
    // Partition array into two groups
    let (evens, odds): (Vec<i32>, Vec<i32>) = numbers
        .iter()
        .partition(|&&x| x % 2 == 0);
    
    println!("Evens: {:?}", evens);
    println!("Odds: {:?}", odds);
    
    // Group consecutive elements
    let groups: Vec<&[i32]> = numbers
        .windows(3)
        .collect();
    
    println!("Sliding windows of 3:");
    for window in groups {
        println!("{:?}", window);
    }
    
    // Array rotation (left shift)
    let mut arr = [1, 2, 3, 4, 5];
    let n = 2; // positions to rotate
    arr[..n].reverse();
    arr[n..].reverse();
    arr.reverse();
    println!("Rotated left by {}: {:?}", n, arr);
}
```

Advanced operations include matrix manipulation, functional programming  
patterns with iterators, partitioning data, sliding windows, and  
in-place algorithms like rotation. Combine multiple operations for  
complex data transformations.
