
# File I/O Examples

## Basic File Operations

### Reading a file

Reading entire file content into a String in one operation. The  
fs::read_to_string() function is the simplest way to read text files  
when you need all content at once.

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    let content = fs::read_to_string("example.txt")?;
    println!("File content: {}", content);
    Ok(())
}
```

This example demonstrates the most straightforward file reading approach.  
The fs::read_to_string() function reads the entire file into memory as a  
UTF-8 String, making it ideal for configuration files or small text  
files. The ? operator propagates any I/O errors up to the caller.

Alternative approach with explicit error handling using pattern matching.  
This provides more control over error handling compared to the ? operator,  
allowing custom responses to different types of failures.

```rust
use std::fs;

fn main() {
    match fs::read_to_string("maybe_missing.txt") {
        Ok(content) => println!("File content: {}", content),
        Err(e) => eprintln!("Error reading file: {}", e),
    }
}
```

The match expression handles both success and failure cases explicitly.  
This pattern is useful when you want to continue program execution even  
if file reading fails, or when you need different error handling logic  
for different error types.


### Reading a file line by line

Memory-efficient file processing for large files. BufReader provides  
buffered reading which improves performance and allows processing files  
line by line without loading everything into memory.

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

This approach is ideal for processing large files or when you need to  
handle each line individually. BufReader wraps the File handle and  
provides efficient buffered I/O. The lines() iterator yields Result<String>  
for each line, requiring error handling with the ? operator.
### Reading a file by buffer

Low-level reading with explicit buffer management. This approach gives  
you control over buffer size and allows processing binary data or  
implementing custom reading logic.

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

This method provides maximum control over the reading process. The buffer  
size can be tuned for performance, and the loop continues until read()  
returns 0, indicating end of file. This approach works with both text  
and binary files, processing data in chunks.

### Writing to a file

Creating and writing content to a new file. File::create() overwrites  
existing files or creates new ones, providing a simple way to output  
text or binary data.

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

File::create() creates a new file or truncates an existing one to zero  
length. The write_all() method ensures all data is written, which is  
more reliable than write() for complete data output. Both byte strings  
(b"...") and regular strings can be written using as_bytes().

### Appending to a file

Adding content to existing files without overwriting. OpenOptions  
provides fine-grained control over file opening behavior, including  
append mode and automatic file creation.

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

OpenOptions builder pattern allows precise control over file opening  
behavior. The append(true) flag positions writes at the end of file,  
create(true) makes new files if they don't exist, preserving existing  
content when files already exist.

### Getting file information

Retrieving file metadata including size, type, and timestamps. File  
metadata provides essential information for file management and  
validation operations.

```rust
use std::fs;
use chrono::{DateTime, Utc};

fn main() -> std::io::Result<()> {
    let metadata = fs::metadata("data.txt")?;
    
    println!("File size: {} bytes", metadata.len());
    println!("Is file: {}", metadata.is_file());
    println!("Is directory: {}", metadata.is_dir());
    
    if let Ok(created) = metadata.created() {
        let created_datetime: DateTime<Utc> = created.into();
        println!("Created: {}", created_datetime.format("%Y-%m-%d %H:%M:%S UTC"));
    }
    
    if let Ok(modified) = metadata.modified() {
        let modified_datetime: DateTime<Utc> = modified.into();
        println!("Modified: {}", modified_datetime.format("%Y-%m-%d %H:%M:%S UTC"));
    }
    
    if let Ok(accessed) = metadata.accessed() {
        let accessed_datetime: DateTime<Utc> = accessed.into();
        println!("Accessed: {}", accessed_datetime.format("%Y-%m-%d %H:%M:%S UTC"));
    }
    
    Ok(())
}
```

The metadata() function returns comprehensive file information including  
size, file type flags, and system timestamps. Timestamp availability  
depends on the filesystem and platform. The chrono crate helps format  
timestamps in human-readable form.

## File Management

### Creating a file

Simple file creation using File::create(). This is the most basic way  
to create new files, automatically overwriting existing ones.

```rust
use std::fs::File;

