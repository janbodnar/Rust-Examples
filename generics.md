# Generics

Rust generics provide powerful abstraction mechanisms that enable writing  
flexible, type-safe code that works with multiple data types. Generics are  
compile-time constructs that create specific implementations for each type  
used, ensuring zero runtime cost while maintaining type safety.  

## Basic generic function

A simple generic function that works with any type.  

```rust
fn identity<T>(value: T) -> T {
    value
}

fn main() {
    let number = identity(42);
    let text = identity("Hello");
    let flag = identity(true);
    
    println!("Number: {}", number);
    println!("Text: {}", text);
    println!("Flag: {}", flag);
}
```

The `identity` function accepts any type `T` and returns the same value.  
The compiler generates separate implementations for each concrete type used,  
ensuring type safety without runtime overhead. Generic parameters are  
specified in angle brackets after the function name.  

## Generic struct with single parameter

Creating a container that can hold any type of value.  

```rust
#[derive(Debug)]
struct Box<T> {
    value: T,
}

impl<T> Box<T> {
    fn new(value: T) -> Self {
        Box { value }
    }
    
    fn get(&self) -> &T {
        &self.value
    }
    
    fn unwrap(self) -> T {
        self.value
    }
}

fn main() {
    let int_box = Box::new(100);
    let string_box = Box::new("Rust".to_string());
    let float_box = Box::new(3.14);
    
    println!("Integer box: {:?}", int_box);
    println!("String box: {:?}", string_box);
    println!("Float box: {:?}", float_box);
    
    println!("Unwrapped: {}", int_box.unwrap());
}
```

Generic structs use type parameters to store values of any type. The `impl`  
block must also declare the generic parameter. Methods can operate on the  
generic type safely, with the compiler ensuring type correctness at compile  
time. This pattern is fundamental to Rust's standard library collections.  

## Generic function with constraints

Using trait bounds to constrain generic types to specific capabilities.  

```rust
use std::fmt::Display;

fn print_twice<T: Display>(value: T) {
    println!("First: {}", value);
    println!("Second: {}", value);
}

fn compare_values<T: PartialOrd + Display>(a: T, b: T) {
    if a > b {
        println!("{} is greater than {}", a, b);
    } else if a < b {
        println!("{} is less than {}", a, b);
    } else {
        println!("{} equals {}", a, b);
    }
}

fn main() {
    print_twice(42);
    print_twice("Hello, World!");
    
    compare_values(10, 5);
    compare_values(3.14, 2.71);
    compare_values('a', 'z');
}
```

Trait bounds restrict generic types to those implementing specific traits.  
The `Display` trait enables printing, while `PartialOrd` enables comparison.  
Multiple bounds are combined with `+`. This ensures generic functions only  
accept types that support the required operations.  

## Generic enum with multiple variants

Creating enums that can hold different types of data.  

```rust
#[derive(Debug)]
enum Either<L, R> {
    Left(L),
    Right(R),
}

impl<L, R> Either<L, R> {
    fn is_left(&self) -> bool {
        matches!(self, Either::Left(_))
    }
    
    fn is_right(&self) -> bool {
        matches!(self, Either::Right(_))
    }
    
    fn left(self) -> Option<L> {
        match self {
            Either::Left(value) => Some(value),
            Either::Right(_) => None,
        }
    }
    
    fn right(self) -> Option<R> {
        match self {
            Either::Left(_) => None,
            Either::Right(value) => Some(value),
        }
    }
}

fn main() {
    let number_or_text = Either::Left(42);
    let text_or_number = Either::Right("Hello".to_string());
    
    println!("First either: {:?}", number_or_text);
    println!("Second either: {:?}", text_or_number);
    
    if number_or_text.is_left() {
        println!("Contains number: {:?}", number_or_text.left());
    }
}
```

Generic enums can represent choice between different types. The `Either` type  
is useful for operations that can return one of two different types. Methods  
can extract values safely using pattern matching, maintaining type safety  
throughout the program.  

## Multiple generic parameters

Working with functions and structs that have multiple type parameters.  

