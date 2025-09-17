# Sets

A set is an unordered collection of unique elements. Rust provides  
`HashSet` for general use and `BTreeSet` for sorted sets.  

## Creating a HashSet

Create a new HashSet and add elements to it.  

```rust
use std::collections::HashSet;

fn main() {
    let mut colors = HashSet::new();
    
    colors.insert("red");
    colors.insert("green");
    colors.insert("blue");
    colors.insert("red"); // Duplicate, won't be added
    
    println!("{:?}", colors);
    println!("Set contains {} unique colors", colors.len());
}
```

The `HashSet::new()` creates an empty set. The `insert()` method adds  
elements, automatically handling duplicates. HashSet doesn't maintain  
insertion order, so output order may vary between runs.  

## HashSet from array

Initialize a HashSet directly from an array of values.  

```rust
use std::collections::HashSet;

fn main() {
    let fruits: HashSet<&str> = ["apple", "banana", "orange", "apple"]
        .iter()
        .cloned()
        .collect();
    
    println!("{:?}", fruits);
    
    // Alternative using from_iter
    let numbers: HashSet<i32> = HashSet::from_iter([1, 2, 3, 2, 4]);
    println!("{:?}", numbers);
}
```

The `collect()` method transforms an iterator into a HashSet, removing  
duplicates automatically. The `from_iter()` function provides a direct  
way to create sets from iterables.  

## Check membership

Test whether specific elements exist in the set.  

```rust
use std::collections::HashSet;

fn main() {
    let animals: HashSet<&str> = ["cat", "dog", "bird"].iter().cloned().collect();
    
    if animals.contains("cat") {
        println!("Found a cat!");
    }
    
    let pets = ["cat", "fish", "rabbit"];
    for pet in pets {
        if animals.contains(pet) {
            println!("{} is in our animal set", pet);
        } else {
            println!("{} is not in our animal set", pet);
        }
    }
}
```

The `contains()` method returns a boolean indicating membership. This  
operation has O(1) average time complexity for HashSet, making it very  
efficient for lookups even with large datasets.  

## Set operations

Perform mathematical set operations like union, intersection, and  
difference.  

```rust
use std::collections::HashSet;

fn main() {
    let set_a: HashSet<i32> = [1, 2, 3, 4].iter().cloned().collect();
    let set_b: HashSet<i32> = [3, 4, 5, 6].iter().cloned().collect();
    
    // Union - all elements from both sets
    let union: HashSet<i32> = set_a.union(&set_b).cloned().collect();
    println!("Union: {:?}", union);
    
    // Intersection - common elements
    let intersection: HashSet<i32> = set_a.intersection(&set_b).cloned().collect();
    println!("Intersection: {:?}", intersection);
    
    // Difference - elements in set_a but not in set_b
    let difference: HashSet<i32> = set_a.difference(&set_b).cloned().collect();
    println!("Difference: {:?}", difference);
    
    // Symmetric difference - elements in either set but not both
    let sym_diff: HashSet<i32> = set_a.symmetric_difference(&set_b).cloned().collect();
    println!("Symmetric difference: {:?}", sym_diff);
}
```

Set operations return iterators that need to be collected into new sets.  
Union combines all unique elements, intersection finds common elements,  
difference shows elements unique to the first set, and symmetric  
difference shows elements unique to either set.  

## Remove elements

Remove elements from a set using various methods.  

```rust
use std::collections::HashSet;

fn main() {
    let mut languages: HashSet<&str> = ["rust", "python", "java", "go"]
        .iter()
        .cloned()
        .collect();
    
    println!("Original: {:?}", languages);
    
    // Remove a specific element
    if languages.remove("java") {
        println!("Removed java successfully");
    }
    
    // Try to remove non-existent element
    if !languages.remove("cpp") {
        println!("cpp was not in the set");
    }
    
    println!("After removal: {:?}", languages);
    
    // Remove all elements
    languages.clear();
    println!("After clear: {:?}", languages);
    println!("Is empty: {}", languages.is_empty());
}
```

The `remove()` method returns true if the element was present and  
removed, false otherwise. The `clear()` method removes all elements,  
and `is_empty()` checks if the set contains no elements.  

## Iterate over set

Loop through set elements using different iteration methods.  

```rust
use std::collections::HashSet;

fn main() {
    let cities: HashSet<&str> = ["tokyo", "london", "paris", "berlin"]
        .iter()
        .cloned()
        .collect();
    
    println!("Cities using for loop:");
    for city in &cities {
        println!("  {}", city);
    }
    
    println!("\nCities using iter().for_each:");
    cities.iter().for_each(|city| println!("  {}", city));
    
    println!("\nUppercase cities:");
    cities.iter()
        .map(|city| city.to_uppercase())
        .for_each(|city| println!("  {}", city));
}
```

