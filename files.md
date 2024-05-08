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