```rust
#[derive(Debug)]
struct Pair<T, U> {
    first: T,
    second: U,
}

impl<T, U> Pair<T, U> {
    fn new(first: T, second: U) -> Self {
        Pair { first, second }
    }
    
    fn swap(self) -> Pair<U, T> {
        Pair {
            first: self.second,
            second: self.first,
        }
    }
}

fn combine<T, U, V>(left: T, right: U, f: impl Fn(T, U) -> V) -> V {
    f(left, right)
}

fn main() {
    let coordinate = Pair::new(10.5, 20.3);
    let mixed = Pair::new(42, "items".to_string());
    
    println!("Coordinate: {:?}", coordinate);
    println!("Mixed pair: {:?}", mixed);
    
    let swapped = mixed.swap();
    println!("Swapped: {:?}", swapped);
    
    let sum = combine(5, 10, |a, b| a + b);
    let concat = combine("Hello, ", "World!", |a, b| format!("{}{}", a, b));
    
    println!("Sum: {}", sum);
    println!("Concatenated: {}", concat);
}
```

Multiple type parameters enable complex generic relationships. Each parameter  
can represent a different type, allowing flexible data structures and  
functions. The `impl Fn` syntax creates generic closures, enabling functional  
programming patterns with type safety.  

## Generic traits and associated types

Defining traits that work with generic types and associated types.  

```rust
trait Container {
    type Item;
    
    fn new() -> Self;
    fn insert(&mut self, item: Self::Item);
    fn get(&self, index: usize) -> Option<&Self::Item>;
    fn len(&self) -> usize;
}

#[derive(Debug)]
struct SimpleVec<T> {
    items: Vec<T>,
}

impl<T> Container for SimpleVec<T> {
    type Item = T;
    
    fn new() -> Self {
        SimpleVec { items: Vec::new() }
    }
    
    fn insert(&mut self, item: Self::Item) {
        self.items.push(item);
    }
    
    fn get(&self, index: usize) -> Option<&Self::Item> {
        self.items.get(index)
    }
    
    fn len(&self) -> usize {
        self.items.len()
    }
}

fn main() {
    let mut numbers = SimpleVec::new();
    numbers.insert(1);
    numbers.insert(2);
    numbers.insert(3);
    
    println!("Container: {:?}", numbers);
    println!("Length: {}", numbers.len());
    println!("First item: {:?}", numbers.get(0));
    
    let mut words: SimpleVec<String> = Container::new();
    words.insert("hello".to_string());
    words.insert("world".to_string());
    
    println!("Words: {:?}", words);
}
```

Associated types allow traits to define placeholder types that implementing  
types must specify. This creates cleaner interfaces than using generic  
parameters on traits directly. The `Container` trait defines a generic  
interface that can be implemented for any collection type.  

## Generic with lifetime parameters

Combining generics with lifetimes for borrowing relationships.  

```rust
#[derive(Debug)]
struct Reference<'a, T> {
    value: &'a T,
}

impl<'a, T> Reference<'a, T> {
    fn new(value: &'a T) -> Self {
        Reference { value }
    }
    
    fn get(&self) -> &T {
        self.value
    }
}

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn find_in_slice<'a, T: PartialEq>(slice: &'a [T], target: &T) -> Option<&'a T> {
    slice.iter().find(|&item| item == target)
}

fn main() {
    let number = 42;
    let reference = Reference::new(&number);
    println!("Reference: {:?}", reference);
    println!("Value: {}", reference.get());
    
    let string1 = "hello";
    let string2 = "world!";
    let longer = longest(string1, string2);
    println!("Longer string: {}", longer);
    
    let numbers = [1, 2, 3, 4, 5];
    if let Some(found) = find_in_slice(&numbers, &3) {
        println!("Found: {}", found);
    }
}
```

Lifetime parameters work with generics to ensure borrowed data remains valid.  
The `'a` lifetime parameter indicates how long references must be valid. This  
combines Rust's ownership system with generic programming, ensuring memory  
safety without sacrificing flexibility.  

## Generic iterator implementation

Creating custom iterators that work with any type.  

```rust
#[derive(Debug)]
struct Counter<T> {
    current: T,
    max: T,
}

impl<T> Counter<T> 
where 
    T: Copy + PartialOrd + std::ops::Add<Output = T> + From<u8>,
{
    fn new(max: T) -> Self {
        Counter {
            current: T::from(0),
            max,
        }
    }
}

impl<T> Iterator for Counter<T>
where
    T: Copy + PartialOrd + std::ops::Add<Output = T> + From<u8>,
{
    type Item = T;
    
    fn next(&mut self) -> Option<Self::Item> {
        if self.current < self.max {
            let current = self.current;
            self.current = current + T::from(1);
            Some(current)
        } else {
            None
        }
    }
}

fn main() {
    println!("Integer counter:");
    let int_counter = Counter::new(5);
    for num in int_counter {
        println!("Count: {}", num);
    }
    
    println!("\nFloat counter:");
    let float_counter = Counter::new(3.0);
    for num in float_counter {
        println!("Count: {:.1}", num);
    }
}
```

