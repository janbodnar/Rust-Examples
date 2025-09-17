# Rust Crates Examples

## serde - JSON serialization

Serialize and deserialize Rust data structures to and from JSON format.  
This is essential for APIs, configuration files, and data exchange.  

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Person {
    name: String,
    age: u32,
    email: String,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let person = Person {
        name: "Alice".to_string(),
        age: 30,
        email: "alice@example.com".to_string(),
    };
    
    // Serialize to JSON
    let json = serde_json::to_string(&person)?;
    println!("JSON: {}", json);
    
    // Deserialize from JSON
    let parsed: Person = serde_json::from_str(&json)?;
    println!("Parsed: {:?}", parsed);
    
    Ok(())
}
```

The Serialize and Deserialize traits are automatically derived for structs,  
enabling seamless conversion between Rust types and JSON. The serde_json  
crate provides to_string() for serialization and from_str() for  
deserialization. Error handling with ? operator ensures robust parsing.  

## clap - Command line arguments

Parse command line arguments with automatic help generation and validation.  
Clap provides a declarative way to define CLI interfaces with subcommands  
and options.  

```rust
use clap::{Arg, Command};

fn main() {
    let matches = Command::new("myapp")
        .version("1.0")
        .about("A simple CLI example")
        .arg(
            Arg::new("input")
                .short('i')
                .long("input")
                .value_name("FILE")
                .help("Sets the input file to use")
                .required(true),
        )
        .arg(
            Arg::new("output")
                .short('o')
                .long("output")
                .value_name("FILE")
                .help("Sets the output file")
                .required(false),
        )
        .arg(
            Arg::new("verbose")
                .short('v')
                .long("verbose")
                .help("Enable verbose output")
                .action(clap::ArgAction::SetTrue),
        )
        .get_matches();

    let input_file = matches.get_one::<String>("input").unwrap();
    println!("Input file: {}", input_file);
    
    if let Some(output_file) = matches.get_one::<String>("output") {
        println!("Output file: {}", output_file);
    }
    
    if matches.get_flag("verbose") {
        println!("Verbose mode enabled");
    }
}
```

Clap automatically generates help text and validates arguments. The builder  
pattern defines arguments with short/long forms, validation rules, and help  
text. get_one() retrieves values with type safety, while get_flag() handles  
boolean flags. The library handles parsing errors gracefully.  

## tokio - Async runtime

Execute asynchronous operations using Rust's async/await syntax with tokio  
runtime. This enables concurrent programming for I/O-bound tasks like  
network requests and file operations.  

```rust
use tokio::time::{sleep, Duration};
use std::time::Instant;

async fn fetch_data(id: u32) -> String {
    // Simulate async work
    sleep(Duration::from_millis(100)).await;
    format!("Data for ID: {}", id)
}

async fn process_multiple() {
    let start = Instant::now();
    
    // Sequential execution
    println!("Sequential execution:");
    for i in 1..=3 {
        let result = fetch_data(i).await;
        println!("{}", result);
    }
    println!("Sequential time: {:?}", start.elapsed());
    
    // Concurrent execution
    println!("\nConcurrent execution:");
    let start = Instant::now();
    let futures = (1..=3).map(|i| fetch_data(i));
    let results = futures::future::join_all(futures).await;
    
    for result in results {
        println!("{}", result);
    }
    println!("Concurrent time: {:?}", start.elapsed());
}

#[tokio::main]
async fn main() {
    process_multiple().await;
}
```

The #[tokio::main] macro sets up the async runtime automatically. Functions  
marked with async return Futures that can be awaited. join_all() executes  
multiple futures concurrently, demonstrating the performance benefits of  
async programming for I/O-bound operations.  

## regex - Regular expressions

Pattern matching and text processing using regular expressions. Regex  
provides powerful text search, validation, and extraction capabilities  
for processing unstructured text data.  

```rust
use regex::Regex;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let text = "Contact us at support@example.com or sales@company.org";
    
    // Basic pattern matching
    let email_regex = Regex::new(r"\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b")?;
    
    // Find all matches
    println!("Email addresses found:");
    for mat in email_regex.find_iter(text) {
        println!("  {}", mat.as_str());
    }
    
    // Capture groups
    let phone_text = "Call me at (555) 123-4567 or (555) 987-6543";
    let phone_regex = Regex::new(r"\((\d{3})\) (\d{3})-(\d{4})")?;
    
    println!("\nPhone numbers with area codes:");
    for cap in phone_regex.captures_iter(phone_text) {
        println!("  Full: {}", &cap[0]);
        println!("  Area code: {}", &cap[1]);
        println!("  Number: {}-{}", &cap[2], &cap[3]);
    }
    
    // Replace patterns
    let cleaned = email_regex.replace_all(text, "[EMAIL]");
    println!("\nWith emails masked: {}", cleaned);
    
    Ok(())
}
```

Regex patterns are compiled once and can be reused efficiently. find_iter()  
returns all matches in the text. Capture groups (parentheses) extract  
specific parts of matches. replace_all() enables text transformation by  
substituting matched patterns with new content.  

## chrono - Date and time

Handle dates, times, and durations with timezone support. Chrono provides  
comprehensive datetime functionality for parsing, formatting, and  
arithmetic operations.  

```rust
use chrono::{DateTime, Local, NaiveDate, TimeZone, Utc, Duration};

