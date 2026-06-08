# Read/Write files

Rust's standard library provides comprehensive primitives for file I/O  
through the `std::fs` and `std::io` modules.  Operations are safe by  
default and make heavy use of `Result<T, E>` for explicit error handling.  
This article progresses from basic writes to advanced patterns.  


## Write text to a file

The simplest way to create a file and write bytes into it.  
`File::create` opens a file for writing only, truncating any existing  
content or creating it if it doesn't exist.  

    use std::{fs::File, io::Result, io::Write};

    fn main() -> Result<()> {
        let data = "The Battle of Thermopylae was fought between an alliance of
    Greek city-states, led by King Leonidas of Sparta, and the Persian Empire of
    Xerxes I over the course of three days, during the second Persian invasion of
    Greece.";

        let mut file = File::create("thermopylae.txt")?;
        file.write_all(data.as_bytes())?;
        Ok(())
    }

`write_all` attempts to write the entire buffer and returns an error if  
it cannot complete the write.  This is preferred over `write`, which may  
perform partial writes and requires a retry loop.  

The `?` is a postfix operator that unwraps `Result<T, E>` and `Option<T>`  
values.  With `Result<T, E>`, it unwraps the result and gives the inner  
value, propagating the error to the calling function.  It is a convenience  
syntax for reducing boilerplate error handling.  

The `?` operator works like this:  

- If it is a success, it unwraps the `Result` to get the success value  
  inside.  The value is then assigned to the variable file.  
- If the result is an error, the error is not assigned to the variable  
  file.  The `Error` is returned by the function to the caller.  

The `?` is syntactic sugar for:  

    let mut file = match File::create("foo.txt") {
        Err(why) => panic!("couldn't create {}: {}", display, why),
        Ok(file) => file,
    };

Note that `?` requires the enclosing function to return a compatible  
`Result` or `Option` type, or the error must implement `From` for the  
function's error type.  


## Read to string

`fs::read_to_string` opens a file, reads its entire content, and returns  
a `String`.  Suitable for smaller text (UTF-8) files.  For large files  
consider buffered or streaming approaches.  

    use std::fs;
    use std::io;

    fn main() -> io::Result<()> {
        let content = fs::read_to_string("thermopylae.txt")?;
        println!("{content}");
        Ok(())
    }

This is a convenience function that combines `File::open`, `read_to_end`,  
and `String::from_utf8`.  It will fail with an error if the file contains  
invalid UTF-8.  Use `fs::read` if you need raw bytes instead.  


## Read into vector

Read file into `Vec<u8>` vector.  Later transform the vector back to  
string using `String::from_utf8`.  This gives you full control over  
error handling when the file is not valid UTF-8.  

    use std::fs::File;
    use std::io::{Error, ErrorKind, Read, Result};

    fn main() -> Result<()> {
        let mut file = File::open("thermopylae.txt")?;
        let mut content = Vec::new();
        file.read_to_end(&mut content)?;

        println!("{:?}", content);

        let text = match String::from_utf8(content) {
            Ok(t) => t,
            Err(e) => return Err(Error::new(ErrorKind::InvalidData, e)),
        };

        println!("{text}");

        Ok(())
    }

`read_to_end` appends all bytes from the reader into the destination  
vector.  The vector grows dynamically, so there is no need to pre-allocate  
a fixed size.  The match on `String::from_utf8` lets you convert the UTF-8  
error into an I/O error with a custom `ErrorKind`.  

### With unwrap_or_else

Use `unwrap_or_else` to provide a custom panic message on invalid UTF-8.  

    use std::fs::File;
    use std::io::{Read, Result};

    fn main() -> Result<()> {
        let mut file = File::open("thermopylae.txt")?;
        let mut content: Vec<u8> = Vec::new();
        file.read_to_end(&mut content)?;

        println!("{:?}", content);

        let text = String::from_utf8(content).unwrap_or_else(|e| {
            panic!("Invalid UTF-8 sequence: {}", e);
        });

        println!("{text}");

        Ok(())
    }

`unwrap_or_else` consumes the `Result` and returns the inner `Ok` value.  
If it is `Err`, the closure is called, which in this case panics with a  
descriptive message.  Use this when invalid UTF-8 is truly unrecoverable.  


## Fluent operations

