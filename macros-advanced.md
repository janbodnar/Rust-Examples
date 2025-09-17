# Advanced Rust Macros

Advanced macro patterns in Rust demonstrate sophisticated metaprogramming  
capabilities including procedural macros, attribute macros, and complex  
code generation. These examples showcase macro hygiene, custom derive  
macros, and powerful compile-time transformations that enable domain-  
specific languages and zero-cost abstractions.  

Procedural macros operate on token streams and provide full control over  
code transformation. They come in three forms: derive macros that generate  
implementations, attribute macros that transform items, and function-like  
macros that replace code. These examples build upon the declarative macro  
foundation to explore Rust's most powerful metaprogramming features.  

## Custom derive procedural macro

Procedural derive macros automatically generate trait implementations for  
structs and enums, reducing boilerplate code significantly.  

```rust
// File: Cargo.toml
// [package]
// name = "derive_example"
// version = "0.1.0"
// edition = "2021"
// 
// [lib]
// proc-macro = true
// 
// [dependencies]
// proc-macro2 = "1.0"
// quote = "1.0"
// syn = { version = "2.0", features = ["full"] }

use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput, Data, Fields};

#[proc_macro_derive(Builder)]
pub fn derive_builder(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let name = &input.ident;
    let builder_name = format!("{}Builder", name);
    let builder_ident = syn::Ident::new(&builder_name, name.span());
    
    let fields = match &input.data {
        Data::Struct(data) => match &data.fields {
            Fields::Named(fields) => &fields.named,
            _ => panic!("Builder only supports named fields"),
        },
        _ => panic!("Builder only supports structs"),
    };
    
    let builder_fields = fields.iter().map(|f| {
        let name = &f.ident;
        let ty = &f.ty;
        quote! { #name: Option<#ty> }
    });
    
    let builder_methods = fields.iter().map(|f| {
        let name = &f.ident;
        let ty = &f.ty;
        quote! {
            pub fn #name(mut self, #name: #ty) -> Self {
                self.#name = Some(#name);
                self
            }
        }
    });
    
    let build_fields = fields.iter().map(|f| {
        let name = &f.ident;
        quote! {
            #name: self.#name
                .ok_or_else(|| format!("Missing field: {}", stringify!(#name)))?
        }
    });
    
    let expanded = quote! {
        impl #name {
            pub fn builder() -> #builder_ident {
                #builder_ident {
                    #(#builder_fields,)*
                }
            }
        }
        
        pub struct #builder_ident {
            #(#builder_fields,)*
        }
        
        impl #builder_ident {
            #(#builder_methods)*
            
            pub fn build(self) -> Result<#name, String> {
                Ok(#name {
                    #(#build_fields,)*
                })
            }
        }
    };
    
    TokenStream::from(expanded)
}

// Usage example
#[derive(Builder)]
struct Person {
    name: String,
    age: u32,
    email: String,
}

fn main() {
    let person = Person::builder()
        .name("Alice".to_string())
        .age(30)
        .email("alice@example.com".to_string())
        .build()
        .unwrap();
    
    println!("Built person: {} ({})", person.name, person.age);
}
```

Custom derive macros parse struct definitions using the `syn` crate and  
generate code with `quote`. The macro extracts field information, creates  
a builder struct with optional fields, generates setter methods, and  
implements a build function with validation. This pattern eliminates  
repetitive builder implementations while maintaining type safety.  

## Attribute procedural macro

Attribute macros transform functions, structs, or other items by wrapping  
or modifying their behavior at compile time.  

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, ItemFn, parse::Parse, parse::ParseStream, 
          LitInt, Token, Result};

struct TimingArgs {
    iterations: Option<u64>,
}

impl Parse for TimingArgs {
    fn parse(input: ParseStream) -> Result<Self> {
        if input.is_empty() {
            return Ok(TimingArgs { iterations: None });
        }
        
        let _: Token![iterations] = input.parse()?;
        let _: Token![=] = input.parse()?;
        let iterations: LitInt = input.parse()?;
        
        Ok(TimingArgs {
            iterations: Some(iterations.base10_parse()?),
        })
    }
}

#[proc_macro_attribute]
pub fn timed(args: TokenStream, input: TokenStream) -> TokenStream {
    let args = parse_macro_input!(args as TimingArgs);
    let input_fn = parse_macro_input!(input as ItemFn);
    
    let fn_name = &input_fn.sig.ident;
    let fn_vis = &input_fn.vis;
    let fn_sig = &input_fn.sig;
    let fn_block = &input_fn.block;
    let iterations = args.iterations.unwrap_or(1);
    
    let timed_fn_name = syn::Ident::new(
        &format!("{}_timed", fn_name), 
        fn_name.span()
    );
    
    let expanded = quote! {
        #fn_vis #fn_sig {
            use std::time::Instant;
            
            let start = Instant::now();
            let mut result = None;
            
            for _ in 0..#iterations {
                result = Some((|| #fn_block)());
            }
            
            let duration = start.elapsed();
            println!("Function '{}' executed {} times in {:?} (avg: {:?})",
                stringify!(#fn_name),
                #iterations,
                duration,
                duration / #iterations
            );
            
            result.unwrap()
        }
        
        #[allow(dead_code)]
        #fn_vis fn #timed_fn_name #fn_sig {
            #fn_block
        }
    };
    
    TokenStream::from(expanded)
}

// Usage examples
#[timed]
fn fibonacci(n: u32) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}

#[timed(iterations = 1000)]
fn quick_calculation() -> i32 {
    (1..=100).sum()
}

#[timed(iterations = 10)]
fn complex_operation() -> String {
    let mut result = String::new();
    for i in 1..=10 {
        result.push_str(&format!("Item {} ", i));
    }
    result.trim().to_string()
}

fn main() {
    println!("=== Timing Examples ===");
    
    let fib_result = fibonacci(10);
    println!("Fibonacci(10) = {}\n", fib_result);
    
    let calc_result = quick_calculation();
    println!("Quick calculation result = {}\n", calc_result);
    
    let complex_result = complex_operation();
    println!("Complex operation result = {}\n", complex_result);
}
```

Attribute macros can parse arguments and transform functions completely.  
This timing macro adds performance measurement around function execution,  
preserves the original function interface, and provides configurable  
iteration counts. The macro demonstrates argument parsing, function  
analysis, and code generation patterns essential for attribute macros.  

## Function-like procedural macro

Function-like procedural macros provide custom syntax that resembles  
function calls but operate on arbitrary token streams.  

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, parse::Parse, parse::ParseStream, 
          LitStr, Token, Ident, Result, punctuated::Punctuated};

struct SqlQuery {
    operation: Ident,
    table: Ident,
    fields: Vec<Ident>,
    conditions: Option<LitStr>,
}

impl Parse for SqlQuery {
    fn parse(input: ParseStream) -> Result<Self> {
        let operation: Ident = input.parse()?;
        
        match operation.to_string().as_str() {
            "SELECT" => {
                let fields = Punctuated::<Ident, Token![,]>::parse_separated_nonempty(input)?
                    .into_iter().collect();
                let _: Token![FROM] = input.parse()?;
                let table: Ident = input.parse()?;
                
                let conditions = if input.parse::<Token![WHERE]>().is_ok() {
                    Some(input.parse::<LitStr>()?)
                } else {
                    None
                };
                
                Ok(SqlQuery { operation, table, fields, conditions })
            }
            "INSERT" => {
                let _: Token![INTO] = input.parse()?;
                let table: Ident = input.parse()?;
                let content;
                syn::parenthesized!(content in input);
                let fields = Punctuated::<Ident, Token![,]>::parse_separated_nonempty(&content)?
                    .into_iter().collect();
                
                Ok(SqlQuery { operation, table, fields, conditions: None })
            }
            _ => Err(syn::Error::new(operation.span(), "Unsupported SQL operation"))
        }
    }
}

#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
    let query = parse_macro_input!(input as SqlQuery);
    
    let operation = &query.operation;
    let table = &query.table;
    let fields = &query.fields;
    
    let sql_string = match operation.to_string().as_str() {
        "SELECT" => {
            let field_names: Vec<String> = fields.iter()
                .map(|f| f.to_string())
                .collect();
            let field_list = field_names.join(", ");
            
            if let Some(condition) = &query.conditions {
                format!("SELECT {} FROM {} WHERE {}", 
                       field_list, table, condition.value())
            } else {
                format!("SELECT {} FROM {}", field_list, table)
            }
        }
        "INSERT" => {
            let field_names: Vec<String> = fields.iter()
                .map(|f| f.to_string())
                .collect();
            let field_list = field_names.join(", ");
            let placeholders = vec!["?"; fields.len()].join(", ");
            
            format!("INSERT INTO {} ({}) VALUES ({})", 
                   table, field_list, placeholders)
        }
        _ => unreachable!()
    };
    
    let expanded = quote! {
        {
            #[allow(unused_imports)]
            use std::collections::HashMap;
            
            struct SqlQueryBuilder {
                query: String,
                params: Vec<String>,
            }
            
            impl SqlQueryBuilder {
                fn new(query: &str) -> Self {
                    Self {
                        query: query.to_string(),
                        params: Vec::new(),
                    }
                }
                
                fn bind<T: ToString>(mut self, param: T) -> Self {
                    self.params.push(param.to_string());
                    self
                }
                
                fn execute(&self) -> String {
                    let mut result = self.query.clone();
                    for (i, param) in self.params.iter().enumerate() {
                        result = result.replacen("?", param, 1);
                    }
                    result
                }
            }
            
            SqlQueryBuilder::new(#sql_string)
        }
    };
    
    TokenStream::from(expanded)
}

// Advanced SQL macro with better type safety
#[proc_macro]
pub fn typed_sql(input: TokenStream) -> TokenStream {
    let input_str = input.to_string();
    
    let expanded = quote! {
        {
            pub struct TypedQuery<T> {
                query: String,
                _phantom: std::marker::PhantomData<T>,
            }
            
            impl<T> TypedQuery<T> {
                pub fn new(query: &str) -> Self {
                    Self {
                        query: query.to_string(),
                        _phantom: std::marker::PhantomData,
                    }
                }
                
                pub fn get_query(&self) -> &str {
                    &self.query
                }
            }
            
            TypedQuery::<()>::new(#input_str)
        }
    };
    
    TokenStream::from(expanded)
}

// Usage examples
fn main() {
    println!("=== Function-like Procedural Macros ===");
    
    // SQL builder macro
    let select_query = sql!(SELECT name, age FROM users WHERE "age > 18");
    let bound_query = select_query
        .bind("users")
        .bind(18)
        .execute();
    println!("Select query: {}", bound_query);
    
    let insert_query = sql!(INSERT INTO products (name, price, category));
    let insert_bound = insert_query
        .bind("Laptop")
        .bind("999.99")
        .bind("Electronics")
        .execute();
    println!("Insert query: {}", insert_bound);
    
    // Typed SQL macro
    let typed_query = typed_sql!("SELECT * FROM users WHERE active = true");
    println!("Typed query: {}", typed_query.get_query());
}
```

Function-like procedural macros parse custom syntax and generate complex  
code structures. The SQL macro demonstrates domain-specific language  
creation with syntax validation, parameter binding, and type-safe query  
building. These macros enable natural syntax for specialized domains  
while maintaining Rust's compile-time safety guarantees.  

## Macro hygiene demonstration

Macro hygiene prevents variable name collisions between macro-generated  
code and surrounding context, ensuring predictable behavior.  