fn main() -> std::io::Result<()> {
    let _file = File::create("new_file.txt")?;
    println!("File created successfully");
    Ok(())
}
```

File::create() creates an empty file or truncates an existing file to  
zero length. The file handle is automatically closed when the variable  
goes out of scope, ensuring proper resource cleanup.

### Creating a file while checking if it exists

Safe file creation that respects existing files. This pattern prevents  
accidental overwrites by checking file existence before creation.

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

Path::new().exists() checks for file existence without attempting to  
open it. This approach is safer for preserving existing files and  
provides feedback about whether a new file was actually created.

## Reading Environment Variables

Accessing system environment variables for configuration and system  
information. Environment variables provide a standard way to pass  
configuration data to programs.

```rust
use std::env;

fn main() {
    if let Ok(path) = env::var("PATH") {
        println!("PATH: {}", path);
    } else {
        println!("PATH variable not set");
    }
}
```

The env::var() function returns a Result<String, VarError> allowing  
graceful handling of missing variables. This pattern is essential for  
reading configuration values, system paths, and user preferences.

## Directory Operations

### Creating a directory

Basic directory creation with fs::create_dir(). This function creates  
a single directory level and fails if parent directories don't exist.

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    fs::create_dir("new_directory")?;
    println!("Directory created successfully");
    Ok(())
}
```

fs::create_dir() creates only one directory level. If you need to create  
multiple directory levels at once, use fs::create_dir_all() instead.  
The function fails if the directory already exists.
### Listing a directory

Reading directory contents to enumerate files and subdirectories.  
The fs::read_dir() function provides an iterator over directory entries.

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

Each directory entry requires error handling as the iterator yields  
Result values. The path.file_name() method extracts just the filename  
without the directory path. The to_string_lossy() handles potential  
Unicode conversion issues safely.
### Listing directory recursively

Walking directory trees to list all files and subdirectories at any  
depth. This example demonstrates recursive directory traversal with  
proper indentation for visualization.

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

The recursive function tracks depth for indentation and calls itself for  
subdirectories. The is_dir() check distinguishes between files and  
directories. This pattern is useful for file system analysis and  
backup operations.

## Console I/O

### Reading from stdin

This example shows how to read user input with proper newline handling  
and trimming. The stdin().read_line() method preserves the newline  
character, so we use trim() to remove it for clean processing.

```rust
use std::io;

fn main() -> std::io::Result<()> {
    println!("Enter your name:");
    
    let mut input = String::new();
    io::stdin().read_line(&mut input)?;
    
    let name = input.trim();
    println!("Hello, {}!", name);
    
    Ok(())
}
```

### Reading multiple lines from stdin

This example demonstrates reading continuous input until EOF. It's useful  
for processing data piped from other programs or when users want to enter  
multiple lines of text before terminating with Ctrl+D (Unix) or Ctrl+Z  
(Windows).

```rust
use std::io;

fn main() -> std::io::Result<()> {
    println!("Enter multiple lines (Ctrl+D/Ctrl+Z to finish):");
    
    let mut lines = Vec::new();
    
    loop {
        let mut input = String::new();
        match io::stdin().read_line(&mut input) {
            Ok(0) => break, // EOF reached
            Ok(_) => lines.push(input.trim().to_string()),
            Err(e) => return Err(e),
        }
    }
    
    println!("You entered {} lines:", lines.len());
    for (i, line) in lines.iter().enumerate() {
        println!("{}: {}", i + 1, line);
    }
    
    Ok(())
}
```

### Writing to stdout and stderr

This example covers both macro shortcuts and explicit stream writing.  
Writing to stderr is important for error messages and logging, while  
stdout is for normal program output. This separation allows for proper  
output redirection in shell commands.