fn main() {
    // Current time
    let now_utc = Utc::now();
    let now_local = Local::now();
    
    println!("UTC time: {}", now_utc.format("%Y-%m-%d %H:%M:%S"));
    println!("Local time: {}", now_local.format("%Y-%m-%d %H:%M:%S %Z"));
    
    // Parse dates from strings
    let date = NaiveDate::from_ymd_opt(2023, 12, 25).unwrap();
    println!("Christmas 2023: {}", date.format("%A, %B %d, %Y"));
    
    // Parse datetime with timezone
    let parsed = DateTime::parse_from_rfc3339("2023-12-25T10:30:00+00:00").unwrap();
    println!("Parsed: {}", parsed);
    
    // Duration arithmetic
    let future = now_utc + Duration::days(30);
    let past = now_utc - Duration::weeks(2);
    
    println!("30 days from now: {}", future.format("%Y-%m-%d"));
    println!("2 weeks ago: {}", past.format("%Y-%m-%d"));
    
    // Calculate duration between dates
    let duration = future.signed_duration_since(now_utc);
    println!("Days until future date: {}", duration.num_days());
    
    // Convert between timezones
    let tokyo_time = now_utc.with_timezone(&chrono_tz::Asia::Tokyo);
    println!("Tokyo time: {}", tokyo_time.format("%Y-%m-%d %H:%M:%S %Z"));
}
```

Chrono distinguishes between naive (timezone-unaware) and timezone-aware  
datetimes. Duration arithmetic allows adding/subtracting time periods.  
format() method provides flexible string formatting using strftime patterns.  
Timezone conversion preserves the instant while changing the representation.  

## uuid - Unique identifiers

Generate universally unique identifiers for database keys, session tokens,  
and distributed systems. UUIDs provide guaranteed uniqueness without  
coordination between systems.  

```rust
use uuid::Uuid;
use std::collections::HashMap;

fn main() {
    // Generate different UUID versions
    let v4_uuid = Uuid::new_v4(); // Random
    println!("UUID v4 (random): {}", v4_uuid);
    
    // Hyphenated format (default)
    println!("Hyphenated: {}", v4_uuid.hyphenated());
    
    // Simple format (no hyphens)
    println!("Simple: {}", v4_uuid.simple());
    
    // URN format
    println!("URN: {}", v4_uuid.urn());
    
    // Parse UUID from string
    let uuid_str = "550e8400-e29b-41d4-a716-446655440000";
    match Uuid::parse_str(uuid_str) {
        Ok(uuid) => println!("Parsed UUID: {}", uuid),
        Err(e) => println!("Parse error: {}", e),
    }
    
    // Use UUIDs as keys in collections
    let mut user_sessions: HashMap<Uuid, String> = HashMap::new();
    
    for i in 1..=3 {
        let session_id = Uuid::new_v4();
        let username = format!("user{}", i);
        user_sessions.insert(session_id, username.clone());
        println!("Session created: {} -> {}", session_id, username);
    }
    
    // Demonstrate UUID as struct field
    #[derive(Debug)]
    struct User {
        id: Uuid,
        name: String,
    }
    
    let user = User {
        id: Uuid::new_v4(),
        name: "Alice".to_string(),
    };
    
    println!("User: {:?}", user);
}
```

UUID v4 generates cryptographically random identifiers with extremely low  
collision probability. Different formatting options support various use  
cases. UUIDs implement standard traits like Hash and Eq, making them  
suitable for use as HashMap keys and in data structures.  

## rand - Random numbers

Generate random numbers, choose random elements, and shuffle collections.  
The rand crate provides cryptographically secure and fast random number  
generation for various distributions.  

```rust
use rand::{Rng, thread_rng, distributions::Uniform, seq::SliceRandom};