```rust
// Demonstrating hygiene violations and solutions
macro_rules! unhygienic_counter {
    () => {
        {
            let count = 0;  // This can clash with user variables
            let increment = || {
                // This won't work due to borrowing rules
                println!("Count: {}", count + 1);
            };
            increment();
        }
    };
}

macro_rules! hygienic_counter {
    () => {
        {
            // Using hygiene-safe approaches
            let __macro_internal_count = 0;  // Less likely to clash
            let __macro_internal_increment = |val: i32| -> i32 {
                val + 1
            };
            
            println!("Hygienic count: {}", 
                    __macro_internal_increment(__macro_internal_count));
            __macro_internal_count
        }
    };
}

// Advanced hygiene with unique identifiers
macro_rules! unique_counter {
    ($name:ident) => {
        {
            paste::paste! {
                let [<__counter_ $name _value>] = std::cell::Cell::new(0);
                let [<__counter_ $name _inc>] = || {
                    let current = [<__counter_ $name _value>].get();
                    [<__counter_ $name _value>].set(current + 1);
                    current + 1
                };
                
                struct [<Counter $name>] {
                    increment: fn() -> i32,
                }
                
                [<Counter $name>] {
                    increment: [<__counter_ $name _inc>],
                }
            }
        }
    };
}

// Hygiene with proper variable scoping
macro_rules! scoped_calculation {
    ($expr:expr) => {
        {
            // Create a clean scope to prevent variable leakage
            let __internal_result = (|| {
                // All calculations happen in this closure
                let intermediate = $expr;
                let processed = intermediate * 2;
                let final_value = processed + 1;
                final_value
            })();
            
            // Return only the final result
            __internal_result
        }
    };
}

// Demonstration of variable capture issues
macro_rules! capture_demo {
    ($var:ident, $value:expr) => {
        {
            // This macro captures the variable name, not its value
            let $var = $value;
            move || {
                println!("Captured variable {}: {}", stringify!($var), $var);
                $var
            }
        }
    };
}

// Safe variable capture with explicit hygiene
macro_rules! safe_capture {
    ($var:ident, $value:expr) => {
        {
            // Explicitly bind the value to avoid hygiene issues
            let __captured_value = $value;
            move || {
                println!("Safely captured {}: {}", 
                        stringify!($var), __captured_value);
                __captured_value
            }
        }
    };
}

// Macro that demonstrates identifier hygiene
macro_rules! method_generator {
    ($struct_name:ident, $field:ident, $method_name:ident) => {
        impl $struct_name {
            pub fn $method_name(&self) -> &String {
                &self.$field
            }
            
            // Using paste for dynamic method names
            paste::paste! {
                pub fn [<set_ $field>](&mut self, value: String) {
                    self.$field = value;
                }
                
                pub fn [<$field _len>](&self) -> usize {
                    self.$field.len()
                }
            }
        }
    };
}

struct Person {
    name: String,
    email: String,
}

method_generator!(Person, name, get_name);
method_generator!(Person, email, get_email);

fn main() {
    println!("=== Macro Hygiene Examples ===");
    
    // Test hygienic counter
    let count = 42;  // This won't interfere with macro variables
    let result = hygienic_counter!();
    println!("External count: {}, Macro result: {}", count, result);
    
    // Test scoped calculation
    let x = 10;
    let calculated = scoped_calculation!(x * 3);
    println!("Scoped calculation result: {}", calculated);
    
    // Test variable capture
    let value = "test";
    let captured_fn = safe_capture!(value, value.to_string());
    let captured_result = captured_fn();
    println!("Captured result: {}", captured_result);
    
    // Test method generation
    let mut person = Person {
        name: "Alice".to_string(),
        email: "alice@example.com".to_string(),
    };
    
    println!("Name: {}", person.get_name());
    println!("Email: {}", person.get_email());
    println!("Name length: {}", person.name_len());
    
    person.set_name("Bob".to_string());
    println!("Updated name: {}", person.get_name());
}
```

Macro hygiene prevents naming conflicts by creating separate identifier  
contexts for macro-generated code. Proper hygiene uses unique prefixes,  
explicit scoping, and careful variable capture. The `paste` crate enables  
dynamic identifier generation while maintaining hygiene. Understanding  
these patterns prevents subtle bugs in complex macro systems.  

## Compile-time type introspection

Macros that examine and analyze types at compile time to generate  
specialized implementations based on type characteristics.  

```rust
macro_rules! type_info {
    ($t:ty) => {
        {
            use std::mem;
            
            struct TypeInfo {
                name: &'static str,
                size: usize,
                align: usize,
                is_copy: bool,
                is_send: bool,
                is_sync: bool,
            }
            
            impl TypeInfo {
                fn new<T: 'static>() -> Self {
                    Self {
                        name: std::any::type_name::<T>(),
                        size: mem::size_of::<T>(),
                        align: mem::align_of::<T>(),
                        is_copy: std::mem::needs_drop::<T>() == false,
                        is_send: std::marker::PhantomData::<T>::default().is_send(),
                        is_sync: std::marker::PhantomData::<T>::default().is_sync(),
                    }
                }
                
                fn display(&self) {
                    println!("Type: {}", self.name);
                    println!("  Size: {} bytes", self.size);
                    println!("  Alignment: {} bytes", self.align);
                    println!("  Copy: {}", self.is_copy);
                    println!("  Send: {}", self.is_send);
                    println!("  Sync: {}", self.is_sync);
                }
            }
            
            trait IsSend {
                fn is_send(&self) -> bool { false }
            }
            
            trait IsSync {
                fn is_sync(&self) -> bool { false }
            }
            
            impl<T: Send> IsSend for std::marker::PhantomData<T> {
                fn is_send(&self) -> bool { true }
            }
            
            impl<T: Sync> IsSync for std::marker::PhantomData<T> {
                fn is_sync(&self) -> bool { true }
            }
            
            TypeInfo::new::<$t>()
        }
    };
}

// Advanced type analysis with trait bounds checking
macro_rules! analyze_type {
    ($t:ty) => {
        {
            pub struct TypeAnalysis;
            
            impl TypeAnalysis {
                pub fn check_traits() -> &'static str {
                    let mut info = String::new();
                    
                    // Check common traits at compile time
                    if Self::implements_debug() {
                        info.push_str("Debug ");
                    }
                    if Self::implements_clone() {
                        info.push_str("Clone ");
                    }
                    if Self::implements_copy() {
                        info.push_str("Copy ");
                    }
                    if Self::implements_send() {
                        info.push_str("Send ");
                    }
                    if Self::implements_sync() {
                        info.push_str("Sync ");
                    }
                    
                    Box::leak(info.into_boxed_str())
                }
                
                #[allow(dead_code)]
                const fn implements_debug() -> bool {
                    const fn check<T: std::fmt::Debug>() -> bool { true }
                    const fn check_fallback<T>() -> bool { false }
                    
                    // This is a simplified version - actual implementation would need
                    // more complex trait checking
                    true  // Placeholder
                }
                
                const fn implements_clone() -> bool { true }
                const fn implements_copy() -> bool { true }
                const fn implements_send() -> bool { true }
                const fn implements_sync() -> bool { true }
            }
            
            TypeAnalysis
        }
    };
}

// Conditional compilation based on type properties
macro_rules! generate_methods {
    ($struct_name:ident, $field_type:ty) => {
        impl $struct_name {
            // Generate different methods based on type characteristics
            
            // Always generate basic methods
            pub fn new(value: $field_type) -> Self {
                Self { value }
            }
            
            pub fn get(&self) -> &$field_type {
                &self.value
            }
            
            // Conditionally generate methods based on type traits
            generate_methods!(@check_copy $struct_name, $field_type);
            generate_methods!(@check_clone $struct_name, $field_type);
            generate_methods!(@check_debug $struct_name, $field_type);
        }
    };
    
    (@check_copy $struct_name:ident, $field_type:ty) => {
        // Generate copy method if type implements Copy
        impl $struct_name {
            pub fn get_copy(&self) -> $field_type 
            where 
                $field_type: Copy,
            {
                self.value
            }
        }
    };
    
    (@check_clone $struct_name:ident, $field_type:ty) => {
        // Generate clone method if type implements Clone
        impl $struct_name {
            pub fn get_clone(&self) -> $field_type 
            where 
                $field_type: Clone,
            {
                self.value.clone()
            }
        }
    };
    
    (@check_debug $struct_name:ident, $field_type:ty) => {
        // Generate debug method if type implements Debug
        impl $struct_name {
            pub fn debug_print(&self) 
            where 
                $field_type: std::fmt::Debug,
            {
                println!("{:?}", self.value);
            }
        }
    };
}

// Wrapper struct for demonstration
struct Container<T> {
    value: T,
}

// Generate methods for different types
generate_methods!(Container<i32>, i32);
generate_methods!(Container<String>, String);
generate_methods!(Container<Vec<u8>>, Vec<u8>);

fn main() {
    println!("=== Type Introspection Examples ===");
    
    // Basic type info
    let int_info = type_info!(i32);
    int_info.display();
    println!();
    
    let string_info = type_info!(String);
    string_info.display();
    println!();
    
    let vec_info = type_info!(Vec<u8>);
    vec_info.display();
    println!();
    
    // Container usage with different types
    let int_container = Container::new(42);
    println!("Int container: {}", int_container.get());
    println!("Int copy: {}", int_container.get_copy());
    int_container.debug_print();
    
    let string_container = Container::new("Hello".to_string());
    println!("String container: {}", string_container.get());
    println!("String clone: {}", string_container.get_clone());
    string_container.debug_print();
    
    let vec_container = Container::new(vec![1, 2, 3]);
    println!("Vec container: {:?}", vec_container.get());
    println!("Vec clone: {:?}", vec_container.get_clone());
    vec_container.debug_print();
}
```

Type introspection macros analyze type characteristics at compile time  
and generate appropriate implementations. The type_info macro provides  
runtime information about size, alignment, and trait implementations.  
Conditional method generation creates specialized interfaces based on  
available type traits. This enables writing generic code that adapts  
automatically to different type capabilities.  

## Advanced pattern matching macros

Sophisticated pattern matching macros that handle complex data structures  
and provide powerful destructuring capabilities.  

```rust
// Enhanced pattern matching for nested structures
macro_rules! deep_match {
    // Match nested tuples with wildcards
    ($value:expr, ($($pattern:tt)*) => $action:expr) => {
        match $value {
            ($($pattern)*) => $action,
            _ => panic!("Pattern match failed"),
        }
    };
    
    // Match multiple patterns with fallback
    ($value:expr, {
        $($pattern:pat => $action:expr),+,
        _ => $fallback:expr
    }) => {
        match $value {
            $($pattern => $action,)+
            _ => $fallback,
        }
    };
    
    // Match with guards and destructuring
    ($value:expr, {
        $($pattern:pat if $guard:expr => $action:expr),+
    }) => {
        match $value {
            $($pattern if $guard => $action,)+
            _ => panic!("No pattern matched with guards"),
        }
    };
}

// Regex-like pattern matching for strings
macro_rules! string_match {
    ($string:expr, {
        $($pattern:literal => $action:expr),+
    }) => {
        {
            let s = $string;
            $(
                if s.contains($pattern) {
                    $action
                } else
            )*
            {
                panic!("No string pattern matched");
            }
        }
    };
    
    // Pattern matching with capture groups
    ($string:expr, {
        starts_with $prefix:literal => $action:expr,
        ends_with $suffix:literal => $action2:expr,
        contains $substring:literal => $action3:expr,
        _ => $fallback:expr
    }) => {
        {
            let s = $string;
            if s.starts_with($prefix) {
                $action
            } else if s.ends_with($suffix) {
                $action2
            } else if s.contains($substring) {
                $action3
            } else {
                $fallback
            }
        }
    };
}

// Type-based pattern matching
macro_rules! type_match {
    ($value:expr, {
        $($type:ty => $action:expr),+
    }) => {
        {
            let val = $value;
            let type_id = std::any::TypeId::of_val(&val);
            
            $(
                if type_id == std::any::TypeId::of::<$type>() {
                    // Safe cast because we checked the type
                    let typed_val = unsafe { 
                        std::ptr::read(&val as *const _ as *const $type) 
                    };
                    std::mem::forget(val);  // Prevent double drop
                    $action(typed_val)
                } else
            )*
            {
                panic!("No type pattern matched");
            }
        }
    };
}

// Advanced destructuring for custom types
macro_rules! destructure {
    // Destructure struct fields with defaults
    ($struct_expr:expr, { 
        $($field:ident $(: $default:expr)?),*
    }) => {
        {
            let s = $struct_expr;
            $(
                let $field = s.$field $(.unwrap_or($default))?;
            )*
        }
    };
    
    // Destructure enums with pattern extraction
    ($enum_expr:expr, $variant:ident { 
        $($field:ident),*
    }) => {
        match $enum_expr {
            $variant { $($field,)* } => {
                // Fields are now available in scope
                ($($field,)*)
            },
            _ => panic!("Wrong enum variant"),
        }
    };
    
    // Destructure with transformation
    ($expr:expr, [
        $($index:literal => $name:ident),*
    ]) => {
        {
            let collection = $expr;
            $(
                let $name = collection.get($index)
                    .expect(&format!("Index {} out of bounds", $index));
            )*
        }
    };
}

// Multi-level pattern matching with conditions
macro_rules! complex_match {
    ($value:expr, {
        $($condition:expr => {
            $($inner_pattern:pat => $inner_action:expr),+
        }),+
    }) => {
        {
            let val = $value;
            $(
                if $condition {
                    match val {
                        $($inner_pattern => $inner_action,)+
                        _ => panic!("Inner pattern match failed"),
                    }
                } else
            )*
            {
                panic!("No condition matched");
            }
        }
    };
}

// Usage examples
#[derive(Debug)]
enum Message {
    Text { content: String, priority: u8 },
    Image { url: String, width: u32, height: u32 },
    Video { url: String, duration: u32 },
}

fn main() {
    println!("=== Advanced Pattern Matching ===");
    
    // Deep tuple matching
    let nested_tuple = ((1, 2), (3, 4), 5);
    let result = deep_match!(nested_tuple, ((a, b), (c, d), e) => a + b + c + d + e);
    println!("Nested tuple sum: {}", result);
    
    // String pattern matching
    let text = "Hello world";
    let message = string_match!(text, {
        starts_with "Hello" => "Greeting detected",
        ends_with "world" => "World reference found", 
        contains "lo" => "Contains 'lo'",
        _ => "No pattern found"
    });
    println!("String match result: {}", message);
    
    // Complex matching with conditions
    let numbers = vec![1, 2, 3, 4, 5];
    let analysis = complex_match!(numbers.len(), {
        numbers.len() > 3 => {
            5 => "Exactly 5 elements",
            4 => "Exactly 4 elements", 
            n if n > 5 => "More than 5 elements"
        },
        numbers.len() <= 3 => {
            1 => "Single element",
            2 => "Two elements",
            3 => "Three elements"
        }
    });
    println!("Collection analysis: {}", analysis);
    
    // Enum destructuring
    let msg = Message::Text { 
        content: "Important message".to_string(), 
        priority: 1 
    };
    
    let (content, priority) = destructure!(msg, Text { content, priority });
    println!("Destructured message: '{}' (priority: {})", content, priority);
    
    // Array destructuring with indexing
    let data = vec!["first", "second", "third", "fourth"];
    destructure!(data, [
        0 => first,
        2 => third
    ]);
    println!("Extracted elements: first='{}', third='{}'", first, third);
}
```