Sets can be iterated using for loops or iterator methods. The iteration  
order is not guaranteed in HashSet. Use `&cities` to borrow the set  
during iteration, allowing continued use after the loop.  

## BTreeSet for sorted sets

Use BTreeSet when you need elements to be kept in sorted order.  

```rust
use std::collections::BTreeSet;

fn main() {
    let mut scores = BTreeSet::new();
    scores.insert(95);
    scores.insert(87);
    scores.insert(92);
    scores.insert(88);
    scores.insert(95); // Duplicate, won't be added
    
    println!("Sorted scores: {:?}", scores);
    
    // Get first and last elements
    if let Some(min) = scores.first() {
        println!("Lowest score: {}", min);
    }
    
    if let Some(max) = scores.last() {
        println!("Highest score: {}", max);
    }
    
    // Range operations
    let good_scores: BTreeSet<_> = scores.range(90..).cloned().collect();
    println!("Scores 90 and above: {:?}", good_scores);
}
```

BTreeSet maintains elements in sorted order, enabling operations like  
`first()`, `last()`, and `range()`. The trade-off is slightly slower  
insertions and lookups compared to HashSet, but predictable ordering.  

## Set comparison

Compare sets for equality, subset, and superset relationships.  

```rust
use std::collections::HashSet;

fn main() {
    let set1: HashSet<i32> = [1, 2, 3].iter().cloned().collect();
    let set2: HashSet<i32> = [1, 2, 3].iter().cloned().collect();
    let set3: HashSet<i32> = [1, 2].iter().cloned().collect();
    let set4: HashSet<i32> = [1, 2, 3, 4].iter().cloned().collect();
    
    // Equality
    println!("set1 == set2: {}", set1 == set2);
    println!("set1 == set3: {}", set1 == set3);
    
    // Subset and superset relationships
    println!("set3 is subset of set1: {}", set3.is_subset(&set1));
    println!("set1 is superset of set3: {}", set1.is_superset(&set3));
    println!("set1 is subset of set4: {}", set1.is_subset(&set4));
    
    // Disjoint sets (no common elements)
    let set5: HashSet<i32> = [5, 6, 7].iter().cloned().collect();
    println!("set1 and set5 are disjoint: {}", set1.is_disjoint(&set5));
}
```

Set comparison methods provide mathematical relationships between sets.  
Two sets are equal if they contain the same elements. A subset contains  
only elements that exist in another set. Disjoint sets share no common  
elements.  

## Filter and transform sets

Apply filtering and transformation operations to sets.  

```rust
use std::collections::HashSet;

fn main() {
    let numbers: HashSet<i32> = (1..=10).collect();
    println!("Original numbers: {:?}", numbers);
    
    // Filter even numbers
    let evens: HashSet<i32> = numbers
        .iter()
        .filter(|&&n| n % 2 == 0)
        .cloned()
        .collect();
    println!("Even numbers: {:?}", evens);
    
    // Transform to squares of odd numbers
    let odd_squares: HashSet<i32> = numbers
        .iter()
        .filter(|&&n| n % 2 == 1)
        .map(|&n| n * n)
        .collect();
    println!("Squares of odd numbers: {:?}", odd_squares);
    
    // Find numbers greater than 5
    let large_numbers: Vec<i32> = numbers
        .iter()
        .filter(|&&n| n > 5)
        .cloned()
        .collect();
    println!("Numbers > 5: {:?}", large_numbers);
}
```

Sets work well with iterator methods for functional programming  
patterns. Use `filter()` to select elements meeting criteria, `map()`  
to transform elements, and `collect()` to create new collections.  

## Custom types in sets

Store custom structs in sets by implementing Hash and Eq traits.  

```rust
use std::collections::HashSet;
use std::hash::{Hash, Hasher};

#[derive(Debug, Clone)]
struct Person {
    name: String,
    age: u32,
}

impl PartialEq for Person {
    fn eq(&self, other: &Self) -> bool {
        self.name == other.name && self.age == other.age
    }
}

impl Eq for Person {}

impl Hash for Person {
    fn hash<H: Hasher>(&self, state: &mut H) {
        self.name.hash(state);
        self.age.hash(state);
    }
}

fn main() {
    let mut people = HashSet::new();
    
    people.insert(Person { name: "Alice".to_string(), age: 30 });
    people.insert(Person { name: "Bob".to_string(), age: 25 });
    people.insert(Person { name: "Alice".to_string(), age: 30 }); // Duplicate
    
    println!("Unique people: {:?}", people);
    println!("Number of people: {}", people.len());
    
    // Check if specific person exists
    let target = Person { name: "Alice".to_string(), age: 30 };
    if people.contains(&target) {
        println!("Found Alice!");
    }
}
```

Custom types in sets require implementing `Hash`, `Eq`, and `PartialEq`  
traits. The hash function determines bucket placement, while equality  
traits determine when two instances are considered the same for  
deduplication purposes.