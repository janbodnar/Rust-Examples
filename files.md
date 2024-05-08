# Read/Write files 


## Write text to a file 

The `?` is a postfix operator that unwraps `Result<T, E>` and `Option<T>` values.  
With `Result<T, E>`, it unwraps the result and gives the inner value, propagating the  
error to the calling function. It is a convenience syntax for reducing code.  

```rust
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
```

The `?` operator works like this: 

- If it is a success, it unwraps the `Result` to get the success value inside. The value is then  
  assigned to the variable file  
- If the result is an error, the error is not assigned to the variable file. The `Error` is returned by  
  the function to the caller.

The `?` is a syntax sugar for: 

```rust
let mut file = match File::create("foo.txt") {
    Err(why) => panic!("couldn't create {}: {}", display, why),
    Ok(file) => file,
};
```

## fs::read_to_string

Suitable for smaller text (UTF8) files.  

```rust
use std::fs;
use std::io;

fn main() -> io::Result<()> {
    let content = fs::read_to_string("thermopylae.txt")?;
    println!("{content}");
    Ok(())
}
```

## Read into vector

Read file into `Vec<u8>` vector. Later transform the vector back to string  
using `String::from_utf8`.   

```rust
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
```

Alternatively, use `unwrap_or_else` for error handling.  

```rust
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
```