```rust
use std::io::{self, Write};

fn main() -> std::io::Result<()> {
    // Using println! macro (writes to stdout)
    println!("This goes to stdout");
    
    // Using eprintln! macro (writes to stderr)
    eprintln!("This goes to stderr");
    
    // Explicit writing to stdout
    let mut stdout = io::stdout();
    stdout.write_all(b"Explicit stdout write\n")?;
    stdout.flush()?;
    
    // Explicit writing to stderr
    let mut stderr = io::stderr();
    stderr.write_all(b"Explicit stderr write\n")?;
    stderr.flush()?;
    
    Ok(())
}
```

## Advanced File Operations

### Copying files

Using fs::copy() for simple file duplication. This function preserves  
the file contents but not necessarily the metadata like permissions or  
timestamps. It's the most straightforward way to copy files in Rust.

```rust
use std::fs;

fn main() -> std::io::Result<()> {

    let source = "source.txt";
    let destination = "copy.txt";
    
    let bytes_copied = fs::copy(source, destination)?;
    println!("Copied {} bytes from {} to {}", bytes_copied, source, destination);
    
    fs::remove_file("file_to_delete.txt")?;
    println!("File deleted successfully");
    
    Ok(())
}
```

### Moving and renaming files

File system operations with fs::rename(). This function can both move  
files to different directories and rename them within the same directory.  
On most systems, this is an atomic operation when source and destination  
are on the same filesystem.

```rust
use std::fs;

fn main() -> std::io::Result<()> {

    // Rename a file in the same directory
    fs::rename("old_name.txt", "new_name.txt")?;
    println!("File renamed successfully");
    
    // Move a file to a different directory
    fs::rename("file.txt", "backup/file.txt")?;
    println!("File moved to backup directory");
    
    fs::rename("old_name.txt", "new_name.txt")?;
    println!("File renamed or moved successfully");
    Ok(())
}
```

### Deleting files and directories

Safe removal of files and directory trees. This example shows how to  
remove individual files and how to recursively remove directories with  
all their contents. Always be careful with remove_dir_all() as it's  
irreversible.

```rust
use std::fs;

fn main() -> std::io::Result<()> {

    // Remove a single file
    fs::remove_file("unwanted_file.txt")?;
    println!("File deleted successfully");
    
    // Remove an empty directory
    fs::remove_dir("empty_directory")?;
    println!("Empty directory removed");
    
    // Remove a directory and all its contents (be careful!)
    fs::remove_dir_all("directory_to_delete")?;
    println!("Directory tree removed");
    
    Ok(())
}
```

## Path Manipulation

### Working with paths

Comprehensive Path and PathBuf usage examples. Path is borrowed and  
PathBuf is owned, similar to str and String. This example shows common  
operations for working with filesystem paths in a cross-platform way.

```rust
use std::path::{Path, PathBuf};

fn main() -> std::io::Result<()> {
    // Creating paths
    let path = Path::new("/home/user/documents/file.txt");
    let mut path_buf = PathBuf::from("/home/user");
    
    // PathBuf operations (owned, can be modified)
    path_buf.push("documents");
    path_buf.push("file.txt");
    
    // Path operations (borrowed, read-only)
    println!("Path: {}", path.display());
    println!("PathBuf: {}", path_buf.display());
    println!("Are they equal? {}", path == path_buf);
    
    // Converting between Path and PathBuf
    let path_from_buf: &Path = &path_buf;
    let buf_from_path: PathBuf = path.to_path_buf();
    
    println!("Converted successfully");
    
    Ok(())
}
```

### Building paths dynamically

Cross-platform path construction patterns. This example demonstrates  
how to build paths programmatically while maintaining compatibility  
across different operating systems with different path separators.

```rust
use std::path::PathBuf;

fn main() -> std::io::Result<()> {
    // Building paths component by component
    let mut path = PathBuf::new();
    path.push("usr");
    path.push("local");
    path.push("bin");
    path.push("program");
    
    println!("Built path: {}", path.display());
    
    // Using join() method for cleaner syntax
    let base_path = PathBuf::from("/var/log");
    let log_file = base_path.join("application").join("debug.log");
    
    println!("Log file path: {}", log_file.display());
    
    // Building from components vector
    let components = vec!["home", "user", "projects", "rust"];
    let project_path = components.iter().collect::<PathBuf>();
    
    println!("Project path: {}", project_path.display());
    
    Ok(())
}
```