fn main() {
    let mut rng = thread_rng();
    
    // Basic random numbers
    println!("Random u32: {}", rng.gen::<u32>());
    println!("Random float [0,1): {}", rng.gen::<f64>());
    
    // Random in range
    println!("Random 1-100: {}", rng.gen_range(1..=100));
    println!("Random float 10.0-20.0: {}", rng.gen_range(10.0..20.0));
    
    // Random boolean
    println!("Random bool: {}", rng.gen_bool(0.7)); // 70% chance of true
    
    // Generate multiple values efficiently
    let uniform = Uniform::from(1..=6); // Dice roll distribution
    println!("Dice rolls:");
    for _ in 0..5 {
        println!("  {}", rng.sample(uniform));
    }
    
    // Random choice from slice
    let colors = ["red", "green", "blue", "yellow"];
    if let Some(color) = colors.choose(&mut rng) {
        println!("Random color: {}", color);
    }
    
    // Shuffle a vector
    let mut numbers: Vec<i32> = (1..=10).collect();
    println!("Original: {:?}", numbers);
    numbers.shuffle(&mut rng);
    println!("Shuffled: {:?}", numbers);
    
    // Generate random string
    let charset: &[u8] = b"ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
    let random_string: String = (0..8)
        .map(|_| {
            let idx = rng.gen_range(0..charset.len());
            charset[idx] as char
        })
        .collect();
    println!("Random string: {}", random_string);
}
```

thread_rng() provides a thread-local random number generator that's  
automatically seeded. Distributions like Uniform optimize repeated  
sampling from the same range. SliceRandom trait adds choose() and  
shuffle() methods to slices and vectors for random selection.  

## log - Logging framework

Structured logging with configurable levels and output formatting. The log  
crate provides a facade that works with various logging implementations  
like env_logger, simplelog, or custom backends.  

```rust
use log::{trace, debug, info, warn, error};

fn divide_numbers(a: f64, b: f64) -> Result<f64, String> {
    debug!("Attempting to divide {} by {}", a, b);
    
    if b == 0.0 {
        error!("Division by zero attempted: {} / {}", a, b);
        return Err("Cannot divide by zero".to_string());
    }
    
    let result = a / b;
    info!("Division successful: {} / {} = {}", a, b, result);
    Ok(result)
}

fn process_data(data: &[f64]) {
    info!("Processing {} data points", data.len());
    trace!("Input data: {:?}", data);
    
    let sum: f64 = data.iter().sum();
    let avg = sum / data.len() as f64;
    
    if avg > 100.0 {
        warn!("Average value {} is unusually high", avg);
    }
    
    info!("Data processing complete. Average: {:.2}", avg);
}

fn main() {
    // Initialize simple logger (in practice, use env_logger or similar)
    env_logger::init();
    
    info!("Application starting");
    
    // Test division function
    match divide_numbers(10.0, 2.0) {
        Ok(result) => info!("Result: {}", result),
        Err(e) => error!("Error: {}", e),
    }
    
    // Test error case
    if let Err(e) = divide_numbers(5.0, 0.0) {
        error!("Division failed: {}", e);
    }
    
    // Process some data
    let data = vec![45.2, 67.8, 123.4, 89.1, 156.7];
    process_data(&data);
    
    info!("Application completed successfully");
}
```

Log levels (trace, debug, info, warn, error) provide hierarchical filtering.  
env_logger reads RUST_LOG environment variable to control output levels.  
Structured logging with formatting macros captures context information  
for debugging and monitoring production systems.  

## anyhow - Error handling

Simplified error handling with context and error chaining. Anyhow provides  
a unified error type that can wrap any error and add contextual information  
for better debugging.  

```rust
use anyhow::{Context, Result, anyhow, bail};
use std::fs;
use std::num::ParseIntError;

fn read_number_from_file(path: &str) -> Result<i32> {
    let content = fs::read_to_string(path)
        .with_context(|| format!("Failed to read file: {}", path))?;
    
    let trimmed = content.trim();
    if trimmed.is_empty() {
        bail!("File is empty or contains only whitespace");
    }
    
    let number = trimmed.parse::<i32>()
        .with_context(|| format!("Failed to parse '{}' as integer", trimmed))?;
    
    if number < 0 {
        return Err(anyhow!("Number must be non-negative, got: {}", number));
    }
    
    Ok(number)
}

fn process_numbers(file_paths: &[&str]) -> Result<()> {
    let mut total = 0;
    
    for (index, path) in file_paths.iter().enumerate() {
        let number = read_number_from_file(path)
            .with_context(|| format!("Processing file {} ({})", index + 1, path))?;
        
        total += number;
        println!("File {}: {} (running total: {})", index + 1, number, total);
    }
    
    println!("Final total: {}", total);
    Ok(())
}

