
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

## Reading from stdin

Reading user input from the standard input stream.  

```rust
use std::io;

fn main() -> io::Result<()> {
    println!("Enter your name:");
    
    let mut input = String::new();
    io::stdin().read_line(&mut input)?;
    
    // Remove the trailing newline
    let name = input.trim();
    println!("Hello, {}!", name);
    
    Ok(())
}
```

The `read_line` method reads a line from stdin and appends it to the provided  
string buffer. The method includes the newline character, so we use `trim()` to  
remove it.  

## Reading multiple lines from stdin

```rust
use std::io::{self, BufRead};

fn main() -> io::Result<()> {
    println!("Enter multiple lines (Ctrl+D to finish):");
    
    let stdin = io::stdin();
    for line in stdin.lock().lines() {
        let line = line?;
        if line.is_empty() {
            break;
        }
        println!("You entered: {}", line);
    }
    
    Ok(())
}
```

## Writing to stdout and stderr

Writing to standard output and standard error streams.  

```rust
use std::io::{self, Write};

fn main() -> io::Result<()> {
    // Write to stdout
    println!("This goes to stdout");
    writeln!(io::stdout(), "This also goes to stdout")?;
    
    // Write to stderr
    eprintln!("This goes to stderr");
    writeln!(io::stderr(), "This also goes to stderr")?;
    
    // Explicitly flush stdout
    io::stdout().flush()?;
    
    Ok(())
}
```

The `println!` and `eprintln!` macros are convenient for writing to stdout and  
stderr respectively. For more control, use `writeln!` with explicit streams.  

## Copying files

Copying files using the standard library.  

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    // Copy a file
    fs::copy("source.txt", "destination.txt")?;
    println!("File copied successfully");
    
    Ok(())
}
```

## Moving/renaming files

Moving or renaming files and directories.  

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    // Rename a file
    fs::rename("old_name.txt", "new_name.txt")?;
    println!("File renamed successfully");
    
    // Move a file to a different directory
    fs::rename("file.txt", "subdirectory/file.txt")?;
    println!("File moved successfully");
    
    Ok(())
}
```

## Deleting files and directories

Removing files and directories from the filesystem.  

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    // Remove a file
    fs::remove_file("unwanted_file.txt")?;
    println!("File deleted successfully");
    
    // Remove an empty directory
    fs::remove_dir("empty_directory")?;
    println!("Directory deleted successfully");
    
    // Remove a directory and all its contents
    fs::remove_dir_all("directory_with_files")?;
    println!("Directory and contents deleted successfully");
    
    Ok(())
}
```

## Working with paths

Manipulating file paths using PathBuf and Path.  

```rust
use std::path::{Path, PathBuf};

fn main() {
    // Create a path from string
    let path = Path::new("/home/user/document.txt");
    
    // Get components of a path
    if let Some(file_name) = path.file_name() {
        println!("File name: {:?}", file_name);
    }
    
    if let Some(parent) = path.parent() {
        println!("Parent directory: {:?}", parent);
    }
    
    if let Some(extension) = path.extension() {
        println!("File extension: {:?}", extension);
    }
    
    // Build paths dynamically
    let mut path_buf = PathBuf::new();
    path_buf.push("home");
    path_buf.push("user");
    path_buf.push("documents");
    path_buf.set_extension("txt");
    
    println!("Built path: {:?}", path_buf);
    
    // Join paths
    let base = Path::new("/home/user");
    let full_path = base.join("documents").join("file.txt");
    println!("Joined path: {:?}", full_path);
}
```

## Binary file I/O

Reading and writing binary data to files.  

```rust
use std::fs::File;
use std::io::{Read, Write};

fn main() -> std::io::Result<()> {
    // Write binary data
    let data: [u8; 4] = [0x48, 0x65, 0x6C, 0x6C]; // "Hell" in ASCII
    let mut file = File::create("binary_file.bin")?;
    file.write_all(&data)?;
    
    // Read binary data
    let mut file = File::open("binary_file.bin")?;
    let mut buffer = Vec::new();
    file.read_to_end(&mut buffer)?;
    
    println!("Read {} bytes: {:?}", buffer.len(), buffer);
    
    Ok(())
}
```

## Working with temporary files

Creating temporary files and directories.  

```rust
use std::fs::File;
use std::io::{Write, Read};
use std::env;

