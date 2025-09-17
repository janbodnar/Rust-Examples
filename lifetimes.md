# Lifetimes

## Basic lifetime annotation with functions

Functions that return references need lifetime annotations to ensure  
memory safety by tracking reference validity.  

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("Hello");
    let string2 = String::from("World!");
    
    let result = longest(&string1, &string2);
    println!("Longest string: {}", result);
}
```

The lifetime annotation `'a` tells the compiler that the returned  
reference will be valid for at least as long as the shorter of the two  
input references. This ensures the reference remains valid when used.  
The lifetime `'a` is a generic lifetime parameter that connects the  
lifetimes of the inputs and output.  

## Lifetime with structs holding references

Structs containing references must declare lifetime parameters to  
ensure referenced data outlives the struct instance.  

```rust
struct Excerpt<'a> {
    part: &'a str,
}

impl<'a> Excerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find '.'");
    
    let excerpt = Excerpt {
        part: first_sentence,
    };
    
    println!("Excerpt: {}", excerpt.part);
}
```

The struct `Excerpt` holds a string slice that must live as long as the  
struct itself. The lifetime parameter `'a` ensures the referenced data  
(`novel` in this case) cannot be dropped while the struct exists. The  
implementation block also needs the lifetime parameter to match.  

## Multiple lifetime parameters

Functions can have multiple lifetime parameters when working with  
references that have different lifetimes.  

```rust
fn announce_and_return_longest<'a, 'b>(
    x: &'a str,
    y: &'a str,
    announcement: &'b str,
) -> &'a str {
    println!("Attention! {}", announcement);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("short");
    let string2 = String::from("much longer string");
    
    {
        let announcement = String::from("Important announcement!");
        let result = announce_and_return_longest(
            &string1, 
            &string2, 
            &announcement
        );
        println!("Result: {}", result);
    }
}
```

Here `'a` connects the two comparison strings with the return value,  
while `'b` is independent for the announcement parameter. The return  
type uses only `'a` since the announcement isn't returned. This  
demonstrates how different lifetime parameters can coexist and serve  
different purposes in the same function signature.  

## Lifetime elision rules

The compiler can often infer lifetimes automatically using elision  
rules, reducing the need for explicit annotations.  

```rust
// These functions have implicit lifetime annotations
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();
    
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    
    &s[..]
}

fn get_slice(slice: &[i32]) -> &[i32] {
    &slice[..2.min(slice.len())]
}

fn main() {
    let sentence = "Hello world from Rust";
    let word = first_word(sentence);
    println!("First word: {}", word);
    
    let numbers = vec![1, 2, 3, 4, 5];
    let first_two = get_slice(&numbers);
    println!("First two: {:?}", first_two);
}
```

The compiler applies elision rules: when there's one input lifetime,  
it's assigned to all output lifetimes. This makes code cleaner by  
eliminating redundant annotations. The functions `first_word` and  
`get_slice` work as if they had explicit lifetime annotations like  
`fn first_word<'a>(s: &'a str) -> &'a str`.  

## Static lifetime

The static lifetime `'static` indicates data that lives for the entire  
program duration, like string literals and global constants.  

```rust
static GREETING: &'static str = "Hello, world!";

fn get_static_str() -> &'static str {
    "This string has static lifetime"
}

fn longest_with_static<'a>(x: &'a str, y: &'static str) -> &'a str {
    println!("Static string: {}", y);
    if x.len() > y.len() {
        x
    } else {
        y  // This coerces 'static to 'a
    }
}

fn main() {
    println!("Global: {}", GREETING);
    
    let static_str = get_static_str();
    println!("Function returned: {}", static_str);
    
    let runtime_string = String::from("Runtime string");
    let result = longest_with_static(&runtime_string, "literal");
    println!("Longest: {}", result);
}
```

Static lifetime is the longest possible lifetime, lasting the entire  
program execution. String literals have static lifetime by default.  
The `'static` lifetime can be coerced to any shorter lifetime, making  
it compatible with other lifetime parameters. This is useful for  
constants and string literals that need to work with dynamic data.  

## Lifetime subtyping and variance

Lifetimes have subtyping relationships where longer lifetimes can be  
used where shorter ones are expected.  