fn main() -> Result<()> {
    // Create test files for demonstration
    fs::write("number1.txt", "42")?;
    fs::write("number2.txt", "58")?;
    fs::write("empty.txt", "")?;
    fs::write("invalid.txt", "not_a_number")?;
    
    let files = ["number1.txt", "number2.txt"];
    
    println!("Processing valid files:");
    process_numbers(&files)
        .context("Failed to process number files")?;
    
    println!("\nTesting error cases:");
    
    // Test empty file
    if let Err(e) = read_number_from_file("empty.txt") {
        println!("Empty file error: {}", e);
        println!("Error chain:");
        for (i, cause) in e.chain().enumerate() {
            println!("  {}: {}", i, cause);
        }
    }
    
    // Test invalid content
    if let Err(e) = read_number_from_file("invalid.txt") {
        println!("\nInvalid content error: {}", e);
        println!("Error chain:");
        for (i, cause) in e.chain().enumerate() {
            println!("  {}: {}", i, cause);
        }
    }
    
    // Cleanup test files
    let _ = fs::remove_file("number1.txt");
    let _ = fs::remove_file("number2.txt");
    let _ = fs::remove_file("empty.txt");
    let _ = fs::remove_file("invalid.txt");
    
    Ok(())
}
```

Result<T> is an alias for Result<T, anyhow::Error>. with_context() adds  
descriptive information to errors without losing the original error.  
chain() method provides access to the full error chain for debugging.  
anyhow! and bail! macros create custom errors with formatted messages.  

## thiserror - Custom error types

Define custom error types with automatic Display and Error trait  
implementations. Thiserror provides derive macros that eliminate  
boilerplate code for error handling.  

```rust
use thiserror::Error;
use std::fs;
use std::num::ParseIntError;

#[derive(Error, Debug)]
pub enum ConfigError {
    #[error("Configuration file not found: {path}")]
    FileNotFound { path: String },
    
    #[error("Invalid configuration format")]
    InvalidFormat,
    
    #[error("Missing required field: {field}")]
    MissingField { field: String },
    
    #[error("Invalid value for {field}: {value}")]
    InvalidValue { field: String, value: String },
    