### Path component extraction

Getting filenames, extensions, and parent directories. This example  
shows how to decompose paths into their constituent parts, which is  
useful for file processing and validation operations.

```rust
use std::path::Path;

fn main() -> std::io::Result<()> {
    let path = Path::new("/home/user/documents/report.pdf");
    
    // Extract different components
    if let Some(file_name) = path.file_name() {
        println!("File name: {}", file_name.to_string_lossy());
    }
    
    if let Some(file_stem) = path.file_stem() {
        println!("File stem: {}", file_stem.to_string_lossy());
    }
    
    if let Some(extension) = path.extension() {
        println!("Extension: {}", extension.to_string_lossy());
    }
    
    if let Some(parent) = path.parent() {
        println!("Parent directory: {}", parent.display());
    }
    
    // Iterate through components
    println!("Path components:");
    for component in path.components() {
        println!("  {:?}", component);
    }
    
    Ok(())
}
```

## Binary and Advanced I/O

### Binary file I/O

Reading and writing raw bytes with proper handling. This example shows  
how to work with binary data, which is essential for images, executables,  
or any non-text file format. Binary mode ensures no character encoding  
transformations occur.

```rust
use std::fs::File;
use std::io::{Read, Write};

fn main() -> std::io::Result<()> {
    // Writing binary data
    let binary_data: Vec<u8> = vec![0x48, 0x65, 0x6C, 0x6C, 0x6F]; // "Hello" in ASCII
    
    let mut output_file = File::create("binary_file.bin")?;
    output_file.write_all(&binary_data)?;
    
    // Reading binary data
    let mut input_file = File::open("binary_file.bin")?;
    let mut buffer = Vec::new();
    input_file.read_to_end(&mut buffer)?;
    
    println!("Read {} bytes", buffer.len());
    println!("Data: {:?}", buffer);
    
    // Converting to string if it's valid UTF-8
    if let Ok(text) = String::from_utf8(buffer) {
        println!("As text: {}", text);
    }
    
    Ok(())
}
```

### Working with temporary files

Working with system temp directories and cleanup. Temporary files are  
useful for intermediate processing and are typically cleaned up  
automatically. This example shows both manual and automatic cleanup  
approaches.

```rust
use std::env;
use std::fs::File;
use std::io::Write;
use std::path::PathBuf;

fn main() -> std::io::Result<()> {
    // Get system temporary directory
    let temp_dir = env::temp_dir();
    println!("System temp directory: {}", temp_dir.display());
    
    // Create a temporary file path
    let temp_file_path = temp_dir.join("rust_temp_file.txt");
    
    // Create and write to temporary file
    {
        let mut temp_file = File::create(&temp_file_path)?;
        temp_file.write_all(b"This is temporary data\n")?;
        println!("Temporary file created: {}", temp_file_path.display());
    } // File handle is dropped here
    
    // Read the temporary file
    let content = std::fs::read_to_string(&temp_file_path)?;
    println!("Temporary file content: {}", content);
    
    // Manual cleanup
    std::fs::remove_file(&temp_file_path)?;
    println!("Temporary file cleaned up");
    
    fs::copy("source.txt", "destination.txt")?;
    println!("File copied successfully");
    Ok(())
}
```

   

### Large file handling with seeking

Efficient streaming and seeking for big files. This example shows how  
to work with large files without loading them entirely into memory,  
using seeking to jump to specific positions for random access.

