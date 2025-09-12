
# File I/O Examples

## Reading a file

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    let content = fs::read_to_string("example.txt")?;
    println!("File content: {}", content);
    Ok(())
}
```


## Reading a file line by line

```rust
use std::fs::File;
use std::io::{BufRead, BufReader};

fn main() -> std::io::Result<()> {
    let file = File::open("example.txt")?;
    let reader = BufReader::new(file);
    
    for (index, line) in reader.lines().enumerate() {
        println!("Line {}: {}", index + 1, line?);
    }
    
    Ok(())
}
```
## Reading a file by buffer

```rust
use std::fs::File;
use std::io::Read;

fn main() -> std::io::Result<()> {
    let mut file = File::open("example.txt")?;
    let mut buffer = [0; 1024]; // 1KB buffer
    
    loop {
        let bytes_read = file.read(&mut buffer)?;
        if bytes_read == 0 {
            break; // End of file
        }
        
        // Process the buffer content
        let content = &buffer[..bytes_read];
        println!("Read {} bytes: {:?}", bytes_read, content);
    }
    
    Ok(())
}
```
## Writing to a file

```rust
use std::fs::File;
use std::io::Write;

fn main() -> std::io::Result<()> {
    let mut file = File::create("output.txt")?;
    file.write_all(b"Hello, world!\n")?;
    file.write_all("This is a test file.\n".as_bytes())?;
    Ok(())
}
```
## Appending to a file

```rust
use std::fs::OpenOptions;
use std::io::Write;

fn main() -> std::io::Result<()> {
    let mut file = OpenOptions::new()
        .append(true)
        .create(true)
        .open("output.txt")?;
        
    file.write_all(b"Appended line\n")?;
    Ok(())
}
```
## Getting file information

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    let metadata = fs::metadata("example.txt")?;
    
    println!("File size: {} bytes", metadata.len());
    println!("Is file: {}", metadata.is_file());
    println!("Is directory: {}", metadata.is_dir());
    println!("Created: {:?}", metadata.created());
    println!("Modified: {:?}", metadata.modified());
    println!("Accessed: {:?}", metadata.accessed());
    
    Ok(())
}
```
## Creating a file

```rust
use std::fs::File;

fn main() -> std::io::Result<()> {
    let _file = File::create("new_file.txt")?;
    println!("File created successfully");
    Ok(())
}
```
## Creating a file while checking if it exists

```rust
use std::fs::File;
use std::path::Path;

fn main() -> std::io::Result<()> {
    let file_path = "new_file.txt";
    
    if Path::new(file_path).exists() {
        println!("File already exists");
    } else {
        let _file = File::create(file_path)?;
        println!("File created successfully");
    }
    
    Ok(())
}
```
## Creating a directory

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    fs::create_dir("new_directory")?;
    println!("Directory created successfully");
    Ok(())
}
```
## Listing a directory

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    let entries = fs::read_dir(".")?;
    
    for entry in entries {
        let entry = entry?;
        let path = entry.path();
        
        if let Some(file_name) = path.file_name() {
            println!("{}", file_name.to_string_lossy());
        }
    }
    
    Ok(())
}
```
## Listing directory recursively

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    let entries = fs::read_dir(".")?;
    
    for entry in entries {
        let entry = entry?;
        let path = entry.path();
        
        if path.is_dir() {
            println!("Directory: {}", path.display());
            // Recursively list subdirectory contents
            list_dir_recursive(&path, 1)?;
        } else {
            println!("File: {}", path.display());
        }
    }
    
    Ok(())
}

fn list_dir_recursive(path: &std::path::Path, depth: usize) -> std::io::Result<()> {
    let entries = fs::read_dir(path)?;
    
    for entry in entries {
        let entry = entry?;
        let path = entry.path();
        
        // Indent based on depth
        let indent = "  ".repeat(depth);
        
        if path.is_dir() {
            println!("{}Directory: {}", indent, path.display());
            // Recursively list subdirectory contents
            list_dir_recursive(&path, depth + 1)?;
        } else {
            println!("{}File: {}", indent, path.display());
        }
    }
    
    Ok(())
}
```

## File operations

File operations include copying, moving, and deleting files using the  
standard library functions.  

### Copying a file

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    fs::copy("source.txt", "destination.txt")?;
    println!("File copied successfully");
    Ok(())
}
```