    #[error("I/O error")]
    Io(#[from] std::io::Error),
    
    #[error("Parse error")]
    Parse(#[from] ParseIntError),
}

#[derive(Debug)]
struct Config {
    server_port: u16,
    database_url: String,
    max_connections: usize,
}

impl Config {
    fn from_file(path: &str) -> Result<Self, ConfigError> {
        let content = fs::read_to_string(path)
            .map_err(|_| ConfigError::FileNotFound { 
                path: path.to_string() 
            })?;
        
        let mut server_port = None;
        let mut database_url = None;
        let mut max_connections = None;
        
        for line in content.lines() {
            let trimmed = line.trim();
            if trimmed.is_empty() || trimmed.starts_with('#') {
                continue;
            }
            
            let parts: Vec<&str> = trimmed.split('=').collect();
            if parts.len() != 2 {
                return Err(ConfigError::InvalidFormat);
            }
            
            let key = parts[0].trim();
            let value = parts[1].trim();
            
            match key {
                "server_port" => {
                    server_port = Some(value.parse::<u16>()?);
                }
                "database_url" => {
                    database_url = Some(value.to_string());
                }
                "max_connections" => {
                    max_connections = Some(value.parse::<usize>()?);
                }
                _ => {
                    return Err(ConfigError::InvalidValue {
                        field: key.to_string(),
                        value: value.to_string(),
                    });
                }
            }
        }
        
        Ok(Config {
            server_port: server_port.ok_or_else(|| ConfigError::MissingField {
                field: "server_port".to_string(),
            })?,
            database_url: database_url.ok_or_else(|| ConfigError::MissingField {
                field: "database_url".to_string(),
            })?,
            max_connections: max_connections.ok_or_else(|| ConfigError::MissingField {
                field: "max_connections".to_string(),
            })?,
        })
    }
}

fn main() {
    // Create test config file
    let config_content = r#"
# Server configuration
server_port=8080
database_url=postgresql://localhost/myapp
max_connections=100
"#;
    fs::write("config.txt", config_content).unwrap();
    
    // Test successful parsing
    match Config::from_file("config.txt") {
        Ok(config) => {
            println!("Config loaded successfully:");
            println!("  Server port: {}", config.server_port);
            println!("  Database URL: {}", config.database_url);
            println!("  Max connections: {}", config.max_connections);
        }
        Err(e) => println!("Failed to load config: {}", e),
    }
    
    // Test error cases
    println!("\nTesting error cases:");
    
    // Missing file
    if let Err(e) = Config::from_file("missing.txt") {
        println!("Missing file: {}", e);
    }
    
    // Invalid format
    fs::write("invalid.txt", "invalid_config_line").unwrap();
    if let Err(e) = Config::from_file("invalid.txt") {
        println!("Invalid format: {}", e);
    }
    
    // Missing field
    fs::write("incomplete.txt", "server_port=8080").unwrap();
    if let Err(e) = Config::from_file("incomplete.txt") {
        println!("Incomplete config: {}", e);
    }
    
    // Cleanup
    let _ = fs::remove_file("config.txt");
    let _ = fs::remove_file("invalid.txt");
    let _ = fs::remove_file("incomplete.txt");
}
```

The Error derive macro automatically implements Display and Error traits.  
#[from] attribute enables automatic conversion from other error types.  
Error messages can include field values using formatting syntax. Custom  
error types provide better type safety and more descriptive error messages.  

## rayon - Data parallelism

Parallel iteration and computation using work-stealing thread pools. Rayon  
provides simple APIs to parallelize CPU-intensive operations across  
multiple cores without manual thread management.  

```rust
use rayon::prelude::*;
use std::time::Instant;

fn fibonacci(n: u64) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}

fn is_prime(n: u64) -> bool {
    if n < 2 {
        return false;
    }
    for i in 2..=(n as f64).sqrt() as u64 {
        if n % i == 0 {
            return false;
        }
    }
    true
}

fn main() {
    let numbers: Vec<u64> = (1..=40).collect();
    
    // Sequential processing
    println!("Sequential processing:");
    let start = Instant::now();
    let sequential_results: Vec<u64> = numbers
        .iter()
        .map(|&n| fibonacci(n))
        .collect();
    let sequential_time = start.elapsed();
    println!("Time: {:?}", sequential_time);
    
    // Parallel processing
    println!("\nParallel processing:");
    let start = Instant::now();
    let parallel_results: Vec<u64> = numbers
        .par_iter()
        .map(|&n| fibonacci(n))
        .collect();
    let parallel_time = start.elapsed();
    println!("Time: {:?}", parallel_time);
    
    // Verify results are identical
    assert_eq!(sequential_results, parallel_results);
    println!("Results verified - parallel gives same output");
    
    // Parallel filtering and reduction
    let large_numbers: Vec<u64> = (100_000..200_000).collect();
    
    println!("\nFinding prime numbers:");
    let start = Instant::now();
    let prime_count = large_numbers
        .par_iter()
        .filter(|&&n| is_prime(n))
        .count();
    let prime_time = start.elapsed();
    
    println!("Found {} primes in range 100,000-200,000", prime_count);
    println!("Time: {:?}", prime_time);
    
    // Parallel sum with custom reduction
    let sum: u64 = (1..=1_000_000u64)
        .into_par_iter()
        .map(|n| n * n) // Square each number
        .reduce(|| 0, |a, b| a + b);
    
    println!("\nSum of squares 1-1,000,000: {}", sum);
    
    // Parallel sort
    let mut data: Vec<i32> = (1..=100_000).rev().collect(); // Reverse sorted
    println!("\nSorting 100,000 numbers in parallel:");
    let start = Instant::now();
    data.par_sort_unstable();
    let sort_time = start.elapsed();
    println!("Parallel sort time: {:?}", sort_time);
    
    // Verify sorted
    assert!(data.windows(2).all(|w| w[0] <= w[1]));
    println!("Sort verified successful");
}
```

Adding par_iter() instead of iter() automatically parallelizes the  
operation across available CPU cores. Rayon handles work distribution,  
load balancing, and thread synchronization automatically. The same  
functional programming patterns work with parallel iterators.  

## csv - CSV file processing

Read and write CSV files with automatic serialization/deserialization.  
The csv crate handles various CSV formats, custom delimiters, and  
integrates seamlessly with serde for struct mapping.  

```rust
use csv::{Reader, Writer, WriterBuilder};
use serde::{Deserialize, Serialize};
use std::error::Error;
use std::fs;

#[derive(Debug, Serialize, Deserialize)]
struct Employee {
    id: u32,
    name: String,
    department: String,
    salary: f64,
    start_date: String,
}

fn main() -> Result<(), Box<dyn Error>> {
    // Create sample CSV data
    let csv_content = r#"id,name,department,salary,start_date
1,Alice Johnson,Engineering,75000.0,2022-01-15
2,Bob Smith,Marketing,65000.0,2021-06-10
3,Carol Davis,Engineering,82000.0,2020-03-22
4,David Wilson,HR,58000.0,2023-02-01
5,Eve Brown,Marketing,68000.0,2022-11-30"#;
    
    fs::write("employees.csv", csv_content)?;
    
    // Read CSV with automatic deserialization
    println!("Reading employee data:");
    let mut reader = Reader::from_path("employees.csv")?;
    let mut employees = Vec::new();
    
    for result in reader.deserialize() {
        let employee: Employee = result?;
        employees.push(employee);
    }
    
    // Display loaded data
    for emp in &employees {
        println!("  {} - {} (${:,.2})", emp.name, emp.department, emp.salary);
    }
    
    // Filter and process data
    let engineering_avg = employees
        .iter()
        .filter(|emp| emp.department == "Engineering")
        .map(|emp| emp.salary)
        .sum::<f64>() / employees
        .iter()
        .filter(|emp| emp.department == "Engineering")
        .count() as f64;
    
    println!("\nEngineering average salary: ${:,.2}", engineering_avg);
    
    // Create modified data
    let mut high_earners: Vec<Employee> = employees
        .into_iter()
        .filter(|emp| emp.salary > 70000.0)
        .collect();
    
    // Give raises
    for emp in &mut high_earners {
        emp.salary *= 1.05; // 5% raise
    }
    
    // Write filtered data to new CSV
    let mut writer = Writer::from_path("high_earners.csv")?;
    
    for employee in &high_earners {
        writer.serialize(employee)?;
    }
    writer.flush()?;
    
    println!("\nHigh earners with raises written to high_earners.csv:");
    for emp in &high_earners {
        println!("  {} - ${:,.2}", emp.name, emp.salary);
    }
    
    // Custom CSV format example
    let custom_data = vec![
        vec!["Product", "Category", "Price"],
        vec!["Laptop", "Electronics", "999.99"],
        vec!["Book", "Education", "24.95"],
        vec!["Coffee Maker", "Appliances", "89.99"],
    ];
    
    let mut writer = WriterBuilder::new()
        .delimiter(b';')
        .quote_style(csv::QuoteStyle::Always)
        .from_path("products.csv")?;
    
    for record in custom_data {
        writer.write_record(&record)?;
    }
    writer.flush()?;
    
    println!("\nCustom format CSV created with semicolon delimiter");
    
    // Read custom format
    let mut reader = csv::ReaderBuilder::new()
        .delimiter(b';')
        .from_path("products.csv")?;
    
    println!("Custom CSV content:");
    for result in reader.records() {
        let record = result?;
        println!("  {:?}", record);
    }
    
    // Cleanup
    fs::remove_file("employees.csv")?;
    fs::remove_file("high_earners.csv")?;
    fs::remove_file("products.csv")?;
    
    Ok(())
}
```

CSV Reader automatically handles headers and type conversion when used with  
serde derive macros. Writer serializes structs directly to CSV format.  
Custom delimiters, quote styles, and escape characters support various  
CSV dialects. Error handling provides detailed information about parsing  
issues.  

## base64 - Encoding and decoding

Encode binary data as text and decode base64 strings back to binary.  
Base64 encoding is essential for embedding binary data in text formats  
like JSON, XML, or URLs.  

```rust
use base64::{Engine as _, engine::general_purpose};

fn main() {
    // Basic encoding and decoding
    let original_data = b"Hello, World! This is a test message.";
    println!("Original data: {:?}", std::str::from_utf8(original_data).unwrap());
    
    // Encode to base64
    let encoded = general_purpose::STANDARD.encode(original_data);
    println!("Base64 encoded: {}", encoded);
    
    // Decode from base64
    match general_purpose::STANDARD.decode(&encoded) {
        Ok(decoded) => {
            let decoded_str = std::str::from_utf8(&decoded).unwrap();
            println!("Decoded: {}", decoded_str);
        }
        Err(e) => println!("Decode error: {}", e),
    }
    
    // URL-safe encoding (for URLs and filenames)
    let url_data = b"This has + and / characters that are problematic in URLs";
    let url_encoded = general_purpose::URL_SAFE.encode(url_data);
    println!("\nURL-safe encoded: {}", url_encoded);
    
    let url_decoded = general_purpose::URL_SAFE.decode(&url_encoded).unwrap();
    println!("URL-safe decoded: {}", std::str::from_utf8(&url_decoded).unwrap());
    
    // Encoding binary data (simulated image data)
    let binary_data: Vec<u8> = (0..=255).cycle().take(1000).collect();
    let binary_encoded = general_purpose::STANDARD.encode(&binary_data);
    
    println!("\nBinary data ({} bytes) encoded to {} characters", 
             binary_data.len(), binary_encoded.len());
    
    // Demonstrate different encoding variants
    let test_data = b"Test data for different encodings";
    
    println!("\nDifferent encoding variants:");
    println!("Standard:    {}", general_purpose::STANDARD.encode(test_data));
    println!("URL-safe:    {}", general_purpose::URL_SAFE.encode(test_data));
    println!("No padding:  {}", general_purpose::STANDARD_NO_PAD.encode(test_data));
    
    // Error handling for invalid base64
    let invalid_base64 = "This is not valid base64!@#$";
    match general_purpose::STANDARD.decode(invalid_base64) {
        Ok(_) => println!("Unexpected success"),
        Err(e) => println!("Expected decode error: {}", e),
    }
    
    // Practical example: encoding JSON for embedding
    let json_data = r#"{"user_id": 123, "permissions": ["read", "write"]}"#;
    let json_encoded = general_purpose::STANDARD.encode(json_data.as_bytes());
    println!("\nJSON embedded as base64: {}", json_encoded);
    
    // Decode and parse
    let json_decoded = general_purpose::STANDARD.decode(&json_encoded).unwrap();
    let json_str = std::str::from_utf8(&json_decoded).unwrap();
    println!("Extracted JSON: {}", json_str);
}
```

Different engines support various base64 standards: STANDARD uses +/  
characters, URL_SAFE uses -_ for URL compatibility. NO_PAD variants omit  
padding characters. The Engine trait provides encode() and decode()  
methods with comprehensive error handling for malformed input.  

## url - URL parsing and manipulation

Parse, validate, and construct URLs with proper encoding handling. The url  
crate ensures correct URL formatting and provides safe methods for  
building URLs with query parameters and fragments.  

```rust
use url::{Url, form_urlencoded};
use std::collections::HashMap;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Parse a complete URL
    let url_str = "https://example.com:8080/api/v1/users?page=2&limit=50#results";
    let url = Url::parse(url_str)?;
    
