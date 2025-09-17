# Vectors

A vector is a contiguous growable collection that can store elements of the  
same type. Vectors are heap-allocated and can dynamically resize during  
runtime.  

## Creating vectors with vec! macro

The `vec!` macro provides a convenient way to create vectors with initial  
values.  

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];
    let words = vec!["hello", "world", "rust"];
    let empty: Vec<i32> = vec![];
    
    println!("Numbers: {:?}", numbers);
    println!("Words: {:?}", words);
    println!("Empty vector length: {}", empty.len());
}
```

The `vec!` macro creates a vector and populates it with the provided elements.  
It can also create empty vectors with explicit type annotations. This is the  
most common way to create vectors with known initial values.  

## Creating vectors with Vec::new

Use `Vec::new()` to create an empty vector that can grow as needed.  

```rust
fn main() {
    let mut numbers: Vec<i32> = Vec::new();
    let mut names: Vec<String> = Vec::new();
    
    numbers.push(10);
    numbers.push(20);
    numbers.push(30);
    
    names.push("Alice".to_string());
    names.push("Bob".to_string());
    
    println!("Numbers: {:?}", numbers);
    println!("Names: {:?}", names);
}
```

`Vec::new()` creates an empty vector with zero capacity. The vector will  
automatically allocate memory when elements are added. Type annotations are  
required since the compiler cannot infer the element type from an empty  
vector.  

## Creating vectors with capacity

Pre-allocate memory using `Vec::with_capacity()` for better performance when  
the approximate size is known.  

```rust
fn main() {
    let mut numbers = Vec::with_capacity(10);
    
    for i in 1..=5 {
        numbers.push(i);
    }
    
    println!("Numbers: {:?}", numbers);
    println!("Length: {}, Capacity: {}", numbers.len(), numbers.capacity());
    
    // Adding more elements won't require reallocation
    numbers.push(6);
    numbers.push(7);
    
    println!("After adding more: {:?}", numbers);
    println!("Length: {}, Capacity: {}", numbers.len(), numbers.capacity());
}
```

Pre-allocating capacity reduces the number of memory reallocations when  
adding elements. The capacity represents the total allocated space, while  
length represents the number of actual elements. This optimization is useful  
when you know approximately how many elements you'll store.  

## Adding elements with push and insert

Add elements to the end with `push()` or at specific positions with  
`insert()`.  

```rust
fn main() {
    let mut fruits = vec!["apple", "banana"];
    
    // Add to the end
    fruits.push("cherry");
    fruits.push("date");
    
    println!("After push: {:?}", fruits);
    
    // Insert at specific position
    fruits.insert(1, "blueberry");
    fruits.insert(0, "avocado");
    
    println!("After insert: {:?}", fruits);
}
```

The `push()` method adds elements to the end in O(1) time. The `insert()`  
method can add elements at any position but requires O(n) time as existing  
elements must be shifted. Index 0 inserts at the beginning, while  
`vec.len()` is equivalent to push.  

## Removing elements

Remove elements using `pop()`, `remove()`, or `swap_remove()` methods.  

```rust
fn main() {
    let mut colors = vec!["red", "green", "blue", "yellow", "purple"];
    
    println!("Original: {:?}", colors);
    
    // Remove from end
    let last = colors.pop();
    println!("Popped: {:?}, Remaining: {:?}", last, colors);
    
    // Remove from specific position
    let removed = colors.remove(1);
    println!("Removed: {}, Remaining: {:?}", removed, colors);
    
    // Swap remove (faster but changes order)
    let swapped = colors.swap_remove(0);
    println!("Swap removed: {}, Remaining: {:?}", swapped, colors);
}
```

`pop()` removes and returns the last element in O(1) time. `remove()` takes  
O(n) time as it maintains order by shifting elements. `swap_remove()` is O(1)  
but changes order by moving the last element to the removed position.  

## Accessing elements safely

Access vector elements using indexing or safe methods like `get()`.  

```rust
fn main() {
    let numbers = vec![10, 20, 30, 40, 50];
    
    // Direct indexing (can panic)
    println!("First element: {}", numbers[0]);
    println!("Last element: {}", numbers[numbers.len() - 1]);
    
    // Safe access with get()
    match numbers.get(2) {
        Some(value) => println!("Element at index 2: {}", value),
        None => println!("Index 2 is out of bounds"),
    }
    
    match numbers.get(10) {
        Some(value) => println!("Element at index 10: {}", value),
        None => println!("Index 10 is out of bounds"),
    }
    
    // Safe access to first and last elements
    if let Some(first) = numbers.first() {
        println!("First: {}", first);
    }
    if let Some(last) = numbers.last() {
        println!("Last: {}", last);
    }
}
```

Direct indexing with `[]` is fast but panics on invalid indices. The `get()`  
method returns `Option<T>` for safe access. The `first()` and `last()`  
methods provide safe access to boundary elements without needing to check  
the length manually.  

## Vector length and capacity operations

Monitor and control vector size and memory allocation.  

```rust
fn main() {
    let mut data = Vec::with_capacity(5);
    
    println!("Initial - Length: {}, Capacity: {}", data.len(), data.capacity());
    
    // Add elements
    for i in 1..=3 {
        data.push(i);
    }
    
    println!("After adding 3 - Length: {}, Capacity: {}", data.len(), data.capacity());
    
    // Reserve more space
    data.reserve(10);
    println!("After reserve(10) - Length: {}, Capacity: {}", data.len(), data.capacity());
    
    // Shrink to fit actual usage
    data.shrink_to_fit();
    println!("After shrink_to_fit - Length: {}, Capacity: {}", data.len(), data.capacity());
    
    // Check if empty
    println!("Is empty: {}", data.is_empty());
    
    // Clear all elements
    data.clear();
    println!("After clear - Length: {}, Capacity: {}", data.len(), data.capacity());
    println!("Is empty: {}", data.is_empty());
}
```

Understanding capacity helps optimize memory usage. `reserve()` ensures  
sufficient space for additional elements. `shrink_to_fit()` reduces memory  
overhead by shrinking capacity to match length. `clear()` removes all  
elements but preserves capacity.  

## Iterating with different methods

Explore various iteration patterns for processing vector elements.  

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];
    let mut mutable_numbers = vec![10, 20, 30];
    
    // Iterate with for loop (borrowing)
    println!("For loop iteration:");
    for num in &numbers {
        println!("Value: {}", num);
    }
    
    // Iterate with enumerate for index access
    println!("\nEnumerate iteration:");
    for (index, value) in numbers.iter().enumerate() {
        println!("Index {}: Value {}", index, value);
    }
    
    // Iterate and modify elements
    println!("\nModifying elements:");
    for num in &mut mutable_numbers {
        *num *= 2;
    }
    println!("Modified: {:?}", mutable_numbers);
    
    // Iterator methods
    println!("\nUsing for_each:");
    numbers.iter().for_each(|n| print!("{} ", n));
    println!();
    
    // Consuming iteration
    println!("\nConsuming iteration:");
    for num in numbers {
        println!("Consumed: {}", num);
    }
    // Note: numbers is no longer accessible here
}
```