```rust
fn print_refs<'a, 'b>(x: &'a str, y: &'b str) 
where 
    'a: 'b,  // 'a outlives 'b
{
    println!("First: {}, Second: {}", x, y);
}

struct Container<'a> {
    data: &'a str,
}

fn use_container<'short>(container: Container<'short>) {
    println!("Container data: {}", container.data);
}

fn main() {
    let long_lived = String::from("I live long");
    
    {
        let short_lived = String::from("Short life");
        
        // This works because 'long_lived outlives 'short_lived
        print_refs(&long_lived, &short_lived);
        
        let container = Container { data: &long_lived };
        use_container(container);  // Lifetime gets shortened
    }
    
    println!("Long lived is still here: {}", long_lived);
}
```

The `'a: 'b` syntax means lifetime `'a` outlives lifetime `'b`. This  
creates a subtyping relationship where longer lifetimes can substitute  
for shorter ones. Variance rules determine how lifetime parameters in  
generic types relate to each other, enabling flexible yet safe code.  

## Higher-ranked trait bounds (HRTB)

Higher-ranked trait bounds allow working with closures and function  
pointers that work with any lifetime.  

```rust
fn apply_to_all<F>(items: Vec<&str>, mut f: F) -> Vec<String>
where
    F: for<'a> FnMut(&'a str) -> String,
{
    items.into_iter().map(|item| f(item)).collect()
}

fn call_with_closure<F>(f: F) -> String
where
    F: for<'a> Fn(&'a str) -> &'a str,
{
    let s = String::from("temporary");
    let result = f(&s);
    result.to_string()  // Convert to owned to return
}

fn main() {
    let words = vec!["hello", "world", "rust"];
    
    let uppercased = apply_to_all(words, |s| s.to_uppercase());
    println!("Uppercased: {:?}", uppercased);
    
    let result = call_with_closure(|s| {
        if s.len() > 5 {
            &s[..5]
        } else {
            s
        }
    });
    println!("Result: {}", result);
}
```

The `for<'a>` syntax creates higher-ranked trait bounds, meaning the  
closure must work for any lifetime `'a`. This is essential when  
working with closures that need to handle references with varying  
lifetimes. HRTBs enable generic functions to work with closures that  
operate on borrowed data without knowing the specific lifetimes.  

## Lifetime bounds in generics

Generic types can have lifetime bounds to ensure they contain only  
data that lives long enough.  

```rust
use std::fmt::Display;

struct Wrapper<'a, T: 'a> {
    value: &'a T,
}

impl<'a, T: Display + 'a> Wrapper<'a, T> {
    fn new(value: &'a T) -> Self {
        Wrapper { value }
    }
    
    fn print(&self) {
        println!("Wrapped value: {}", self.value);
    }
}

fn store_if_long_enough<'a, T: Display + 'a>(
    items: &'a [T], 
    min_lifetime: &'a str
) -> Option<Wrapper<'a, T>> {
    if items.len() > 2 {
        Some(Wrapper::new(&items[0]))
    } else {
        None
    }
}

fn main() {
    let numbers = vec![42, 84, 126];
    let lifetime_marker = "marker";
    
    if let Some(wrapper) = store_if_long_enough(&numbers, lifetime_marker) {
        wrapper.print();
    }
    
    let text = String::from("Hello, Rust!");
    let text_wrapper = Wrapper::new(&text);
    text_wrapper.print();
}
```

The `T: 'a` bound ensures the generic type `T` lives at least as long  
as lifetime `'a`. This is necessary when storing references to `T` in  
structs with lifetime parameters. The bound guarantees that the  
referenced data won't be dropped while the wrapper exists, maintaining  
memory safety across generic abstractions.  

## Anonymous lifetimes and underscore syntax

Anonymous lifetimes using `'_` let the compiler infer lifetimes while  
maintaining explicit control over lifetime relationships.  

```rust
struct Parser<'a> {
    input: &'a str,
    position: usize,
}

impl<'a> Parser<'a> {
    fn new(input: &'a str) -> Self {
        Parser { input, position: 0 }
    }
    
    // Anonymous lifetime - compiler infers it
    fn current_char(&self) -> Option<char> {
        self.input.chars().nth(self.position)
    }
    
    // Explicit lifetime relationship
    fn remaining(&self) -> &'a str {
        &self.input[self.position..]
    }
    
    // Anonymous lifetime in parameter
    fn skip_until(&mut self, target: char) -> Option<&'_ str> {
        if let Some(pos) = self.input[self.position..].find(target) {
            let start = self.position;
            self.position += pos;
            Some(&self.input[start..self.position])
        } else {
            None
        }
    }
}

fn main() {
    let text = "Hello, world! This is Rust.";
    let mut parser = Parser::new(text);
    
    if let Some(skipped) = parser.skip_until(',') {
        println!("Skipped: '{}'", skipped);
    }
    
    println!("Remaining: '{}'", parser.remaining());
}
```