    println!("URL parsing:");
    println!("  Full URL: {}", url);
    println!("  Scheme: {}", url.scheme());
    println!("  Host: {:?}", url.host_str());
    println!("  Port: {:?}", url.port());
    println!("  Path: {}", url.path());
    println!("  Query: {:?}", url.query());
    println!("  Fragment: {:?}", url.fragment());
    
    // Parse query parameters
    println!("\nQuery parameters:");
    for (key, value) in url.query_pairs() {
        println!("  {} = {}", key, value);
    }
    
    // Build URL programmatically
    let mut new_url = Url::parse("https://api.example.com")?;
    new_url.set_path("/search");
    
    // Add query parameters safely
    new_url.query_pairs_mut()
        .append_pair("q", "rust programming")
        .append_pair("category", "tutorials")
        .append_pair("sort", "date");
    
    println!("\nConstructed URL: {}", new_url);
    
    // URL joining and resolution
    let base = Url::parse("https://example.com/docs/")?;
    let relative_urls = [
        "tutorial.html",
        "../api/reference.html",
        "/download/install.html",
        "https://other.com/external.html",
    ];
    
    println!("\nURL resolution from base '{}':", base);
    for relative in &relative_urls {
        match base.join(relative) {
            Ok(resolved) => println!("  '{}' -> {}", relative, resolved),
            Err(e) => println!("  '{}' -> Error: {}", relative, e),
        }
    }
    