Different iteration patterns serve different purposes. Borrowing with `&`  
preserves the vector, mutable borrowing with `&mut` allows modification,  
and consuming iteration takes ownership. `enumerate()` provides both index  
and value during iteration.  

## Using map for transformations

Transform vector elements using the `map()` function.  

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];
    
    // Square all numbers
    let squared: Vec<i32> = numbers.iter().map(|x| x * x).collect();
    println!("Original: {:?}", numbers);
    println!("Squared: {:?}", squared);
    
    // Convert to strings
    let strings: Vec<String> = numbers.iter().map(|x| x.to_string()).collect();
    println!("As strings: {:?}", strings);
    
    // Chain multiple transformations
    let processed: Vec<i32> = numbers
        .iter()
        .map(|x| x * 2)
        .map(|x| x + 1)
        .collect();
    println!("Processed (2x + 1): {:?}", processed);
    
    // Working with strings
    let words = vec!["hello", "world", "rust"];
    let lengths: Vec<usize> = words.iter().map(|s| s.len()).collect();
    let uppercase: Vec<String> = words.iter().map(|s| s.to_uppercase()).collect();
    
    println!("Words: {:?}", words);
    println!("Lengths: {:?}", lengths);
    println!("Uppercase: {:?}", uppercase);
}
```

The `map()` function creates a new iterator that applies a transformation  
to each element. It's lazy and only executes when collected. Multiple map  
operations can be chained for complex transformations. The original vector  
remains unchanged unless consumed.  

## Filtering elements

Filter vector elements based on conditions using `filter()`.  

```rust
fn main() {
    let numbers = vec![-3, -1, 0, 2, 4, 6, 7, 9];
    
    // Filter positive numbers
    let positive: Vec<i32> = numbers.iter().filter(|&&x| x > 0).copied().collect();
    println!("Original: {:?}", numbers);
    println!("Positive: {:?}", positive);
    
    // Filter even numbers
    let even: Vec<i32> = numbers.iter().filter(|&&x| x % 2 == 0).copied().collect();
    println!("Even: {:?}", even);
    
    // Combine filter and map
    let positive_doubled: Vec<i32> = numbers
        .iter()
        .filter(|&&x| x > 0)
        .map(|x| x * 2)
        .collect();
    println!("Positive doubled: {:?}", positive_doubled);
    
    // Filter strings by length
    let words = vec!["a", "hello", "hi", "world", "rust", "programming"];
    let long_words: Vec<&str> = words
        .iter()
        .filter(|&&word| word.len() > 3)
        .copied()
        .collect();
    println!("Words: {:?}", words);
    println!("Long words (>3 chars): {:?}", long_words);
}
```

The `filter()` method creates an iterator over elements that satisfy a  
predicate. The `&&x` pattern extracts the value from the double reference  
created by `iter().filter()`. `copied()` converts references back to values  
for collecting into a new vector.  

## Reducing vectors with fold and reduce

Aggregate vector elements into a single value using reduction operations.  

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];
    
    // Sum using fold
    let sum = numbers.iter().fold(0, |acc, x| acc + x);
    println!("Numbers: {:?}", numbers);
    println!("Sum using fold: {}", sum);
    
    // Product using fold
    let product = numbers.iter().fold(1, |acc, x| acc * x);
    println!("Product using fold: {}", product);
    
    // Find maximum using fold
    let max = numbers.iter().fold(i32::MIN, |acc, &x| acc.max(x));
    println!("Maximum using fold: {}", max);
    
    // Sum using reduce (no initial value)
    let sum_reduce = numbers.iter().reduce(|acc, x| acc + x);
    println!("Sum using reduce: {:?}", sum_reduce);
    
    // Concatenate strings
    let words = vec!["Hello", " ", "world", "!"];
    let sentence = words.iter().fold(String::new(), |acc, &word| acc + word);
    println!("Words: {:?}", words);
    println!("Concatenated: '{}'", sentence);
    
    // Built-in reduction methods
    let total: i32 = numbers.iter().sum();
    let product_builtin: i32 = numbers.iter().product();
    println!("Sum (built-in): {}", total);
    println!("Product (built-in): {}", product_builtin);
}
```

