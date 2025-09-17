# Hashmaps

A HashMap stores key-value pairs and provides fast lookups, insertions, and  
deletions. Keys must implement the Hash and Eq traits.  

## Basic creation and insertion

Create a HashMap and insert key-value pairs using various methods.  

```rust
use std::collections::HashMap;

fn main() {
    // Create empty HashMap and insert values
    let mut capitals = HashMap::new();
    capitals.insert("Slovakia", "Bratislava");
    capitals.insert("Hungary", "Budapest");
    capitals.insert("Poland", "Warsaw");
    
    println!("{:?}", capitals);
    
    // Create HashMap with initial capacity
    let mut scores: HashMap<String, i32> = HashMap::with_capacity(10);
    scores.insert("Alice".to_string(), 95);
    scores.insert("Bob".to_string(), 87);
    
    println!("{:?}", scores);
}
```

The `HashMap::new()` creates an empty HashMap that will automatically grow as  
needed. The `insert()` method adds key-value pairs, returning `None` for new  
keys or the previous value if the key already existed. Using  
`HashMap::with_capacity()` pre-allocates space for better performance when  
you know the approximate size.  

## HashMap from arrays and iterators

Initialize HashMap from existing data using various collection methods.  

```rust
use std::collections::HashMap;

fn main() {
    // Create from array of tuples
    let countries = HashMap::from([
        ("sk", "Slovakia"),
        ("hu", "Hungary"), 
        ("pl", "Poland"),
        ("cz", "Czechia"),
    ]);
    
    println!("From array: {:?}", countries);
    
    // Create from iterator of tuples
    let data = vec![("red", "#FF0000"), ("green", "#00FF00"), ("blue", "#0000FF")];
    let colors: HashMap<&str, &str> = data.into_iter().collect();
    
    println!("From iterator: {:?}", colors);
    
    // Create from two vectors
    let keys = vec!["apple", "banana", "cherry"];
    let values = vec![1.20, 0.80, 2.50];
    let prices: HashMap<&str, f64> = keys.into_iter().zip(values).collect();
    
    println!("From zip: {:?}", prices);
}
```

The `HashMap::from()` method creates a HashMap from an array of tuples.  
The `collect()` method can convert any iterator of key-value pairs into a  
HashMap. The `zip()` method pairs elements from two iterators, creating  
tuples that can be collected into a HashMap.  

## Accessing values

Retrieve values from HashMap using different access methods.  

```rust
use std::collections::HashMap;

fn main() {
    let mut inventory = HashMap::from([
        ("apples", 50),
        ("bananas", 30),
        ("oranges", 25),
    ]);
    
    // Using get() method (returns Option)
    match inventory.get("apples") {
        Some(count) => println!("We have {} apples", count),
        None => println!("No apples in inventory"),
    }
    
    // Using get() with if let
    if let Some(count) = inventory.get("bananas") {
        println!("Bananas in stock: {}", count);
    }
    
    // Using get() with unwrap_or for default value
    let grape_count = inventory.get("grapes").unwrap_or(&0);
    println!("Grapes: {}", grape_count);
    
    // Using index operator (panics if key doesn't exist)
    println!("Oranges: {}", inventory["oranges"]);
    
    // Get mutable reference
    if let Some(count) = inventory.get_mut("apples") {
        *count += 10;
        println!("Updated apples: {}", count);
    }
}
```

The `get()` method returns an `Option<&V>` that's `Some(&value)` if the key  
exists or `None` if it doesn't. This prevents panics when accessing  
non-existent keys. The index operator `[]` provides direct access but panics  
if the key is missing. The `get_mut()` method returns a mutable reference  
for updating values in place.  

## Checking for keys and values

Test for the presence of keys and values using various query methods.  

