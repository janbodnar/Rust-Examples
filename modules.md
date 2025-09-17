# Rust Modules

## Basic module definition

Creating a simple module with functions and organizing code into logical  
units. Modules provide namespacing and encapsulation for related  
functionality.

```rust
mod greetings {
    pub fn hello() {
        println!("Hello from the greetings module!");
    }
    
    pub fn goodbye() {
        println!("Goodbye from the greetings module!");
    }
    
    // Private function - only accessible within this module
    fn private_function() {
        println!("This is private");
    }
}

fn main() {
    greetings::hello();
    greetings::goodbye();
    // greetings::private_function(); // This would cause a compile error
}
```

The `mod` keyword defines a module containing related functions. The `pub`  
keyword makes functions publicly accessible from outside the module. By  
default, all items in a module are private. The `::` syntax is used to  
access items within a module. Private functions are only accessible within  
the same module, providing encapsulation and hiding implementation details.

## Nested modules

Organizing code with multiple levels of modules to create hierarchical  
structure. Nested modules allow for more granular organization of related  
functionality.

```rust
mod database {
    pub mod users {
        pub fn create_user(name: &str) {
            println!("Creating user: {}", name);
        }
        
        pub fn delete_user(id: u32) {
            println!("Deleting user with ID: {}", id);
        }
    }
    
    pub mod posts {
        pub fn create_post(title: &str) {
            println!("Creating post: {}", title);
        }
        
        pub fn list_posts() {
            println!("Listing all posts");
        }
    }
    
    // Private module - not accessible outside database module
    mod config {
        pub fn get_connection_string() -> &'static str {
            "localhost:5432"
        }
    }
}

fn main() {
    database::users::create_user("Alice");
    database::users::delete_user(42);
    database::posts::create_post("My First Post");
    database::posts::list_posts();
    
    // database::config::get_connection_string(); // Compile error - private module
}
```

Nested modules create a hierarchical namespace structure. Each level adds  
another layer of organization. The `database::users::create_user` path  
shows how to access deeply nested functions. Private modules like `config`  
are only accessible within their parent module, allowing internal  
organization without exposing implementation details.

## Module visibility and privacy

Demonstrating different visibility levels and privacy controls within  
modules. Understanding `pub`, `pub(crate)`, `pub(super)`, and private  
visibility.

```rust
mod outer {
    pub fn public_function() {
        println!("This is public");
        inner::crate_visible();
        inner::super_visible();
        inner::private_function();
    }
    
    pub mod inner {
        pub fn public_function() {
            println!("Inner public function");
        }
        
        pub(crate) fn crate_visible() {
            println!("Visible within the crate");
        }
        
        pub(super) fn super_visible() {
            println!("Visible to parent module");
        }
        
        fn private_function() {
            println!("Private to this module");
        }
    }
}

fn main() {
    outer::public_function();
    outer::inner::public_function();
    outer::inner::crate_visible(); // Works - same crate
    // outer::inner::super_visible(); // Compile error - only visible to parent
    // outer::inner::private_function(); // Compile error - private
}
```

Different visibility modifiers control where code can be accessed. `pub`  
makes items publicly available everywhere. `pub(crate)` restricts access  
to the current crate. `pub(super)` allows access only from the parent  
module. Items without `pub` are private to the current module. This system  
provides fine-grained control over code visibility and encapsulation.

## Module with structs and implementations

Creating modules containing structs, enums, and their implementations.  
Modules can encapsulate data structures and their related methods together.

```rust
mod geometry {
    #[derive(Debug)]
    pub struct Rectangle {
        width: f64,
        height: f64,
    }
    
    #[derive(Debug)]
    pub enum Shape {
        Circle(f64),
        Rectangle(Rectangle),
        Triangle { base: f64, height: f64 },
    }
    
    impl Rectangle {
        pub fn new(width: f64, height: f64) -> Self {
            Rectangle { width, height }
        }
        
        pub fn area(&self) -> f64 {
            self.width * self.height
        }
        
        pub fn perimeter(&self) -> f64 {
            2.0 * (self.width + self.height)
        }
    }
    
    impl Shape {
        pub fn area(&self) -> f64 {
            match self {
                Shape::Circle(radius) => std::f64::consts::PI * radius * radius,
                Shape::Rectangle(rect) => rect.area(),
                Shape::Triangle { base, height } => 0.5 * base * height,
            }
        }
    }
}

fn main() {
    let rect = geometry::Rectangle::new(10.0, 5.0);
    println!("Rectangle: {:?}", rect);
    println!("Area: {}", rect.area());
    println!("Perimeter: {}", rect.perimeter());
    
    let shapes = vec![
        geometry::Shape::Circle(3.0),
        geometry::Shape::Rectangle(rect),
        geometry::Shape::Triangle { base: 4.0, height: 6.0 },
    ];
    
    for shape in shapes {
        println!("Shape area: {:.2}", shape.area());
    }
}
```