fn main() -> std::io::Result<()> {
    // Get system temporary directory
    let temp_dir = env::temp_dir();
    println!("Temp directory: {:?}", temp_dir);
    
    // Create a temporary file path
    let temp_file_path = temp_dir.join("my_temp_file.txt");
    
    // Write to temporary file
    let mut temp_file = File::create(&temp_file_path)?;
    temp_file.write_all(b"This is temporary data")?;
    
    // Read from temporary file
    let mut temp_file = File::open(&temp_file_path)?;
    let mut content = String::new();
    temp_file.read_to_string(&mut content)?;
    
    println!("Temp file content: {}", content);
    
    // Clean up temporary file
    std::fs::remove_file(&temp_file_path)?;
    println!("Temporary file cleaned up");
    
    Ok(())
}
```

## File permissions

Checking and modifying file permissions on Unix systems.  

```rust
use std::fs;
use std::os::unix::fs::PermissionsExt;

fn main() -> std::io::Result<()> {
    let file_path = "test_file.txt";
    
    // Create a test file
    fs::write(file_path, "test content")?;
    
    // Get current permissions
    let metadata = fs::metadata(file_path)?;
    let permissions = metadata.permissions();
    println!("Current permissions: {:o}", permissions.mode());
    
    // Modify permissions (make file read-only)
    let mut perms = permissions;
    perms.set_mode(0o444); // Read-only for all
    fs::set_permissions(file_path, perms)?;
    
    println!("Permissions changed to read-only");
    
    // Clean up
    // Note: Need to restore write permissions to delete
    let mut perms = fs::metadata(file_path)?.permissions();
    perms.set_mode(0o644);
    fs::set_permissions(file_path, perms)?;
    fs::remove_file(file_path)?;
    
    Ok(())
}
```

## CSV file handling

Reading and writing CSV files using the csv crate.  

```rust
// Add to Cargo.toml: csv = "1.1"
use csv::{Reader, Writer};
use std::error::Error;

#[derive(Debug)]
struct Person {
    name: String,
    age: u32,
    city: String,
}

fn main() -> Result<(), Box<dyn Error>> {
    // Write CSV data
    let mut writer = Writer::from_path("people.csv")?;
    writer.write_record(&["Name", "Age", "City"])?;
    writer.write_record(&["Alice", "30", "New York"])?;
    writer.write_record(&["Bob", "25", "San Francisco"])?;
    writer.flush()?;
    
    // Read CSV data
    let mut reader = Reader::from_path("people.csv")?;
    println!("CSV contents:");
    
    for result in reader.records() {
        let record = result?;
        println!("{:?}", record);
    }
    
    Ok(())
}
```

## JSON file handling

Reading and writing JSON files using the serde_json crate.  

```rust
// Add to Cargo.toml: serde = { version = "1.0", features = ["derive"] }
// serde_json = "1.0"
use serde::{Deserialize, Serialize};
use std::fs;

#[derive(Serialize, Deserialize, Debug)]
struct Config {
    name: String,
    version: String,
    features: Vec<String>,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Create sample data
    let config = Config {
        name: "My App".to_string(),
        version: "1.0.0".to_string(),
        features: vec!["feature1".to_string(), "feature2".to_string()],
    };
    
    // Write JSON to file
    let json_string = serde_json::to_string_pretty(&config)?;
    fs::write("config.json", json_string)?;
    println!("JSON written to file");
    
    // Read JSON from file
    let json_content = fs::read_to_string("config.json")?;
    let loaded_config: Config = serde_json::from_str(&json_content)?;
    
    println!("Loaded config: {:?}", loaded_config);
    
    Ok(())
}
```

## Error handling patterns for I/O

Different approaches to handle I/O errors gracefully.  

```rust
use std::fs;
use std::io::{self, ErrorKind};

fn read_file_with_custom_error(path: &str) -> Result<String, String> {
    match fs::read_to_string(path) {
        Ok(content) => Ok(content),
        Err(error) => match error.kind() {
            ErrorKind::NotFound => Err(format!("File '{}' not found", path)),
            ErrorKind::PermissionDenied => Err("Permission denied".to_string()),
            _ => Err(format!("Unknown error: {}", error)),
        },
    }
}