The `fs::copy` function copies the contents and most metadata of a file  
from source to destination. It returns the number of bytes copied on  
success.  

### Moving a file

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    fs::rename("old_name.txt", "new_name.txt")?;
    println!("File moved/renamed successfully");
    Ok(())
}
```

The `fs::rename` function moves a file from one location to another or  
renames it. This operation is atomic on most platforms.  

### Deleting a file

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    fs::remove_file("file_to_delete.txt")?;
    println!("File deleted successfully");
    Ok(())
}
```

The `fs::remove_file` function permanently deletes a file from the  
filesystem. Use with caution as this operation cannot be undone.  

### Deleting a directory

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    // Remove empty directory
    fs::remove_dir("empty_directory")?;
    
    // Remove directory and all its contents
    fs::remove_dir_all("directory_with_contents")?;
    
    println!("Directories deleted successfully");
    Ok(())
}
```

Use `remove_dir` for empty directories and `remove_dir_all` for  
directories containing files and subdirectories.  

## Path manipulation

Path manipulation provides safe ways to work with file and directory  
paths across different operating systems.  

### Working with paths

```rust
use std::path::Path;

fn main() {
    let path = Path::new("/home/user/documents/file.txt");
    
    println!("File name: {:?}", path.file_name());
    println!("Parent directory: {:?}", path.parent());
    println!("Extension: {:?}", path.extension());
    println!("File stem: {:?}", path.file_stem());
    println!("Is absolute: {}", path.is_absolute());
    println!("Is relative: {}", path.is_relative());
}
```

The `Path` type provides methods to extract components and information  
from file paths in a platform-independent way.  

### Building paths

```rust
use std::path::PathBuf;

fn main() {
    let mut path = PathBuf::new();
    path.push("home");
    path.push("user");
    path.push("documents");
    path.push("file.txt");
    
    println!("Built path: {}", path.display());
    
    // Alternative using join
    let path2 = PathBuf::from("home").join("user").join("file.txt");
    println!("Joined path: {}", path2.display());
}
```

`PathBuf` is the owned version of `Path` that allows building and  
modifying paths programmatically.  

### Getting current directory and absolute paths

```rust
use std::env;
use std::path::PathBuf;

fn main() -> std::io::Result<()> {
    let current_dir = env::current_dir()?;
    println!("Current directory: {}", current_dir.display());
    
    let relative_path = PathBuf::from("relative_file.txt");
    let absolute_path = current_dir.join(relative_path);
    println!("Absolute path: {}", absolute_path.display());
    
    // Canonicalize resolves symlinks and relative components
    if let Ok(canonical) = absolute_path.canonicalize() {
        println!("Canonical path: {}", canonical.display());
    }
    
    Ok(())
}
```

Use `env::current_dir()` to get the current working directory and  
`canonicalize()` to resolve symlinks and relative path components.  

## Binary file I/O

Binary file operations handle raw bytes without text encoding  
assumptions, useful for images, executables, and custom formats.  

### Reading binary data

```rust
use std::fs::File;
use std::io::{Read, Result};

fn main() -> Result<()> {
    let mut file = File::open("binary_file.dat")?;
    let mut buffer = Vec::new();
    
    // Read entire file into buffer
    file.read_to_end(&mut buffer)?;
    
    println!("Read {} bytes", buffer.len());
    println!("First 10 bytes: {:?}", &buffer[..10.min(buffer.len())]);
    
    Ok(())
}
```

The `read_to_end` method reads all bytes from a file into a vector,  
making it suitable for binary data of unknown size.  

### Writing binary data

```rust
use std::fs::File;
use std::io::{Write, Result};