The `'_` syntax tells the compiler to infer the lifetime, which is  
useful when you want to be explicit about having a lifetime without  
specifying exactly which one. This is particularly helpful in complex  
types where some lifetimes are obvious and others need explicit  
control. It makes code more readable while maintaining type safety.  

## Lifetime in closures and function pointers

Closures capture references with specific lifetimes, and function  
pointers must specify lifetime relationships explicitly.  

```rust
fn create_closure<'a>(data: &'a str) -> impl Fn() -> &'a str + 'a {
    move || {
        println!("Accessing: {}", data);
        data
    }
}

fn process_with_callback<'a, F>(
    data: &'a [&'a str], 
    callback: F
) -> Vec<&'a str>
where
    F: Fn(&'a str) -> &'a str,
{
    data.iter().map(|&item| callback(item)).collect()
}

type StringProcessor = for<'a> fn(&'a str) -> &'a str;

fn truncate_string(s: &str) -> &str {
    if s.len() > 10 {
        &s[..10]
    } else {
        s
    }
}

fn main() {
    let message = String::from("Hello from closure");
    let closure = create_closure(&message);
    println!("Closure result: {}", closure());
    
    let words = vec!["hello", "world", "rust", "programming"];
    let processed = process_with_callback(&words, |s| {
        if s.len() > 4 { &s[..4] } else { s }
    });
    println!("Processed: {:?}", processed);
    
    let processor: StringProcessor = truncate_string;
    let truncated = processor("This is a very long string");
    println!("Truncated: {}", truncated);
}
```

Closures that capture references need lifetime annotations to ensure  
the captured data lives long enough. The `impl Fn() -> &'a str + 'a`  
syntax means the closure returns a reference with lifetime `'a` and  
itself must live for `'a`. Function pointers with lifetimes use  
higher-ranked trait bounds to work with any lifetime.  

## Complex lifetime relationships

Real-world scenarios often involve multiple structs with interconnected  
lifetime dependencies requiring careful analysis.  

```rust
struct Database<'a> {
    data: &'a [String],
}

struct Query<'a, 'b> {
    db: &'a Database<'b>,
    filter: &'a str,
}

struct QueryResult<'a, 'b, 'c> {
    query: &'a Query<'b, 'c>,
    results: Vec<&'c str>,
}

impl<'a> Database<'a> {
    fn new(data: &'a [String]) -> Self {
        Database { data }
    }
    
    fn search<'b>(&'b self, pattern: &str) -> Vec<&'a str> {
        self.data
            .iter()
            .filter(|item| item.contains(pattern))
            .map(|s| s.as_str())
            .collect()
    }
}

impl<'a, 'b> Query<'a, 'b> {
    fn new(db: &'a Database<'b>, filter: &'a str) -> Self {
        Query { db, filter }
    }
    
    fn execute(&self) -> QueryResult<'_, 'b, 'b> {
        let results = self.db.search(self.filter);
        QueryResult {
            query: self,
            results,
        }
    }
}

fn main() {
    let data = vec![
        String::from("rust programming"),
        String::from("system programming"),
        String::from("web development"),
    ];
    
    let database = Database::new(&data);
    let filter = "programming";
    let query = Query::new(&database, filter);
    
    let result = query.execute();
    println!("Found {} results for '{}':", 
             result.results.len(), result.query.filter);
    for item in result.results {
        println!("  - {}", item);
    }
}
```

This example demonstrates complex lifetime relationships where multiple  
lifetimes interact. The `Database` has lifetime `'a` for its data, the  
`Query` has lifetime `'a` for database reference and `'b` for the  
database's data lifetime. The `QueryResult` connects all three  
lifetimes, ensuring the results remain valid relative to the original  
data source.  

## Lifetime with iterators and lazy evaluation

Iterators that return references must maintain lifetime relationships  
with their source data throughout the iteration chain.  