fn main() {
    // Using unwrap_or_else for default values
    let content = fs::read_to_string("config.txt")
        .unwrap_or_else(|_| "default config".to_string());
    println!("Config: {}", content);
    
    // Using custom error handling
    match read_file_with_custom_error("missing.txt") {
        Ok(content) => println!("File content: {}", content),
        Err(error) => eprintln!("Error: {}", error),
    }
}
```

## Checking file existence and type

Verifying file existence and determining file types.  

```rust
use std::path::Path;
use std::fs;

fn main() -> std::io::Result<()> {
    let path = Path::new("example.txt");
    
    // Check if path exists
    if path.exists() {
        println!("Path exists");
        
        // Check what type it is
        if path.is_file() {
            println!("It's a file");
            
            // Get file size
            let metadata = fs::metadata(path)?;
            println!("File size: {} bytes", metadata.len());
        } else if path.is_dir() {
            println!("It's a directory");
        } else {
            println!("It's something else (symlink, etc.)");
        }
    } else {
        println!("Path does not exist");
    }
    
    Ok(())
}
```

## Working with symlinks

Creating and reading symbolic links.  

```rust
use std::fs;
use std::os::unix::fs as unix_fs;

fn main() -> std::io::Result<()> {
    // Create a regular file
    fs::write("original.txt", "Original content")?;
    
    // Create a symbolic link
    unix_fs::symlink("original.txt", "link.txt")?;
    
    // Read through the symlink
    let content = fs::read_to_string("link.txt")?;
    println!("Content through symlink: {}", content);
    
    // Check if it's a symlink
    let metadata = fs::symlink_metadata("link.txt")?;
    if metadata.is_symlink() {
        // Read the link target
        let target = fs::read_link("link.txt")?;
        println!("Symlink points to: {:?}", target);
    }
    
    // Clean up
    fs::remove_file("link.txt")?;
    fs::remove_file("original.txt")?;
    
    Ok(())
}
```

## Reading large files efficiently

Strategies for reading large files without loading everything into memory.  

```rust
use std::fs::{self, File};
use std::io::{BufRead, BufReader, Seek, SeekFrom, Write};

fn main() -> std::io::Result<()> {
    // Create a large test file
    let mut file = File::create("large_file.txt")?;
    for i in 0..10000 {
        writeln!(file, "Line number {}", i)?;
    }
    
    // Read file chunk by chunk
    let file = File::open("large_file.txt")?;
    let mut reader = BufReader::new(file);
    let mut line_count = 0;
    
    // Process line by line without loading entire file
    for line in reader.lines() {
        let _line = line?;
        line_count += 1;
        
        // Process first 10 lines only for demo
        if line_count <= 10 {
            println!("Processed line {}", line_count);
        }
    }
    
    println!("Total lines processed: {}", line_count);
    
    // Seeking to specific position
    let mut file = File::open("large_file.txt")?;
    file.seek(SeekFrom::Start(100))?; // Skip first 100 bytes
    
    let mut reader = BufReader::new(file);
    let mut buffer = String::new();
    reader.read_line(&mut buffer)?;
    println!("Line after seeking: {}", buffer.trim());
    
    // Clean up
    fs::remove_file("large_file.txt")?;
    
    Ok(())
}
```

## File locking

Preventing concurrent access to files using file locking.  

```rust
use std::fs::{File, OpenOptions};
use std::io::{Write, Read};
use std::os::unix::fs::OpenOptionsExt;

fn main() -> std::io::Result<()> {
    // Create or open a file with exclusive access
    let mut file = OpenOptions::new()
        .create(true)
        .write(true)
        .read(true)
        .mode(0o644)
        .open("locked_file.txt")?;
    
    // Write some data
    file.write_all(b"This file is being accessed exclusively")?;
    
    // Simulate some work while file is locked
    println!("Working with locked file...");
    std::thread::sleep(std::time::Duration::from_secs(1));
    
    // Read the data back
    let mut content = String::new();
    file.read_to_string(&mut content)?;
    println!("File content: {}", content);
    
    // File is automatically unlocked when it goes out of scope
    drop(file);
    
    // Clean up
    std::fs::remove_file("locked_file.txt")?;
    
    Ok(())
}
```