```rust
use std::collections::HashMap;

fn main() {
    let grades = HashMap::from([
        ("Alice", 95),
        ("Bob", 87),
        ("Charlie", 92),
        ("Diana", 88),
    ]);
    
    // Check if key exists
    if grades.contains_key("Alice") {
        println!("Alice's grade is recorded");
    }
    
    // Check multiple keys
    let students = ["Alice", "Eve", "Bob"];
    for student in students {
        if grades.contains_key(student) {
            println!("{} has a grade: {}", student, grades[student]);
        } else {
            println!("{} is not in the gradebook", student);
        }
    }
    
    // Check if HashMap is empty
    if !grades.is_empty() {
        println!("Gradebook has {} entries", grades.len());
    }
    
    // Check for specific value (less efficient than key lookup)
    let target_grade = 95;
    let has_perfect = grades.values().any(|&grade| grade == target_grade);
    println!("Someone got {}? {}", target_grade, has_perfect);
}
```

The `contains_key()` method efficiently checks if a key exists without  
retrieving the value. The `is_empty()` and `len()` methods provide size  
information. The `values()` method returns an iterator over all values,  
which can be used with methods like `any()` to check for value existence.  

## Iteration patterns

Iterate over HashMap keys, values, and key-value pairs.  

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::from([
        ("Alice", 95),
        ("Bob", 87),
        ("Charlie", 92),
        ("Diana", 88),
    ]);
    
    println!("Iterating over key-value pairs:");
    for (name, score) in &scores {
        println!("{}: {}", name, score);
    }
    
    println!("\nIterating over keys only:");
    for name in scores.keys() {
        println!("Student: {}", name);
    }
    
    println!("\nIterating over values only:");
    for score in scores.values() {
        println!("Score: {}", score);
    }
    
    println!("\nMutable iteration to update values:");
    for (name, score) in scores.iter_mut() {
        if *score < 90 {
            *score += 5; // Bonus points
            println!("Bonus applied to {}: new score {}", name, score);
        }
    }
    
    println!("\nFinal scores: {:?}", scores);
}
```

HashMap iteration doesn't guarantee any particular order since hash tables  
are unordered collections. The `iter()` method (or `&hashmap`) yields  
`(&K, &V)` pairs, `keys()` yields `&K`, and `values()` yields `&V`. The  
`iter_mut()` method yields `(&K, &mut V)` for modifying values in place.  

## Updating and modifying values

Update existing values and handle cases where keys may or may not exist.  

```rust
use std::collections::HashMap;

fn main() {
    let mut word_count = HashMap::new();
    let text = "the quick brown fox jumps over the lazy dog the fox";
    
    // Count word occurrences
    for word in text.split_whitespace() {
        // Method 1: Using entry API
        let count = word_count.entry(word).or_insert(0);
        *count += 1;
    }
    
    println!("Word counts: {:?}", word_count);
    
    // Method 2: Manual check and update
    let mut scores = HashMap::from([("Alice", 85), ("Bob", 90)]);
    
    // Update existing value
    if let Some(score) = scores.get_mut("Alice") {
        *score += 10;
    }
    
    // Insert new or update existing
    scores.insert("Charlie", scores.get("Charlie").unwrap_or(&0) + 95);
    
    // Method 3: Using entry API for complex logic
    scores.entry("Diana").and_modify(|score| *score += 5).or_insert(80);
    
    println!("Updated scores: {:?}", scores);
}
```

The entry API provides efficient ways to handle conditional insertion and  
updates. The `or_insert()` method inserts a value only if the key doesn't  
exist. The `and_modify()` method allows updating existing values while  
`or_insert()` provides a fallback for missing keys. This pattern avoids  
double lookups that would occur with separate `contains_key()` and `insert()`  
calls.  

## Removing elements

Remove elements from HashMap using various methods.  

```rust
use std::collections::HashMap;

