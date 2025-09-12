
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

## Deleting a File

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    fs::remove_file("file_to_delete.txt")?;
    println!("File deleted successfully");
    Ok(())
}
```

## Renaming or Moving a File

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    fs::rename("old_name.txt", "new_name.txt")?;
    println!("File renamed or moved successfully");
    Ok(())
}
```

## Copying a File

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    fs::copy("source.txt", "destination.txt")?;
    println!("File copied successfully");
    Ok(())
}
```

## Deleting a Directory

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    fs::remove_dir("directory_to_delete")?;
    println!("Directory deleted successfully");
    Ok(())
}
```

## Deleting a Directory Recursively

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    fs::remove_dir_all("directory_to_delete")?;
    println!("Directory and its contents deleted successfully");
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

## Reading a File with Error Handling

```rust
use std::fs;

fn main() {
    match fs::read_to_string("maybe_missing.txt") {
        Ok(content) => println!("File content: {}", content),
        Err(e) => eprintln!("Error reading file: {}", e),
    }
}
```

## Reading Environment Variables

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