fn main() -> Result<()> {
    let mut file = File::create("output.dat")?;
    
    // Write raw bytes
    let data = vec![0x48, 0x65, 0x6C, 0x6C, 0x6F]; // "Hello" in bytes
    file.write_all(&data)?;
    
    // Write integer as bytes
    let number: u32 = 42;
    file.write_all(&number.to_le_bytes())?;
    
    println!("Binary data written successfully");
    Ok(())
}
```

Use `write_all` to write byte slices and convert numbers to bytes using  
methods like `to_le_bytes()` for little-endian byte order.  

### Reading structured binary data

```rust
use std::fs::File;
use std::io::{Read, Result};

fn main() -> Result<()> {
    let mut file = File::open("structured.dat")?;
    
    // Read a 32-bit integer
    let mut int_buffer = [0u8; 4];
    file.read_exact(&mut int_buffer)?;
    let value = u32::from_le_bytes(int_buffer);
    println!("Read integer: {}", value);
    
    // Read a string length and then the string
    let mut len_buffer = [0u8; 4];
    file.read_exact(&mut len_buffer)?;
    let str_len = u32::from_le_bytes(len_buffer) as usize;
    
    let mut str_buffer = vec![0u8; str_len];
    file.read_exact(&mut str_buffer)?;
    let text = String::from_utf8(str_buffer).expect("Invalid UTF-8");
    println!("Read string: {}", text);
    
    Ok(())
}
```

Use `read_exact` to read a specific number of bytes and convert bytes  
back to native types using `from_le_bytes()` and similar methods.  

## CSV file handling

CSV (Comma-Separated Values) file handling requires the `csv` crate  
for robust parsing and writing capabilities.  

### Reading CSV files

```rust
// Add to Cargo.toml: csv = "1.3"
use csv::Reader;
use std::error::Error;

fn main() -> Result<(), Box<dyn Error>> {
    let mut reader = Reader::from_path("data.csv")?;
    
    // Read headers
    let headers = reader.headers()?.clone();
    println!("Headers: {:?}", headers);
    
    // Read records
    for result in reader.records() {
        let record = result?;
        println!("Record: {:?}", record);
    }
    
    Ok(())
}
```

The csv crate provides robust CSV parsing with header support and  
automatic field parsing. Add `csv = "1.3"` to your Cargo.toml.  

### Writing CSV files

```rust
// Add to Cargo.toml: csv = "1.3"
use csv::Writer;
use std::error::Error;

fn main() -> Result<(), Box<dyn Error>> {
    let mut writer = Writer::from_path("output.csv")?;
    
    // Write headers
    writer.write_record(&["Name", "Age", "City"])?;
    
    // Write data records
    writer.write_record(&["Alice", "30", "New York"])?;
    writer.write_record(&["Bob", "25", "Los Angeles"])?;
    writer.write_record(&["Charlie", "35", "Chicago"])?;
    
    writer.flush()?;
    println!("CSV file written successfully");
    
    Ok(())
}
```

Use `Writer` to create CSV files with proper escaping and formatting.  
Call `flush()` to ensure all data is written to disk.  

### Reading CSV with deserialization

```rust
// Add to Cargo.toml: csv = "1.3", serde = { version = "1.0", features = ["derive"] }
use csv::Reader;
use serde::Deserialize;
use std::error::Error;

#[derive(Debug, Deserialize)]
struct Person {
    name: String,
    age: u32,
    city: String,
}

fn main() -> Result<(), Box<dyn Error>> {
    let mut reader = Reader::from_path("people.csv")?;
    
    for result in reader.deserialize() {
        let person: Person = result?;
        println!("{:?}", person);
    }
    
    Ok(())
}
```

Combine csv with serde to automatically deserialize CSV records into  
Rust structs, making data handling more type-safe and convenient.  

## JSON file handling

JSON file handling uses the `serde_json` crate for serialization and  
deserialization of JSON data structures.  

### Reading JSON files

```rust
// Add to Cargo.toml: serde_json = "1.0"
use serde_json::Value;
use std::fs;
use std::error::Error;