fn main() {
    let mut inventory = HashMap::from([
        ("apples", 50),
        ("bananas", 30),
        ("oranges", 25),
        ("grapes", 15),
        ("mangoes", 8),
    ]);
    
    println!("Initial inventory: {:?}", inventory);
    
    // Remove specific key-value pair
    if let Some(removed_count) = inventory.remove("bananas") {
        println!("Removed {} bananas", removed_count);
    }
    
    // Remove items with low stock (< 10)
    let to_remove: Vec<_> = inventory
        .iter()
        .filter(|(_, &count)| count < 10)
        .map(|(&fruit, _)| fruit)
        .collect();
    
    for fruit in to_remove {
        inventory.remove(fruit);
        println!("Removed {} from inventory", fruit);
    }
    
    // Clear all elements
    let mut temp_map = inventory.clone();
    temp_map.clear();
    println!("After clear: {:?}", temp_map);
    
    // Remove and return entry
    if let Some((fruit, count)) = inventory.remove_entry("apples") {
        println!("Removed entry: {} -> {}", fruit, count);
    }
    
    println!("Final inventory: {:?}", inventory);
}
```

The `remove()` method removes a key-value pair and returns the value if the  
key existed. When removing multiple items based on conditions, collect the  
keys first to avoid borrowing conflicts during iteration. The `clear()`  
method removes all elements efficiently. The `remove_entry()` method returns  
both the key and value as a tuple.  

## HashMap with different data types

Demonstrate HashMap usage with various key and value types.  

```rust
use std::collections::HashMap;

fn main() {
    // String keys with integer values
    let mut populations: HashMap<String, u64> = HashMap::new();
    populations.insert("New York".to_string(), 8_336_817);
    populations.insert("Los Angeles".to_string(), 3_979_576);
    
    // Integer keys with string values
    let status_codes = HashMap::from([
        (200, "OK"),
        (404, "Not Found"),
        (500, "Internal Server Error"),
    ]);
    
    // Character keys with boolean values
    let vowels: HashMap<char, bool> = "aeiou"
        .chars()
        .map(|c| (c, true))
        .collect();
    
    // Tuple keys with complex values
    let mut coordinates: HashMap<(i32, i32), String> = HashMap::new();
    coordinates.insert((0, 0), "Origin".to_string());
    coordinates.insert((1, 1), "Northeast".to_string());
    coordinates.insert((-1, -1), "Southwest".to_string());
    
    // Vector values
    let mut groups: HashMap<&str, Vec<&str>> = HashMap::new();
    groups.insert("fruits", vec!["apple", "banana", "orange"]);
    groups.insert("colors", vec!["red", "green", "blue"]);
    
    println!("Populations: {:?}", populations);
    println!("Status codes: {:?}", status_codes);
    println!("Vowels: {:?}", vowels);
    println!("Coordinates: {:?}", coordinates);
    println!("Groups: {:?}", groups);
}
```

HashMap keys must implement `Hash + Eq`. Most primitive types work as keys,  
including strings, numbers, characters, and tuples of hashable types.  
Values can be any type, including complex structures like vectors or other  
collections. When using `String` as keys, consider using `&str` if possible  
to avoid unnecessary allocations.  

## HashMap with custom structs

Use custom structs as keys and values in HashMap.  

```rust
use std::collections::HashMap;
use std::hash::{Hash, Hasher};

#[derive(Debug, Clone)]
struct Person {
    name: String,
    age: u32,
}

// Custom struct that implements Hash and Eq for use as HashMap key
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
struct ProductId {
    category: String,
    id: u32,
}

#[derive(Debug, Clone)]
struct Product {
    name: String,
    price: f64,
    in_stock: bool,
}