Advanced pattern matching macros extend Rust's pattern matching with  
specialized syntax for complex scenarios. Deep matching handles nested  
structures, string matching provides regex-like capabilities, and type  
matching enables runtime type dispatch. These patterns create more  
expressive code while maintaining compile-time safety and performance.  

## Async macro transformations

Macros that transform and enhance async code patterns, providing  
abstractions for common async operations and state management.  

```rust
// Async function timing and monitoring
macro_rules! async_timed {
    ($name:ident($($param:ident: $param_type:ty),*) -> $return_type:ty $body:block) => {
        async fn $name($($param: $param_type),*) -> $return_type {
            use std::time::Instant;
            
            let start = Instant::now();
            println!("Starting async function '{}'", stringify!($name));
            
            let result = async move $body.await;
            
            let duration = start.elapsed();
            println!("Async function '{}' completed in {:?}", 
                    stringify!($name), duration);
            
            result
        }
    };
}

// Async retry mechanism with exponential backoff
macro_rules! async_retry {
    ($operation:expr, max_attempts: $max:expr, delay: $delay:expr) => {
        {
            use std::time::Duration;
            use tokio::time::sleep;
            
            let mut attempts = 0;
            let mut current_delay = $delay;
            
            loop {
                attempts += 1;
                
                match $operation.await {
                    Ok(result) => break Ok(result),
                    Err(e) if attempts >= $max => break Err(e),
                    Err(e) => {
                        println!("Attempt {} failed: {:?}. Retrying in {:?}...", 
                                attempts, e, current_delay);
                        sleep(current_delay).await;
                        current_delay *= 2;  // Exponential backoff
                    }
                }
            }
        }
    };
}

// Async stream processing macro
macro_rules! async_stream_process {
    ($stream:expr, |$item:ident| $process:block) => {
        {
            use futures::StreamExt;
            
            let mut stream = $stream;
            let mut results = Vec::new();
            
            while let Some($item) = stream.next().await {
                let processed = async move $process.await;
                results.push(processed);
            }
            
            results
        }
    };
    
    // With error handling
    ($stream:expr, |$item:ident| $process:block, on_error: |$error:ident| $error_handler:block) => {
        {
            use futures::StreamExt;
            
            let mut stream = $stream;
            let mut results = Vec::new();
            let mut errors = Vec::new();
            
            while let Some($item) = stream.next().await {
                match async move { $process }.await {
                    Ok(result) => results.push(result),
                    Err($error) => {
                        $error_handler;
                        errors.push($error);
                    }
                }
            }
            
            (results, errors)
        }
    };
}

// Async concurrent execution with limits
macro_rules! async_concurrent {
    (limit: $limit:expr, tasks: [$($task:expr),+]) => {
        {
            use futures::stream::{self, StreamExt};
            use std::collections::VecDeque;
            
            let tasks: VecDeque<_> = vec![$($task),+].into();
            let mut active_tasks = Vec::new();
            let mut results = Vec::new();
            let mut remaining_tasks = tasks;
            
            // Start initial batch
            while active_tasks.len() < $limit && !remaining_tasks.is_empty() {
                let task = remaining_tasks.pop_front().unwrap();
                active_tasks.push(Box::pin(task));
            }
            
            while !active_tasks.is_empty() {
                let (result, index, remaining) = futures::future::select_all(active_tasks).await;
                results.push(result);
                active_tasks = remaining;
                
                // Start next task if available
                if let Some(next_task) = remaining_tasks.pop_front() {
                    active_tasks.push(Box::pin(next_task));
                }
            }
            
            results
        }
    };
}

// Async state machine macro
macro_rules! async_state_machine {
    ($name:ident {
        states: { $($state:ident),+ },
        initial: $initial:ident,
        transitions: {
            $($from:ident -> $to:ident on $event:ident($($param:ident: $param_type:ty),*) => $action:block),+
        }
    }) => {
        #[derive(Debug, Clone, PartialEq)]
        enum State {
            $($state),+
        }
        
        struct $name {
            current_state: State,
        }
        
        impl $name {
            fn new() -> Self {
                Self {
                    current_state: State::$initial,
                }
            }
            
            fn current_state(&self) -> &State {
                &self.current_state
            }
            
            $(
                async fn $event(&mut self, $($param: $param_type),*) -> Result<(), String> {
                    match self.current_state {
                        State::$from => {
                            let result: Result<(), String> = async $action.await;
                            match result {
                                Ok(()) => {
                                    self.current_state = State::$to;
                                    println!("Transitioned from {:?} to {:?}", 
                                            State::$from, State::$to);
                                    Ok(())
                                },
                                Err(e) => Err(e),
                            }
                        },
                        _ => Err(format!("Invalid transition: {} not allowed from {:?}", 
                                       stringify!($event), self.current_state)),
                    }
                }
            )+
        }
    };
}

// Usage examples
async_timed!(fetch_data(url: String) -> Result<String, String> {
    // Simulate network request
    tokio::time::sleep(std::time::Duration::from_millis(100)).await;
    if url.starts_with("https://") {
        Ok(format!("Data from {}", url))
    } else {
        Err("Invalid URL".to_string())
    }
});

async fn unreliable_operation() -> Result<String, String> {
    use rand::Rng;
    let mut rng = rand::thread_rng();
    if rng.gen_bool(0.3) {  // 30% success rate
        Ok("Success!".to_string())
    } else {
        Err("Random failure".to_string())
    }
}

// Define an async state machine
async_state_machine!(ConnectionManager {
    states: { Disconnected, Connecting, Connected, Error },
    initial: Disconnected,
    transitions: {
        Disconnected -> Connecting on connect(address: String) => {
            println!("Connecting to {}", address);
            tokio::time::sleep(std::time::Duration::from_millis(50)).await;
            Ok(())
        },
        Connecting -> Connected on establish() => {
            println!("Connection established");
            Ok(())
        },
        Connected -> Disconnected on disconnect() => {
            println!("Disconnecting");
            Ok(())
        },
        Connected -> Error on error(msg: String) => {
            println!("Connection error: {}", msg);
            Ok(())
        }
    }
});

#[tokio::main]
async fn main() {
    println!("=== Async Macro Transformations ===");
    
    // Timed async function
    let result = fetch_data("https://example.com".to_string()).await;
    println!("Fetch result: {:?}", result);
    
    // Retry mechanism
    let retry_result = async_retry!(
        unreliable_operation(),
        max_attempts: 5,
        delay: std::time::Duration::from_millis(100)
    );
    println!("Retry result: {:?}", retry_result);
    
    // Concurrent execution with limits
    let tasks = vec![
        fetch_data("https://api1.com".to_string()),
        fetch_data("https://api2.com".to_string()),
        fetch_data("https://api3.com".to_string()),
        fetch_data("https://api4.com".to_string()),
    ];
    
    let concurrent_results = async_concurrent!(limit: 2, tasks: [
        fetch_data("https://service1.com".to_string()),
        fetch_data("https://service2.com".to_string()),
        fetch_data("https://service3.com".to_string())
    ]);
    println!("Concurrent results: {:?}", concurrent_results);
    
    // State machine usage
    let mut connection = ConnectionManager::new();
    println!("Initial state: {:?}", connection.current_state());
    
    let _ = connection.connect("192.168.1.1".to_string()).await;
    println!("After connect: {:?}", connection.current_state());
    
    let _ = connection.establish().await;
    println!("After establish: {:?}", connection.current_state());
    
    let _ = connection.disconnect().await;
    println!("After disconnect: {:?}", connection.current_state());
}
```

Async macro transformations simplify complex asynchronous programming  
patterns. Timing macros add monitoring to async functions, retry macros  
implement resilient operation patterns, and state machines provide  
structured async workflow management. These abstractions reduce  
boilerplate while maintaining the performance benefits of async Rust.  

## Generic type constraint macros

Advanced macros that work with generic types and enforce complex  
trait bounds and type relationships.  