Generic iterators enable creating reusable iteration patterns. Complex trait  
bounds ensure the type supports required operations like addition and  
comparison. The `where` clause improves readability for complex bounds. This  
pattern enables creating domain-specific iterators for any compatible type.  

## Default generic type parameters

Using default types to simplify generic usage.  

```rust
#[derive(Debug)]
struct HashMap<K, V, H = std::collections::hash_map::DefaultHasher> {
    _key: std::marker::PhantomData<K>,
    _value: std::marker::PhantomData<V>,
    _hasher: std::marker::PhantomData<H>,
}

#[derive(Debug)]
struct Result<T, E = Box<dyn std::error::Error>> {
    value: std::result::Result<T, E>,
}

impl<T, E> Result<T, E> {
    fn ok(value: T) -> Self {
        Result { value: Ok(value) }
    }
    
    fn err(error: E) -> Self {
        Result { value: Err(error) }
    }
    
    fn is_ok(&self) -> bool {
        self.value.is_ok()
    }
    
    fn unwrap(self) -> T {
        self.value.unwrap()
    }
}

#[derive(Debug)]
struct Vector<T, const N: usize = 3> {
    data: [T; N],
}

impl<T: Default + Copy, const N: usize> Vector<T, N> {
    fn new() -> Self {
        Vector {
            data: [T::default(); N],
        }
    }
}

fn main() {
    // Uses default error type
    let success: Result<i32> = Result::ok(42);
    let failure: Result<i32> = Result::err("Something went wrong".into());
    
    println!("Success: {:?}", success);
    println!("Failure: {:?}", failure);
    
    // Uses default size of 3
    let vec3d: Vector<f64> = Vector::new();
    let vec4d: Vector<f64, 4> = Vector::new();
    
    println!("3D vector: {:?}", vec3d);
    println!("4D vector: {:?}", vec4d);
}
```

Default type parameters reduce boilerplate for common use cases. Users can  
specify only the required parameters while defaults handle the rest. Const  
generics enable compile-time parameterization by values like array sizes.  
This provides type safety for numeric parameters without runtime overhead.  

## Generic builder pattern

Implementing the builder pattern with generics for type-safe construction.  

```rust
#[derive(Debug)]
struct Config {
    name: String,
    port: u16,
    debug: bool,
}

struct ConfigBuilder<N, P, D> {
    name: N,
    port: P,
    debug: D,
}

struct Set<T>(T);
struct Unset;

impl ConfigBuilder<Unset, Unset, Unset> {
    fn new() -> Self {
        ConfigBuilder {
            name: Unset,
            port: Unset,
            debug: Unset,
        }
    }
}

impl<P, D> ConfigBuilder<Unset, P, D> {
    fn name(self, name: String) -> ConfigBuilder<Set<String>, P, D> {
        ConfigBuilder {
            name: Set(name),
            port: self.port,
            debug: self.debug,
        }
    }
}

impl<N, D> ConfigBuilder<N, Unset, D> {
    fn port(self, port: u16) -> ConfigBuilder<N, Set<u16>, D> {
        ConfigBuilder {
            name: self.name,
            port: Set(port),
            debug: self.debug,
        }
    }
}

impl<N, P> ConfigBuilder<N, P, Unset> {
    fn debug(self, debug: bool) -> ConfigBuilder<N, P, Set<bool>> {
        ConfigBuilder {
            name: self.name,
            port: self.port,
            debug: Set(debug),
        }
    }
}

impl ConfigBuilder<Set<String>, Set<u16>, Set<bool>> {
    fn build(self) -> Config {
        Config {
            name: self.name.0,
            port: self.port.0,
            debug: self.debug.0,
        }
    }
}

fn main() {
    let config = ConfigBuilder::new()
        .name("MyApp".to_string())
        .port(8080)
        .debug(true)
        .build();
    
    println!("Config: {:?}", config);
    
    // This would cause a compile error - missing required fields:
    // let incomplete = ConfigBuilder::new().name("test".to_string()).build();
}
```

