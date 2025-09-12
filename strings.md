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

Convert string representations of numbers to their numeric types using the  
`parse()` method.  

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

The `parse()` method attempts to convert a string slice into a specified  
numeric type. It returns a `Result<T, ParseIntError>` where `T` is the target  
type. The first example uses turbofish syntax `::<i32>` to specify the type  
explicitly. The second example uses type inference by declaring the variable  
type. Both approaches use pattern matching with `if let Ok()` to handle the  
potential parsing failure gracefully.  

## String lines

Split multi-line strings into individual lines and process each line  
separately using the `lines()` method.  

```rust
fn main() {
    // Define a multi-line string with words
    let words = "
club
sky
blue
cup
coin
new
cent
owl
falcon
brave
war
ice
paint
water";

    // Split the string into lines and iterate through them
    for word in words.lines() {
        // Check if the word length is 3
        if word.len() == 3 {
            println!("{}", word);
        }
    }
}
```

The `lines()` method returns an iterator over the lines of a string, splitting  
on line terminators. Each line is a string slice that excludes the line  
terminator. This example demonstrates filtering lines by length, printing only  
three-character words. The method handles both Unix (`\n`) and Windows  
(`\r\n`) line endings automatically.  


## Multi-line string

Create multi-line string literals without explicit escape sequences, similar  
to text blocks in other languages.  

```rust
fn main() {
    // Define a multi-line string similar to Java's text block
    let lyrics = "I cheated myself
like I knew I would
I told ya, I was trouble
you know that I'm no good";
    
    // Print the lyrics
    println!("{}", lyrics);
}
```

Rust allows multi-line string literals by simply writing the string across  
multiple lines. Line breaks are preserved as part of the string content.  
This approach is useful for embedding formatted text, poetry, or configuration  
data directly in source code without using escape sequences like `\n`.  

## Modifying string

Demonstrate various methods for modifying mutable strings including adding,  
removing, and inserting characters.  

```rust
fn main() {
    // Create a mutable string similar to Java's StringBuilder
    let mut sb = String::from("Misty mountains");
    println!("{}", sb);

    // Delete the last character (similar to sb.deleteCharAt(sb.length()-1))
    sb.pop();
    println!("{}", sb);

    // Append 's'
    sb.push('s');
    println!("{}", sb);

    // Insert 'T', 'h', 'e', ' ' at the beginning
    sb.insert(0, 'T');
    sb.insert(1, 'h');
    sb.insert(2, 'e');
    sb.insert(3, ' ');
    println!("{}", sb);

    // Set the character at index 4 to 'm'
    // In Rust, we need to remove the character at index 4 and insert 'm' at that position
    sb.remove(4);
    sb.insert(4, 'm');
    println!("{}", sb);
}
```

The `String` type provides methods for in-place modification: `pop()` removes  
the last character, `push()` appends a character, `insert()` adds a character  
at a specific position, and `remove()` deletes a character at an index.  
Unlike string slices (`&str`), `String` is growable and can be modified.  
These operations require the string to be declared as mutable (`mut`).  

## Blank string

Check if strings are blank (empty or contain only whitespace characters)  
using a custom function and string methods.  

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

The `is_blank()` function combines `trim()` and `is_empty()` to detect strings  
that are either empty or contain only whitespace. The `trim()` method removes  
leading and trailing whitespace (spaces, tabs, newlines), then `is_empty()`  
checks if any content remains. This pattern is useful for validating user  
input or processing text data that may contain formatting whitespace.  


## String interpolation

Format strings with variables and expressions using Rust's powerful  
formatting macros and syntax.  

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

Rust's `println!` macro supports various formatting options: direct variable  
interpolation with `{variable}`, positional arguments with `{0}`, named  
parameters, precision control with `{:.2}`, number base conversion with  
`{:b}` (binary), `{:o}` (octal), `{:x}` (hexadecimal), debug formatting with  
`{:?}`, and pretty-printing with `{:#?}`. This system provides type-safe  
string formatting with compile-time checking.  

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