fn main() -> Result<(), Box<dyn Error>> {
    let content = fs::read_to_string("data.json")?;
    let json: Value = serde_json::from_str(&content)?;
    
    println!("JSON data: {:#}", json);
    
    // Access specific fields
    if let Some(name) = json["name"].as_str() {
        println!("Name: {}", name);
    }
    
    Ok(())
}
```

Use `serde_json::Value` for dynamic JSON parsing when the structure  
is unknown, or deserialize into specific types for known structures.  

### Writing JSON files

```rust
// Add to Cargo.toml: serde_json = "1.0"
use serde_json::{json, Value};
use std::fs;
use std::error::Error;

fn main() -> Result<(), Box<dyn Error>> {
    let data = json!({
        "name": "Alice",
        "age": 30,
        "hobbies": ["reading", "swimming", "coding"],
        "address": {
            "street": "123 Main St",
            "city": "Anytown"
        }
    });
    
    let json_string = serde_json::to_string_pretty(&data)?;
    fs::write("output.json", json_string)?;
    
    println!("JSON file written successfully");
    Ok(())
}
```

The `json!` macro creates JSON values easily, and `to_string_pretty`  
formats the output with proper indentation for readability.  

### JSON with structs

```rust
// Add to Cargo.toml: serde_json = "1.0", serde = { version = "1.0", features = ["derive"] }
use serde::{Deserialize, Serialize};
use std::fs;
use std::error::Error;

#[derive(Serialize, Deserialize, Debug)]
struct Person {
    name: String,
    age: u32,
    email: Option<String>,
}

fn main() -> Result<(), Box<dyn Error>> {
    // Create and serialize
    let person = Person {
        name: "Bob".to_string(),
        age: 25,
        email: Some("bob@example.com".to_string()),
    };
    
    let json = serde_json::to_string_pretty(&person)?;
    fs::write("person.json", &json)?;
    
    // Read and deserialize
    let content = fs::read_to_string("person.json")?;
    let loaded_person: Person = serde_json::from_str(&content)?;
    
    println!("Loaded person: {:?}", loaded_person);
    Ok(())
}
```

Using serde's derive macros provides automatic serialization and  
deserialization for custom structs, ensuring type safety.  

## Temporary files

Temporary files are useful for intermediate data storage and testing  
without cluttering the filesystem permanently.  

### Creating temporary files

```rust
// Add to Cargo.toml: tempfile = "3.8"
use tempfile::NamedTempFile;
use std::io::{Write, Read, Seek, SeekFrom};
use std::error::Error;

fn main() -> Result<(), Box<dyn Error>> {
    let mut temp_file = NamedTempFile::new()?;
    
    // Write data
    writeln!(temp_file, "This is temporary data")?;
    writeln!(temp_file, "Line 2")?;
    
    // Get the path
    println!("Temp file path: {:?}", temp_file.path());
    
    // Read back data
    temp_file.seek(SeekFrom::Start(0))?;
    let mut contents = String::new();
    temp_file.read_to_string(&mut contents)?;
    println!("Contents: {}", contents);
    
    // File is automatically deleted when temp_file goes out of scope
    Ok(())
}
```

`NamedTempFile` creates a temporary file with a unique name that is  
automatically deleted when the variable goes out of scope.  

### Creating temporary directories

```rust
// Add to Cargo.toml: tempfile = "3.8"
use tempfile::tempdir;
use std::fs::File;
use std::io::Write;
use std::error::Error;

fn main() -> Result<(), Box<dyn Error>> {
    let temp_dir = tempdir()?;
    println!("Temp directory: {:?}", temp_dir.path());
    
    // Create files in temp directory
    let file_path = temp_dir.path().join("temp_file.txt");
    let mut file = File::create(&file_path)?;
    writeln!(file, "Temporary file content")?;
    
    println!("Created file: {:?}", file_path);
    
    // Directory and all contents are deleted when temp_dir goes out of scope
    Ok(())
}
```

Use `tempdir()` to create temporary directories for testing or storing  
multiple temporary files that need to be organized.  

### Persistent temporary files

```rust
// Add to Cargo.toml: tempfile = "3.8"
use tempfile::NamedTempFile;
use std::io::Write;
use std::error::Error;