```rust
use std::fs::File;
use std::io::{Read, Seek, SeekFrom, Write};

fn main() -> std::io::Result<()> {
    let file_path = "large_file_test.txt";
    
    // Create a test file with some content
    {
        let mut file = File::create(file_path)?;
        for i in 0..1000 {
            writeln!(file, "Line {}: This is some sample content for testing", i)?;
        }
    }
    
    // Open file for random access
    let mut file = File::open(file_path)?;
    
    // Get file size
    let file_size = file.metadata()?.len();
    println!("File size: {} bytes", file_size);
    
    // Seek to the middle of the file
    let middle_position = file_size / 2;
    file.seek(SeekFrom::Start(middle_position))?;
    
    // Read a small chunk from the middle
    let mut buffer = vec![0; 100];
    let bytes_read = file.read(&mut buffer)?;
    let content = String::from_utf8_lossy(&buffer[..bytes_read]);
    println!("Content from middle: {}", content.trim());
    
    // Seek to end and then back 200 bytes
    file.seek(SeekFrom::End(-200))?;
    buffer.clear();
    buffer.resize(200, 0);
    let bytes_read = file.read(&mut buffer)?;
    let end_content = String::from_utf8_lossy(&buffer[..bytes_read]);
    println!("Content near end: {}", end_content.trim());
    
    // Cleanup
    std::fs::remove_file(file_path)?;
    

### File locking

Preventing concurrent access issues. This example demonstrates file  
locking to prevent multiple processes from modifying the same file  
simultaneously. Note that file locking behavior varies between  
operating systems.

```rust
use std::fs::File;
use std::io::Write;

// Note: For production use, consider using the `fs2` crate for better cross-platform support
fn main() -> std::io::Result<()> {
    let file_path = "locked_file.txt";
    
    // Open file for writing
    let mut file = File::create(file_path)?;
    
    println!("File opened, attempting to write with exclusive access...");
    
    // Simulate exclusive file access
    // In a real scenario, you'd use platform-specific locking
    #[cfg(unix)]
    {
        use std::os::unix::io::AsRawFd;
        
        let fd = file.as_raw_fd();
        // In production, use flock() system call here
        println!("File descriptor: {}", fd);
    }
    
    // Write data while "locked"
    file.write_all(b"This content was written with exclusive access\n")?;
    file.flush()?;
    
    println!("Data written successfully");
    
    // File is automatically "unlocked" when dropped
    drop(file);
    
    // Verify content
    let content = std::fs::read_to_string(file_path)?;
    println!("File content: {}", content);
    
    // Cleanup
    std::fs::remove_file(file_path)?;
    
    Ok(())
}
```

## Structured Data

### CSV file handling

Complete read/write examples using the csv crate. CSV (Comma-Separated  
Values) is a common format for data exchange. This example shows how to  
read and write CSV files with proper header handling and type conversion.

```rust
// Add to Cargo.toml: csv = "1.1", serde = { version = "1.0", features = ["derive"] }
use csv::{Reader, Writer};
use serde::{Deserialize, Serialize};
use std::error::Error;

#[derive(Debug, Deserialize, Serialize)]
struct Person {
    name: String,
    age: u32,
    city: String,
}

fn main() -> Result<(), Box<dyn Error>> {
    let csv_file = "people.csv";
    
    // Write CSV data
    let people = vec![
        Person { name: "Alice".to_string(), age: 30, city: "New York".to_string() },
        Person { name: "Bob".to_string(), age: 25, city: "Los Angeles".to_string() },
        Person { name: "Charlie".to_string(), age: 35, city: "Chicago".to_string() },
    ];
    
    let mut writer = Writer::from_path(csv_file)?;
    for person in &people {
        writer.serialize(person)?;
    }
    writer.flush()?;
    
    println!("CSV file written successfully");
    
    // Read CSV data
    let mut reader = Reader::from_path(csv_file)?;
    println!("Reading CSV data:");
    
    for result in reader.deserialize() {
        let person: Person = result?;
        println!("{:?}", person);
    }
    
    // Cleanup
    std::fs::remove_file(csv_file)?;
    
    Ok(())
}
```

### JSON file handling

Serialization/deserialization with serde_json. JSON is widely used for  
configuration files and data exchange. This example demonstrates reading  
and writing structured data to JSON files with proper error handling.

```rust
// Add to Cargo.toml: serde_json = "1.0", serde = { version = "1.0", features = ["derive"] }
use serde::{Deserialize, Serialize};
use std::fs::File;
use std::io::{BufReader, BufWriter};