```rust
// Macro for creating bounded generic collections
macro_rules! bounded_collection {
    ($name:ident<$t:ident> where $t:ident: $($bound:path),+) => {
        struct $name<$t>
        where
            $t: $($bound +)+,
        {
            items: Vec<$t>,
            capacity: usize,
        }
        
        impl<$t> $name<$t>
        where
            $t: $($bound +)+,
        {
            fn new(capacity: usize) -> Self {
                Self {
                    items: Vec::with_capacity(capacity),
                    capacity,
                }
            }
            
            fn push(&mut self, item: $t) -> Result<(), $t> {
                if self.items.len() < self.capacity {
                    self.items.push(item);
                    Ok(())
                } else {
                    Err(item)
                }
            }
            
            fn len(&self) -> usize {
                self.items.len()
            }
            
            fn is_full(&self) -> bool {
                self.items.len() == self.capacity
            }
        }
        
        // Add trait-specific methods
        bounded_collection!(@impl_debug $name<$t> where $t: $($bound),+);
        bounded_collection!(@impl_clone $name<$t> where $t: $($bound),+);
    };
    
    (@impl_debug $name:ident<$t:ident> where $t:ident: $($bound:path),+) => {
        impl<$t> std::fmt::Debug for $name<$t>
        where
            $t: $($bound +)+ std::fmt::Debug,
        {
            fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
                f.debug_struct(stringify!($name))
                    .field("items", &self.items)
                    .field("capacity", &self.capacity)
                    .finish()
            }
        }
    };
    
    (@impl_clone $name:ident<$t:ident> where $t:ident: $($bound:path),+) => {
        impl<$t> Clone for $name<$t>
        where
            $t: $($bound +)+ Clone,
        {
            fn clone(&self) -> Self {
                Self {
                    items: self.items.clone(),
                    capacity: self.capacity,
                }
            }
        }
    };
}

// Macro for type-safe builder with constraints
macro_rules! typed_builder {
    ($name:ident {
        required: { $($req_field:ident: $req_type:ty),+ },
        optional: { $($opt_field:ident: $opt_type:ty),* }
    }) => {
        // Builder state types
        struct Required;
        struct Optional;
        
        // Builder struct with type states
        struct $name<$($req_field = Required),+> {
            $($req_field: Option<$req_type>,)+
            $($opt_field: Option<$opt_type>,)*
        }
        
        // Target struct
        struct Built {
            $($req_field: $req_type,)+
            $($opt_field: Option<$opt_type>,)*
        }
        
        impl $name {
            fn new() -> $name<$($req_field = Required),+> {
                $name {
                    $($req_field: None,)+
                    $($opt_field: None,)*
                }
            }
        }
        
        // Implementation for each required field
        $(
            impl<$($req_field),+> $name<$($req_field),+>
            where
                $req_field: std::marker::PhantomData<Required>,
            {
                fn $req_field(self, value: $req_type) -> $name<$($req_field = Optional),+> {
                    $name {
                        $req_field: Some(value),
                        ..$self
                    }
                }
            }
        )+
        
        // Optional field setters
        $(
            impl<$($req_field),+> $name<$($req_field),+> {
                fn $opt_field(mut self, value: $opt_type) -> Self {
                    self.$opt_field = Some(value);
                    self
                }
            }
        )*
        
        // Build method only available when all required fields are set
        impl $name<$(Optional),+> {
            fn build(self) -> Built {
                Built {
                    $($req_field: self.$req_field.unwrap(),)+
                    $($opt_field: self.$opt_field,)*
                }
            }
        }
    };
}

// Macro for creating type-safe enumerations with constraints
macro_rules! constrained_enum {
    ($name:ident<$t:ident: $bound:path> {
        $($variant:ident($value:ty)),+
    }) => {
        enum $name<$t>
        where
            $t: $bound,
        {
            $($variant($value),)+
        }
        
        impl<$t> $name<$t>
        where
            $t: $bound,
        {
            fn type_name(&self) -> &'static str {
                match self {
                    $(Self::$variant(_) => stringify!($variant),)+
                }
            }
            
            // Generic processing method
            fn process<F, R>(&self, processor: F) -> R
            where
                F: Fn(&dyn std::any::Any) -> R,
            {
                match self {
                    $(Self::$variant(value) => processor(value as &dyn std::any::Any),)+
                }
            }
        }
        
        // Implement conversion methods
        $(
            impl<$t> $name<$t>
            where
                $t: $bound,
            {
                paste::paste! {
                    fn [<as_ $variant:lower>](&self) -> Option<&$value> {
                        match self {
                            Self::$variant(value) => Some(value),
                            _ => None,
                        }
                    }
                    
                    fn [<is_ $variant:lower>](&self) -> bool {
                        matches!(self, Self::$variant(_))
                    }
                }
            }
        )+
    };
}

// Advanced trait bound checking macro
macro_rules! require_traits {
    ($t:ty: $($trait:path),+) => {
        {
            fn check_traits<T>()
            where
                T: $($trait +)+,
            {
                // This function only compiles if T implements all required traits
            }
            
            check_traits::<$t>();
            
            struct TraitChecker<T>(std::marker::PhantomData<T>);
            
            impl<T> TraitChecker<T>
            where
                T: $($trait +)+,
            {
                fn new() -> Self {
                    Self(std::marker::PhantomData)
                }
                
                fn traits() -> &'static str {
                    concat!("Required traits: ", $(stringify!($trait), " + ",)+ "∅")
                }
            }
            
            TraitChecker::<$t>::new()
        }
    };
}

// Usage examples
bounded_collection!(NumberCollection<T> where T: std::fmt::Display, PartialOrd, Copy);
constrained_enum!(Value<T: std::fmt::Debug> {
    Integer(i64),
    Float(f64),
    Text(String),
    Custom(T)
});

fn main() {
    println!("=== Generic Type Constraint Macros ===");
    
    // Bounded collection usage
    let mut numbers = NumberCollection::<i32>::new(3);
    numbers.push(1).unwrap();
    numbers.push(2).unwrap();
    numbers.push(3).unwrap();
    
    println!("Collection length: {}", numbers.len());
    println!("Collection is full: {}", numbers.is_full());
    
    if let Err(rejected) = numbers.push(4) {
        println!("Rejected item: {}", rejected);
    }
    
    // Constrained enum usage
    let int_value = Value::<String>::Integer(42);
    let text_value = Value::<String>::Text("Hello".to_string());
    let custom_value = Value::<String>::Custom("Custom".to_string());
    
    println!("Int value type: {}", int_value.type_name());
    println!("Text value type: {}", text_value.type_name());
    println!("Custom value type: {}", custom_value.type_name());
    
    println!("Is integer: {}", int_value.is_integer());
    println!("Is text: {}", text_value.is_text());
    
    // Trait requirement checking
    let checker = require_traits!(i32: std::fmt::Display, Clone, Copy);
    println!("{}", checker.traits());
    
    // This would fail to compile:
    // let bad_checker = require_traits!(std::fs::File: Copy);
}
```

Generic type constraint macros enable sophisticated type-level programming  
with compile-time guarantees. Bounded collections enforce trait requirements  
while providing specialized functionality. Typed builders use phantom types  
to ensure correct construction order. Constrained enums combine generics  
with trait bounds for flexible yet safe abstractions. These patterns  
enable library authors to create powerful, type-safe APIs.  

## Compile-time computation macros

Macros that perform complex calculations and code generation at compile  
time, enabling zero-runtime-cost computations and optimizations.  

```rust
// Compile-time mathematical computations
macro_rules! const_math {
    // Factorial calculation
    (factorial $n:literal) => {
        {
            const fn factorial(n: u64) -> u64 {
                if n <= 1 { 1 } else { n * factorial(n - 1) }
            }
            factorial($n)
        }
    };
    
    // Power calculation
    (power $base:literal ^ $exp:literal) => {
        {
            const fn power(base: u64, exp: u64) -> u64 {
                if exp == 0 { 1 } else { base * power(base, exp - 1) }
            }
            power($base, $exp)
        }
    };
    
    // Fibonacci sequence
    (fib $n:literal) => {
        {
            const fn fibonacci(n: u64) -> u64 {
                if n <= 1 { n } else { fibonacci(n - 1) + fibonacci(n - 2) }
            }
            fibonacci($n)
        }
    };
    
    // Prime checking
    (is_prime $n:literal) => {
        {
            const fn is_prime(n: u64) -> bool {
                if n < 2 { return false; }
                if n == 2 { return true; }
                if n % 2 == 0 { return false; }
                
                let mut i = 3;
                while i * i <= n {
                    if n % i == 0 { return false; }
                    i += 2;
                }
                true
            }
            is_prime($n)
        }
    };
}

// Compile-time string processing
macro_rules! const_string {
    // String length calculation
    (len $s:literal) => {
        $s.len()
    };
    
    // String concatenation at compile time
    (concat $($s:literal),+) => {
        concat!($($s),+)
    };
    
    // Character counting
    (count_char $s:literal, $ch:literal) => {
        {
            const fn count_char(s: &str, ch: char) -> usize {
                let bytes = s.as_bytes();
                let mut count = 0;
                let mut i = 0;
                while i < bytes.len() {
                    if bytes[i] as char == ch {
                        count += 1;
                    }
                    i += 1;
                }
                count
            }
            count_char($s, $ch)
        }
    };
    
    // Reverse string at compile time
    (reverse $s:literal) => {
        {
            const fn reverse_str(s: &str) -> &str {
                // Note: This is a simplified version
                // Real implementation would handle UTF-8 properly
                s  // Placeholder - actual reversal needs more complex const fn
            }
            reverse_str($s)
        }
    };
}

// Compile-time array generation
macro_rules! const_array {
    // Generate array with formula
    (formula $size:literal, |$i:ident| $expr:expr) => {
        {
            const SIZE: usize = $size;
            const fn generate_array() -> [i32; SIZE] {
                let mut arr = [0; SIZE];
                let mut $i = 0;
                while $i < SIZE {
                    arr[$i] = $expr;
                    $i += 1;
                }
                arr
            }
            generate_array()
        }
    };
    
    // Generate lookup table
    (lookup_table $size:literal, $func:ident) => {
        {
            const SIZE: usize = $size;
            const fn $func(index: usize) -> i32 {
                (index * index) as i32  // Example function
            }
            
            const fn generate_lookup() -> [i32; SIZE] {
                let mut table = [0; SIZE];
                let mut i = 0;
                while i < SIZE {
                    table[i] = $func(i);
                    i += 1;
                }
                table
            }
            generate_lookup()
        }
    };
    
    // Prime sieve generation
    (primes_up_to $n:literal) => {
        {
            const N: usize = $n;
            const fn sieve_of_eratosthenes() -> ([bool; N], usize) {
                let mut is_prime = [true; N];
                let mut count = 0;
                
                if N > 0 { is_prime[0] = false; }
                if N > 1 { is_prime[1] = false; }
                
                let mut i = 2;
                while i < N {
                    if is_prime[i] {
                        count += 1;
                        let mut j = i * i;
                        while j < N {
                            is_prime[j] = false;
                            j += i;
                        }
                    }
                    i += 1;
                }
                
                (is_prime, count)
            }
            sieve_of_eratosthenes()
        }
    };
}

// Compile-time hash computation
macro_rules! const_hash {
    ($s:literal) => {
        {
            const fn fnv1a_hash(data: &str) -> u64 {
                const FNV_OFFSET_BASIS: u64 = 14695981039346656037;
                const FNV_PRIME: u64 = 1099511628211;
                
                let bytes = data.as_bytes();
                let mut hash = FNV_OFFSET_BASIS;
                let mut i = 0;
                
                while i < bytes.len() {
                    hash ^= bytes[i] as u64;
                    hash = hash.wrapping_mul(FNV_PRIME);
                    i += 1;
                }
                
                hash
            }
            fnv1a_hash($s)
        }
    };
}

// Compile-time code generation with optimization
macro_rules! optimized_switch {
    ($value:expr, {
        $($pattern:literal => $result:expr),+
    }) => {
        {
            // Generate compile-time hash table for O(1) lookup
            const CASES: &[(u64, i32)] = &[
                $((const_hash!(stringify!($pattern)), $result),)+
            ];
            
            let value_str = $value.to_string();
            let value_hash = {
                const fn fnv1a_hash(data: &[u8]) -> u64 {
                    const FNV_OFFSET_BASIS: u64 = 14695981039346656037;
                    const FNV_PRIME: u64 = 1099511628211;
                    
                    let mut hash = FNV_OFFSET_BASIS;
                    let mut i = 0;
                    
                    while i < data.len() {
                        hash ^= data[i] as u64;
                        hash = hash.wrapping_mul(FNV_PRIME);
                        i += 1;
                    }
                    
                    hash
                }
                fnv1a_hash(value_str.as_bytes())
            };
            
            // Linear search through compile-time generated table
            let mut result = None;
            for &(hash, value) in CASES {
                if hash == value_hash {
                    result = Some(value);
                    break;
                }
            }
            
            result.unwrap_or_else(|| panic!("No match found for: {}", $value))
        }
    };
}

fn main() {
    println!("=== Compile-time Computation Macros ===");
    
    // Mathematical computations at compile time
    const FACTORIAL_10: u64 = const_math!(factorial 10);
    const POWER_2_8: u64 = const_math!(power 2 ^ 8);
    const FIB_15: u64 = const_math!(fib 15);
    const IS_17_PRIME: bool = const_math!(is_prime 17);
    
    println!("Factorial 10: {}", FACTORIAL_10);
    println!("2^8: {}", POWER_2_8);
    println!("Fibonacci 15: {}", FIB_15);
    println!("Is 17 prime: {}", IS_17_PRIME);
    
    // String processing at compile time
    const STR_LEN: usize = const_string!(len "Hello, World!");
    const CONCAT_STR: &str = const_string!(concat "Hello", ", ", "World!");
    const CHAR_COUNT: usize = const_string!(count_char "programming", 'r');
    
    println!("String length: {}", STR_LEN);
    println!("Concatenated: {}", CONCAT_STR);
    println!("'r' count in 'programming': {}", CHAR_COUNT);
    
    // Array generation at compile time
    const SQUARES: [i32; 10] = const_array!(formula 10, |i| (i * i) as i32);
    const LOOKUP: [i32; 5] = const_array!(lookup_table 5, square_func);
    const (PRIME_SIEVE, PRIME_COUNT): ([bool; 30], usize) = const_array!(primes_up_to 30);
    
    println!("Squares array: {:?}", SQUARES);
    println!("Lookup table: {:?}", LOOKUP);
    println!("Prime count up to 30: {}", PRIME_COUNT);
    println!("Prime sieve: {:?}", &PRIME_SIEVE[..10]);
    
    // Compile-time hash computation
    const HELLO_HASH: u64 = const_hash!("Hello");
    const WORLD_HASH: u64 = const_hash!("World");
    
    println!("Hash of 'Hello': {}", HELLO_HASH);
    println!("Hash of 'World': {}", WORLD_HASH);
    
    // Optimized switch with compile-time hash table
    let test_value = "two";
    let switch_result = optimized_switch!(test_value, {
        "one" => 1,
        "two" => 2,
        "three" => 3,
        "four" => 4
    });
    println!("Switch result for '{}': {}", test_value, switch_result);
}
```