fn main() -> Result<(), Box<dyn Error>> {
    let mut temp_file = NamedTempFile::new()?;
    writeln!(temp_file, "This will persist")?;
    
    // Convert to persistent file
    let (file, path) = temp_file.keep()?;
    drop(file); // Close the file
    
    println!("File persisted at: {:?}", path);
    println!("Remember to clean up manually!");
    
    // You must manually delete the file later
    // std::fs::remove_file(path)?;
    
    Ok(())
}
```

Use the `keep()` method to prevent automatic cleanup and make the  
temporary file persistent, but remember to clean it up manually.  

## Error handling patterns

Proper error handling is crucial for robust I/O operations that can  
fail due to missing files, permissions, or system issues.  

### Basic error handling

```rust
use std::fs;
use std::io;

fn read_file_safe(path: &str) -> Result<String, io::Error> {
    match fs::read_to_string(path) {
        Ok(content) => Ok(content),
        Err(error) => {
            eprintln!("Failed to read file '{}': {}", path, error);
            Err(error)
        }
    }
}

fn main() {
    match read_file_safe("example.txt") {
        Ok(content) => println!("File content: {}", content),
        Err(_) => println!("Could not read the file"),
    }
}
```

Always handle I/O operations that can fail using `Result` types and  
provide meaningful error messages for debugging.  

### Custom error types

```rust
use std::fs;
use std::fmt;
use std::error::Error;

#[derive(Debug)]
enum FileOperationError {
    NotFound(String),
    PermissionDenied(String),
    Other(String),
}

impl fmt::Display for FileOperationError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            FileOperationError::NotFound(file) => 
                write!(f, "File not found: {}", file),
            FileOperationError::PermissionDenied(file) => 
                write!(f, "Permission denied: {}", file),
            FileOperationError::Other(msg) => 
                write!(f, "File operation error: {}", msg),
        }
    }
}

impl Error for FileOperationError {}

fn read_file_with_context(path: &str) -> Result<String, FileOperationError> {
    fs::read_to_string(path).map_err(|error| {
        match error.kind() {
            std::io::ErrorKind::NotFound => 
                FileOperationError::NotFound(path.to_string()),
            std::io::ErrorKind::PermissionDenied => 
                FileOperationError::PermissionDenied(path.to_string()),
            _ => FileOperationError::Other(error.to_string()),
        }
    })
}

fn main() {
    match read_file_with_context("example.txt") {
        Ok(content) => println!("Content: {}", content),
        Err(error) => eprintln!("Error: {}", error),
    }
}
```

Create custom error types for better error categorization and handling  
specific to your application's needs.  

### Error chaining and context

```rust
// Add to Cargo.toml: anyhow = "1.0"
use anyhow::{Context, Result};
use std::fs;

fn process_config_file(path: &str) -> Result<()> {
    let content = fs::read_to_string(path)
        .with_context(|| format!("Failed to read config file: {}", path))?;
    
    if content.is_empty() {
        anyhow::bail!("Config file is empty");
    }
    
    // Process the content
    println!("Config loaded successfully");
    Ok(())
}

fn main() -> Result<()> {
    process_config_file("config.toml")
        .context("Failed to process configuration")?;
    
    Ok(())
}
```

Use libraries like `anyhow` for better error context and chaining,  
making debugging easier with detailed error traces.  

### Retry patterns for I/O operations

```rust
use std::fs;
use std::thread;
use std::time::Duration;

fn read_with_retry(path: &str, max_attempts: u32) -> std::io::Result<String> {
    for attempt in 1..=max_attempts {
        match fs::read_to_string(path) {
            Ok(content) => return Ok(content),
            Err(error) => {
                if attempt == max_attempts {
                    return Err(error);
                }
                
                eprintln!("Attempt {} failed: {}. Retrying...", attempt, error);
                thread::sleep(Duration::from_millis(100 * attempt as u64));
            }
        }
    }
    
    unreachable!()
}

fn main() {
    match read_with_retry("unstable_file.txt", 3) {
        Ok(content) => println!("Success: {}", content),
        Err(error) => eprintln!("Failed after retries: {}", error),
    }
}
```

Implement retry logic for operations that might fail temporarily due  
to system conditions like network issues or resource contention.  