Rust has a Fluent API for working with files through `OpenOptions`.  
`File::options()` returns a builder that lets you chain configuration  
calls before opening the file.  

    use std::{fs::File, io::Read, io::Result};

    fn main() -> Result<()> {
        let path = "thermopylae.txt";

        let mut buf: [u8; 1024] = [0; 1024];

        let mut file = File::options().read(true).open(path)?;
        file.read(&mut buf)?;

        println!("{}", String::from_utf8_lossy(&buf));

        Ok(())
    }

`File::options()` is shorthand for `OpenOptions::new()`.  The builder  
pattern supports `.read(true)`, `.write(true)`, `.append(true)`,  
`.create(true)`, and `.truncate(true)`.  Chaining them lets you express  
open modes clearly without bitflags.  

`String::from_utf8_lossy` replaces invalid UTF-8 sequences with the  
Unicode replacement character (U+FFFD) instead of returning an error.  
Useful for displaying content when perfect UTF-8 fidelity isn't required,  
such as for debugging or logging.  


## Buffered reading

`BufReader` wraps a reader and adds an internal buffer, reducing the  
number of system calls when reading small chunks.  The `lines()` method  
returns an iterator over lines, each split at the newline character.  

    use std::fs::File;
    use std::io::{BufRead, BufReader, Result};

    fn main() -> Result<()> {
        let file = File::open("thermopylae.txt")?;
        let reader = BufReader::new(file);

        for line in reader.lines() {
            println!("{}", line?);
        }

        Ok(())
    }

Each call to `reader.lines()` allocates a new `String`.  The iterator  
yields `io::Result<String>`, so the `?` on `line?` propagates read errors  
up to `main`.  Newline characters are stripped from each line.  


## Buffered writing

`BufWriter` buffers writes in memory before flushing to the underlying  
writer, reducing system calls for many small writes.  Always call  
`flush()` or let it drop to ensure all data is written.  

    use std::fs::File;
    use std::io::{BufWriter, Result, Write};

    fn main() -> Result<()> {
        let file = File::create("log.txt")?;
        let mut writer = BufWriter::new(file);

        for i in 0..100 {
            writeln!(writer, "Line number {}", i)?;
        }
        writer.flush()?;
        Ok(())
    }

The `writeln!` macro works like `println!` but writes to any type that  
implements `Write`.  The buffer capacity defaults to 8 KiB and can be  
customized with `BufWriter::with_capacity(cap, file)`.  


## Appending to a file

Open a file in append mode with `OpenOptions`.  All writes go to the  
end of the file; existing content is preserved.  

    use std::fs::OpenOptions;
    use std::io::{Result, Write};

    fn main() -> Result<()> {
        let mut file = OpenOptions::new()
            .append(true)
            .create(true)
            .open("log.txt")?;

        writeln!(file, "A new log entry")?;
        Ok(())
    }

Setting `.create(true)` ensures the file is created if it doesn't exist.  
Without it, opening a non-existent file returns an error.  `.append(true)`  
guarantees atomic append semantics on POSIX systems when writing up to  
`PIPE_BUF` bytes.  


## Read lines into a collection

Collect all lines from a file into a `Vec<String>` for processing.  
Useful when you need random access to lines or repeated iteration.  

    use std::fs::File;
    use std::io::{BufRead, BufReader, Result};

    fn main() -> Result<()> {
        let file = File::open("thermopylae.txt")?;
        let reader = BufReader::new(file);

        let lines: Vec<String> = reader
            .lines()
            .collect::<Result<_, _>>()?;

        println!("File has {} lines", lines.len());

        for (i, line) in lines.iter().enumerate() {
            println!("{}: {}", i + 1, line);
        }

        Ok(())
    }

The turbofish `collect::<Result<Vec<String>, _>>()` converts  
`Iterator<Item = Result<String>>` into `Result<Vec<String>>`.  If any  
item is an error, the entire collection short-circuits and returns that  
error.  This is a concise pattern for "collect or fail".  


## Working with file metadata

`std::fs::metadata` returns a `Metadata` struct with properties like  
file length, permissions, modification time, and file type.  

    use std::fs;

    fn main() -> std::io::Result<()> {
        let meta = fs::metadata("thermopylae.txt")?;

        println!("Size:        {} bytes", meta.len());
        println!("Read-only:   {}", meta.permissions().readonly());
        println!("Is file:     {}", meta.is_file());
        println!("Is dir:      {}", meta.is_dir());

        if let Ok(modified) = meta.modified() {
            println!("Modified:    {:?}", modified);
        }

        Ok(())
    }

`modified()` returns `Result<SystemTime>` because the platform may not  
support modification timestamps.  `len()` returns the file size in bytes.  
Use `symlink_metadata` to query a symlink without following it.  