The typestate pattern uses generics to encode required builder steps in the  
type system. Each field has `Set` or `Unset` states, preventing compilation  
of incomplete configurations. The `build` method only exists when all fields  
are set, ensuring compile-time validation of builder usage.  

## Higher-ranked trait bounds

Using generic functions with complex lifetime relationships.  

```rust
fn apply_to_all<F>(items: &mut [String], f: F)
where
    F: for<'a> Fn(&'a mut String),
{
    for item in items {
        f(item);
    }
}

fn process_with_closure<F, T>(value: T, f: F) -> T
where
    F: for<'a> Fn(&'a T) -> T,
    T: Clone,
{
    f(&value)
}

trait Closure<Args> {
    type Output;
    fn call(&self, args: Args) -> Self::Output;
}

impl<F, T> Closure<T> for F
where
    F: Fn(T) -> T,
{
    type Output = T;
    
    fn call(&self, args: T) -> Self::Output {
        self(args)
    }
}

fn use_closure<C, T>(closure: C, value: T) -> T
where
    C: Closure<T, Output = T>,
{
    closure.call(value)
}

fn main() {
    let mut words = vec![
        "hello".to_string(),
        "world".to_string(),
        "rust".to_string(),
    ];
    
    println!("Before: {:?}", words);
    
    apply_to_all(&mut words, |s| {
        s.make_ascii_uppercase();
    });
    
    println!("After: {:?}", words);
    
    let number = 42;
    let doubled = process_with_closure(number, |x| x * 2);
    println!("Doubled: {}", doubled);
    
    let tripled = use_closure(|x| x * 3, 10);
    println!("Tripled: {}", tripled);
}
```

Higher-ranked trait bounds (`for<'a>`) enable generic functions to work with  
closures that have flexible lifetime requirements. This is essential for  
functions that must work with borrowed data of any lifetime. The pattern  
enables powerful abstractions while maintaining Rust's lifetime safety.  

## Generic smart pointer

Creating custom smart pointers with generic ownership semantics.  

```rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
struct SharedPtr<T> {
    inner: Rc<RefCell<T>>,
}

impl<T> SharedPtr<T> {
    fn new(value: T) -> Self {
        SharedPtr {
            inner: Rc::new(RefCell::new(value)),
        }
    }
    
    fn borrow(&self) -> std::cell::Ref<T> {
        self.inner.borrow()
    }
    
    fn borrow_mut(&self) -> std::cell::RefMut<T> {
        self.inner.borrow_mut()
    }
    
    fn clone(&self) -> Self {
        SharedPtr {
            inner: Rc::clone(&self.inner),
        }
    }
    
    fn strong_count(&self) -> usize {
        Rc::strong_count(&self.inner)
    }
}

impl<T: std::fmt::Display> SharedPtr<T> {
    fn print(&self) {
        println!("Value: {}", self.borrow());
    }
}

fn main() {
    let shared_num = SharedPtr::new(42);
    let shared_num_clone = shared_num.clone();
    
    println!("Strong count: {}", shared_num.strong_count());
    
    shared_num.print();
    
    {
        let mut borrowed = shared_num.borrow_mut();
        *borrowed = 100;
    }
    
    shared_num_clone.print();
    
    let shared_string = SharedPtr::new("Hello".to_string());
    {
        let mut borrowed = shared_string.borrow_mut();
        borrowed.push_str(", World!");
    }
    
    shared_string.print();
}
```

Custom smart pointers combine multiple ownership patterns. `Rc` provides  
shared ownership while `RefCell` enables interior mutability. Generic  
implementations work with any type, and conditional implementations add  
type-specific functionality. This pattern is fundamental to many Rust  
libraries and frameworks.  

## Generic async patterns

Working with generic asynchronous code and futures.  

```rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll};

struct AsyncContainer<T> {
    value: Option<T>,
    ready: bool,
}

impl<T> AsyncContainer<T> {
    fn new(value: T) -> Self {
        AsyncContainer {
            value: Some(value),
            ready: false,
        }
    }
}

impl<T> Future for AsyncContainer<T> {
    type Output = T;
    
    fn poll(mut self: Pin<&mut Self>, _cx: &mut Context<'_>) -> Poll<Self::Output> {
        if self.ready {
            Poll::Ready(self.value.take().unwrap())
        } else {
            self.ready = true;
            Poll::Pending
        }
    }
}

async fn process_async<T, F, Fut>(value: T, f: F) -> T
where
    F: FnOnce(T) -> Fut,
    Fut: Future<Output = T>,
{
    f(value).await
}

async fn double_async(x: i32) -> i32 {
    // Simulate async work
    x * 2
}

async fn concat_async(s: String) -> String {
    // Simulate async work
    format!("{} (processed)", s)
}

// Note: This is a simplified example for demonstration
// In real async code, you'd use tokio or async-std
fn block_on<F: Future>(future: F) -> F::Output {
    // Simplified executor - not for production use
    use std::task::{Context, Poll, RawWaker, RawWakerVTable, Waker};
    
    let mut future = Box::pin(future);
    let waker = unsafe {
        Waker::from_raw(RawWaker::new(
            std::ptr::null(),
            &RawWakerVTable::new(|_| RawWaker::new(std::ptr::null(), &VTABLE), |_| {}, |_| {}, |_| {}),
        ))
    };
    let mut context = Context::from_waker(&waker);
    
    loop {
        match future.as_mut().poll(&mut context) {
            Poll::Ready(result) => return result,
            Poll::Pending => continue,
        }
    }
}

const VTABLE: RawWakerVTable = RawWakerVTable::new(
    |_| RawWaker::new(std::ptr::null(), &VTABLE),
    |_| {},
    |_| {},
    |_| {},
);

fn main() {
    let container = AsyncContainer::new(42);
    let result = block_on(container);
    println!("Async result: {}", result);
    
    let doubled = block_on(process_async(10, double_async));
    println!("Doubled: {}", doubled);
    
    let processed = block_on(process_async("Hello".to_string(), concat_async));
    println!("Processed: {}", processed);
}
```

Generic async programming enables type-safe asynchronous operations. Futures  
can be parameterized by their output type, and generic async functions work  
with any compatible future type. This pattern is essential for building  
scalable async applications with strong type guarantees.  

## Generic error handling

Creating flexible error handling systems with generics.  

```rust
use std::fmt;

#[derive(Debug)]
struct AppError<T> {
    kind: T,
    message: String,
}

impl<T: fmt::Display> fmt::Display for AppError<T> {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{}: {}", self.kind, self.message)
    }
}

impl<T: fmt::Debug + fmt::Display> std::error::Error for AppError<T> {}

#[derive(Debug)]
enum NetworkError {
    Timeout,
    ConnectionRefused,
    InvalidResponse,
}

impl fmt::Display for NetworkError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            NetworkError::Timeout => write!(f, "Network timeout"),
            NetworkError::ConnectionRefused => write!(f, "Connection refused"),
            NetworkError::InvalidResponse => write!(f, "Invalid response"),
        }
    }
}

#[derive(Debug)]
enum ParseError {
    InvalidFormat,
    MissingField,
    TypeMismatch,
}

impl fmt::Display for ParseError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            ParseError::InvalidFormat => write!(f, "Invalid format"),
            ParseError::MissingField => write!(f, "Missing field"),
            ParseError::TypeMismatch => write!(f, "Type mismatch"),
        }
    }
}

fn network_operation() -> Result<String, AppError<NetworkError>> {
    Err(AppError {
        kind: NetworkError::Timeout,
        message: "Request timed out after 30 seconds".to_string(),
    })
}

fn parse_data(data: &str) -> Result<i32, AppError<ParseError>> {
    if data.is_empty() {
        return Err(AppError {
            kind: ParseError::MissingField,
            message: "Data is empty".to_string(),
        });
    }
    
    data.parse().map_err(|_| AppError {
        kind: ParseError::TypeMismatch,
        message: format!("Cannot parse '{}' as integer", data),
    })
}

fn handle_result<T, E>(result: Result<T, E>) -> String
where
    T: fmt::Display,
    E: fmt::Display,
{
    match result {
        Ok(value) => format!("Success: {}", value),
        Err(error) => format!("Error: {}", error),
    }
}

fn main() {
    let network_result = network_operation();
    println!("{}", handle_result(network_result));
    
    let parse_result = parse_data("not_a_number");
    println!("{}", handle_result(parse_result));
    
    let success_result = parse_data("42");
    println!("{}", handle_result(success_result));
}
```

Generic error types enable creating domain-specific error handling while  
maintaining type safety. The `AppError<T>` type can wrap any error kind,  
providing consistent error structure across different error domains. Generic  
error handling functions work with any error type implementing required  
traits.  

## Generic conversion traits

Implementing generic conversion patterns for type transformation.  