Modules can contain struct definitions, enums, and their implementations  
together, creating cohesive units of functionality. The `impl` blocks  
define methods for the types. Public structs allow external creation and  
usage, while keeping field access controlled. This pattern promotes good  
code organization by grouping related data structures and their behavior.

## File-based modules

Organizing modules across multiple files for better code organization.  
Large modules can be split into separate files to improve maintainability.

```rust
// File: src/main.rs
mod math_utils;  // Declares a module from math_utils.rs file
mod string_utils; // Declares a module from string_utils.rs file

fn main() {
    let result = math_utils::add(5, 3);
    println!("5 + 3 = {}", result);
    
    let factorial = math_utils::factorial(5);
    println!("5! = {}", factorial);
    
    let reversed = string_utils::reverse("hello");
    println!("Reversed: {}", reversed);
    
    let capitalized = string_utils::capitalize("world");
    println!("Capitalized: {}", capitalized);
}

// File: src/math_utils.rs
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

pub fn multiply(a: i32, b: i32) -> i32 {
    a * b
}

pub fn factorial(n: u32) -> u64 {
    match n {
        0 | 1 => 1,
        _ => n as u64 * factorial(n - 1),
    }
}

// File: src/string_utils.rs
pub fn reverse(s: &str) -> String {
    s.chars().rev().collect()
}

pub fn capitalize(s: &str) -> String {
    let mut chars = s.chars();
    match chars.next() {
        None => String::new(),
        Some(first) => first.to_uppercase().collect::<String>() + chars.as_str(),
    }
}

pub fn word_count(s: &str) -> usize {
    s.split_whitespace().count()
}
```

File-based modules allow splitting code across multiple files for better  
organization. The `mod module_name;` declaration tells Rust to look for  
a file named `module_name.rs` in the same directory. Each file becomes  
a module with the same name as the file. This approach keeps related  
functionality together while preventing files from becoming too large.  
Public functions are accessible using the `module::function` syntax.

## Module re-exports

Using `pub use` to re-export items and create convenient APIs. Re-exports  
allow creating simpler interfaces and hiding internal module structure.

```rust
mod internal {
    pub mod network {
        pub fn connect() {
            println!("Connecting to network...");
        }
        
        pub fn disconnect() {
            println!("Disconnecting from network...");
        }
    }
    
    pub mod database {
        pub fn connect() {
            println!("Connecting to database...");
        }
        
        pub fn query(sql: &str) {
            println!("Executing query: {}", sql);
        }
    }
    
    pub mod utils {
        pub fn log(message: &str) {
            println!("[LOG] {}", message);
        }
        
        pub fn format_timestamp() -> String {
            "2024-01-01 12:00:00".to_string()
        }
    }
}

// Re-export commonly used items at the crate root
pub use internal::network::{connect as network_connect, disconnect as network_disconnect};
pub use internal::database::{connect as db_connect, query as db_query};
pub use internal::utils::log;

// Create a convenience module with all utilities
pub mod prelude {
    pub use crate::internal::network::*;
    pub use crate::internal::database::*;
    pub use crate::internal::utils::*;
}

fn main() {
    // Using re-exported functions directly
    network_connect();
    db_connect();
    db_query("SELECT * FROM users");
    log("Application started");
    
    // Using prelude for wildcard imports
    use prelude::*;
    disconnect();
    format_timestamp();
}
```

The `pub use` statement re-exports items from other modules, making them  
available at a different path. This allows creating simplified APIs by  
hiding complex internal module structures. The `as` keyword provides  
aliases to avoid naming conflicts. Prelude modules offer convenient  
wildcard imports for commonly used functionality, similar to the standard  
library's approach.

## Conditional compilation with modules

Using conditional compilation to include different modules based on target  
platform or features. This allows platform-specific or feature-specific  
code organization.