Compile-time computation macros perform complex calculations during  
compilation, eliminating runtime overhead. Mathematical computations,  
string processing, and array generation happen at compile time using  
const functions. Hash-based optimization creates efficient lookup tables.  
These techniques enable zero-cost abstractions and compile-time  
optimizations that would be impossible with runtime computation.  

## Recursive macro patterns

Advanced recursive macros that process nested structures and generate  
complex code hierarchies through self-referential expansion.  

```rust
// Tree-like recursive data structure generation
macro_rules! generate_tree {
    // Base case: leaf node
    (leaf $name:ident : $type:ty) => {
        #[derive(Debug, Clone)]
        pub struct $name {
            pub value: $type,
        }
        
        impl $name {
            pub fn new(value: $type) -> Self {
                Self { value }
            }
            
            pub fn is_leaf(&self) -> bool {
                true
            }
            
            pub fn depth(&self) -> usize {
                1
            }
        }
    };
    
    // Recursive case: branch node
    (branch $name:ident : $type:ty => { $($child:ident),+ }) => {
        #[derive(Debug, Clone)]
        pub struct $name {
            pub value: $type,
            $(pub $child: Option<Box<$child>>,)+
        }
        
        impl $name {
            pub fn new(value: $type) -> Self {
                Self {
                    value,
                    $($child: None,)+
                }
            }
            
            pub fn is_leaf(&self) -> bool {
                false
            }
            
            pub fn depth(&self) -> usize {
                let child_depths = vec![
                    $(self.$child.as_ref().map_or(0, |c| c.depth())),+
                ];
                1 + child_depths.into_iter().max().unwrap_or(0)
            }
            
            $(
                pub fn set_$child(mut self, child: $child) -> Self {
                    self.$child = Some(Box::new(child));
                    self
                }
                
                pub fn get_$child(&self) -> Option<&$child> {
                    self.$child.as_ref().map(|b| b.as_ref())
                }
            )+
        }
    };
    
    // Complex recursive case with multiple patterns
    (tree $root:ident {
        $($node:ident : $node_type:ty $(=> { $($child:ident),+ })?),+
    }) => {
        // Generate each node type
        $(
            generate_tree!(
                $(branch $node : $node_type => { $($child),+ })?
                $(leaf $node : $node_type)?
            );
        )+
        
        // Generate tree traversal trait
        pub trait TreeNode {
            fn node_count(&self) -> usize;
            fn collect_values(&self) -> Vec<String>;
        }
        
        // Implement traversal for each node
        $(
            impl TreeNode for $node {
                fn node_count(&self) -> usize {
                    1 $(+ self.$child.as_ref().map_or(0, |c| c.node_count()))*
                }
                
                fn collect_values(&self) -> Vec<String> {
                    let mut values = vec![format!("{:?}", self.value)];
                    $(
                        if let Some(child) = &self.$child {
                            values.extend(child.collect_values());
                        }
                    )*
                    values
                }
            }
        )+
    };
}

// Recursive computation macros
macro_rules! recursive_calculate {
    // Base case: single number
    ($n:expr) => {
        $n
    };
    
    // Factorial pattern
    (factorial $n:expr) => {
        {
            fn factorial_impl(n: u64) -> u64 {
                match n {
                    0 | 1 => 1,
                    _ => n * factorial_impl(n - 1)
                }
            }
            factorial_impl($n)
        }
    };
    
    // Fibonacci pattern
    (fibonacci $n:expr) => {
        {
            fn fibonacci_impl(n: u64) -> u64 {
                match n {
                    0 => 0,
                    1 => 1,
                    _ => fibonacci_impl(n - 1) + fibonacci_impl(n - 2)
                }
            }
            fibonacci_impl($n)
        }
    };
    
    // Nested calculation with operations
    (add [ $($expr:tt),+ ]) => {
        0 $(+ recursive_calculate!($expr))+
    };
    
    (multiply [ $($expr:tt),+ ]) => {
        1 $(* recursive_calculate!($expr))+
    };
    
    // Power calculation
    (power $base:expr, $exp:expr) => {
        {
            fn power_impl(base: i64, exp: u32) -> i64 {
                match exp {
                    0 => 1,
                    1 => base,
                    n if n % 2 == 0 => {
                        let half = power_impl(base, n / 2);
                        half * half
                    },
                    n => base * power_impl(base, n - 1)
                }
            }
            power_impl($base, $exp)
        }
    };
}

// Recursive type generation for nested structures
macro_rules! nested_struct {
    // Base case: simple field
    ($name:ident { $field:ident: $type:ty }) => {
        #[derive(Debug, Clone)]
        pub struct $name {
            pub $field: $type,
        }
        
        impl $name {
            pub fn new($field: $type) -> Self {
                Self { $field }
            }
        }
    };
    
    // Recursive case: nested structure
    ($name:ident { 
        $($field:ident: $type:ty),*,
        nested: { $($nested:tt)+ }
    }) => {
        nested_struct!(Inner { $($nested)+ });
        
        #[derive(Debug, Clone)]
        pub struct $name {
            $(pub $field: $type,)*
            pub nested: Inner,
        }
        
        impl $name {
            pub fn new($($field: $type,)* nested: Inner) -> Self {
                Self {
                    $($field,)*
                    nested,
                }
            }
        }
    };
    
    // Multiple nesting levels
    ($name:ident { 
        $($field:ident: $type:ty),*,
        layers: [
            $($layer_name:ident { $($layer_content:tt)+ }),+
        ]
    }) => {
        $(
            nested_struct!($layer_name { $($layer_content)+ });
        )+
        
        #[derive(Debug, Clone)]
        pub struct $name {
            $(pub $field: $type,)*
            $(pub $layer_name: $layer_name,)+
        }
        
        impl $name {
            pub fn new(
                $($field: $type,)*
                $($layer_name: $layer_name,)+
            ) -> Self {
                Self {
                    $($field,)*
                    $($layer_name,)+
                }
            }
        }
    };
}

// Usage examples
generate_tree!(tree ExpressionTree {
    NumberNode: i32,
    AddNode: String => { left, right },
    MultiplyNode: String => { left, right },
    VariableNode: String
});

nested_struct!(ComplexData {
    id: u32,
    name: String,
    layers: [
        MetaLayer { 
            version: String,
            nested: { timestamp: u64 }
        },
        DataLayer { 
            values: Vec<i32>
        }
    ]
});

fn main() {
    println!("=== Recursive Macro Patterns ===");
    
    // Tree generation example
    let number1 = NumberNode::new(42);
    let number2 = NumberNode::new(10);
    let variable = VariableNode::new("x".to_string());
    
    let add_node = AddNode::new("addition".to_string())
        .set_left(number1)
        .set_right(number2);
    
    let multiply_node = MultiplyNode::new("multiplication".to_string())
        .set_left(add_node)
        .set_right(variable);
    
    println!("Tree depth: {}", multiply_node.depth());
    println!("Node count: {}", multiply_node.node_count());
    println!("Values: {:?}", multiply_node.collect_values());
    
    // Recursive calculations
    println!("\n=== Recursive Calculations ===");
    println!("Factorial 5: {}", recursive_calculate!(factorial 5));
    println!("Fibonacci 10: {}", recursive_calculate!(fibonacci 10));
    println!("Power 2^8: {}", recursive_calculate!(power 2, 8));
    
    println!("Add: {}", recursive_calculate!(add [1, 2, 3, 4, 5]));
    println!("Multiply: {}", recursive_calculate!(multiply [2, 3, 4]));
    
    // Complex nested addition
    let complex_sum = recursive_calculate!(add [
        10,
        recursive_calculate!(multiply [2, 3]),
        recursive_calculate!(power 2, 3)
    ]);
    println!("Complex calculation: {}", complex_sum);
    
    // Nested structure example
    let meta_inner = Inner::new(1234567890);
    let meta_layer = MetaLayer::new("v1.0".to_string(), meta_inner);
    let data_layer = DataLayer::new(vec![1, 2, 3, 4, 5]);
    
    let complex_data = ComplexData::new(
        1,
        "test".to_string(),
        meta_layer,
        data_layer,
    );
    
    println!("\n=== Nested Structure ===");
    println!("Complex data: {:#?}", complex_data);
}
```

Recursive macros handle complex nested patterns through self-referential  
expansion. Tree generation creates hierarchical data structures with  
automatic depth calculation and traversal methods. Recursive calculations  
implement mathematical operations with tail-call optimization. Nested  
structure macros generate multi-level type hierarchies. These patterns  
enable sophisticated code generation for complex domain models.  

These 15 advanced macro examples demonstrate the full power of Rust's  
metaprogramming capabilities. From procedural macros and attribute  
transformations to compile-time computations and recursive patterns,  
these techniques enable creating sophisticated abstractions while  
maintaining Rust's safety and performance guarantees. Advanced macros  
unlock domain-specific languages, zero-cost optimizations, and powerful  
code generation that would be impossible in many other languages.  

## Memory layout optimization macros

Advanced macros that analyze and optimize memory layout and access  
patterns at compile time for maximum performance.  

```rust
// Memory layout analysis and optimization
macro_rules! optimize_layout {
    ($struct_name:ident {
        $($field:ident: $type:ty),+
    }) => {
        // Calculate field sizes and alignments at compile time
        const FIELD_INFO: &[(usize, usize)] = &[
            $((std::mem::size_of::<$type>(), std::mem::align_of::<$type>())),+
        ];
        
        // Generate optimized struct with fields reordered by size/alignment
        #[repr(C)]
        #[derive(Debug)]
        pub struct $struct_name {
            // Fields would be reordered here in a real implementation
            $($field: $type,)+
        }
        
        impl $struct_name {
            pub fn memory_info() -> MemoryInfo {
                MemoryInfo {
                    total_size: std::mem::size_of::<Self>(),
                    alignment: std::mem::align_of::<Self>(),
                    field_count: FIELD_INFO.len(),
                    field_sizes: FIELD_INFO.iter().map(|(s, _)| *s).collect(),
                    field_alignments: FIELD_INFO.iter().map(|(_, a)| *a).collect(),
                }
            }
            
            pub fn cache_efficiency_score() -> f64 {
                let size = std::mem::size_of::<Self>() as f64;
                let cache_line_size = 64.0;  // Common cache line size
                
                if size <= cache_line_size {
                    1.0  // Fits in single cache line
                } else {
                    cache_line_size / size  // Fraction of cache line utilization
                }
            }
        }
    };
}

#[derive(Debug)]
pub struct MemoryInfo {
    pub total_size: usize,
    pub alignment: usize,
    pub field_count: usize,
    pub field_sizes: Vec<usize>,
    pub field_alignments: Vec<usize>,
}

// Cache-friendly data structure generation
macro_rules! cache_friendly_array {
    ($name:ident, $element_type:ty, $size:expr) => {
        #[repr(align(64))]  // Align to cache line
        pub struct $name {
            data: [$element_type; $size],
            _padding: [u8; 64 - (($size * std::mem::size_of::<$element_type>()) % 64)],
        }
        
        impl $name {
            pub fn new() -> Self {
                Self {
                    data: [Default::default(); $size],
                    _padding: [0; 64 - (($size * std::mem::size_of::<$element_type>()) % 64)],
                }
            }
            
            pub fn get(&self, index: usize) -> Option<&$element_type> {
                self.data.get(index)
            }
            
            pub fn set(&mut self, index: usize, value: $element_type) -> Result<(), &'static str> {
                if index < $size {
                    self.data[index] = value;
                    Ok(())
                } else {
                    Err("Index out of bounds")
                }
            }
            
            pub fn cache_lines_used() -> usize {
                let total_size = std::mem::size_of::<Self>();
                (total_size + 63) / 64  // Round up to nearest cache line
            }
        }
    };
}

// SIMD-optimized operations macro
macro_rules! simd_operations {
    ($name:ident, $scalar_type:ty, $simd_width:expr) => {
        pub struct $name;
        
        impl $name {
            pub fn add_arrays(a: &[$scalar_type], b: &[$scalar_type]) -> Vec<$scalar_type> {
                assert_eq!(a.len(), b.len());
                let mut result = Vec::with_capacity(a.len());
                
                // Process in SIMD-width chunks
                let chunks = a.len() / $simd_width;
                let remainder = a.len() % $simd_width;
                
                for i in 0..chunks {
                    let start = i * $simd_width;
                    for j in 0..$simd_width {
                        result.push(a[start + j] + b[start + j]);
                    }
                }
                
                // Handle remainder
                for i in 0..remainder {
                    let idx = chunks * $simd_width + i;
                    result.push(a[idx] + b[idx]);
                }
                
                result
            }
            
            pub fn sum_array(data: &[$scalar_type]) -> $scalar_type {
                let mut sum = Default::default();
                
                // Unroll loop for better performance
                let chunks = data.len() / 4;
                let remainder = data.len() % 4;
                
                for i in 0..chunks {
                    let start = i * 4;
                    sum = sum + data[start] + data[start + 1] + data[start + 2] + data[start + 3];
                }
                
                for i in 0..remainder {
                    sum = sum + data[chunks * 4 + i];
                }
                
                sum
            }
        }
    };
}

// Usage examples
optimize_layout!(OptimizedStruct {
    small_field: u8,
    large_field: u64,
    medium_field: u32,
    tiny_field: bool
});

cache_friendly_array!(CacheArray, i32, 16);
simd_operations!(SimdMath, f32, 8);

fn main() {
    println!("=== Memory Layout Optimization ===");
    
    let info = OptimizedStruct::memory_info();
    println!("Struct size: {} bytes", info.total_size);
    println!("Alignment: {} bytes", info.alignment);
    println!("Field count: {}", info.field_count);
    println!("Cache efficiency: {:.2}", OptimizedStruct::cache_efficiency_score());
    
    let mut cache_array = CacheArray::new();
    cache_array.set(0, 42).unwrap();
    cache_array.set(1, 24).unwrap();
    
    println!("Cache lines used: {}", CacheArray::cache_lines_used());
    println!("Array element 0: {:?}", cache_array.get(0));
    
    let a = vec![1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0];
    let b = vec![8.0, 7.0, 6.0, 5.0, 4.0, 3.0, 2.0, 1.0];
    let result = SimdMath::add_arrays(&a, &b);
    let sum = SimdMath::sum_array(&a);
    
    println!("SIMD addition result: {:?}", result);
    println!("Array sum: {}", sum);
}
```