fn main() {
    // HashMap with custom struct values
    let mut people: HashMap<u32, Person> = HashMap::new();
    people.insert(1, Person { name: "Alice".to_string(), age: 30 });
    people.insert(2, Person { name: "Bob".to_string(), age: 25 });
    
    println!("People: {:#?}", people);
    
    // HashMap with custom struct keys and values
    let mut inventory: HashMap<ProductId, Product> = HashMap::new();
    
    let laptop_id = ProductId { 
        category: "Electronics".to_string(), 
        id: 1001 
    };
    let laptop = Product {
        name: "Gaming Laptop".to_string(),
        price: 1299.99,
        in_stock: true,
    };
    
    inventory.insert(laptop_id.clone(), laptop);
    
    let book_id = ProductId {
        category: "Books".to_string(),
        id: 2001,
    };
    inventory.insert(book_id, Product {
        name: "Rust Programming".to_string(),
        price: 49.99,
        in_stock: false,
    });
    
    // Access product by custom key
    if let Some(product) = inventory.get(&laptop_id) {
        println!("Found product: {} - ${}", product.name, product.price);
    }
    
    println!("Inventory: {:#?}", inventory);
}
```

To use custom structs as HashMap keys, they must implement `Hash`, `Eq`, and  
`PartialEq`. The `#[derive(Hash, Eq, PartialEq)]` attribute automatically  
implements these traits for structs with fields that also implement them.  
Custom values don't need these traits but should implement `Debug` and  
`Clone` for convenience.  

## HashMap capacity and performance

Manage HashMap capacity and understand performance characteristics.  

```rust
use std::collections::HashMap;

fn main() {
    // Create HashMap with initial capacity
    let mut scores: HashMap<String, i32> = HashMap::with_capacity(100);
    
    println!("Initial capacity: {}", scores.capacity());
    println!("Length: {}", scores.len());
    
    // Add elements
    for i in 0..50 {
        scores.insert(format!("player_{}", i), i * 10);
    }
    
    println!("After 50 insertions:");
    println!("Capacity: {}", scores.capacity());
    println!("Length: {}", scores.len());
    
    // Reserve additional capacity
    scores.reserve(100);
    println!("After reserve(100): capacity = {}", scores.capacity());
    
    // Shrink to fit actual size
    scores.shrink_to_fit();
    println!("After shrink_to_fit: capacity = {}", scores.capacity());
    
    // Performance demonstration: lookup timing
    let key = "player_25";
    let start = std::time::Instant::now();
    for _ in 0..1000 {
        let _ = scores.get(key);
    }
    let duration = start.elapsed();
    println!("1000 lookups took: {:?}", duration);
    
    // Memory usage estimation
    let memory_per_entry = std::mem::size_of::<(String, i32)>();
    let estimated_memory = scores.len() * memory_per_entry;
    println!("Estimated memory usage: {} bytes", estimated_memory);
}
```

HashMap automatically resizes when load factor becomes too high, but  
pre-allocating with `with_capacity()` or `reserve()` avoids expensive  
reallocations. The `shrink_to_fit()` method reduces capacity to save memory.  
HashMap provides O(1) average-case lookup, insertion, and deletion  
performance, making it excellent for frequent key-based operations.  

## Merging and combining HashMaps

Combine multiple HashMaps using different strategies.  

```rust
use std::collections::HashMap;

fn main() {
    let scores1 = HashMap::from([
        ("Alice", 95),
        ("Bob", 87),
        ("Charlie", 92),
    ]);
    
    let scores2 = HashMap::from([
        ("Diana", 88),
        ("Bob", 90),  // Different value for existing key
        ("Eve", 94),
    ]);
    
    // Method 1: Extend (overwrites existing keys)
    let mut combined1 = scores1.clone();
    combined1.extend(scores2.clone());
    println!("Extended (overwrites): {:?}", combined1);
    
    // Method 2: Merge with custom logic (keep higher score)
    let mut combined2 = scores1.clone();
    for (key, value) in scores2.clone() {
        combined2.entry(key)
            .and_modify(|existing| *existing = (*existing).max(value))
            .or_insert(value);
    }
    println!("Merged (keep max): {:?}", combined2);
    
    // Method 3: Merge with addition for existing keys
    let mut combined3 = scores1.clone();
    for (key, value) in scores2.clone() {
        *combined3.entry(key).or_insert(0) += value;
    }
    println!("Merged (sum values): {:?}", combined3);
    
    // Method 4: Chain iterators to create new HashMap
    let combined4: HashMap<&str, i32> = scores1
        .iter()
        .chain(scores2.iter())
        .map(|(&k, &v)| (k, v))
        .collect();
    println!("Chained (first wins): {:?}", combined4);
}
```