    // Form URL encoding
    let form_data = [
        ("username", "john_doe"),
        ("email", "john@example.com"),
        ("message", "Hello World! Special chars: +&="),
    ];
    
    let encoded_form: String = form_urlencoded::Serializer::new(String::new())
        .extend_pairs(&form_data)
        .finish();
    
    println!("\nForm URL encoded data: {}", encoded_form);
    
    // Decode form data
    println!("Decoded form data:");
    for (key, value) in form_urlencoded::parse(encoded_form.as_bytes()) {
        println!("  {} = {}", key, value);
    }
    
    // URL modification
    let mut api_url = Url::parse("https://api.service.com/v1/data")?;
    println!("\nURL modification:");
    println!("Original: {}", api_url);
    
    // Change components
    api_url.set_scheme("http").map_err(|_| "Invalid scheme")?;
    api_url.set_host(Some("localhost"))?;
    api_url.set_port(Some(3000))?;
    api_url.set_path("/dev/api/data");
    api_url.set_fragment(Some("section1"));
    
    println!("Modified: {}", api_url);
    
    // Build complex query string
    let mut params = HashMap::new();
    params.insert("filters", vec!["active", "verified"]);
    params.insert("fields", vec!["id", "name", "email"]);
    params.insert("sort", vec!["created_at:desc"]);
    