#[derive(Debug, Deserialize, Serialize)]
struct Config {
    database_url: String,
    port: u16,
    debug: bool,
    features: Vec<String>,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let json_file = "config.json";
    
    // Create sample configuration
    let config = Config {
        database_url: "postgresql://localhost/myapp".to_string(),
        port: 8080,
        debug: true,
        features: vec!["auth".to_string(), "logging".to_string(), "metrics".to_string()],
    };
    
    // Write JSON to file
    let file = File::create(json_file)?;
    let writer = BufWriter::new(file);
    serde_json::to_writer_pretty(writer, &config)?;
    
    println!("JSON configuration written successfully");
    
    // Read JSON from file
    let file = File::open(json_file)?;
    let reader = BufReader::new(file);
    let loaded_config: Config = serde_json::from_reader(reader)?;
    
    println!("Loaded configuration: {:#?}", loaded_config);
    
    // Verify data integrity
    assert_eq!(config.database_url, loaded_config.database_url);
    assert_eq!(config.port, loaded_config.port);
    println!("Data integrity verified");
    
    // Cleanup
    std::fs::remove_file(json_file)?;
    
    Ok(())
}
```

## Error Handling and Utilities

### Custom error types

Custom error types and graceful failure handling. This example shows  
how to create application-specific error types that provide better  
context and error handling than generic I/O errors.

```rust
use std::fmt;
use std::fs;
use std::io;

#[derive(Debug)]
enum FileProcessingError {
    IoError(io::Error),
    InvalidFormat(String),
    EmptyFile,
}

impl fmt::Display for FileProcessingError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            FileProcessingError::IoError(err) => write!(f, "I/O error: {}", err),
            FileProcessingError::InvalidFormat(msg) => write!(f, "Invalid format: {}", msg),
            FileProcessingError::EmptyFile => write!(f, "File is empty"),
        }
    }
}

impl std::error::Error for FileProcessingError {}

impl From<io::Error> for FileProcessingError {
    fn from(error: io::Error) -> Self {
        FileProcessingError::IoError(error)
    }
}

fn process_file(path: &str) -> Result<Vec<String>, FileProcessingError> {
    let content = fs::read_to_string(path)?;
    
    if content.trim().is_empty() {
        return Err(FileProcessingError::EmptyFile);
    }
    
    let lines: Vec<String> = content
        .lines()
        .map(|line| line.trim().to_string())
        .filter(|line| !line.is_empty())
        .collect();
    
    if lines.is_empty() {
        return Err(FileProcessingError::InvalidFormat("No valid lines found".to_string()));
    }
    
    Ok(lines)
}

fn main() {
    // Create test files
    fs::write("valid.txt", "line1\nline2\nline3\n").unwrap();
    fs::write("empty.txt", "").unwrap();
    fs::write("whitespace.txt", "   \n\n   \n").unwrap();
    
    // Test with different files
    let test_files = ["valid.txt", "empty.txt", "whitespace.txt", "nonexistent.txt"];
    
    for file in &test_files {
        match process_file(file) {
            Ok(lines) => println!("{}: Successfully processed {} lines", file, lines.len()),
            Err(e) => println!("{}: Error - {}", file, e),
        }
    }
    
    // Cleanup
    let _ = fs::remove_file("valid.txt");
    let _ = fs::remove_file("empty.txt");
    let _ = fs::remove_file("whitespace.txt");
}
```

### File existence and validation

Robust file type and existence validation. This example demonstrates  
comprehensive file system checks including existence, type validation,  
and accessibility testing before performing operations.

```rust
use std::fs;
use std::path::Path;