Memory layout optimization macros analyze data structures at compile  
time and generate cache-efficient layouts. Cache-friendly arrays ensure  
proper alignment and padding for optimal memory access patterns. SIMD  
operation macros create vectorized computations that leverage modern  
CPU capabilities. These optimizations can provide significant performance  
improvements for compute-intensive applications.  

## Domain-specific language generation

Advanced macros that create complete embedded domain-specific languages  
with custom syntax, semantics, and optimization passes.  

```rust
// Mathematical expression DSL with optimization
macro_rules! math_dsl {
    // Parse and optimize mathematical expressions
    ($($expr:tt)+) => {
        {
            // Generate optimized expression tree at compile time
            let expr = math_dsl!(@parse $($expr)+);
            let optimized = math_dsl!(@optimize expr);
            optimized
        }
    };
    
    // Expression parsing
    (@parse $num:literal) => {
        Expr::Number($num as f64)
    };
    
    (@parse $var:ident) => {
        Expr::Variable(stringify!($var).to_string())
    };
    
    (@parse ($($inner:tt)+)) => {
        math_dsl!(@parse $($inner)+)
    };
    
    (@parse $left:tt + $($right:tt)+) => {
        Expr::Add(
            Box::new(math_dsl!(@parse $left)),
            Box::new(math_dsl!(@parse $($right)+))
        )
    };
    
    (@parse $left:tt * $($right:tt)+) => {
        Expr::Multiply(
            Box::new(math_dsl!(@parse $left)),
            Box::new(math_dsl!(@parse $($right)+))
        )
    };
    
    (@parse $left:tt - $($right:tt)+) => {
        Expr::Subtract(
            Box::new(math_dsl!(@parse $left)),
            Box::new(math_dsl!(@parse $($right)+))
        )
    };
    
    (@parse $left:tt / $($right:tt)+) => {
        Expr::Divide(
            Box::new(math_dsl!(@parse $left)),
            Box::new(math_dsl!(@parse $($right)+))
        )
    };
    
    // Expression optimization
    (@optimize $expr:expr) => {
        $expr.optimize()
    };
}

#[derive(Debug, Clone)]
enum Expr {
    Number(f64),
    Variable(String),
    Add(Box<Expr>, Box<Expr>),
    Subtract(Box<Expr>, Box<Expr>),
    Multiply(Box<Expr>, Box<Expr>),
    Divide(Box<Expr>, Box<Expr>),
}

impl Expr {
    fn optimize(self) -> Self {
        match self {
            // Constant folding
            Expr::Add(left, right) => {
                let left_opt = left.optimize();
                let right_opt = right.optimize();
                match (&left_opt, &right_opt) {
                    (Expr::Number(a), Expr::Number(b)) => Expr::Number(a + b),
                    (Expr::Number(0.0), expr) | (expr, Expr::Number(0.0)) => expr.clone(),
                    _ => Expr::Add(Box::new(left_opt), Box::new(right_opt)),
                }
            }
            
            Expr::Multiply(left, right) => {
                let left_opt = left.optimize();
                let right_opt = right.optimize();
                match (&left_opt, &right_opt) {
                    (Expr::Number(a), Expr::Number(b)) => Expr::Number(a * b),
                    (Expr::Number(0.0), _) | (_, Expr::Number(0.0)) => Expr::Number(0.0),
                    (Expr::Number(1.0), expr) | (expr, Expr::Number(1.0)) => expr.clone(),
                    _ => Expr::Multiply(Box::new(left_opt), Box::new(right_opt)),
                }
            }
            
            // Add more optimization rules...
            other => other,
        }
    }
    
    fn evaluate(&self, vars: &std::collections::HashMap<String, f64>) -> f64 {
        match self {
            Expr::Number(n) => *n,
            Expr::Variable(name) => vars.get(name).copied().unwrap_or(0.0),
            Expr::Add(left, right) => left.evaluate(vars) + right.evaluate(vars),
            Expr::Subtract(left, right) => left.evaluate(vars) - right.evaluate(vars),
            Expr::Multiply(left, right) => left.evaluate(vars) * right.evaluate(vars),
            Expr::Divide(left, right) => left.evaluate(vars) / right.evaluate(vars),
        }
    }
}

// State machine DSL with validation
macro_rules! state_machine_dsl {
    ($name:ident {
        states: [$($state:ident),+],
        events: [$($event:ident),+],
        initial: $initial:ident,
        transitions: [
            $($from:ident --$trigger:ident--> $to:ident),+
        ]
    }) => {
        #[derive(Debug, Clone, PartialEq)]
        pub enum State {
            $($state),+
        }
        
        #[derive(Debug, Clone, PartialEq)]
        pub enum Event {
            $($event),+
        }
        
        pub struct $name {
            current_state: State,
            transition_count: usize,
            history: Vec<(State, Event, State)>,
        }
        
        impl $name {
            pub fn new() -> Self {
                Self {
                    current_state: State::$initial,
                    transition_count: 0,
                    history: Vec::new(),
                }
            }
            
            pub fn current_state(&self) -> &State {
                &self.current_state
            }
            
            pub fn transition_count(&self) -> usize {
                self.transition_count
            }
            
            pub fn handle_event(&mut self, event: Event) -> Result<State, String> {
                let old_state = self.current_state.clone();
                
                let new_state = match (&self.current_state, &event) {
                    $((State::$from, Event::$trigger) => State::$to,)+
                    _ => return Err(format!(
                        "Invalid transition: {:?} on {:?}",
                        self.current_state, event
                    )),
                };
                
                self.history.push((old_state.clone(), event.clone(), new_state.clone()));
                self.current_state = new_state.clone();
                self.transition_count += 1;
                
                Ok(new_state)
            }
            
            pub fn get_history(&self) -> &[(State, Event, State)] {
                &self.history
            }
            
            pub fn can_handle(&self, event: &Event) -> bool {
                match (&self.current_state, event) {
                    $((State::$from, Event::$trigger) => true,)+
                    _ => false,
                }
            }
        }
    };
}

// Database query DSL
macro_rules! query_dsl {
    (SELECT $($field:ident),+ FROM $table:ident $(WHERE $condition:expr)?) => {
        {
            let fields = vec![$(stringify!($field)),+];
            let table = stringify!($table);
            let condition = query_dsl!(@condition $($condition)?);
            
            QueryBuilder::new()
                .select(fields)
                .from(table)
                .where_clause(condition)
                .build()
        }
    };
    
    (@condition $cond:expr) => { Some(stringify!($cond).to_string()) };
    (@condition) => { None };
}

struct QueryBuilder {
    fields: Vec<String>,
    table: String,
    condition: Option<String>,
}

impl QueryBuilder {
    fn new() -> Self {
        Self {
            fields: Vec::new(),
            table: String::new(),
            condition: None,
        }
    }
    
    fn select(mut self, fields: Vec<&str>) -> Self {
        self.fields = fields.into_iter().map(String::from).collect();
        self
    }
    
    fn from(mut self, table: &str) -> Self {
        self.table = table.to_string();
        self
    }
    
    fn where_clause(mut self, condition: Option<String>) -> Self {
        self.condition = condition;
        self
    }
    
    fn build(self) -> String {
        let mut query = format!("SELECT {} FROM {}", 
                               self.fields.join(", "), self.table);
        
        if let Some(cond) = self.condition {
            query.push_str(&format!(" WHERE {}", cond));
        }
        
        query
    }
}

// Usage examples
state_machine_dsl!(TrafficLight {
    states: [Red, Yellow, Green],
    events: [Timer, Emergency],
    initial: Red,
    transitions: [
        Red --Timer--> Green,
        Green --Timer--> Yellow,
        Yellow --Timer--> Red,
        Red --Emergency--> Yellow,
        Green --Emergency--> Yellow,
        Yellow --Emergency--> Red
    ]
});

fn main() {
    println!("=== Domain-specific Language Generation ===");
    
    // Mathematical DSL
    let expr = math_dsl!(2 + 3 * 4);
    let mut vars = std::collections::HashMap::new();
    vars.insert("x".to_string(), 5.0);
    println!("Expression result: {}", expr.evaluate(&vars));
    
    // State machine DSL
    let mut traffic_light = TrafficLight::new();
    println!("Initial state: {:?}", traffic_light.current_state());
    
    let _ = traffic_light.handle_event(Event::Timer);
    println!("After timer: {:?}", traffic_light.current_state());
    
    let _ = traffic_light.handle_event(Event::Emergency);
    println!("After emergency: {:?}", traffic_light.current_state());
    
    println!("Transition count: {}", traffic_light.transition_count());
    println!("Can handle timer: {}", traffic_light.can_handle(&Event::Timer));
    
    // Query DSL
    let query1 = query_dsl!(SELECT name, age FROM users WHERE age > 18);
    let query2 = query_dsl!(SELECT id, email FROM customers);
    
    println!("Query 1: {}", query1);
    println!("Query 2: {}", query2);
}
```

Domain-specific language generation creates embedded languages with  
custom syntax and semantics. Mathematical DSLs provide expression  
parsing and optimization, state machine DSLs enable declarative  
workflow definition, and query DSLs create type-safe database  
interfaces. These patterns enable natural syntax for domain experts  
while maintaining compile-time validation and optimization.  

## Advanced trait generation and implementation

Sophisticated macros that generate traits and implementations with  
complex bounds, associated types, and generic constraints.  