`fold()` requires an initial value and always returns a value. `reduce()`  
uses the first element as initial value and returns `Option<T>`. Both are  
powerful for aggregating data. Rust provides built-in `sum()` and  
`product()` methods for common numeric reductions.  

## Sorting vectors

Sort vectors in-place or create sorted copies using various methods.  

```rust
fn main() {
    let mut numbers = vec![64, 34, 25, 12, 22, 11, 90];
    let words = vec!["zebra", "apple", "banana", "cherry"];
    
    println!("Original numbers: {:?}", numbers);
    
    // Sort in-place
    numbers.sort();
    println!("Sorted numbers: {:?}", numbers);
    
    // Sort in reverse order
    numbers.sort_by(|a, b| b.cmp(a));
    println!("Reverse sorted: {:?}", numbers);
    
    // Create sorted copy without modifying original
    let mut words_copy = words.clone();
    words_copy.sort();
    println!("Original words: {:?}", words);
    println!("Sorted words: {:?}", words_copy);
    
    // Sort by custom criteria (length)
    let mut words_by_length = words.clone();
    words_by_length.sort_by(|a, b| a.len().cmp(&b.len()));
    println!("Sorted by length: {:?}", words_by_length);
    
    // Sort using sort_by_key
    let mut words_by_key = words.clone();
    words_by_key.sort_by_key(|s| s.len());
    println!("Sorted by key (length): {:?}", words_by_key);
    
    // Unstable sort (potentially faster)
    let mut more_numbers = vec![5, 2, 8, 1, 9, 3];
    more_numbers.sort_unstable();
    println!("Unstable sorted: {:?}", more_numbers);
}
```

