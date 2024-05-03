# Vector 

A vector is a contiguous growable collection of data.  

## for_each function

The `for_each` function is used for declarative iteration.  

```rust
fn main() {
    let mut words = Vec::<&str>::new();
    words.push("sky");
    words.push("road");
    words.push("cloud");

    words.iter().for_each(|word| println!("{word}"));
}
```

## for loop

The `for/in` loop is used for classic, procedural iteration.  

```rust
fn main() {
    let words = vec!["rock", "pub", "cloud", "book", "chair", "glass"];

    println!("the collection has {} elements", words.len());

    for word in &words {
        println!("{word} has {} latin characters", word.len());
    }
}
```

## Vec::from

The `Vec::from` creates a vector from an array. The elements are accessed via  
index.  

```rust
fn main() {
    let words = Vec::from(["rock", "pub", "cloud", "book", "chair", "glass"]);

    let n = words.len();

    let w1 = words[0];
    println!("{w1}");

    let w2 = words[n-1];
    println!("{w2}");
}
```

## Sort vector in-place

```rust
fn main() {
    let mut words = Vec::new();

    words.push("sky");
    words.push("cloud");
    words.push("cup");
    words.push("tent");
    words.push("war");
    words.push("atom");
    words.push("bear");
    words.push("den");

    println!("{words:?}");
    words.sort();
    println!("{words:?}");
}
```

## Create a sorted copy

Create a copy with `clone` and then sort it.  

```rust
fn main() {
    let mut words = Vec::new();

    words.push("sky");
    words.push("cloud");
    words.push("cup");
    words.push("tent");
    words.push("war");
    words.push("atom");
    words.push("bear");
    words.push("den");

    println!("{words:?}");

    let mut words2 = words.clone();
    words2.sort();
    println!("{words2:?}");
}
```

## Sum of integers

Create a vector from a range of integers and calculate its sum.  

```rust
fn main() {
    let vals = (15..35).collect::<Vec<i32>>();

    let sum: i32 = vals.iter().sum();
    println!("{sum}");
}
```

## map function

`map` applies the given function to each element of the vector. It returns a modified vector.  

```rust
fn main() {
    let vals: Vec<i32> = (1..10).collect();
    
    let res: Vec<i32> = vals.iter().map(|e| e * 2).collect();

    println!("{vals:?}");
    println!("{res:?}");
}
```

## filter function

The filter method applies the filtering function on the vector elements.  

```rust
fn main() {
    let vals = vec![-3, -2, 1, 0, -1, 3, 2, 4];

    let res: Vec<i32> = vals.iter().filter(|&e| *e > 0).copied().collect();
   
    println!("{vals:?}");
    println!("{res:?}");
}
```

The `iter` method creates an iterator that yields references to the elements of the `vals` vector.  
In addition, the `filter` method takes a closure that receives a reference to the current element  
being iterated, so we end up with a reference to a reference to the element.  

The `|&e| *e > 0` closure than uses the *pattern matching* operation to extract a reference, which is    
later dereferenced in the `*e > 0` expression. The `copied` function then copies the values so that  
end up with a `Vec<i32>`.  

Other options are:  

`let res: Vec<i32> = vals.iter().copied().filter(|&e| e > 0).collect();`

Here we first call `copied` and we then deal with only single references.  

`let res: Vec<i32> = vals.iter().filter(|&&e| e > 0).copied().collect();`

Here we extract the value from `e` using `&&e` pattern match.  

### Recap:  

The `iter` method on a collection like a vector yields an iterator that produces references to the elements  
of the collection. The `filter` function then takes a closure that receives each element as a reference from  
the iterator. If the iterator is already producing references, the closure will receive a double reference  
(a reference to a reference).  

For example, if you have a vector of integers `Vec<i32>` and you call `iter` on it, the items produced by the  
iterator are of type `&i32`. When you pass these items to the `filter` closure, the closure receives `&&i32`  
because it's a reference to the items produced by `iter`, which are themselves references.  

## filter_map 

applies both filter and map operations.  

```rust
fn main() {
    let vals = vec![-3, -2, 1, 0, -1, 3, 2, 4];

    let res: Vec<i32> = vals
        .iter()
        .filter_map(|&x| {
            let doubled = x * 2;
            if doubled > 0 {
                Some(doubled)
            } else {
                None
            }
        })
        .collect();

    println!("{res:?}");

    let res: Vec<i32> = vals
        .iter()
        .filter_map(|&x| if x > 0 { Some(x * 2) } else { None })
        .collect();

        println!("{res:?}");
}
```