The `extend()` method adds all key-value pairs from another HashMap,  
overwriting existing keys. For custom merge logic, use the entry API to  
handle conflicts. The `chain()` iterator method can combine HashMap  
iterators, with earlier values taking precedence for duplicate keys when  
collected.  

## Grouping and aggregation

Use HashMap for grouping data and performing aggregations.  

```rust
use std::collections::HashMap;

fn main() {
    let sales_data = vec![
        ("Electronics", 1200.0),
        ("Books", 45.0),
        ("Electronics", 800.0),
        ("Clothing", 230.0),
        ("Books", 67.0),
        ("Electronics", 450.0),
        ("Clothing", 89.0),
    ];
    
    // Group by category and sum values
    let mut category_totals: HashMap<&str, f64> = HashMap::new();
    for (category, amount) in &sales_data {
        *category_totals.entry(category).or_insert(0.0) += amount;
    }
    
    println!("Category totals: {:?}", category_totals);
    
    // Group by category and collect all values
    let mut category_sales: HashMap<&str, Vec<f64>> = HashMap::new();
    for (category, amount) in &sales_data {
        category_sales.entry(category).or_insert_with(Vec::new).push(*amount);
    }
    
    println!("Category sales: {:?}", category_sales);
    
    // Calculate statistics per category
    let mut category_stats: HashMap<&str, (f64, f64, usize)> = HashMap::new();
    for (category, sales) in &category_sales {
        let sum: f64 = sales.iter().sum();
        let count = sales.len();
        let average = sum / count as f64;
        category_stats.insert(category, (sum, average, count));
    }
    
    println!("Category statistics (sum, avg, count):");
    for (category, (sum, avg, count)) in &category_stats {
        println!("{}: total=${:.2}, avg=${:.2}, count={}", 
                 category, sum, avg, count);
    }
}
```

HashMap is excellent for grouping operations. Use `or_insert()` for simple  
accumulation, `or_insert_with(Vec::new)` for collecting multiple values per  
key, and more complex logic for calculating statistics. The entry API  
ensures efficient single-lookup operations for both checking and updating.  

## HashMap with default values

Handle missing keys gracefully using default values and the entry API.  

```rust
use std::collections::HashMap;

#[derive(Debug, Default)]
struct PlayerStats {
    games_played: u32,
    wins: u32,
    losses: u32,
}

impl PlayerStats {
    fn win_rate(&self) -> f64 {
        if self.games_played == 0 {
            0.0
        } else {
            self.wins as f64 / self.games_played as f64
        }
    }
}

fn main() {
    let mut player_stats: HashMap<String, PlayerStats> = HashMap::new();
    
    // Simulate game results
    let games = vec![
        ("Alice", true),
        ("Bob", false),
        ("Alice", true),
        ("Charlie", true),
        ("Bob", true),
        ("Alice", false),
    ];
    
    for (player, won) in games {
        let stats = player_stats
            .entry(player.to_string())
            .or_insert_with(PlayerStats::default);
        
        stats.games_played += 1;
        if won {
            stats.wins += 1;
        } else {
            stats.losses += 1;
        }
    }
    
    println!("Player statistics:");
    for (player, stats) in &player_stats {
        println!("{}: {} games, {:.2}% win rate", 
                 player, stats.games_played, stats.win_rate() * 100.0);
    }
    
    // Using or_default() for simple default values
    let mut counters: HashMap<String, i32> = HashMap::new();
    let words = ["hello", "world", "hello", "rust", "world", "hello"];
    
    for word in words {
        *counters.entry(word.to_string()).or_default() += 1;
    }
    
    println!("Word counts: {:?}", counters);
}
```