```rust
trait TryIntoWith<T, E> {
    fn try_into_with<F>(self, converter: F) -> Result<T, E>
    where
        F: FnOnce(Self) -> Result<T, E>,
        Self: Sized;
}

impl<S, T, E> TryIntoWith<T, E> for S {
    fn try_into_with<F>(self, converter: F) -> Result<T, E>
    where
        F: FnOnce(Self) -> Result<T, E>,
    {
        converter(self)
    }
}

trait Convert<From, To> {
    type Error;
    
    fn convert(from: From) -> Result<To, Self::Error>;
}

struct StringToInt;

impl Convert<String, i32> for StringToInt {
    type Error = std::num::ParseIntError;
    
    fn convert(from: String) -> Result<i32, Self::Error> {
        from.parse()
    }
}

struct IntToString;

impl Convert<i32, String> for IntToString {
    type Error = ();
    
    fn convert(from: i32) -> Result<String, Self::Error> {
        Ok(from.to_string())
    }
}

fn safe_convert<C, From, To>(value: From) -> Result<To, C::Error>
where
    C: Convert<From, To>,
{
    C::convert(value)
}

#[derive(Debug)]
struct Wrapper<T>(T);

impl<T> Wrapper<T> {
    fn new(value: T) -> Self {
        Wrapper(value)
    }
    
    fn map<U, F>(self, f: F) -> Wrapper<U>
    where
        F: FnOnce(T) -> U,
    {
        Wrapper(f(self.0))
    }
    
    fn and_then<U, F>(self, f: F) -> Wrapper<U>
    where
        F: FnOnce(T) -> Wrapper<U>,
    {
        f(self.0)
    }
}

fn main() {
    // Using custom conversion trait
    let string_value = "42".to_string();
    let converted: Result<i32, _> = string_value.try_into_with(|s| s.parse());
    println!("Converted: {:?}", converted);
    
    // Using generic converter
    let int_result: Result<i32, _> = safe_convert::<StringToInt, _, _>("123".to_string());
    println!("Safe convert int: {:?}", int_result);
    
    let string_result: Result<String, _> = safe_convert::<IntToString, _, _>(456);
    println!("Safe convert string: {:?}", string_result);
    
    // Using wrapper with transformations
    let wrapped = Wrapper::new(10)
        .map(|x| x * 2)
        .and_then(|x| Wrapper::new(x + 5));
    
    println!("Wrapped result: {:?}", wrapped);
}
```

Generic conversion traits provide flexible type transformation patterns. The  
`Convert` trait defines a consistent interface for type conversions with  
error handling. Generic wrapper types enable monadic operations like `map`  
and `and_then`, providing functional programming patterns with type safety.  

## Phantom types and type-level programming

Using phantom types for compile-time type safety without runtime cost.  

```rust
use std::marker::PhantomData;

#[derive(Debug)]
struct Measurement<Unit> {
    value: f64,
    _phantom: PhantomData<Unit>,
}

struct Meters;
struct Feet;
struct Seconds;

impl<Unit> Measurement<Unit> {
    fn new(value: f64) -> Self {
        Measurement {
            value,
            _phantom: PhantomData,
        }
    }
    
    fn value(&self) -> f64 {
        self.value
    }
}

impl Measurement<Meters> {
    fn to_feet(self) -> Measurement<Feet> {
        Measurement::new(self.value * 3.28084)
    }
}

impl Measurement<Feet> {
    fn to_meters(self) -> Measurement<Meters> {
        Measurement::new(self.value / 3.28084)
    }
}

// Type-safe arithmetic
impl<Unit> std::ops::Add for Measurement<Unit> {
    type Output = Self;
    
    fn add(self, other: Self) -> Self::Output {
        Measurement::new(self.value + other.value)
    }
}

// Type-level state machine
#[derive(Debug)]
struct StateMachine<State> {
    _state: PhantomData<State>,
}

struct Idle;
struct Running;
struct Stopped;

impl StateMachine<Idle> {
    fn new() -> Self {
        StateMachine { _state: PhantomData }
    }
    
    fn start(self) -> StateMachine<Running> {
        println!("Starting machine...");
        StateMachine { _state: PhantomData }
    }
}

impl StateMachine<Running> {
    fn stop(self) -> StateMachine<Stopped> {
        println!("Stopping machine...");
        StateMachine { _state: PhantomData }
    }
    
    fn pause(self) -> StateMachine<Idle> {
        println!("Pausing machine...");
        StateMachine { _state: PhantomData }
    }
}

impl StateMachine<Stopped> {
    fn reset(self) -> StateMachine<Idle> {
        println!("Resetting machine...");
        StateMachine { _state: PhantomData }
    }
}

fn main() {
    // Type-safe measurements
    let distance_m = Measurement::<Meters>::new(100.0);
    let distance_ft = distance_m.to_feet();
    
    println!("Distance: {:.2} meters", Measurement::<Meters>::new(100.0).value());
    println!("Distance: {:.2} feet", distance_ft.value());
    
    let total_distance = Measurement::<Meters>::new(50.0) + Measurement::<Meters>::new(75.0);
    println!("Total distance: {:.2} meters", total_distance.value());
    
    // This would cause a compile error - cannot add different units:
    // let invalid = Measurement::<Meters>::new(10.0) + Measurement::<Feet>::new(5.0);
    
    // Type-safe state machine
    let machine = StateMachine::<Idle>::new();
    let running_machine = machine.start();
    let stopped_machine = running_machine.stop();
    let reset_machine = stopped_machine.reset();
    
    // This would cause a compile error - can't stop an idle machine:
    // let invalid = reset_machine.stop();
    
    println!("State machine operations completed successfully");
}
```