`sort()` performs a stable sort in O(n log n) time. `sort_by()` allows  
custom comparison logic. `sort_by_key()` sorts by a computed key value.  
`sort_unstable()` is potentially faster but doesn't preserve the relative  
order of equal elements.  

## Searching in vectors

Search for elements using various methods including binary search.  

```rust
fn main() {
    let numbers = vec![10, 20, 30, 40, 50];
    let words = vec!["apple", "banana", "cherry", "date"];
    
    // Check if element exists
    println!("Numbers: {:?}", numbers);
    println!("Contains 30: {}", numbers.contains(&30));
    println!("Contains 35: {}", numbers.contains(&35));
    
    // Find position of element
    match numbers.iter().position(|&x| x == 30) {
        Some(index) => println!("Found 30 at index: {}", index),
        None => println!("30 not found"),
    }
    
    // Binary search (requires sorted vector)
    let mut sorted_numbers = vec![5, 15, 25, 35, 45, 55];
    sorted_numbers.sort(); // Ensure it's sorted
    
    match sorted_numbers.binary_search(&25) {
        Ok(index) => println!("Binary search found 25 at index: {}", index),
        Err(index) => println!("25 would be inserted at index: {}", index),
    }
    
    match sorted_numbers.binary_search(&30) {
        Ok(index) => println!("Binary search found 30 at index: {}", index),
        Err(index) => println!("30 would be inserted at index: {}", index),
    }
    
    // Find first element matching condition
    let first_even = numbers.iter().find(|&&x| x % 2 == 0);
    println!("First even number: {:?}", first_even);
    
    // Count elements matching condition
    let even_count = numbers.iter().filter(|&&x| x % 2 == 0).count();
    println!("Count of even numbers: {}", even_count);
    
    // Check if any/all elements match condition
    let has_large = numbers.iter().any(|&&x| x > 40);
    let all_positive = numbers.iter().all(|&&x| x > 0);
    println!("Has number > 40: {}", has_large);
    println!("All numbers positive: {}", all_positive);
}
```

`contains()` checks for existence in O(n) time. `position()` finds the index  
of an element. `binary_search()` works on sorted vectors in O(log n) time.  
`find()`, `any()`, and `all()` provide flexible searching with predicates.  

## Working with vector slices