The `or_insert_with()` method takes a closure that creates the default value,  
useful for complex types. The `or_default()` method uses the type's `Default`  
implementation. This pattern ensures that you can always safely access and  
modify values without checking if keys exist first.  

## Converting between HashMap and other collections

Transform HashMap to and from vectors, arrays, and other data structures.  

```rust
use std::collections::HashMap;

fn main() {
    let original_map = HashMap::from([
        ("apple", 1.20),
        ("banana", 0.80),
        ("cherry", 2.50),
        ("date", 3.00),
    ]);
    
    println!("Original map: {:?}", original_map);
    
    // Convert to vector of tuples
    let mut pairs: Vec<(&str, f64)> = original_map.iter()
        .map(|(&k, &v)| (k, v))
        .collect();
    pairs.sort_by(|a, b| a.0.cmp(b.0)); // Sort by key
    println!("Sorted pairs: {:?}", pairs);
    
    // Convert back to HashMap
    let sorted_map: HashMap<&str, f64> = pairs.into_iter().collect();
    println!("Recreated map: {:?}", sorted_map);
    
    // Separate keys and values into vectors
    let keys: Vec<&str> = original_map.keys().cloned().collect();
    let values: Vec<f64> = original_map.values().cloned().collect();
    println!("Keys: {:?}", keys);
    println!("Values: {:?}", values);
    
    // Filter and transform to new HashMap
    let expensive_items: HashMap<&str, f64> = original_map
        .iter()
        .filter(|(_, &price)| price > 1.50)
        .map(|(&name, &price)| (name, price))
        .collect();
    println!("Expensive items: {:?}", expensive_items);
    
    // Transform values while keeping keys
    let discounted: HashMap<&str, f64> = original_map
        .iter()
        .map(|(&name, &price)| (name, price * 0.9)) // 10% discount
        .collect();
    println!("Discounted prices: {:?}", discounted);
    
    // Create frequency map from vector
    let words = vec!["apple", "banana", "apple", "cherry", "banana", "apple"];
    let frequency: HashMap<&str, usize> = words
        .iter()
        .fold(HashMap::new(), |mut acc, &word| {
            *acc.entry(word).or_insert(0) += 1;
            acc
        });
    println!("Word frequency: {:?}", frequency);
}
```

HashMap integrates well with iterators and can be easily converted to other  
collections. The `collect()` method works with any iterator of key-value  
pairs. Use `filter()` and `map()` to transform HashMaps selectively. The  
`fold()` method is powerful for creating HashMaps from sequences with  
custom logic.  

## Nested HashMaps and complex structures

Work with multi-level data structures using nested HashMaps.  

```rust
use std::collections::HashMap;

type CityData = HashMap<String, i32>;
type CountryData = HashMap<String, CityData>;

fn main() {
    let mut world_populations: CountryData = HashMap::new();
    
    // Initialize country data
    let slovakia_cities = HashMap::from([
        ("Bratislava".to_string(), 432_000),
        ("Košice".to_string(), 240_000),
        ("Prešov".to_string(), 91_000),
    ]);
    
    let hungary_cities = HashMap::from([
        ("Budapest".to_string(), 1_752_000),
        ("Debrecen".to_string(), 201_000),
        ("Szeged".to_string(), 161_000),
    ]);
    
    world_populations.insert("Slovakia".to_string(), slovakia_cities);
    world_populations.insert("Hungary".to_string(), hungary_cities);
    
    // Add new city to existing country
    world_populations
        .entry("Slovakia".to_string())
        .or_insert_with(HashMap::new)
        .insert("Žilina".to_string(), 80_000);
    
    // Access nested data
    if let Some(slovakia) = world_populations.get("Slovakia") {
        if let Some(population) = slovakia.get("Bratislava") {
            println!("Bratislava population: {}", population);
        }
    }
    
    // Calculate total population per country
    for (country, cities) in &world_populations {
        let total_population: i32 = cities.values().sum();
        println!("{} total population: {}", country, total_population);
        
        // Show all cities
        println!("  Cities:");
        for (city, population) in cities {
            println!("    {}: {}", city, population);
        }
    }
    
    // Find largest city across all countries
    let mut largest_city = ("", 0);
    for cities in world_populations.values() {
        for (city, &population) in cities {
            if population > largest_city.1 {
                largest_city = (city, population);
            }
        }
    }
    println!("Largest city: {} with {} people", largest_city.0, largest_city.1);
}
```