Phantom types enable compile-time type safety without runtime overhead. They  
encode type-level information that the compiler uses for type checking but  
doesn't affect the runtime representation. This pattern is powerful for  
creating type-safe APIs that prevent misuse at compile time.  

## Generic macros integration

Combining generics with macros for powerful code generation patterns.  

```rust
macro_rules! impl_newtype {
    ($name:ident<$generic:ident>) => {
        #[derive(Debug, Clone, PartialEq)]
        struct $name<$generic>($generic);
        
        impl<$generic> $name<$generic> {
            fn new(value: $generic) -> Self {
                $name(value)
            }
            
            fn into_inner(self) -> $generic {
                self.0
            }
            
            fn as_ref(&self) -> &$generic {
                &self.0
            }
        }
        
        impl<$generic> std::ops::Deref for $name<$generic> {
            type Target = $generic;
            
            fn deref(&self) -> &Self::Target {
                &self.0
            }
        }
    };
}

macro_rules! impl_collection {
    ($name:ident, $inner:ty) => {
        #[derive(Debug)]
        struct $name<T> {
            items: $inner,
        }
        
        impl<T> $name<T> {
            fn new() -> Self {
                $name {
                    items: <$inner>::new(),
                }
            }
            
            fn push(&mut self, item: T) {
                self.items.push(item);
            }
            
            fn len(&self) -> usize {
                self.items.len()
            }
            
            fn iter(&self) -> impl Iterator<Item = &T> {
                self.items.iter()
            }
        }
        
        impl<T> std::ops::Index<usize> for $name<T> {
            type Output = T;
            
            fn index(&self, index: usize) -> &Self::Output {
                &self.items[index]
            }
        }
    };
}

// Generate generic newtypes
impl_newtype!(Id<T>);
impl_newtype!(Wrapper<T>);

// Generate generic collections
impl_collection!(MyVec, Vec<T>);

macro_rules! generic_trait_impl {
    ($trait_name:ident for $type_name:ident<$generic:ident> {
        fn $method:ident($($param:ident: $param_type:ty),*) -> $return_type:ty $body:block
    }) => {
        trait $trait_name<$generic> {
            fn $method(&self, $($param: $param_type),*) -> $return_type;
        }
        
        impl<$generic> $trait_name<$generic> for $type_name<$generic> 
        where 
            $generic: Clone,
        {
            fn $method(&self, $($param: $param_type),*) -> $return_type $body
        }
    };
}

generic_trait_impl!(
    Processable for Wrapper<T> {
        fn process(multiplier: i32) -> String {
            format!("Processing {:?} with multiplier {}", self.0, multiplier)
        }
    }
);

fn main() {
    // Using generated newtypes
    let user_id = Id::new(12345);
    let product_id = Id::new("PROD-001".to_string());
    
    println!("User ID: {:?}", user_id);
    println!("Product ID: {:?}", product_id);
    println!("Unwrapped: {}", *user_id);
    
    // Using generated collection
    let mut my_vec = MyVec::new();
    my_vec.push("hello");
    my_vec.push("world");
    my_vec.push("rust");
    
    println!("Collection: {:?}", my_vec);
    println!("Length: {}", my_vec.len());
    println!("First item: {}", my_vec[0]);
    
    for item in my_vec.iter() {
        println!("Item: {}", item);
    }
    
    // Using generated trait
    let wrapped_number = Wrapper::new(42);
    let result = wrapped_number.process(10);
    println!("Process result: {}", result);
}
```