    let mut query_url = Url::parse("https://api.example.com/users")?;
    {
        let mut query_pairs = query_url.query_pairs_mut();
        for (key, values) in params {
            for value in values {
                query_pairs.append_pair(key, &value);
            }
        }
    }
    
    println!("\nComplex query URL: {}", query_url);
    
    Ok(())
}
```

Url::parse() validates and parses complete URLs with comprehensive error  
reporting. Methods like join() handle relative URL resolution according  
to RFC standards. query_pairs_mut() provides safe query parameter  
manipulation with automatic encoding of special characters.  

## flate2 - Compression and decompression

Compress and decompress data using gzip, deflate, and zlib algorithms.  
Flate2 provides both streaming and one-shot compression interfaces for  
reducing file sizes and network bandwidth.  

```rust
use flate2::{Compression, read::GzDecoder, write::GzEncoder};
use std::io::{Read, Write};
use std::fs;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Sample data to compress
    let original_data = "This is a test message that will be compressed. ".repeat(100);
    println!("Original data size: {} bytes", original_data.len());
    println!("Sample content: {}", &original_data[..50]);
    
    // Compress data using gzip
    let mut encoder = GzEncoder::new(Vec::new(), Compression::default());
    encoder.write_all(original_data.as_bytes())?;
    let compressed_data = encoder.finish()?;
    
    println!("\nCompressed size: {} bytes", compressed_data.len());
    println!("Compression ratio: {:.1}%", 
             (compressed_data.len() as f64 / original_data.len() as f64) * 100.0);
    
    // Decompress the data
    let mut decoder = GzDecoder::new(&compressed_data[..]);
    let mut decompressed_data = String::new();
    decoder.read_to_string(&mut decompressed_data)?;
    
    println!("Decompressed size: {} bytes", decompressed_data.len());
    println!("Data matches original: {}", original_data == decompressed_data);
    
    // File compression example
    fs::write("large_file.txt", &original_data)?;
    
    // Compress file
    let file_content = fs::read("large_file.txt")?;
    let mut encoder = GzEncoder::new(Vec::new(), Compression::best());
    encoder.write_all(&file_content)?;
    let compressed_file = encoder.finish()?;
    
    fs::write("large_file.txt.gz", &compressed_file)?;
    
    println!("\nFile compression:");
    println!("Original file: {} bytes", file_content.len());
    println!("Compressed file: {} bytes", compressed_file.len());
    
    // Different compression levels
    let test_data = "A".repeat(10000); // Highly compressible data
    let levels = [
        ("None", Compression::none()),
        ("Fast", Compression::fast()),
        ("Default", Compression::default()),
        ("Best", Compression::best()),
    ];
    
    println!("\nCompression level comparison for {} bytes of 'A':", test_data.len());
    for (name, level) in levels {
        let mut encoder = GzEncoder::new(Vec::new(), level);
        encoder.write_all(test_data.as_bytes())?;
        let compressed = encoder.finish()?;
        
        println!("  {}: {} bytes ({:.1}%)", 
                 name, 
                 compressed.len(),
                 (compressed.len() as f64 / test_data.len() as f64) * 100.0);
    }
    
    // Stream compression for large data
    println!("\nStreaming compression demo:");
    let chunks = vec![
        "First chunk of data. ",
        "Second chunk with more information. ",
        "Third chunk to complete the stream. ",
    ];
    
    let mut encoder = GzEncoder::new(Vec::new(), Compression::default());
    let mut total_input = 0;
    
    for chunk in &chunks {
        encoder.write_all(chunk.as_bytes())?;
        total_input += chunk.len();
        println!("  Added {} bytes (total input: {})", chunk.len(), total_input);
    }
    
    let final_compressed = encoder.finish()?;
    println!("  Final compressed size: {} bytes", final_compressed.len());
    
    // Verify streaming decompression
    let mut decoder = GzDecoder::new(&final_compressed[..]);
    let mut stream_decompressed = String::new();
    decoder.read_to_string(&mut stream_decompressed)?;
    
    let expected: String = chunks.iter().collect();
    println!("  Stream decompression successful: {}", 
             stream_decompressed == expected);
    
    // Cleanup
    fs::remove_file("large_file.txt")?;
    fs::remove_file("large_file.txt.gz")?;
    
    Ok(())
}
```

GzEncoder and GzDecoder provide streaming compression interfaces that work  
with any Read/Write implementation. Compression levels trade speed for  
size reduction. The finish() method must be called to flush remaining  
data and get the complete compressed output. Streaming enables processing  
large datasets without loading everything into memory.