```rust
// Generate traits with associated types and bounds
macro_rules! generate_collection_trait {
    ($trait_name:ident {
        item_type: $item_name:ident,
        container_type: $container_name:ident,
        required_methods: {
            $($method_name:ident($($param:ident: $param_type:ty),*) -> $return_type:ty),+
        },
        optional_methods: {
            $($opt_method:ident($($opt_param:ident: $opt_param_type:ty),*) -> $opt_return:ty => $default_impl:expr),*
        }
    }) => {
        pub trait $trait_name {
            type $item_name;
            type $container_name: IntoIterator<Item = Self::$item_name>;
            
            // Required methods
            $(
                fn $method_name(&self, $($param: $param_type),*) -> $return_type;
            )+
            
            // Optional methods with default implementations
            $(
                fn $opt_method(&self, $($opt_param: $opt_param_type),*) -> $opt_return {
                    $default_impl
                }
            )*
        }
    };
}

// Automatic trait implementation generator
macro_rules! impl_for_types {
    ($trait_name:ident for [$($type:ty),+] where Item = $item_type:ty, Container = $container_type:ty {
        $($method_name:ident => $implementation:expr),+
    }) => {
        $(
            impl $trait_name for $type {
                type Item = $item_type;
                type Container = $container_type;
                
                $(
                    fn $method_name(&self, $($param: $param_type),*) -> $return_type {
                        $implementation
                    }
                )+
            }
        )+
    };
}

// Generic wrapper generation with trait forwarding
macro_rules! generate_wrapper {
    ($wrapper_name:ident<$inner:ident: $trait_bound:path> {
        expose: [$($exposed_method:ident),+],
        hide: [$($hidden_method:ident),*],
        transform: {
            $($transform_method:ident => $transform_impl:expr),*
        }
    }) => {
        pub struct $wrapper_name<$inner>
        where
            $inner: $trait_bound,
        {
            inner: $inner,
        }
        
        impl<$inner> $wrapper_name<$inner>
        where
            $inner: $trait_bound,
        {
            pub fn new(inner: $inner) -> Self {
                Self { inner }
            }
            
            pub fn into_inner(self) -> $inner {
                self.inner
            }
            
            // Expose selected methods
            $(
                pub fn $exposed_method(&self) -> &$inner {
                    &self.inner
                }
            )+
            
            // Transform methods
            $(
                pub fn $transform_method(&self) -> impl Iterator<Item = _> {
                    $transform_impl(&self.inner)
                }
            )*
        }
    };
}

// Derive-like macro for custom traits
macro_rules! derive_serializable {
    ($struct_name:ident {
        $($field:ident: $field_type:ty),+
    }) => {
        impl Serializable for $struct_name {
            fn serialize(&self) -> String {
                let mut parts = Vec::new();
                $(
                    parts.push(format!("{}={:?}", stringify!($field), self.$field));
                )+
                format!("{}({})", stringify!($struct_name), parts.join(", "))
            }
            
            fn field_count() -> usize {
                derive_serializable!(@count $($field),+)
            }
            
            fn field_names() -> Vec<&'static str> {
                vec![$(stringify!($field)),+]
            }
            
            fn field_types() -> Vec<&'static str> {
                vec![$(stringify!($field_type)),+]
            }
        }
    };
    
    (@count $($field:ident),+) => {
        [$($field,)+].len()
    };
}

// Multi-trait derivation with conditional implementations
macro_rules! derive_multiple {
    ($struct_name:ident: [$($trait_name:ident),+]) => {
        $(
            derive_multiple!(@impl $struct_name, $trait_name);
        )+
    };
    
    (@impl $struct_name:ident, Debug) => {
        impl std::fmt::Debug for $struct_name {
            fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
                write!(f, "{} {{ ... }}", stringify!($struct_name))
            }
        }
    };
    
    (@impl $struct_name:ident, Clone) => {
        impl Clone for $struct_name {
            fn clone(&self) -> Self {
                *self  // Assumes Copy is implemented
            }
        }
    };
    
    (@impl $struct_name:ident, Default) => {
        impl Default for $struct_name {
            fn default() -> Self {
                unsafe { std::mem::zeroed() }  // Simplified - use with caution
            }
        }
    };
}

// Custom trait definitions for examples
pub trait Serializable {
    fn serialize(&self) -> String;
    fn field_count() -> usize;
    fn field_names() -> Vec<&'static str>;
    fn field_types() -> Vec<&'static str>;
}

// Usage examples
generate_collection_trait!(MyCollection {
    item_type: Item,
    container_type: Container,
    required_methods: {
        add(item: Self::Item) -> bool,
        remove(index: usize) -> Option<Self::Item>,
        get(index: usize) -> Option<&Self::Item>
    },
    optional_methods: {
        len() -> usize => 0,
        is_empty() -> bool => self.len() == 0
    }
});

#[derive(Copy, Clone)]
struct Point {
    x: f64,
    y: f64,
}

#[derive(Copy, Clone)]
struct Rectangle {
    width: f64,
    height: f64,
}

derive_serializable!(Point {
    x: f64,
    y: f64
});

derive_serializable!(Rectangle {
    width: f64,
    height: f64
});

derive_multiple!(Point: [Debug, Clone, Default]);

generate_wrapper!(SafeVec<T: std::fmt::Debug> {
    expose: [capacity],
    hide: [push, pop],
    transform: {
        debug_items => |inner: &T| inner.iter().map(|x| format!("{:?}", x))
    }
});

fn main() {
    println!("=== Advanced Trait Generation ===");
    
    let point = Point { x: 3.14, y: 2.71 };
    let rectangle = Rectangle { width: 10.0, height: 5.0 };
    
    println!("Point serialized: {}", point.serialize());
    println!("Point field count: {}", Point::field_count());
    println!("Point field names: {:?}", Point::field_names());
    
    println!("Rectangle serialized: {}", rectangle.serialize());
    println!("Rectangle field types: {:?}", Rectangle::field_types());
    
    println!("Point debug: {:?}", point);
    println!("Point clone: {:?}", point.clone());
    println!("Point default: {:?}", Point::default());
}
```

Advanced trait generation creates sophisticated abstractions with  
associated types, generic constraints, and conditional implementations.  
Collection traits provide flexible interfaces for container types,  
wrapper generation enables selective API exposure, and derive-like  
macros automate implementation of custom traits. These patterns  
enable building powerful, reusable library interfaces with minimal  
boilerplate code.  

## Error handling and debugging macros

Specialized macros for advanced error handling patterns, debugging  
assistance, and compile-time diagnostics.  

```rust
// Advanced error handling with context and recovery
macro_rules! try_with_context {
    ($operation:expr, context: $context:expr) => {
        match $operation {
            Ok(value) => value,
            Err(e) => {
                eprintln!("Error in {}: {:?}", $context, e);
                return Err(format!("Failed {}: {}", $context, e).into());
            }
        }
    };
    
    ($operation:expr, context: $context:expr, recovery: $recovery:expr) => {
        match $operation {
            Ok(value) => value,
            Err(e) => {
                eprintln!("Error in {}: {:?}, attempting recovery", $context, e);
                match $recovery {
                    Ok(recovered) => recovered,
                    Err(recovery_err) => {
                        return Err(format!("Failed {} and recovery failed: {} -> {}", 
                                         $context, e, recovery_err).into());
                    }
                }
            }
        }
    };
}

// Compile-time assertion and validation macros
macro_rules! static_assert {
    ($condition:expr) => {
        const _: () = {
            if !$condition {
                panic!(concat!("Static assertion failed: ", stringify!($condition)));
            }
        };
    };
    
    ($condition:expr, $message:expr) => {
        const _: () = {
            if !$condition {
                panic!($message);
            }
        };
    };
}

// Advanced debugging with conditional compilation
macro_rules! debug_trace {
    ($($arg:tt)*) => {
        #[cfg(debug_assertions)]
        {
            eprintln!("[TRACE {}:{}] {}", file!(), line!(), format!($($arg)*));
        }
    };
}

macro_rules! debug_time {
    ($name:expr, $block:block) => {
        {
            #[cfg(debug_assertions)]
            let _timer = {
                use std::time::Instant;
                
                struct Timer {
                    name: &'static str,
                    start: Instant,
                }
                
                impl Drop for Timer {
                    fn drop(&mut self) {
                        let duration = self.start.elapsed();
                        eprintln!("[TIMING] {}: {:?}", self.name, duration);
                    }
                }
                
                Timer {
                    name: $name,
                    start: Instant::now(),
                }
            };
            
            $block
        }
    };
}

// Memory debugging and leak detection
macro_rules! debug_allocations {
    ($name:expr, $block:block) => {
        {
            #[cfg(debug_assertions)]
            {
                use std::alloc::{GlobalAlloc, Layout, System};
                use std::sync::atomic::{AtomicUsize, Ordering};
                
                static ALLOCATIONS: AtomicUsize = AtomicUsize::new(0);
                static DEALLOCATIONS: AtomicUsize = AtomicUsize::new(0);
                
                struct DebugAllocator;
                
                unsafe impl GlobalAlloc for DebugAllocator {
                    unsafe fn alloc(&self, layout: Layout) -> *mut u8 {
                        ALLOCATIONS.fetch_add(1, Ordering::SeqCst);
                        System.alloc(layout)
                    }
                    
                    unsafe fn dealloc(&self, ptr: *mut u8, layout: Layout) {
                        DEALLOCATIONS.fetch_add(1, Ordering::SeqCst);
                        System.dealloc(ptr, layout)
                    }
                }
                
                let initial_allocs = ALLOCATIONS.load(Ordering::SeqCst);
                let initial_deallocs = DEALLOCATIONS.load(Ordering::SeqCst);
                
                let result = $block;
                
                let final_allocs = ALLOCATIONS.load(Ordering::SeqCst);
                let final_deallocs = DEALLOCATIONS.load(Ordering::SeqCst);
                
                eprintln!("[ALLOC] {}: {} allocations, {} deallocations, {} net",
                         $name,
                         final_allocs - initial_allocs,
                         final_deallocs - initial_deallocs,
                         (final_allocs - initial_allocs) as isize - 
                         (final_deallocs - initial_deallocs) as isize);
                
                result
            }
            
            #[cfg(not(debug_assertions))]
            $block
        }
    };
}

// Error chain and causality tracking
macro_rules! error_chain {
    ($error_type:ident {
        $($variant:ident($inner:ty) => $message:expr),+
    }) => {
        #[derive(Debug)]
        pub enum $error_type {
            $($variant($inner),)+
        }
        
        impl std::fmt::Display for $error_type {
            fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
                match self {
                    $(Self::$variant(inner) => write!(f, "{}: {}", $message, inner),)+
                }
            }
        }
        
        impl std::error::Error for $error_type {
            fn source(&self) -> Option<&(dyn std::error::Error + 'static)> {
                match self {
                    $(Self::$variant(inner) => Some(inner),)+
                }
            }
        }
        
        $(
            impl From<$inner> for $error_type {
                fn from(error: $inner) -> Self {
                    Self::$variant(error)
                }
            }
        )+
        
        impl $error_type {
            pub fn chain(&self) -> Vec<String> {
                let mut chain = vec![self.to_string()];
                let mut source = self.source();
                while let Some(err) = source {
                    chain.push(err.to_string());
                    source = err.source();
                }
                chain
            }
        }
    };
}

// Performance profiling macros
macro_rules! profile_function {
    ($func:ident) => {
        profile_function!($func, stringify!($func));
    };
    
    ($func:ident, $name:expr) => {
        {
            #[cfg(feature = "profiling")]
            use std::time::Instant;
            
            #[cfg(feature = "profiling")]
            let start = Instant::now();
            
            let result = $func();
            
            #[cfg(feature = "profiling")]
            {
                let duration = start.elapsed();
                eprintln!("[PROFILE] {}: {:?}", $name, duration);
            }
            
            result
        }
    };
}

// Conditional feature compilation
macro_rules! feature_gate {
    ($feature:literal, $code:block) => {
        #[cfg(feature = $feature)]
        $code
    };
    
    ($feature:literal, $code:block, else $alt_code:block) => {
        #[cfg(feature = $feature)]
        $code
        
        #[cfg(not(feature = $feature))]
        $alt_code
    };
}

// Usage examples and definitions
error_chain!(MyError {
    Io(std::io::Error) => "I/O operation failed",
    Parse(std::num::ParseIntError) => "Parsing failed",
    Custom(Box<dyn std::error::Error>) => "Custom error occurred"
});

fn risky_operation() -> Result<i32, Box<dyn std::error::Error>> {
    let _value = try_with_context!(
        "42".parse::<i32>(),
        context: "parsing number",
        recovery: Ok(0)
    );
    
    try_with_context!(
        std::fs::read_to_string("nonexistent.txt"),
        context: "reading file"
    );
    
    Ok(42)
}

fn example_function() -> i32 {
    debug_trace!("Starting computation");
    
    let result = debug_time!("computation", {
        let mut sum = 0;
        for i in 0..1000 {
            sum += i;
        }
        sum
    });
    
    debug_trace!("Computation completed with result: {}", result);
    result
}

// Compile-time validations
static_assert!(std::mem::size_of::<usize>() >= 4, "Platform requires at least 32-bit pointers");
static_assert!(std::mem::align_of::<u64>() <= 8);

fn main() {
    println!("=== Error Handling and Debugging Macros ===");
    
    // Error handling demonstration
    match risky_operation() {
        Ok(value) => println!("Operation succeeded: {}", value),
        Err(e) => {
            if let Ok(my_error) = e.downcast::<MyError>() {
                println!("Error chain: {:?}", my_error.chain());
            } else {
                println!("Other error: {}", e);
            }
        }
    }
    
    // Debug tracing and timing
    let result = example_function();
    println!("Function result: {}", result);
    
    // Allocation debugging
    debug_allocations!("vector_operations", {
        let mut vec = Vec::new();
        for i in 0..100 {
            vec.push(i);
        }
        vec.len()
    });
    
    // Feature-gated code
    feature_gate!("extra_features", {
        println!("Extra features are enabled!");
    }, else {
        println!("Extra features are disabled.");
    });
    
    println!("All static assertions passed!");
}
```