Create views into vectors using slices for efficient data access.  

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    
    println!("Original vector: {:?}", numbers);
    
    // Get slice of first 5 elements
    let first_half = &numbers[0..5];
    println!("First half: {:?}", first_half);
    
    // Get slice of last 5 elements
    let second_half = &numbers[5..];
    println!("Second half: {:?}", second_half);
    
    // Get middle elements
    let middle = &numbers[3..7];
    println!("Middle elements: {:?}", middle);
    
    // Split at position
    let (left, right) = numbers.split_at(5);
    println!("Split at 5 - Left: {:?}, Right: {:?}", left, right);
    
    // Get chunks of specific size
    let chunks: Vec<&[i32]> = numbers.chunks(3).collect();
    println!("Chunks of 3: {:?}", chunks);
    
    // Get windows (overlapping slices)
    let windows: Vec<&[i32]> = numbers.windows(3).collect();
    println!("Windows of 3: {:?}", windows);
    
    // Work with slice methods
    println!("First element of slice: {:?}", first_half.first());
    println!("Last element of slice: {:?}", first_half.last());
    
    // Convert slice back to vector
    let slice_copy = first_half.to_vec();
    println!("Slice copied to vector: {:?}", slice_copy);
}
```

Slices provide views into vectors without copying data. Range syntax `[a..b]`  
creates slices efficiently. `split_at()` divides at a position. `chunks()`  
creates non-overlapping groups while `windows()` creates overlapping views.  
Slices have their own useful methods and can be converted back to vectors.  

## Converting between vectors and other types

Transform vectors to and from arrays, strings, and other collections.  

```rust
fn main() {
    // Vector to array (fixed size)
    let vector = vec![1, 2, 3, 4, 5];
    
    // Try to convert to array
    let array_result: Result<[i32; 5], _> = vector.clone().try_into();
    match array_result {
        Ok(array) => println!("Vector to array: {:?}", array),
        Err(_) => println!("Conversion failed - size mismatch"),
    }
    
    // Array to vector
    let array = [10, 20, 30, 40];
    let from_array = array.to_vec();
    println!("Array to vector: {:?}", from_array);
    
    // String to vector of characters
    let text = "Hello";
    let chars: Vec<char> = text.chars().collect();
    println!("String '{}' to char vector: {:?}", text, chars);
    
    // Vector of characters to string
    let char_vec = vec!['R', 'u', 's', 't'];
    let string_from_chars: String = char_vec.iter().collect();
    println!("Char vector to string: '{}'", string_from_chars);
    
    // Vector of strings to single string
    let words = vec!["Hello", "world", "from", "Rust"];
    let sentence = words.join(" ");
    println!("Words joined: '{}'", sentence);
    
    // String to vector of words
    let text = "apple banana cherry";
    let word_vec: Vec<&str> = text.split_whitespace().collect();
    println!("Text '{}' split to words: {:?}", text, word_vec);
    
    // Numbers to strings and back
    let numbers = vec![1, 2, 3, 4, 5];
    let number_strings: Vec<String> = numbers.iter().map(|n| n.to_string()).collect();
    println!("Numbers as strings: {:?}", number_strings);
    
    let parsed_numbers: Result<Vec<i32>, _> = number_strings
        .iter()
        .map(|s| s.parse::<i32>())
        .collect();
    
    match parsed_numbers {
        Ok(nums) => println!("Parsed back to numbers: {:?}", nums),
        Err(e) => println!("Parse error: {}", e),
    }
}
```

Converting between types is common in Rust programming. `try_into()` safely  
converts vectors to arrays when sizes match. `join()` and `split()` work  
with strings. `collect()` is versatile for gathering iterators into  
collections. Error handling is important when parsing strings to numbers.  

## Memory management operations

Control vector memory allocation and optimize performance.  

```rust
fn main() {
    let mut data = Vec::new();
    
    println!("Initial state:");
    println!("Length: {}, Capacity: {}", data.len(), data.capacity());
    
    // Reserve space in advance
    data.reserve(100);
    println!("\nAfter reserve(100):");
    println!("Length: {}, Capacity: {}", data.len(), data.capacity());
    
    // Add some elements
    for i in 1..=50 {
        data.push(i);
    }
    
    println!("\nAfter adding 50 elements:");
    println!("Length: {}, Capacity: {}", data.len(), data.capacity());
    
    // Reserve exact additional space
    data.reserve_exact(25);
    println!("\nAfter reserve_exact(25):");
    println!("Length: {}, Capacity: {}", data.len(), data.capacity());
    
    // Shrink to fit current usage
    data.shrink_to_fit();
    println!("\nAfter shrink_to_fit():");
    println!("Length: {}, Capacity: {}", data.len(), data.capacity());
    
    // Create with specific capacity
    let mut optimized = Vec::with_capacity(1000);
    println!("\nVec::with_capacity(1000):");
    println!("Length: {}, Capacity: {}", optimized.len(), optimized.capacity());
    
    // Fill to capacity
    for i in 1..=1000 {
        optimized.push(i);
    }
    
    println!("\nAfter filling to capacity:");
    println!("Length: {}, Capacity: {}", optimized.len(), optimized.capacity());
    
    // Truncate vector
    optimized.truncate(500);
    println!("\nAfter truncate(500):");
    println!("Length: {}, Capacity: {}", optimized.len(), optimized.capacity());
    
    // Check spare capacity
    println!("Spare capacity: {}", optimized.spare_capacity_mut().len());
}
```

Memory management is crucial for performance. `reserve()` allocates at least  
the requested space while `reserve_exact()` allocates exactly that amount.  
`shrink_to_fit()` reduces memory usage. Pre-allocating with  
`with_capacity()` prevents reallocations during growth.  

## Advanced vector operations

Explore sophisticated vector manipulation techniques.  

```rust
fn main() {
    let mut numbers = vec![1, 2, 2, 3, 2, 4, 5, 2, 6];
    
    println!("Original: {:?}", numbers);
    
    // Retain only elements matching condition
    let mut retained = numbers.clone();
    retained.retain(|&x| x != 2);
    println!("After retain (remove 2s): {:?}", retained);
    
    // Remove duplicates (requires sorting first)
    let mut deduped = numbers.clone();
    deduped.sort();
    deduped.dedup();
    println!("After sort and dedup: {:?}", deduped);
    
    // Custom dedup with comparison function
    let mut words = vec!["apple", "APPLE", "banana", "BANANA", "apple"];
    words.dedup_by(|a, b| a.to_lowercase() == b.to_lowercase());
    println!("Words after case-insensitive dedup: {:?}", words);
    
    // Rotate elements
    let mut sequence = vec![1, 2, 3, 4, 5];
    sequence.rotate_left(2);
    println!("After rotate_left(2): {:?}", sequence);
    
    sequence.rotate_right(1);
    println!("After rotate_right(1): {:?}", sequence);
    
    // Reverse vector
    let mut reversed = vec![1, 2, 3, 4, 5];
    reversed.reverse();
    println!("Reversed: {:?}", reversed);
    
    // Swap elements
    let mut swappable = vec!["first", "second", "third", "fourth"];
    swappable.swap(0, 3);
    println!("After swap(0, 3): {:?}", swappable);
    
    // Partition vector based on predicate
    let mixed = vec![1, 3, 2, 4, 5, 6, 7, 8];
    let (evens, odds): (Vec<i32>, Vec<i32>) = mixed
        .iter()
        .partition(|&&x| x % 2 == 0);
    println!("Original: {:?}", mixed);
    println!("Evens: {:?}, Odds: {:?}", evens, odds);
}
```

Advanced operations provide powerful vector manipulation capabilities.  
`retain()` filters in-place efficiently. `dedup()` removes consecutive  
duplicates after sorting. `rotate_left/right()` shifts elements circularly.  
`partition()` splits elements into two collections based on a predicate.  

## Working with nested vectors

Handle vectors containing other vectors for matrix-like data structures.  

```rust
fn main() {
    // Create a matrix (vector of vectors)
    let mut matrix = vec![
        vec![1, 2, 3],
        vec![4, 5, 6],
        vec![7, 8, 9],
    ];
    
    println!("Matrix:");
    for (i, row) in matrix.iter().enumerate() {
        println!("Row {}: {:?}", i, row);
    }
    
    // Access specific element
    println!("Element at [1][2]: {}", matrix[1][2]);
    
    // Modify element
    matrix[0][1] = 99;
    println!("After modifying [0][1] to 99:");
    for row in &matrix {
        println!("{:?}", row);
    }
    
    // Add new row
    matrix.push(vec![10, 11, 12]);
    println!("After adding new row: {:?}", matrix);
    
    // Flatten nested vector
    let flattened: Vec<i32> = matrix.iter().flatten().copied().collect();
    println!("Flattened: {:?}", flattened);
    
    // Create jagged array (rows of different lengths)
    let jagged = vec![
        vec![1],
        vec![2, 3],
        vec![4, 5, 6],
        vec![7, 8, 9, 10],
    ];
    
    println!("\nJagged array:");
    for (i, row) in jagged.iter().enumerate() {
        println!("Row {} (length {}): {:?}", i, row.len(), row);
    }
    
    // Find all elements greater than 5
    let large_elements: Vec<i32> = jagged
        .iter()
        .flatten()
        .filter(|&&x| x > 5)
        .copied()
        .collect();
    println!("Elements > 5: {:?}", large_elements);
    
    // Create 2D grid with specific dimensions
    let rows = 3;
    let cols = 4;
    let grid: Vec<Vec<i32>> = (0..rows)
        .map(|i| (0..cols).map(|j| i * cols + j).collect())
        .collect();
    
    println!("\nGenerated {}x{} grid:", rows, cols);
    for row in &grid {
        println!("{:?}", row);
    }
}
```

Nested vectors enable matrix and grid representations. Access elements with  
double indexing `matrix[row][col]`. The `flatten()` method converts nested  
structures to flat vectors. Jagged arrays have rows of different lengths.  
Generator expressions can create structured data efficiently.  

## Vector from iterators and ranges

Create vectors efficiently from iterators, ranges, and repeat patterns.  

```rust
fn main() {
    // Create vector from range
    let range_vec: Vec<i32> = (1..=10).collect();
    println!("Range vector: {:?}", range_vec);
    
    // Create vector from step range
    let step_vec: Vec<i32> = (0..20).step_by(3).collect();
    println!("Step vector: {:?}", step_vec);
    
    // Repeat single value
    let repeated: Vec<&str> = std::iter::repeat("hello").take(5).collect();
    println!("Repeated: {:?}", repeated);
    
    // Generate with a function
    let generated: Vec<i32> = (0..10).map(|x| x * x).collect();
    println!("Generated squares: {:?}", generated);
    
    // Create from other iterators
    let text = "the quick brown fox";
    let word_lengths: Vec<usize> = text
        .split_whitespace()
        .map(|word| word.len())
        .collect();
    println!("Word lengths: {:?}", word_lengths);
    
    // Cycle through values
    let colors = ["red", "green", "blue"];
    let cycled: Vec<&str> = colors
        .iter()
        .cycle()
        .take(10)
        .copied()
        .collect();
    println!("Cycled colors: {:?}", cycled);
    
    // Chain multiple iterators
    let first = vec![1, 2, 3];
    let second = vec![4, 5, 6];
    let chained: Vec<i32> = first.iter().chain(second.iter()).copied().collect();
    println!("Chained: {:?}", chained);
}
```

Iterator methods provide powerful ways to create vectors. The `collect()`  
method converts any iterator into a vector. `step_by()` creates arithmetic  
progressions, `repeat()` duplicates values, and `cycle()` creates infinite  
repeating patterns. `chain()` combines multiple iterators sequentially.  

## Vector with custom types and collections

Work with vectors containing custom structs, enums, and complex data types.  

```rust
#[derive(Debug, Clone)]
struct Person {
    name: String,
    age: u32,
}