```rust
#[cfg(target_os = "windows")]
mod windows_specific {
    pub fn get_system_info() -> String {
        "Windows system information".to_string()
    }
    
    pub fn run_command(cmd: &str) {
        println!("Running Windows command: {}", cmd);
    }
}

#[cfg(target_os = "linux")]
mod linux_specific {
    pub fn get_system_info() -> String {
        "Linux system information".to_string()
    }
    
    pub fn run_command(cmd: &str) {
        println!("Running Linux command: {}", cmd);
    }
}

#[cfg(feature = "networking")]
mod network {
    pub fn send_request(url: &str) {
        println!("Sending request to: {}", url);
    }
    
    pub fn download_file(url: &str, path: &str) {
        println!("Downloading {} to {}", url, path);
    }
}

#[cfg(debug_assertions)]
mod debug_utils {
    pub fn debug_print(msg: &str) {
        println!("[DEBUG] {}", msg);
    }
    
    pub fn assert_invariant(condition: bool, msg: &str) {
        if !condition {
            panic!("Invariant violated: {}", msg);
        }
    }
}

fn main() {
    #[cfg(target_os = "windows")]
    {
        let info = windows_specific::get_system_info();
        println!("{}", info);
        windows_specific::run_command("dir");
    }
    
    #[cfg(target_os = "linux")]
    {
        let info = linux_specific::get_system_info();
        println!("{}", info);
        linux_specific::run_command("ls");
    }
    
    #[cfg(feature = "networking")]
    {
        network::send_request("https://api.example.com");
        network::download_file("https://files.example.com/data.txt", "data.txt");
    }
    
    #[cfg(debug_assertions)]
    {
        debug_utils::debug_print("Application starting");
        debug_utils::assert_invariant(true, "This should not fail");
    }
}
```

The `#[cfg()]` attribute enables conditional compilation of modules based  
on various conditions. `target_os` allows platform-specific modules.  
`feature` enables optional functionality based on Cargo features.  
`debug_assertions` includes debug-only code. This pattern allows creating  
flexible, cross-platform applications with optional features while keeping  
the codebase organized and maintainable.

## Module with traits

Defining traits within modules to create reusable interfaces. Modules can  
encapsulate trait definitions and their implementations together for better  
organization.

```rust
mod drawable {
    pub trait Draw {
        fn draw(&self);
        fn area(&self) -> f64;
        
        // Default implementation
        fn describe(&self) {
            println!("This shape has an area of {:.2}", self.area());
        }
    }
    
    pub trait Colorable {
        fn set_color(&mut self, color: &str);
        fn get_color(&self) -> &str;
    }
    
    #[derive(Debug)]
    pub struct Circle {
        radius: f64,
        color: String,
    }
    
    #[derive(Debug)]
    pub struct Rectangle {
        width: f64,
        height: f64,
        color: String,
    }
    
    impl Circle {
        pub fn new(radius: f64) -> Self {
            Circle {
                radius,
                color: "black".to_string(),
            }
        }
    }
    
    impl Rectangle {
        pub fn new(width: f64, height: f64) -> Self {
            Rectangle {
                width,
                height,
                color: "black".to_string(),
            }
        }
    }
    
    impl Draw for Circle {
        fn draw(&self) {
            println!("Drawing a {} circle with radius {}", self.color, self.radius);
        }
        
        fn area(&self) -> f64 {
            std::f64::consts::PI * self.radius * self.radius
        }
    }
    
    impl Draw for Rectangle {
        fn draw(&self) {
            println!("Drawing a {} rectangle {}x{}", self.color, self.width, self.height);
        }
        
        fn area(&self) -> f64 {
            self.width * self.height
        }
    }
    
    impl Colorable for Circle {
        fn set_color(&mut self, color: &str) {
            self.color = color.to_string();
        }
        
        fn get_color(&self) -> &str {
            &self.color
        }
    }
    
    impl Colorable for Rectangle {
        fn set_color(&mut self, color: &str) {
            self.color = color.to_string();
        }
        
        fn get_color(&self) -> &str {
            &self.color
        }
    }
}

fn main() {
    use drawable::{Draw, Colorable, Circle, Rectangle};
    
    let mut circle = Circle::new(5.0);
    let mut rectangle = Rectangle::new(10.0, 8.0);
    
    // Use trait methods
    circle.draw();
    circle.describe();
    
    rectangle.draw();
    rectangle.describe();
    
    // Change colors
    circle.set_color("red");
    rectangle.set_color("blue");
    
    println!("Circle color: {}", circle.get_color());
    println!("Rectangle color: {}", rectangle.get_color());
    
    circle.draw();
    rectangle.draw();
}
```

Modules can define traits and their implementations together, creating  
cohesive interfaces and behavior. The `Draw` trait defines a common  
interface for drawing operations with a default implementation for  
`describe()`. The `Colorable` trait adds color functionality. This  
pattern allows creating flexible, extensible APIs where new types can  
implement the traits to gain the defined behavior.

## Workspace modules

Organizing multiple related modules within a workspace structure. This  
demonstrates how to structure larger projects with multiple interconnected  
modules.