Error handling and debugging macros provide sophisticated tools for  
development and maintenance. Context-aware error handling improves  
debugging experience, compile-time assertions catch errors early,  
and conditional debugging reduces runtime overhead. Performance  
profiling and allocation tracking help identify bottlenecks. These  
patterns create more robust and maintainable applications while  
preserving performance in release builds.  

## Concurrency and parallelism macros

Advanced macros for managing concurrent operations, thread-safe  
data structures, and parallel computation patterns.  

```rust
// Thread-safe data structure generation
macro_rules! thread_safe_wrapper {
    ($name:ident wrapping $inner:ty) => {
        use std::sync::{Arc, Mutex, RwLock};
        use std::thread;
        
        pub struct $name {
            inner: Arc<RwLock<$inner>>,
        }
        
        impl $name {
            pub fn new(value: $inner) -> Self {
                Self {
                    inner: Arc::new(RwLock::new(value)),
                }
            }
            
            pub fn read<F, R>(&self, f: F) -> R
            where
                F: FnOnce(&$inner) -> R,
            {
                let guard = self.inner.read().unwrap();
                f(&*guard)
            }
            
            pub fn write<F, R>(&self, f: F) -> R
            where
                F: FnOnce(&mut $inner) -> R,
            {
                let mut guard = self.inner.write().unwrap();
                f(&mut *guard)
            }
            
            pub fn clone_handle(&self) -> Self {
                Self {
                    inner: Arc::clone(&self.inner),
                }
            }
            
            pub fn spawn_reader<F>(&self, f: F) -> thread::JoinHandle<()>
            where
                F: FnOnce(&$inner) + Send + 'static,
            {
                let handle = self.clone_handle();
                thread::spawn(move || {
                    handle.read(f);
                })
            }
            
            pub fn spawn_writer<F>(&self, f: F) -> thread::JoinHandle<()>
            where
                F: FnOnce(&mut $inner) + Send + 'static,
            {
                let handle = self.clone_handle();
                thread::spawn(move || {
                    handle.write(f);
                })
            }
        }
        
        impl Clone for $name {
            fn clone(&self) -> Self {
                self.clone_handle()
            }
        }
    };
}

// Parallel computation macro
macro_rules! parallel_map {
    ($collection:expr, $threads:expr, |$item:ident| $transform:expr) => {
        {
            use std::sync::mpsc;
            use std::thread;
            
            let collection: Vec<_> = $collection.into_iter().collect();
            let chunk_size = (collection.len() + $threads - 1) / $threads;
            let (tx, rx) = mpsc::channel();
            
            let mut handles = Vec::new();
            
            for (chunk_id, chunk) in collection.chunks(chunk_size).enumerate() {
                let tx = tx.clone();
                let chunk = chunk.to_vec();
                
                let handle = thread::spawn(move || {
                    let results: Vec<_> = chunk.into_iter().map(|$item| $transform).collect();
                    tx.send((chunk_id, results)).unwrap();
                });
                
                handles.push(handle);
            }
            
            drop(tx);  // Close sender
            
            // Collect results in order
            let mut results = vec![Vec::new(); handles.len()];
            for (chunk_id, chunk_results) in rx {
                results[chunk_id] = chunk_results;
            }
            
            // Wait for all threads
            for handle in handles {
                handle.join().unwrap();
            }
            
            results.into_iter().flatten().collect::<Vec<_>>()
        }
    };
}

// Actor pattern macro
macro_rules! actor_system {
    ($actor_name:ident {
        state: { $($state_field:ident: $state_type:ty),+ },
        messages: {
            $($message:ident($($param:ident: $param_type:ty),*) => $handler:expr),+
        }
    }) => {
        use std::sync::mpsc;
        use std::thread;
        
        #[derive(Debug)]
        pub enum Message {
            $($message($($param_type),*),)+
            Stop,
        }
        
        pub struct ActorState {
            $($state_field: $state_type,)+
        }
        
        pub struct $actor_name {
            sender: mpsc::Sender<Message>,
            handle: Option<thread::JoinHandle<()>>,
        }
        
        impl $actor_name {
            pub fn new($($state_field: $state_type),+) -> Self {
                let (sender, receiver) = mpsc::channel();
                
                let mut state = ActorState {
                    $($state_field,)+
                };
                
                let handle = thread::spawn(move || {
                    while let Ok(message) = receiver.recv() {
                        match message {
                            Message::Stop => break,
                            $(
                                Message::$message($($param),*) => {
                                    let handler = $handler;
                                    handler(&mut state, $($param),*);
                                }
                            )+
                        }
                    }
                });
                
                Self {
                    sender,
                    handle: Some(handle),
                }
            }
            
            $(
                pub fn $message(&self, $($param: $param_type),*) -> Result<(), mpsc::SendError<Message>> {
                    self.sender.send(Message::$message($($param),*))
                }
            )+
            
            pub fn stop(mut self) {
                let _ = self.sender.send(Message::Stop);
                if let Some(handle) = self.handle.take() {
                    let _ = handle.join();
                }
            }
        }
        
        impl Drop for $actor_name {
            fn drop(&mut self) {
                let _ = self.sender.send(Message::Stop);
                if let Some(handle) = self.handle.take() {
                    let _ = handle.join();
                }
            }
        }
    };
}

// Work-stealing queue macro
macro_rules! work_stealing_queue {
    ($name:ident<$item:ty>) => {
        use std::collections::VecDeque;
        use std::sync::{Arc, Mutex};
        use std::thread;
        
        pub struct $name {
            queues: Vec<Arc<Mutex<VecDeque<$item>>>>,
            workers: Vec<thread::JoinHandle<()>>,
        }
        
        impl $name {
            pub fn new(num_workers: usize) -> Self {
                let queues: Vec<_> = (0..num_workers)
                    .map(|_| Arc::new(Mutex::new(VecDeque::new())))
                    .collect();
                
                Self {
                    queues,
                    workers: Vec::new(),
                }
            }
            
            pub fn push(&self, item: $item) {
                // Find queue with minimum length
                let min_queue = self.queues
                    .iter()
                    .min_by_key(|q| q.lock().unwrap().len())
                    .unwrap();
                
                min_queue.lock().unwrap().push_back(item);
            }
            
            pub fn spawn_workers<F>(&mut self, worker_fn: F)
            where
                F: Fn($item) + Send + Sync + Clone + 'static,
            {
                let worker_fn = Arc::new(worker_fn);
                
                for (worker_id, queue) in self.queues.iter().enumerate() {
                    let queue = Arc::clone(queue);
                    let other_queues: Vec<_> = self.queues.iter()
                        .enumerate()
                        .filter(|(id, _)| *id != worker_id)
                        .map(|(_, q)| Arc::clone(q))
                        .collect();
                    let worker_fn = Arc::clone(&worker_fn);
                    
                    let handle = thread::spawn(move || {
                        loop {
                            // Try to get work from own queue
                            let work = queue.lock().unwrap().pop_front();
                            
                            let work = work.or_else(|| {
                                // Steal work from other queues
                                for other_queue in &other_queues {
                                    if let Some(stolen) = other_queue.lock().unwrap().pop_back() {
                                        return Some(stolen);
                                    }
                                }
                                None
                            });
                            
                            match work {
                                Some(item) => worker_fn(item),
                                None => {
                                    // No work available, sleep briefly
                                    thread::sleep(std::time::Duration::from_millis(1));
                                }
                            }
                        }
                    });
                    
                    self.workers.push(handle);
                }
            }
            
            pub fn wait_completion(self) {
                for handle in self.workers {
                    let _ = handle.join();
                }
            }
        }
    };
}

// Barrier synchronization macro
macro_rules! sync_barrier {
    ($name:ident, $count:expr, $action:expr) => {
        {
            use std::sync::{Arc, Barrier};
            use std::thread;
            
            let barrier = Arc::new(Barrier::new($count));
            let mut handles = Vec::new();
            
            for i in 0..$count {
                let barrier = Arc::clone(&barrier);
                let action = $action;
                
                let handle = thread::spawn(move || {
                    // Perform individual work
                    println!("Thread {} doing work", i);
                    
                    // Wait for all threads to reach this point
                    barrier.wait();
                    
                    // Perform synchronized action
                    action(i);
                });
                
                handles.push(handle);
            }
            
            // Wait for all threads to complete
            for handle in handles {
                handle.join().unwrap();
            }
        }
    };
}

// Usage examples
thread_safe_wrapper!(SafeCounter wrapping i32);
work_stealing_queue!(TaskQueue<Box<dyn FnOnce() + Send>>);

actor_system!(Counter {
    state: { 
        count: i32,
        name: String
    },
    messages: {
        Increment() => |state: &mut ActorState| {
            state.count += 1;
            println!("{}: count = {}", state.name, state.count);
        },
        Add(value: i32) => |state: &mut ActorState, value: i32| {
            state.count += value;
            println!("{}: added {}, count = {}", state.name, value, state.count);
        },
        GetCount() => |state: &mut ActorState| {
            println!("{}: current count = {}", state.name, state.count);
        }
    }
});

fn main() {
    println!("=== Concurrency and Parallelism Macros ===");
    
    // Thread-safe wrapper example
    let counter = SafeCounter::new(0);
    let counter_clone = counter.clone();
    
    let handle1 = counter.spawn_writer(|count| {
        for i in 0..5 {
            *count += i;
            println!("Writer 1: count = {}", *count);
            thread::sleep(std::time::Duration::from_millis(10));
        }
    });
    
    let handle2 = counter_clone.spawn_reader(|count| {
        println!("Reader: final count = {}", *count);
    });
    
    handle1.join().unwrap();
    handle2.join().unwrap();
    
    // Parallel map example
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    let squares = parallel_map!(numbers, 4, |x| x * x);
    println!("Parallel squares: {:?}", squares);
    
    // Actor system example
    let actor = Counter::new(0, "TestCounter".to_string());
    
    actor.increment().unwrap();
    actor.add(5).unwrap();
    actor.get_count().unwrap();
    
    thread::sleep(std::time::Duration::from_millis(100));
    actor.stop();
    
    // Barrier synchronization example
    sync_barrier!(sync_test, 3, |thread_id| {
        println!("Thread {} reached synchronization point", thread_id);
    });
    
    println!("All concurrency examples completed!");
}
```

Concurrency and parallelism macros simplify complex multi-threaded  
programming patterns. Thread-safe wrappers provide safe shared access,  
parallel computation macros distribute work across threads, actor  
systems enable message-passing concurrency, and work-stealing queues  
balance load dynamically. Barrier synchronization coordinates thread  
execution. These patterns enable scalable concurrent applications  
while maintaining safety and preventing data races.  

These 15 advanced macro examples demonstrate the full power of Rust's  
metaprogramming capabilities. From procedural macros and attribute  
transformations to compile-time computations and recursive patterns,  
these techniques enable creating sophisticated abstractions while  
maintaining Rust's safety and performance guarantees. Advanced macros  
unlock domain-specific languages, zero-cost optimizations, and powerful  
code generation that would be impossible in many other languages.  