```rust
struct TextAnalyzer<'a> {
    content: &'a str,
}

impl<'a> TextAnalyzer<'a> {
    fn new(content: &'a str) -> Self {
        TextAnalyzer { content }
    }
    
    fn words(&self) -> impl Iterator<Item = &'a str> {
        self.content.split_whitespace()
    }
    
    fn lines(&self) -> impl Iterator<Item = &'a str> {
        self.content.lines()
    }
    
    fn long_words(&self, min_length: usize) -> impl Iterator<Item = &'a str> {
        self.words().filter(move |word| word.len() >= min_length)
    }
}

fn process_text<'a>(analyzer: &TextAnalyzer<'a>) -> Vec<(&'a str, usize)> {
    analyzer
        .long_words(5)
        .map(|word| (word, word.len()))
        .collect()
}

fn main() {
    let text = "The quick brown fox jumps over the lazy dog. \
                Programming in Rust provides memory safety \
                without garbage collection.";
    
    let analyzer = TextAnalyzer::new(text);
    
    println!("Long words (5+ chars):");
    for (word, length) in process_text(&analyzer) {
        println!("  {} ({})", word, length);
    }
    
    // Chaining iterators with lifetimes
    let first_line_words: Vec<&str> = analyzer
        .lines()
        .next()
        .unwrap_or("")
        .split_whitespace()
        .take(3)
        .collect();
    
    println!("First three words: {:?}", first_line_words);
}
```

Iterator chains preserve lifetime relationships from the source data.  
The `impl Iterator<Item = &'a str>` return type ensures all iterator  
operations maintain the connection to the original string slice. This  
enables lazy evaluation while guaranteeing memory safety - the  
iterator results remain valid as long as the source data exists.  

## Self-referential structs and lifetime issues

Self-referential structs create complex lifetime challenges that often  
require careful design or alternative approaches.  

```rust
// This won't compile - self-referential struct
// struct SelfRef<'a> {
//     data: String,
//     reference: &'a str,  // Can't reference self.data
// }

// Alternative 1: Separate the data and reference
struct DataHolder {
    data: String,
}

struct DataRef<'a> {
    reference: &'a str,
}

// Alternative 2: Using indices instead of references
struct IndexedData {
    data: String,
    important_range: (usize, usize),
}

impl IndexedData {
    fn new(data: String, start: usize, end: usize) -> Self {
        IndexedData {
            data,
            important_range: (start, end),
        }
    }
    
    fn get_important_part(&self) -> &str {
        let (start, end) = self.important_range;
        &self.data[start..end.min(self.data.len())]
    }
}

// Alternative 3: Using smart pointers (Pin, Box)
use std::pin::Pin;

struct PinnedData {
    data: String,
}

impl PinnedData {
    fn get_pinned_slice(self: Pin<&Self>) -> &str {
        &self.data[..5.min(self.data.len())]
    }
}

fn main() {
    // Using separate structures
    let holder = DataHolder {
        data: String::from("Hello, Rust programming world!"),
    };
    let data_ref = DataRef {
        reference: &holder.data[7..11],  // "Rust"
    };
    println!("Reference: {}", data_ref.reference);
    
    // Using indices
    let indexed = IndexedData::new(
        String::from("Important data here"),
        10,
        14
    );
    println!("Important part: {}", indexed.get_important_part());
    
    // Using Pin for safe self-reference
    let pinned_data = Box::pin(PinnedData {
        data: String::from("Pinned string data"),
    });
    let slice = pinned_data.as_ref().get_pinned_slice();
    println!("Pinned slice: {}", slice);
}
```

Self-referential structs cannot be created directly because Rust's  
ownership model prevents references to fields within the same struct.  
Common solutions include separating data and references into different  
structs, using indices instead of references, or employing smart  
pointers like `Pin` for controlled self-reference scenarios.  

## Lifetime coercion examples

Rust automatically coerces longer lifetimes to shorter ones when safe,  
and provides explicit ways to handle lifetime relationships.  

```rust
fn demonstrate_coercion<'long, 'short>(
    long_data: &'long str,
    short_data: &'short str,
) -> &'short str
where
    'long: 'short,  // 'long outlives 'short
{
    // Automatic coercion from 'long to 'short
    if long_data.len() > short_data.len() {
        long_data  // Coerced from &'long str to &'short str
    } else {
        short_data
    }
}

// Coercion with mutable references
fn coerce_mut_refs<'a>(long_data: &'a mut String) -> &'a str {
    // &mut String coerces to &str through deref coercion
    long_data
}

// Subtyping example
fn accepts_any_lifetime(s: &str) {
    println!("Received: {}", s);
}

fn main() {
    let long_lived = String::from("This lives for the entire main function");
    
    {
        let short_lived = String::from("Short");
        
        // 'long_lived has longer lifetime than 'short_lived
        let result = demonstrate_coercion(&long_lived, &short_lived);
        println!("Coercion result: {}", result);
        
        // Static lifetime coerces to any shorter lifetime
        accepts_any_lifetime("static string");  // &'static str -> &'? str
        accepts_any_lifetime(&short_lived);     // &'short str
        accepts_any_lifetime(&long_lived);      // &'long str
    }
    
    // Mutable reference coercion
    let mut mutable_data = String::from("Mutable data");
    let immutable_ref = coerce_mut_refs(&mut mutable_data);
    println!("Coerced ref: {}", immutable_ref);
    
    // Subtype relationship in action
    let shorter_lifetime = {
        let temp = String::from("temporary");
        &temp[..4]  // This won't compile - temp doesn't live long enough
    };
    // println!("{}", shorter_lifetime);  // Would fail
}
```