```rust
// Simulating a workspace-like structure within a single file
mod app {
    pub mod core {
        pub trait Service {
            fn start(&self);
            fn stop(&self);
            fn status(&self) -> ServiceStatus;
        }
        
        #[derive(Debug, PartialEq)]
        pub enum ServiceStatus {
            Running,
            Stopped,
            Error(String),
        }
        
        pub struct Application {
            name: String,
            services: Vec<Box<dyn Service>>,
        }
        
        impl Application {
            pub fn new(name: &str) -> Self {
                Application {
                    name: name.to_string(),
                    services: Vec::new(),
                }
            }
            
            pub fn add_service(&mut self, service: Box<dyn Service>) {
                self.services.push(service);
            }
            
            pub fn start_all(&self) {
                println!("Starting application: {}", self.name);
                for service in &self.services {
                    service.start();
                }
            }
            
            pub fn stop_all(&self) {
                println!("Stopping application: {}", self.name);
                for service in &self.services {
                    service.stop();
                }
            }
        }
    }
    
    pub mod services {
        use super::core::{Service, ServiceStatus};
        
        pub struct DatabaseService {
            name: String,
            status: ServiceStatus,
        }
        
        pub struct WebServer {
            name: String,
            port: u16,
            status: ServiceStatus,
        }
        
        impl DatabaseService {
            pub fn new(name: &str) -> Self {
                DatabaseService {
                    name: name.to_string(),
                    status: ServiceStatus::Stopped,
                }
            }
        }
        
        impl WebServer {
            pub fn new(name: &str, port: u16) -> Self {
                WebServer {
                    name: name.to_string(),
                    port,
                    status: ServiceStatus::Stopped,
                }
            }
        }
        
        impl Service for DatabaseService {
            fn start(&self) {
                println!("Starting database service: {}", self.name);
            }
            
            fn stop(&self) {
                println!("Stopping database service: {}", self.name);
            }
            
            fn status(&self) -> ServiceStatus {
                ServiceStatus::Running
            }
        }
        
        impl Service for WebServer {
            fn start(&self) {
                println!("Starting web server: {} on port {}", self.name, self.port);
            }
            
            fn stop(&self) {
                println!("Stopping web server: {}", self.name);
            }
            
            fn status(&self) -> ServiceStatus {
                ServiceStatus::Running
            }
        }
    }
    
    pub mod config {
        use std::collections::HashMap;
        
        pub struct Configuration {
            settings: HashMap<String, String>,
        }
        
        impl Configuration {
            pub fn new() -> Self {
                Configuration {
                    settings: HashMap::new(),
                }
            }
            
            pub fn set(&mut self, key: &str, value: &str) {
                self.settings.insert(key.to_string(), value.to_string());
            }
            
            pub fn get(&self, key: &str) -> Option<&String> {
                self.settings.get(key)
            }
            
            pub fn load_defaults(&mut self) {
                self.set("database.host", "localhost");
                self.set("database.port", "5432");
                self.set("web.port", "8080");
                self.set("app.name", "MyApp");
            }
        }
    }
}

fn main() {
    use app::{core::Application, services::{DatabaseService, WebServer}, config::Configuration};
    
    let mut config = Configuration::new();
    config.load_defaults();
    
    let app_name = config.get("app.name").unwrap_or(&"DefaultApp".to_string());
    let web_port: u16 = config.get("web.port")
        .and_then(|p| p.parse().ok())
        .unwrap_or(8080);
    
    let mut app = Application::new(app_name);
    
    let db_service = DatabaseService::new("PostgreSQL");
    let web_service = WebServer::new("HTTP Server", web_port);
    
    app.add_service(Box::new(db_service));
    app.add_service(Box::new(web_service));
    
    app.start_all();
    println!("Application is running...");
    app.stop_all();
}
```

This example demonstrates a workspace-like organization with multiple  
interconnected modules. The `core` module defines common interfaces,  
`services` provides concrete implementations, and `config` handles  
configuration management. Modules use each other through well-defined  
interfaces, promoting loose coupling and high cohesion. This pattern  
scales well for larger applications with multiple components.

## Advanced module patterns

Demonstrating sophisticated module organization patterns including module  
hierarchies, type aliases, and advanced re-export strategies for complex  
applications.