#[derive(Debug)]
enum Status {
    Active,
    Inactive,
    Pending,
}

fn main() {
    // Vector of custom structs
    let mut people = vec![
        Person { name: "Alice".to_string(), age: 30 },
        Person { name: "Bob".to_string(), age: 25 },
        Person { name: "Charlie".to_string(), age: 35 },
    ];
    
    println!("People: {:?}", people);
    
    // Add new person
    people.push(Person { name: "Diana".to_string(), age: 28 });
    
    // Filter people by age
    let adults: Vec<&Person> = people.iter().filter(|p| p.age >= 30).collect();
    println!("Adults (30+): {:?}", adults);
    
    // Sort by age
    people.sort_by(|a, b| a.age.cmp(&b.age));
    println!("Sorted by age: {:?}", people);
    
    // Vector of enums
    let statuses = vec![Status::Active, Status::Pending, Status::Inactive];
    println!("Statuses: {:?}", statuses);
    
    // Vector of vectors with different types
    let mixed_data: Vec<Vec<String>> = vec![
        vec!["name".to_string(), "Alice".to_string()],
        vec!["age".to_string(), "30".to_string()],
        vec!["city".to_string(), "New York".to_string()],
    ];
    
    println!("Mixed data:");
    for (i, row) in mixed_data.iter().enumerate() {
        println!("Row {}: {:?}", i, row);
    }
    
    // Vector of tuples
    let coordinates: Vec<(f64, f64)> = vec![
        (0.0, 0.0),
        (1.5, 2.3),
        (-1.0, 4.2),
        (3.7, -2.1),
    ];
    
    println!("Coordinates: {:?}", coordinates);
    
    // Extract x coordinates
    let x_coords: Vec<f64> = coordinates.iter().map(|(x, _)| *x).collect();
    println!("X coordinates: {:?}", x_coords);
    
    // Find points in first quadrant
    let first_quadrant: Vec<&(f64, f64)> = coordinates
        .iter()
        .filter(|(x, y)| *x > 0.0 && *y > 0.0)
        .collect();
    println!("First quadrant points: {:?}", first_quadrant);
}
```

Vectors work seamlessly with custom types including structs, enums, and  
tuples. Custom types should derive `Clone` for vector operations that need  
copying. Sorting custom types requires implementing comparison logic or  
using `sort_by()`. Pattern matching in closures helps extract data from  
complex types like tuples and structs.
