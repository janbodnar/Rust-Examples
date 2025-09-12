# Rust strings 

## Simple example 

```rust
fn main() {
    let mut word = String::from("falcon");

    let s1 = &word;
    let s2 = &word;

    println!("{s1} {s2} = {}", s1.to_owned() + " " + s2);

    let s3 = &mut word;

    s3.push('s');
    println!("{s3}");
}
```
The line `println!("{s1} {s2} = {}", s1.to_owned() + " " + s2);` cannot go after  
`let s3 = &mut word;`  b/c there cannot be both mutable and immutable references in Rust.  

## Parse string

```rust
fn main() {
    if let Ok(n) = "123".parse::<i32>() {
        println!("{}", n);
    }

    let parsed: Result<i32, _> = "123".parse();
    if let Ok(n) = parsed {
        println!("{}", n);
    }
}
```

## Blank string

```rust
// Function to check if a string is blank (empty or whitespace only)
fn is_blank(s: &str) -> bool {
    s.trim().is_empty()
}

fn main() {
    // Create a vector of strings similar to the Java List.of()
    let data = vec!["sky", "\n\n", "  ", "blue", "\t\t", "", "sky"];
    
    // Iterate through the vector with indices
    for (i, e) in data.iter().enumerate() {
        if is_blank(e) {
            println!("element with index {} is blank", i);
        } else {
            println!("{}", e);
        }
    }
}
```


## String interpolation

```rust
fn main() {
    // Basic variable interpolation
    let name = "John Doe";
    let occupation = "gardener";
    println!("My name is {name}, I am a {occupation}.");
    
    // Positional arguments
    println!("My name is {0}, I am a {1}.", name, occupation);
    
    // Named arguments
    println!("My name is {fullname}, I am a {job}.", fullname = name, job = occupation);
    
    // Formatting numbers with precision
    let price = 12.3456;
    println!("The price is {:.2}", price);
    
    // Formatting with different bases
    let number = 42;
    println!("Binary: {:b}, Octal: {:o}, Hexadecimal: {:x}", number, number, number);
    
    // Debug formatting
    let numbers = vec![1, 2, 3];
    println!("Debug output: {:?}", numbers);
    
    // Pretty print for complex data structures
    let map = std::collections::HashMap::from([
        ("sk", "Slovakia"),
        ("cz", "Czechia"),
        ("pl", "Poland"),
        ("hu", "Hungary"),
    ]);
    println!("Pretty print: {:#?}", map);
}
```

## The format! macro

```rust
use std::collections::HashMap;

fn main() {
    // The format! macro is used to create formatted strings in Rust.
    // It works similarly to the println! macro but returns a String
    // instead of printing to the console.

    let name = "John Doe";
    let occupation = "gardener";
    let number = 42;
    let price = 12.34;
    let numbers = vec![1, 2, 3, 4, 5];
    let map = HashMap::from([
        ("sk", "Slovakia"),
        ("cz", "Czechia"),
        ("pl", "Poland"),
        ("hu", "Hungary"),
    ]);

    // Detailed format! macro examples
    println!("\n=== format! macro examples ===");

    // Basic usage of format!
    let formatted_string = format!("My name is {name}, I am a {occupation}.");
    println!("{}", formatted_string);

    // Format with positional arguments
    let formatted_string = format!("My name is {0}, I am a {1}.", name, occupation);
    println!("{}", formatted_string);

    // Format with named arguments
    let formatted_string = format!(
        "My name is {fullname}, I am a {job}.",
        fullname = name,
        job = occupation
    );
    println!("{}", formatted_string);

    // Format with number formatting
    let formatted_string = format!("The price is {:.2}", price);
    println!("{}", formatted_string);

    // Format with different bases
    let formatted_string = format!(
        "Binary: {:b}, Octal: {:o}, Hexadecimal: {:x}",
        number, number, number
    );
    println!("{}", formatted_string);

    // Format with padding and alignment
    let text = "Hello";
    let formatted_string = format!("Left aligned: {:<10}!", text);
    println!("{}", formatted_string);

    let formatted_string = format!("Right aligned: {:>10}!", text);
    println!("{}", formatted_string);

    let formatted_string = format!("Center aligned: {:^10}!", text);
    println!("{}", formatted_string);

    // Format with padding character
    let formatted_string = format!("Padded with zeros: {:0>5}!", 7);
    println!("{}", formatted_string);

    // Format with debug and pretty print
    let formatted_string = format!("Debug output: {:?}", numbers);
    println!("{}", formatted_string);

    let formatted_string = format!("Pretty print: {:#?}", map);
    println!("{}", formatted_string);

    // Complex formatting example
    let score = 95.789;
    let player = "Alice";
    let level = 3;
    let formatted_string = format!(
        "Player: {:<10} | Score: {:>8.1} | Level: {:0>2} | Status: {:?}",
        player,
        score,
        level,
        if score > 90.0 { "Excellent" } else { "Good" }
    );
    println!("{}", formatted_string);
}
```

## Counting characters

Thr "नमस्ते" has 3 Grapheme Clusters

The word "नमस्ते" is composed of several Unicode codepoints that combine to  
form what we see.

1.  `न` (U+0928) - The letter 'na'. This is the **first** grapheme cluster.
2.  `म` (U+092E) - The letter 'ma'. This is the **second** grapheme cluster.
3.  `स` + `्` + `त` + `े` - This sequence forms the single visual unit "स्ते".
      * `स` (U+0938) is the letter 'sa'.
      * `्` (U+094D) is a **Virama** (or halant). This special sign "kills" the inherent vowel
        of the preceding consonant (`स`) and fuses it with the next consonant (`त`).
      * `त` (U+0924) is the letter 'ta'.
      * `े` (U+0947) is the vowel sign for 'e', which modifies the consonant cluster `स्त`.

Because the Virama links `स` and `त`, the entire sequence `स्ते` is correctly identified as  
the **third** grapheme cluster. It's a single, indivisible user-perceived character.


```rust
// Import the unicode segmentation functionality
use unicode_segmentation::UnicodeSegmentation;

// Function to count grapheme clusters in a string
fn grapheme_length(text: &str) -> usize {
    text.graphemes(true).count()
}

fn main() {
    // Test with ASCII string
    let text1 = "falcon";
    let n1 = text1.len();
    println!("{} has {} bytes", text1, n1);
    
    println!("----------------------------");
    
    // Test with Cyrillic string
    let text2 = "вишня";
    let n2 = text2.len();
    println!("{} has {} bytes", text2, n2);
    
    println!("----------------------------");
    
    // Test with emoji string
    let text3 = "🐺🦊🦝";
    let n3 = text3.len();
    println!("{} has {} bytes", text3, n3);
    
    let n3_grapheme = grapheme_length(text3);
    println!("{} has {} grapheme clusters", text3, n3_grapheme);
    
    println!("----------------------------");
    
    // Test with Devanagari string
    let text4 = "नमस्ते";
    let n4 = text4.len();
    println!("{} has {} bytes", text4, n4);
    
    let n4_grapheme = grapheme_length(text4);
    println!("{} has {} grapheme clusters", text4, n4_grapheme);
    
    // Print individual graphemes for debugging
    println!("Individual graphemes:");
    for (i, grapheme) in text4.graphemes(true).enumerate() {
        println!("  {}: '{}'", i, grapheme);
    }
    
    // Print bytes for debugging
    println!("Bytes:");
    for (i, byte) in text4.bytes().enumerate() {
        println!("  {}: {:#04x}", i, byte);
    }
}
```