fn validate_file(path: &str) -> std::io::Result<()> {
    let path = Path::new(path);
    
    // Check if path exists
    if !path.exists() {
        return Err(std::io::Error::new(
            std::io::ErrorKind::NotFound,
            format!("Path does not exist: {}", path.display())
        ));
    }
    
    // Get metadata for detailed checks
    let metadata = fs::metadata(path)?;
    
    // Check file type
    if metadata.is_file() {
        println!("✓ {} is a regular file", path.display());
    } else if metadata.is_dir() {
        println!("✓ {} is a directory", path.display());
    } else {
        println!("✓ {} is a special file (symlink, device, etc.)", path.display());
    }
    
    // Check file size
    if metadata.is_file() {
        let size = metadata.len();
        println!("  File size: {} bytes", size);
        
        if size == 0 {
            println!("  Warning: File is empty");
        }
    }
    
    // Check permissions (basic check)
    let permissions = metadata.permissions();
    if permissions.readonly() {
        println!("  File is read-only");
    } else {
        println!("  File is writable");
    }
    
    Ok(())
}

fn main() -> std::io::Result<()> {
    // Create test files with different characteristics
    fs::write("regular_file.txt", "This is a regular file with content")?;
    fs::write("empty_file.txt", "")?;
    fs::create_dir_all("test_directory")?;
    
    let test_paths = [
        "regular_file.txt",
        "empty_file.txt", 
        "test_directory",
        "nonexistent_file.txt",
        ".",  // Current directory
    ];
    
    for path in &test_paths {
        println!("\nValidating: {}", path);
        match validate_file(path) {
            Ok(()) => println!("Validation successful"),
            Err(e) => println!("Validation failed: {}", e),
        }
    }
    
    // Cleanup
    fs::remove_file("regular_file.txt")?;
    fs::remove_file("empty_file.txt")?;
    fs::remove_dir("test_directory")?;
    
    Ok(())
}
```

### Symlink operations (Unix-specific)

Creating and reading symbolic links on Unix systems. Symbolic links  
are filesystem entries that point to other files or directories. This  
example shows how to create, read, and work with symlinks safely.

```rust
#[cfg(unix)]
use std::os::unix::fs as unix_fs;
use std::fs;
use std::path::Path;

fn main() -> std::io::Result<()> {
    let target_file = "target_file.txt";
    let symlink_path = "link_to_target.txt";
    
    // Create a target file
    fs::write(target_file, "This is the target file content")?;
    
    #[cfg(unix)]
    {
        // Create a symbolic link
        unix_fs::symlink(target_file, symlink_path)?;
        println!("Symbolic link created: {} -> {}", symlink_path, target_file);
        
        // Check if path is a symlink
        let symlink_metadata = fs::symlink_metadata(symlink_path)?;
        if symlink_metadata.file_type().is_symlink() {
            println!("✓ {} is indeed a symbolic link", symlink_path);
            
            // Read the symlink target
            let link_target = fs::read_link(symlink_path)?;
            println!("Link target: {}", link_target.display());
        }
        
        // Read content through the symlink
        let content = fs::read_to_string(symlink_path)?;
        println!("Content through symlink: {}", content);
        
        // Compare metadata: symlink vs target
        let target_metadata = fs::metadata(target_file)?;
        let symlink_resolved_metadata = fs::metadata(symlink_path)?; // Follows symlink
        
        println!("Target file size: {} bytes", target_metadata.len());
        println!("Symlink resolved size: {} bytes", symlink_resolved_metadata.len());
        
        // Cleanup
        fs::remove_file(symlink_path)?;
    }
    
    #[cfg(not(unix))]
    {
        println!("Symbolic link operations are Unix-specific");
        println!("On Windows, use std::os::windows::fs for junction points and symlinks");
    }
    
    fs::remove_file(target_file)?;
    
    Ok(())
}
```

## Changing File Permissions

```rust
use std::fs;
use std::os::unix::fs::PermissionsExt;

fn main() -> std::io::Result<()> {
    let metadata = fs::metadata("example.txt")?;
    let mut permissions = metadata.permissions();
    permissions.set_mode(0o644); // rw-r--r--
    fs::set_permissions("example.txt", permissions)?;
    println!("Permissions changed successfully");
    Ok(())
}
```

*Note: The permissions example is for Unix-like systems.*