Lifetime coercion allows longer lifetimes to be used where shorter  
ones are expected, following subtyping rules. The `'long: 'short`  
bound establishes the subtyping relationship. Deref coercion also  
works with lifetimes, allowing `&mut T` to coerce to `&T` while  
preserving lifetime relationships. These coercions happen  
automatically when safe.  

## Real-world patterns and best practices

Practical lifetime patterns for building robust applications with  
clear ownership and borrowing strategies.  

```rust
use std::collections::HashMap;

// Pattern 1: Builder with lifetime for configuration
struct ConfigBuilder<'a> {
    name: &'a str,
    settings: HashMap<&'a str, &'a str>,
}

impl<'a> ConfigBuilder<'a> {
    fn new(name: &'a str) -> Self {
        ConfigBuilder {
            name,
            settings: HashMap::new(),
        }
    }
    
    fn set(&mut self, key: &'a str, value: &'a str) -> &mut Self {
        self.settings.insert(key, value);
        self
    }
    
    fn build(self) -> Config<'a> {
        Config {
            name: self.name,
            settings: self.settings,
        }
    }
}

struct Config<'a> {
    name: &'a str,
    settings: HashMap<&'a str, &'a str>,
}

// Pattern 2: Cache with borrowed keys
struct Cache<'a, T> {
    data: HashMap<&'a str, T>,
}

impl<'a, T> Cache<'a, T> {
    fn new() -> Self {
        Cache {
            data: HashMap::new(),
        }
    }
    
    fn get(&self, key: &str) -> Option<&T> {
        self.data.get(key)
    }
    
    fn insert(&mut self, key: &'a str, value: T) {
        self.data.insert(key, value);
    }
}

// Pattern 3: View types for temporary data access
struct DataView<'a> {
    slice: &'a [u8],
    offset: usize,
}

impl<'a> DataView<'a> {
    fn new(data: &'a [u8]) -> Self {
        DataView { slice: data, offset: 0 }
    }
    
    fn read_u32(&mut self) -> Option<u32> {
        if self.offset + 4 <= self.slice.len() {
            let bytes = &self.slice[self.offset..self.offset + 4];
            self.offset += 4;
            Some(u32::from_le_bytes([bytes[0], bytes[1], bytes[2], bytes[3]]))
        } else {
            None
        }
    }
    
    fn remaining(&self) -> &'a [u8] {
        &self.slice[self.offset..]
    }
}

fn main() {
    // Builder pattern usage
    let app_name = "MyApplication";
    let version = "1.0.0";
    let author = "Developer";
    
    let config = ConfigBuilder::new(app_name)
        .set("version", version)
        .set("author", author)
        .build();
    
    println!("Config for: {}", config.name);
    
    // Cache pattern usage
    let mut cache: Cache<i32> = Cache::new();
    let key1 = "first";
    let key2 = "second";
    
    cache.insert(key1, 42);
    cache.insert(key2, 84);
    
    if let Some(value) = cache.get("first") {
        println!("Cached value: {}", value);
    }
    
    // Data view pattern usage
    let data = vec![0x2A, 0x00, 0x00, 0x00, 0x54, 0x00, 0x00, 0x00];
    let mut view = DataView::new(&data);
    
    if let Some(first) = view.read_u32() {
        println!("First u32: {}", first);
    }
    
    println!("Remaining bytes: {:?}", view.remaining());
}
```

These patterns demonstrate practical lifetime usage: builders with  
borrowed configuration data, caches with string slice keys, and view  
types for parsing binary data. The key principles are: use borrowed  
data when you don't need ownership, establish clear lifetime  
relationships between related types, and prefer composition over  
complex self-referential structures for maintainable code.