## Creating and removing directories

The `std::fs` module provides functions for directory management.  

    use std::fs;
    use std::path::Path;

    fn main() -> std::io::Result<()> {
        // Create a single directory
        fs::create_dir("output")?;

        // Create all parent directories as needed
        fs::create_dir_all("output/reports/2025")?;

        // Write a file into the new directory
        fs::write("output/reports/2025/summary.txt", b"Q1 results\n")?;

        // Remove the empty file
        fs::remove_file("output/reports/2025/summary.txt")?;

        // Remove the deepest empty directory
        fs::remove_dir("output/reports/2025")?;

        // Remove the entire tree
        fs::remove_dir_all("output")?;

        Ok(())
    }

`create_dir` fails if the parent doesn't exist; `create_dir_all` creates  
all missing ancestors.  `remove_dir` only removes empty directories; use  
`remove_dir_all` to delete recursively (use with caution).  


## Listing directory contents

`std::fs::read_dir` returns an iterator over directory entries.  Each  
entry exposes the file name, path, and metadata.  

    use std::fs;

    fn main() -> std::io::Result<()> {
        let entries = fs::read_dir(".")?;

        for entry in entries {
            let entry = entry?;
            let path = entry.path();
            let meta = entry.metadata()?;

            let kind = if meta.is_dir() { "DIR" } else { "FILE" };
            println!("{:>4}  {:>8}  {}", kind, meta.len(), path.display());
        }

        Ok(())
    }

`entry?` propagates errors from individual directory entries (e.g.,  
permission denied on a single file).  The iterator is lazy; errors  
occur only as entries are consumed.  `path.display()` returns a  
`Display`-able object safe for printing and logging.  


## Copying files

`std::fs::copy` copies the content of one file to another.  It does  
not copy metadata like permissions or timestamps.  

    use std::fs;

    fn main() -> std::io::Result<()> {
        fs::copy("thermopylae.txt", "thermopylae_backup.txt")?;
        println!("Copied {} bytes", fs::metadata("thermopylae_backup.txt")?.len());
        Ok(())
    }

For copying large files, `fs::copy` uses OS-level copy mechanisms  
where available (e.g., `copy_file_range` on Linux, `fclonefileat` on  
macOS).  Use `std::io::copy` for streaming copy between readers and  
writers when you need more control over buffering.  


## Renaming and moving files

`std::fs::rename` moves or renames a file or directory atomically when  
source and destination are on the same filesystem.  

    use std::fs;

    fn main() -> std::io::Result<()> {
        fs::write("draft.txt", b"Work in progress\n")?;
        fs::rename("draft.txt", "final.txt")?;
        println!("File moved successfully");
        Ok(())
    }

If the destination exists, it is overwritten on most platforms.  Cross-  
filesystem moves require a copy-then-delete pattern (not provided by  
`std::fs`).  For portable cross-filesystem moves, consider the `fs_extra`  
crate.  


## Reading with a fixed-size buffer

When you know the exact number of bytes to read, use `read_exact`.  
This ensures the buffer is filled completely or returns an error.  

    use std::fs::File;
    use std::io::{Read, Result};

    fn main() -> Result<()> {
        let mut file = File::open("thermopylae.txt")?;

        // Read exactly the first 16 bytes
        let mut header = [0u8; 16];
        file.read_exact(&mut header)?;

        println!("Header bytes: {:?}", header);
        println!("As string:    {}", String::from_utf8_lossy(&header));

        Ok(())
    }

If the file has fewer bytes than the buffer size, `read_exact` returns  
`ErrorKind::UnexpectedEof`.  This is useful for reading binary formats  
with fixed-size headers, magic numbers, or checksums.  


## Seek operations

`Seek` lets you move the read/write cursor within a file.  Combine with  
`SeekFrom` to set absolute or relative positions.  

    use std::fs::File;
    use std::io::{Read, Result, Seek, SeekFrom, Write};

    fn main() -> Result<()> {
        let mut file = File::options()
            .read(true)
            .write(true)
            .create(true)
            .open("data.bin")?;

        // Write at the beginning
        file.write_all(b"ABCDEFGHIJ")?;

        // Seek to offset 3, then overwrite
        file.seek(SeekFrom::Start(3))?;
        file.write_all(b"XYZ")?;
        // File now contains "ABCXYZHIJ"

        // Seek back to start and read everything
        file.seek(SeekFrom::Start(0))?;
        let mut content = String::new();
        file.read_to_string(&mut content)?;
        println!("Content: {content}");

        Ok(())
    }