```rust
mod advanced {
    // Type aliases within modules
    pub type UserId = u64;
    pub type Result<T> = std::result::Result<T, Box<dyn std::error::Error>>;
    
    pub mod errors {
        use std::fmt;
        
        #[derive(Debug)]
        pub enum AppError {
            InvalidInput(String),
            NetworkError(String),
            DatabaseError(String),
        }
        
        impl fmt::Display for AppError {
            fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
                match self {
                    AppError::InvalidInput(msg) => write!(f, "Invalid input: {}", msg),
                    AppError::NetworkError(msg) => write!(f, "Network error: {}", msg),
                    AppError::DatabaseError(msg) => write!(f, "Database error: {}", msg),
                }
            }
        }
        
        impl std::error::Error for AppError {}
    }
    
    pub mod user {
        use super::{UserId, Result, errors::AppError};
        
        #[derive(Debug, Clone)]
        pub struct User {
            id: UserId,
            name: String,
            email: String,
        }
        
        impl User {
            pub fn new(id: UserId, name: String, email: String) -> Result<Self> {
                if name.is_empty() {
                    return Err(Box::new(AppError::InvalidInput("Name cannot be empty".to_string())));
                }
                if !email.contains('@') {
                    return Err(Box::new(AppError::InvalidInput("Invalid email format".to_string())));
                }
                
                Ok(User { id, name, email })
            }
            
            pub fn id(&self) -> UserId {
                self.id
            }
            
            pub fn name(&self) -> &str {
                &self.name
            }
            
            pub fn email(&self) -> &str {
                &self.email
            }
        }
        
        pub mod repository {
            use super::{User, UserId, Result};
            use super::super::errors::AppError;
            use std::collections::HashMap;
            
            pub struct UserRepository {
                users: HashMap<UserId, User>,
                next_id: UserId,
            }
            
            impl UserRepository {
                pub fn new() -> Self {
                    UserRepository {
                        users: HashMap::new(),
                        next_id: 1,
                    }
                }
                
                pub fn create_user(&mut self, name: String, email: String) -> Result<UserId> {
                    let id = self.next_id;
                    let user = User::new(id, name, email)?;
                    self.users.insert(id, user);
                    self.next_id += 1;
                    Ok(id)
                }
                
                pub fn get_user(&self, id: UserId) -> Option<&User> {
                    self.users.get(&id)
                }
                
                pub fn update_user(&mut self, id: UserId, name: String, email: String) -> Result<()> {
                    if let Some(user) = self.users.get_mut(&id) {
                        let updated_user = User::new(id, name, email)?;
                        *user = updated_user;
                        Ok(())
                    } else {
                        Err(Box::new(AppError::DatabaseError("User not found".to_string())))
                    }
                }
                
                pub fn delete_user(&mut self, id: UserId) -> bool {
                    self.users.remove(&id).is_some()
                }
                
                pub fn list_users(&self) -> Vec<&User> {
                    self.users.values().collect()
                }
            }
        }
    }
    
    // Re-export commonly used items
    pub use errors::AppError;
    pub use user::{User, repository::UserRepository};
    
    // Create a prelude module with the most common imports
    pub mod prelude {
        pub use super::{UserId, Result, AppError, User, UserRepository};
    }
}

fn main() {
    use advanced::prelude::*;
    
    let mut repo = UserRepository::new();
    
    // Create users
    match repo.create_user("Alice".to_string(), "alice@example.com".to_string()) {
        Ok(id) => println!("Created user with ID: {}", id),
        Err(e) => println!("Error creating user: {}", e),
    }
    
    match repo.create_user("Bob".to_string(), "bob@example.com".to_string()) {
        Ok(id) => println!("Created user with ID: {}", id),
        Err(e) => println!("Error creating user: {}", e),
    }
    
    // Try to create invalid user
    match repo.create_user("".to_string(), "invalid-email".to_string()) {
        Ok(id) => println!("Created user with ID: {}", id),
        Err(e) => println!("Error creating user: {}", e),
    }
    
    // List all users
    println!("\nAll users:");
    for user in repo.list_users() {
        println!("  ID: {}, Name: {}, Email: {}", user.id(), user.name(), user.email());
    }
    
    // Update a user
    if let Err(e) = repo.update_user(1, "Alice Smith".to_string(), "alice.smith@example.com".to_string()) {
        println!("Error updating user: {}", e);
    }
    
    // Get specific user
    if let Some(user) = repo.get_user(1) {
        println!("Updated user: {}, {}", user.name(), user.email());
    }
}
```

This advanced example demonstrates sophisticated module organization with  
nested modules, type aliases, custom error types, and strategic re-exports.  
The pattern uses module hierarchies to create clear separation of concerns  
while providing convenient access through prelude modules. Type aliases  
like `UserId` and `Result<T>` create domain-specific types that improve  
code clarity and maintainability. The repository pattern shows how modules  
can encapsulate complex business logic while maintaining clean interfaces.