Nested HashMaps enable hierarchical data structures. Use type aliases for  
complex nested types to improve readability. The entry API works well with  
nested structures, allowing safe initialization of intermediate levels.  
When traversing nested HashMaps, consider error handling with `Option`  
chaining or the `?` operator.  

## HashMap performance patterns and best practices

Demonstrate efficient HashMap usage patterns and common optimizations.  

```rust
use std::collections::HashMap;
use std::time::Instant;

fn main() {
    // Pre-sizing for known data size
    let capacity = 10000;
    let mut efficient_map: HashMap<i32, String> = HashMap::with_capacity(capacity);
    
    let start = Instant::now();
    for i in 0..capacity {
        efficient_map.insert(i as i32, format!("value_{}", i));
    }
    let efficient_time = start.elapsed();
    
    // Without pre-sizing (multiple reallocations)
    let mut growing_map: HashMap<i32, String> = HashMap::new();
    let start = Instant::now();
    for i in 0..capacity {
        growing_map.insert(i as i32, format!("value_{}", i));
    }
    let growing_time = start.elapsed();
    
    println!("Pre-sized insertion: {:?}", efficient_time);
    println!("Growing insertion: {:?}", growing_time);
    
    // Efficient string key usage
    let string_keys = vec!["key1", "key2", "key3", "key4", "key5"];
    
    // Good: Use &str when possible
    let str_map: HashMap<&str, i32> = string_keys
        .iter()
        .enumerate()
        .map(|(i, &key)| (key, i as i32))
        .collect();
    
    // Less efficient: Converting to String unnecessarily
    let string_map: HashMap<String, i32> = string_keys
        .iter()
        .enumerate()
        .map(|(i, &key)| (key.to_string(), i as i32))
        .collect();
    
    println!("str_map memory efficiency: {} entries", str_map.len());
    println!("string_map overhead: {} entries", string_map.len());
    
    // Batch operations are more efficient
    let mut batch_map: HashMap<i32, i32> = HashMap::with_capacity(1000);
    let data: Vec<(i32, i32)> = (0..1000).map(|i| (i, i * i)).collect();
    
    let start = Instant::now();
    batch_map.extend(data);
    let batch_time = start.elapsed();
    
    println!("Batch insertion time: {:?}", batch_time);
    
    // Reserve space before adding many elements
    let mut reserve_map: HashMap<i32, i32> = HashMap::new();
    reserve_map.reserve(500); // Avoid multiple reallocations
    
    // Use entry API to avoid double lookups
    let mut counter_map: HashMap<char, i32> = HashMap::new();
    let text = "hello world hello rust";
    
    let start = Instant::now();
    for ch in text.chars() {
        if ch != ' ' {
            *counter_map.entry(ch).or_insert(0) += 1;
        }
    }
    let entry_time = start.elapsed();
    
    println!("Entry API counting: {:?}", entry_time);
    println!("Character counts: {:?}", counter_map);
}
```

Pre-allocating HashMap capacity prevents expensive reallocations during  
growth. Use `&str` keys instead of `String` when the string data is static  
or borrowed. The entry API is more efficient than separate `contains_key()`  
and `insert()` operations. Batch operations with `extend()` are faster than  
individual insertions for large datasets.