Macros combined with generics enable powerful code generation patterns. The  
macro system can generate generic types, implementations, and traits based  
on patterns. This approach reduces boilerplate while maintaining type safety  
and enables domain-specific language features within Rust's type system.  

## Performance considerations with generics

Understanding monomorphization and generic performance characteristics.  

```rust
use std::time::Instant;

// Generic function - will be monomorphized
fn generic_add<T>(a: T, b: T) -> T
where
    T: std::ops::Add<Output = T>,
{
    a + b
}

// Trait object version - dynamic dispatch
trait Addable {
    fn add_to(&self, other: &Self) -> Box<dyn Addable>;
    fn value(&self) -> i32;
}

impl Addable for i32 {
    fn add_to(&self, other: &Self) -> Box<dyn Addable> {
        Box::new(self + other)
    }
    
    fn value(&self) -> i32 {
        *self
    }
}

impl Addable for f64 {
    fn add_to(&self, other: &Self) -> Box<dyn Addable> {
        Box::new(self + other)
    }
    
    fn value(&self) -> i32 {
        *self as i32
    }
}

fn dynamic_add(a: &dyn Addable, b: &dyn Addable) -> i32 {
    // This is simplified - would need same type checking in real code
    a.value() + b.value()
}

// Zero-cost abstraction example
#[derive(Debug)]
struct Vector<T, const N: usize> {
    data: [T; N],
}

impl<T: Copy + Default, const N: usize> Vector<T, N> {
    fn new() -> Self {
        Vector {
            data: [T::default(); N],
        }
    }
    
    fn set(&mut self, index: usize, value: T) {
        if index < N {
            self.data[index] = value;
        }
    }
    
    fn get(&self, index: usize) -> Option<T> {
        if index < N {
            Some(self.data[index])
        } else {
            None
        }
    }
    
    fn dot_product(&self, other: &Self) -> T
    where
        T: std::ops::Add<Output = T> + std::ops::Mul<Output = T>,
    {
        let mut result = T::default();
        for i in 0..N {
            result = result + self.data[i] * other.data[i];
        }
        result
    }
}

fn benchmark_generic() {
    let start = Instant::now();
    
    for _ in 0..1_000_000 {
        let _result = generic_add(10i32, 20i32);
    }
    
    let duration = start.elapsed();
    println!("Generic (monomorphized): {:?}", duration);
}

fn benchmark_dynamic() {
    let start = Instant::now();
    let a: Box<dyn Addable> = Box::new(10i32);
    let b: Box<dyn Addable> = Box::new(20i32);
    
    for _ in 0..1_000_000 {
        let _result = dynamic_add(a.as_ref(), b.as_ref());
    }
    
    let duration = start.elapsed();
    println!("Dynamic dispatch: {:?}", duration);
}

fn main() {
    println!("Performance comparison:");
    benchmark_generic();
    benchmark_dynamic();
    
    // Zero-cost abstractions
    let mut v1: Vector<f64, 3> = Vector::new();
    let mut v2: Vector<f64, 3> = Vector::new();
    
    v1.set(0, 1.0);
    v1.set(1, 2.0);
    v1.set(2, 3.0);
    
    v2.set(0, 4.0);
    v2.set(1, 5.0);
    v2.set(2, 6.0);
    
    let dot_product = v1.dot_product(&v2);
    println!("Dot product: {:.1}", dot_product);
    
    // The Vector operations compile to simple array operations
    // with no abstraction overhead
    println!("Vector v1: {:?}", v1);
    println!("Vector v2: {:?}", v2);
    
    // Demonstrate compile-time optimization
    let numbers = [1, 2, 3, 4, 5];
    let sum: i32 = numbers.iter().map(|&x| x * 2).sum();
    println!("Optimized sum: {}", sum);
}
```

Rust generics are zero-cost abstractions through monomorphization. The  
compiler generates specialized versions for each concrete type used,  
eliminating runtime overhead. This contrasts with dynamic dispatch through  
trait objects, which have runtime cost. Generic code often optimizes better  
than hand-written specialized code due to compiler analysis.