`SeekFrom::Start(n)` sets the position to `n` bytes from the beginning.  
`SeekFrom::End(n)` sets it relative to the end (use negative values).  
`SeekFrom::Current(n)` advances by `n` bytes from the current position.  


## Streaming copy with io::copy

`std::io::copy` reads all bytes from a reader and writes them to a  
writer.  It uses an internal buffer and is efficient for large files.  

    use std::fs::File;
    use std::io::{self, BufReader, BufWriter};

    fn main() -> io::Result<()> {
        let src = File::open("thermopylae.txt")?;
        let dst = File::create("copy.txt")?;

        let mut reader = BufReader::new(src);
        let mut writer = BufWriter::new(dst);

        let bytes = io::copy(&mut reader, &mut writer)?;
        println!("Copied {} bytes", bytes);

        Ok(())
    }

`io::copy` returns the number of bytes copied.  The `BufReader`/`BufWriter`  
wrapping is optional but recommended; `copy` uses its own internal buffer  
regardless.  The explicit wrappers help when you chain multiple operations  
on the same reader or writer.  


## Convenience: fs::read and fs::write

For simple cases, `fs::read` and `fs::write` are one-liners that handle  
open, read/write, and close in a single call.  

    use std::fs;

    fn main() -> std::io::Result<()> {
        // Write bytes (overwrites if file exists)
        fs::write("note.txt", b"Sparta stands\n")?;

        // Read bytes back
        let bytes = fs::read("note.txt")?;

        println!("Read {} bytes: {:?}", bytes.len(), bytes);
        Ok(())
    }

These are not suitable for large files because they load the entire  
content into memory.  For appending or controlling truncation, use  
`OpenOptions` or `File::options()` instead.  


## Reading from standard input

`std::io::stdin()` gives access to the standard input stream.  Use  
`lock()` for efficient buffered reading.  

    use std::io::{self, BufRead, Write};

    fn main() -> io::Result<()> {
        let stdin = io::stdin();
        let stdout = io::stdout();
        let mut handle = stdout.lock();

        writeln!(handle, "Type something and press Enter:")?;
        handle.flush()?;

        for line in stdin.lock().lines() {
            let line = line?;
            if line.trim().is_empty() {
                break;
            }
            writeln!(handle, "You wrote: {line}")?;
            handle.flush()?;
        }

        Ok(())
    }

`stdin.lock()` returns a `StdinLock` that implements `BufRead`.  The  
lock is released when the guard goes out of scope.  Always flush  
`stdout` before reading from `stdin` when printing prompts — stdout  
is line-buffered by default and the prompt may not appear otherwise.  


## Writing to standard output vs standard error

Use `std::io::stdout()` for normal output and `std::io::stderr()` for  
error messages.  This lets shell users redirect them independently.  

    use std::io::{self, Write};

    fn main() -> io::Result<()> {
        writeln!(io::stdout(), "Operation completed successfully")?;
        writeln!(io::stderr(), "Warning: this is a demo")?;
        Ok(())
    }

When writing many small fragments, lock the handle first to avoid  
repeated locking overhead: `io::stdout().lock()`.  


## Hard links and symlinks

Create filesystem links for the same content at multiple paths.  

    use std::fs;

    fn main() -> std::io::Result<()> {
        fs::write("original.txt", b"Shared content\n")?;

        // Hard link: same inode, must be on same filesystem
        fs::hard_link("original.txt", "alias.txt")?;

        // Symlink (soft link): separate inode, can cross filesystems
        // (Unix only — use symlink_metadata to read without following)
        #[cfg(unix)]
        fs::symlink("original.txt", "shortcut.txt")?;

        println!("Hard link created: alias.txt");

        Ok(())
    }

`symlink` and `read_link` are Unix-only.  Hard links share the same data;  
deleting `original.txt` does not affect `alias.txt` as long as at least  
one link remains.  


## Path and PathBuf basics

`Path` (borrowed) and `PathBuf` (owned) are the standard types for  
filesystem paths.  They abstract over OS-specific separators.  

    use std::path::{Path, PathBuf};

    fn main() {
        // Build a path from components
        let dir = Path::new("/home/user/docs");
        let file = dir.join("notes.txt");
        println!("Full path:  {}", file.display());

        // Inspect components
        println!("File name:  {:?}", file.file_name());
        println!("Extension:  {:?}", file.extension());
        println!("Parent:     {:?}", file.parent());
        println!("Is absolute: {}", file.is_absolute());

        // Modify a PathBuf in place
        let mut pb = PathBuf::from("/tmp/scratch.txt");
        pb.set_extension("log");
        println!("Changed ext: {}", pb.display());
    }

`Path` is to `PathBuf` as `str` is to `String`.  Use `Path` for function  
parameters that don't need ownership and `PathBuf` when you need to store  
or modify the path.  


## Working with file permissions

`std::fs::Permissions` and `set_permissions` let you inspect and modify  
file permissions.  The API surface is limited for portability; use  
platform-specific crates for fine-grained control.  

    use std::fs;

    fn main() -> std::io::Result<()> {
        fs::write("config.cfg", b"key=value\n")?;

        let mut perms = fs::metadata("config.cfg")?.permissions();
        println!("Read-only before: {}", perms.readonly());

        // Set to read-only
        perms.set_readonly(true);
        fs::set_permissions("config.cfg", perms)?;

        println!("Read-only after:  {}",
            fs::metadata("config.cfg")?.permissions().readonly());

        // Reset so we can clean up
        let mut perms = fs::metadata("config.cfg")?.permissions();
        perms.set_readonly(false);
        fs::set_permissions("config.cfg", perms)?;

        fs::remove_file("config.cfg")?;
        Ok(())
    }

On Unix, `set_readonly` toggles the owner-write bit.  On Windows, it  
sets the read-only attribute.  For more granular permission control,  
use the `file-per-thread` or platform `libc` calls.  


## Error handling patterns

Explicit matching on `io::ErrorKind` lets you take different actions  
based on the type of error — for example, creating a file only if it  
doesn't exist.  

    use std::fs::{self, File};
    use std::io::{self, ErrorKind, Read};

    fn read_or_default(path: &str) -> io::Result<String> {
        let mut content = String::new();
        match File::open(path) {
            Ok(mut f) => {
                f.read_to_string(&mut content)?;
                Ok(content)
            }
            Err(e) if e.kind() == ErrorKind::NotFound => {
                Ok(String::from("(file not found)\n"))
            }
            Err(e) => Err(e),
        }
    }

    fn main() -> io::Result<()> {
        let text = read_or_default("nonexistent.txt")?;
        println!("{text}");
        Ok(())
    }

The `if e.kind() == ErrorKind::NotFound` guard lets you match only  
specific error variants.  This pattern avoids creating a default file  
when the error is something other than "not found" (e.g., permission  
denied).  


## Chunked reading

For very large files, read in fixed-size chunks instead of loading the  
entire file into memory.  

    use std::fs::File;
    use std::io::{Read, Result};

    const CHUNK_SIZE: usize = 4096;

    fn main() -> Result<()> {
        let mut file = File::open("/dev/urandom")?;
        let mut buf = [0u8; CHUNK_SIZE];
        let mut total = 0usize;

        loop {
            let n = file.read(&mut buf)?;
            if n == 0 {
                break; // EOF
            }
            total += n;
            // Process buf[..n] here
            if total >= 16384 {
                break; // Stop after 16 KiB for this demo
            }
        }

        println!("Read {} bytes total", total);
        Ok(())
    }

`read` returns zero only at EOF.  The loop accumulates bytes and  
processes each chunk independently.  For even larger files, consider  
memory-mapped I/O via the `memmap2` crate.  


## Reading and writing with serde

Serialization crates like `serde` and `serde_json` make structured  
file I/O straightforward.  Add to `Cargo.toml`:  
`serde = { version = "1", features = ["derive"] }` and `serde_json = "1"`.  

    use serde::{Deserialize, Serialize};
    use std::fs;

    #[derive(Serialize, Deserialize, Debug)]
    struct Hero {
        name: String,
        homeland: String,
        soldiers: u32,
    }

    fn main() -> std::io::Result<()> {
        let leonidas = Hero {
            name: "Leonidas".into(),
            homeland: "Sparta".into(),
            soldiers: 300,
        };

        // Serialize to JSON and write to file
        let json = serde_json::to_string_pretty(&leonidas)
            .expect("serialization failed");
        fs::write("hero.json", &json)?;

        // Read back and deserialize
        let json = fs::read_to_string("hero.json")?;
        let hero: Hero = serde_json::from_str(&json)
            .expect("deserialization failed");

        println!("{:?}", hero);

        Ok(())
    }

This pattern works with JSON, YAML, TOML, MessagePack, and many other  
formats supported by the serde ecosystem.  For binary formats with  
smaller output, consider `bincode` or `postcard`.  

