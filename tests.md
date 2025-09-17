# Writing Tests

Rust provides a built-in testing framework that makes it easy to write and  
run tests alongside your code. Tests help ensure code correctness, catch  
regressions, and enable confident refactoring. Good tests serve as living  
documentation and improve code quality.  

Key principles of testing in Rust:  
- **Built-in framework**: No external dependencies needed for basic testing  
- **Unit tests**: Test individual functions and methods in isolation  
- **Integration tests**: Test how different parts work together  
- **Documentation tests**: Ensure code examples in docs actually work  
- **Benchmarks**: Measure and track performance over time  

Rust tests are written using the `#[test]` attribute and assert macros like  
`assert!`, `assert_eq!`, and `assert_ne!`. Tests run in parallel by default  
and provide detailed failure information when something goes wrong.  

## Basic unit test

A simple unit test demonstrating the fundamental testing structure.  
Use the `#[test]` attribute to mark functions as tests.  

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[test]
fn test_add() {
    let result = add(2, 3);
    assert_eq!(result, 5);
}

#[test]
fn test_add_negative() {
    assert_eq!(add(-1, 1), 0);
    assert_eq!(add(-5, -3), -8);
}

fn main() {
    println!("Run tests with: cargo test");
}
```

The `#[test]` attribute tells Rust that this is a test function. Tests  
typically follow a pattern: setup, execution, and assertion. Use `cargo test`  
to run all tests in your project.  

## Testing with assertions

Different assertion macros for various testing scenarios.  
Choose the right assertion for clear, descriptive test failures.  

```rust
fn is_even(n: i32) -> bool {
    n % 2 == 0
}

fn divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err("Division by zero".to_string())
    } else {
        Ok(a / b)
    }
}

#[test]
fn test_basic_assertions() {
    // Basic boolean assertion
    assert!(is_even(4));
    assert!(!is_even(3));
    
    // Equality assertions
    assert_eq!(10, 5 + 5);
    assert_ne!(10, 5 + 4);
    
    // Floating point comparison (be careful with precision)
    let result = 0.1 + 0.2;
    assert!((result - 0.3).abs() < f64::EPSILON);
}

#[test]
fn test_result_assertions() {
    let success = divide(10.0, 2.0);
    assert!(success.is_ok());
    assert_eq!(success.unwrap(), 5.0);
    
    let failure = divide(10.0, 0.0);
    assert!(failure.is_err());
}

fn main() {
    println!("Testing various assertion types");
}
```

Different assertions provide different levels of detail when tests fail.  
`assert_eq!` and `assert_ne!` show the actual vs expected values, making  
debugging easier than plain `assert!`.  

## Testing collections

Testing arrays, vectors, and other collections with various approaches.  
Collections often require specific testing strategies for thorough coverage.  

```rust
fn filter_even(numbers: Vec<i32>) -> Vec<i32> {
    numbers.into_iter().filter(|&x| x % 2 == 0).collect()
}

fn sum_of_squares(numbers: &[i32]) -> i32 {
    numbers.iter().map(|&x| x * x).sum()
}

#[test]
fn test_vector_operations() {
    let input = vec![1, 2, 3, 4, 5, 6];
    let evens = filter_even(input);
    
    assert_eq!(evens, vec![2, 4, 6]);
    assert_eq!(evens.len(), 3);
    assert!(evens.contains(&4));
}

#[test]
fn test_array_calculations() {
    let numbers = [1, 2, 3, 4];
    let result = sum_of_squares(&numbers);
    
    // 1² + 2² + 3² + 4² = 1 + 4 + 9 + 16 = 30
    assert_eq!(result, 30);
}

#[test]
fn test_empty_collections() {
    let empty: Vec<i32> = vec![];
    assert!(filter_even(empty).is_empty());
    assert_eq!(sum_of_squares(&[]), 0);
}

fn main() {
    println!("Testing collection operations");
}
```

When testing collections, consider edge cases like empty collections,  
single elements, and large datasets. Test both the content and properties  
like length and membership.  

## Testing with custom error types

Testing functions that return custom error types requires specific  
approaches to verify both success and failure cases.  

```rust
#[derive(Debug, PartialEq)]
enum MathError {
    DivisionByZero,
    NegativeSquareRoot,
    Overflow,
}

fn safe_divide(a: f64, b: f64) -> Result<f64, MathError> {
    if b == 0.0 {
        Err(MathError::DivisionByZero)
    } else {
        Ok(a / b)
    }
}

fn safe_sqrt(x: f64) -> Result<f64, MathError> {
    if x < 0.0 {
        Err(MathError::NegativeSquareRoot)
    } else {
        Ok(x.sqrt())
    }
}

#[test]
fn test_successful_operations() {
    assert_eq!(safe_divide(10.0, 2.0), Ok(5.0));
    assert_eq!(safe_sqrt(9.0), Ok(3.0));
}

#[test]
fn test_error_conditions() {
    assert_eq!(safe_divide(5.0, 0.0), Err(MathError::DivisionByZero));
    assert_eq!(safe_sqrt(-4.0), Err(MathError::NegativeSquareRoot));
}

#[test]
fn test_error_matching() {
    match safe_divide(1.0, 0.0) {
        Err(MathError::DivisionByZero) => (), // Expected
        _ => panic!("Expected DivisionByZero error"),
    }
}

fn main() {
    println!("Testing custom error types");
}
```

Custom error types should implement `Debug` and `PartialEq` for easy  
testing. Test both the success path and all possible error conditions  
to ensure comprehensive coverage.  

## Testing panics

Testing code that should panic under certain conditions.  
Use `#[should_panic]` to verify panic behavior.  

```rust
fn divide_panicking(a: i32, b: i32) -> i32 {
    if b == 0 {
        panic!("Cannot divide by zero!");
    }
    a / b
}

fn validate_age(age: u8) -> u8 {
    if age > 150 {
        panic!("Age cannot be greater than 150");
    }
    age
}

#[test]
#[should_panic]
fn test_division_panic() {
    divide_panicking(10, 0);
}

#[test]
#[should_panic(expected = "Age cannot be greater than 150")]
fn test_age_validation_panic() {
    validate_age(200);
}

#[test]
fn test_normal_operation() {
    assert_eq!(divide_panicking(10, 2), 5);
    assert_eq!(validate_age(25), 25);
}

// Alternative: Testing panic with std::panic::catch_unwind
#[test]
fn test_panic_catching() {
    let result = std::panic::catch_unwind(|| {
        divide_panicking(5, 0)
    });
    assert!(result.is_err());
}

fn main() {
    println!("Testing panic behavior");
}
```

The `#[should_panic]` attribute expects the test to panic. You can specify  
the expected panic message for more precise testing. Use panic tests  
sparingly and prefer returning `Result` types.  

## Testing structs and methods

Testing struct implementations and their methods requires testing both  
state changes and behavior correctness.  

```rust
#[derive(Debug, PartialEq)]
struct Counter {
    value: i32,
}

impl Counter {
    fn new() -> Self {
        Self { value: 0 }
    }
    
    fn increment(&mut self) {
        self.value += 1;
    }
    
    fn add(&mut self, amount: i32) {
        self.value += amount;
    }
    
    fn reset(&mut self) {
        self.value = 0;
    }
    
    fn get_value(&self) -> i32 {
        self.value
    }
}

#[test]
fn test_counter_creation() {
    let counter = Counter::new();
    assert_eq!(counter.get_value(), 0);
}

#[test]
fn test_counter_increment() {
    let mut counter = Counter::new();
    counter.increment();
    assert_eq!(counter.get_value(), 1);
    
    counter.increment();
    assert_eq!(counter.get_value(), 2);
}

#[test]
fn test_counter_add() {
    let mut counter = Counter::new();
    counter.add(5);
    assert_eq!(counter.get_value(), 5);
    
    counter.add(-2);
    assert_eq!(counter.get_value(), 3);
}

#[test]
fn test_counter_reset() {
    let mut counter = Counter::new();
    counter.add(10);
    counter.reset();
    assert_eq!(counter.get_value(), 0);
}

fn main() {
    println!("Testing struct methods");
}
```

When testing structs, verify the initial state, test each method's behavior,  
and ensure methods interact correctly. Test both the public interface and  
any invariants your struct should maintain.  

## Testing with fixtures and setup

Creating reusable test data and setup code for consistent testing.  
Fixtures help maintain test isolation and reduce duplication.  

```rust
struct BankAccount {
    balance: f64,
    owner: String,
}

impl BankAccount {
    fn new(owner: String) -> Self {
        Self { balance: 0.0, owner }
    }
    
    fn deposit(&mut self, amount: f64) -> Result<(), String> {
        if amount <= 0.0 {
            return Err("Amount must be positive".to_string());
        }
        self.balance += amount;
        Ok(())
    }
    
    fn withdraw(&mut self, amount: f64) -> Result<(), String> {
        if amount <= 0.0 {
            return Err("Amount must be positive".to_string());
        }
        if amount > self.balance {
            return Err("Insufficient funds".to_string());
        }
        self.balance -= amount;
        Ok(())
    }
    
    fn get_balance(&self) -> f64 {
        self.balance
    }
}

// Test fixture function
fn setup_account_with_balance() -> BankAccount {
    let mut account = BankAccount::new("Alice".to_string());
    account.deposit(100.0).unwrap();
    account
}

#[test]
fn test_account_creation() {
    let account = BankAccount::new("Bob".to_string());
    assert_eq!(account.get_balance(), 0.0);
    assert_eq!(account.owner, "Bob");
}

#[test]
fn test_deposit() {
    let mut account = BankAccount::new("Charlie".to_string());
    assert!(account.deposit(50.0).is_ok());
    assert_eq!(account.get_balance(), 50.0);
}

#[test]
fn test_withdraw_with_fixture() {
    let mut account = setup_account_with_balance();
    assert!(account.withdraw(30.0).is_ok());
    assert_eq!(account.get_balance(), 70.0);
}

#[test]
fn test_insufficient_funds() {
    let mut account = setup_account_with_balance();
    assert!(account.withdraw(150.0).is_err());
    assert_eq!(account.get_balance(), 100.0); // Balance unchanged
}

fn main() {
    println!("Testing with fixtures");
}
```

Fixtures reduce test code duplication and ensure consistent test setup.  
Create helper functions that return commonly needed test objects or  
perform standard setup operations.  

## Parameterized testing

Testing multiple inputs with the same test logic using data-driven  
approaches for comprehensive coverage.  

```rust
fn is_prime(n: u32) -> bool {
    if n < 2 {
        return false;
    }
    if n == 2 {
        return true;
    }
    if n % 2 == 0 {
        return false;
    }
    
    let limit = (n as f64).sqrt() as u32;
    for i in (3..=limit).step_by(2) {
        if n % i == 0 {
            return false;
        }
    }
    true
}

fn fibonacci(n: u32) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}

#[test]
fn test_prime_numbers() {
    let test_cases = [
        (0, false),
        (1, false),
        (2, true),
        (3, true),
        (4, false),
        (5, true),
        (6, false),
        (7, true),
        (8, false),
        (9, false),
        (10, false),
        (11, true),
        (17, true),
        (25, false),
        (29, true),
    ];
    
    for (input, expected) in test_cases {
        assert_eq!(
            is_prime(input), 
            expected, 
            "is_prime({}) should return {}", 
            input, 
            expected
        );
    }
}

#[test]
fn test_fibonacci_sequence() {
    let expected_values = [
        (0, 0),
        (1, 1),
        (2, 1),
        (3, 2),
        (4, 3),
        (5, 5),
        (6, 8),
        (7, 13),
        (8, 21),
        (9, 34),
        (10, 55),
    ];
    
    for (n, expected) in expected_values {
        assert_eq!(fibonacci(n), expected, "fibonacci({}) should be {}", n, expected);
    }
}

fn main() {
    println!("Parameterized testing examples");
}
```

Parameterized tests allow you to test many inputs efficiently. Use arrays  
or vectors of test cases with descriptive assertion messages that include  
the input values for easier debugging.  

## Testing modules and visibility

Testing private functions and module internals by organizing tests  
within the same module or using the `super` keyword.  

```rust
mod calculator {
    pub struct Calculator {
        memory: f64,
    }
    
    impl Calculator {
        pub fn new() -> Self {
            Self { memory: 0.0 }
        }
        
        pub fn add(&mut self, value: f64) -> f64 {
            self.memory = self.add_internal(self.memory, value);
            self.memory
        }
        
        pub fn get_memory(&self) -> f64 {
            self.memory
        }
        
        pub fn clear(&mut self) {
            self.memory = 0.0;
        }
        
        // Private function we want to test
        fn add_internal(&self, a: f64, b: f64) -> f64 {
            a + b
        }
        
        // Another private helper
        fn validate_input(&self, value: f64) -> bool {
            value.is_finite()
        }
    }
    
    // Tests within the same module can access private items
    #[cfg(test)]
    mod tests {
        use super::*;
        
        #[test]
        fn test_add_internal() {
            let calc = Calculator::new();
            assert_eq!(calc.add_internal(2.0, 3.0), 5.0);
            assert_eq!(calc.add_internal(-1.0, 1.0), 0.0);
        }
        
        #[test]
        fn test_validate_input() {
            let calc = Calculator::new();
            assert!(calc.validate_input(42.0));
            assert!(calc.validate_input(-42.0));
            assert!(!calc.validate_input(f64::NAN));
            assert!(!calc.validate_input(f64::INFINITY));
        }
        
        #[test]
        fn test_public_interface() {
            let mut calc = Calculator::new();
            assert_eq!(calc.get_memory(), 0.0);
            
            calc.add(5.0);
            assert_eq!(calc.get_memory(), 5.0);
            
            calc.add(3.0);
            assert_eq!(calc.get_memory(), 8.0);
            
            calc.clear();
            assert_eq!(calc.get_memory(), 0.0);
        }
    }
}

fn main() {
    use calculator::Calculator;
    
    let mut calc = Calculator::new();
    calc.add(10.0);
    println!("Calculator memory: {}", calc.get_memory());
}
```

Place tests in the same module as the code to test private functions.  
Use `#[cfg(test)]` to ensure test code only compiles during testing.  
The `use super::*;` imports items from the parent module.  

## Testing with external dependencies

Creating testable code that depends on external services or resources  
using dependency injection and mocking patterns.  

```rust
trait HttpClient {
    fn get(&self, url: &str) -> Result<String, String>;
}

struct ApiService<T: HttpClient> {
    client: T,
    base_url: String,
}

impl<T: HttpClient> ApiService<T> {
    fn new(client: T, base_url: String) -> Self {
        Self { client, base_url }
    }
    
    fn get_user(&self, id: u32) -> Result<User, String> {
        let url = format!("{}/users/{}", self.base_url, id);
        let response = self.client.get(&url)?;
        
        // Simplified JSON parsing
        if response.contains("\"id\":") {
            Ok(User {
                id,
                name: "Test User".to_string(),
                email: "test@example.com".to_string(),
            })
        } else {
            Err("Invalid response".to_string())
        }
    }
}

#[derive(Debug, PartialEq)]
struct User {
    id: u32,
    name: String,
    email: String,
}

// Mock implementation for testing
struct MockHttpClient {
    responses: std::collections::HashMap<String, String>,
}

impl MockHttpClient {
    fn new() -> Self {
        Self {
            responses: std::collections::HashMap::new(),
        }
    }
    
    fn set_response(&mut self, url: String, response: String) {
        self.responses.insert(url, response);
    }
}

impl HttpClient for MockHttpClient {
    fn get(&self, url: &str) -> Result<String, String> {
        self.responses
            .get(url)
            .cloned()
            .ok_or_else(|| format!("No mock response for {}", url))
    }
}

#[test]
fn test_successful_user_fetch() {
    let mut mock_client = MockHttpClient::new();
    mock_client.set_response(
        "https://api.example.com/users/123".to_string(),
        r#"{"id": 123, "name": "Test User"}"#.to_string(),
    );
    
    let service = ApiService::new(mock_client, "https://api.example.com".to_string());
    let user = service.get_user(123).unwrap();
    
    assert_eq!(user.id, 123);
    assert_eq!(user.name, "Test User");
}

#[test]
fn test_failed_user_fetch() {
    let mock_client = MockHttpClient::new();
    let service = ApiService::new(mock_client, "https://api.example.com".to_string());
    
    let result = service.get_user(456);
    assert!(result.is_err());
}

fn main() {
    println!("Testing with dependency injection and mocks");
}
```

Use traits to abstract external dependencies, then create mock  
implementations for testing. This allows you to test your logic  
without relying on external services or resources.  

## Testing async code

Testing asynchronous functions and operations using tokio's test  
runtime and async-specific testing patterns.  

```rust
use std::time::Duration;

async fn fetch_data(delay_ms: u64) -> Result<String, String> {
    // Simulate network delay
    tokio::time::sleep(Duration::from_millis(delay_ms)).await;
    
    if delay_ms > 1000 {
        Err("Timeout".to_string())
    } else {
        Ok(format!("Data fetched after {}ms", delay_ms))
    }
}

async fn process_multiple_requests() -> Vec<Result<String, String>> {
    let futures = vec![
        fetch_data(100),
        fetch_data(200),
        fetch_data(50),
    ];
    
    // Wait for all futures to complete
    let results = futures::future::join_all(futures).await;
    results
}

// Async tests require the tokio::test attribute
#[tokio::test]
async fn test_successful_fetch() {
    let result = fetch_data(100).await;
    assert!(result.is_ok());
    assert_eq!(result.unwrap(), "Data fetched after 100ms");
}

#[tokio::test]
async fn test_timeout_error() {
    let result = fetch_data(1500).await;
    assert!(result.is_err());
    assert_eq!(result.unwrap_err(), "Timeout");
}

#[tokio::test]
async fn test_concurrent_requests() {
    let start = std::time::Instant::now();
    let results = process_multiple_requests().await;
    let duration = start.elapsed();
    
    // All requests should succeed
    assert_eq!(results.len(), 3);
    for result in &results {
        assert!(result.is_ok());
    }
    
    // Should complete in roughly 200ms (not 350ms sequentially)
    assert!(duration < Duration::from_millis(300));
}

#[tokio::test]
async fn test_timeout_behavior() {
    let timeout_duration = Duration::from_millis(500);
    
    let result = tokio::time::timeout(timeout_duration, fetch_data(100)).await;
    assert!(result.is_ok()); // Should complete within timeout
    
    let result = tokio::time::timeout(timeout_duration, fetch_data(800)).await;
    assert!(result.is_ok()); // Should complete within timeout
}

fn main() {
    println!("Use 'cargo test' to run async tests");
}
```

Use `#[tokio::test]` for async test functions. Test both success and  
failure cases, verify timing behavior, and use `tokio::time::timeout`  
to test timeout scenarios.  

## Property-based testing

Testing with automatically generated inputs to find edge cases and  
verify properties that should hold for all valid inputs.  

```rust
fn gcd(mut a: u32, mut b: u32) -> u32 {
    while b != 0 {
        let temp = b;
        b = a % b;
        a = temp;
    }
    a
}

fn lcm(a: u32, b: u32) -> u32 {
    if a == 0 || b == 0 {
        0
    } else {
        (a / gcd(a, b)) * b
    }
}

fn quicksort(mut arr: Vec<i32>) -> Vec<i32> {
    if arr.len() <= 1 {
        return arr;
    }
    
    let pivot = arr.remove(0);
    let mut less = Vec::new();
    let mut greater = Vec::new();
    
    for item in arr {
        if item <= pivot {
            less.push(item);
        } else {
            greater.push(item);
        }
    }
    
    let mut result = quicksort(less);
    result.push(pivot);
    result.extend(quicksort(greater));
    result
}

#[test]
fn test_gcd_properties() {
    // Property: gcd(a, b) divides both a and b
    let test_cases = [
        (12, 8),
        (15, 25),
        (17, 19),
        (100, 50),
        (13, 13),
    ];
    
    for (a, b) in test_cases {
        let g = gcd(a, b);
        assert_eq!(a % g, 0, "gcd({}, {}) = {} should divide {}", a, b, g, a);
        assert_eq!(b % g, 0, "gcd({}, {}) = {} should divide {}", a, b, g, b);
    }
}

#[test]
fn test_lcm_properties() {
    let test_cases = [(4, 6), (12, 18), (7, 14), (5, 7)];
    
    for (a, b) in test_cases {
        let l = lcm(a, b);
        let g = gcd(a, b);
        
        // Property: lcm(a, b) * gcd(a, b) = a * b
        assert_eq!(l * g, a * b, "lcm({}, {}) * gcd({}, {}) should equal {} * {}", 
                  a, b, a, b, a, b);
        
        // Property: lcm(a, b) is divisible by both a and b
        assert_eq!(l % a, 0, "lcm({}, {}) = {} should be divisible by {}", a, b, l, a);
        assert_eq!(l % b, 0, "lcm({}, {}) = {} should be divisible by {}", a, b, l, b);
    }
}

#[test]
fn test_quicksort_properties() {
    let test_arrays = [
        vec![3, 1, 4, 1, 5, 9, 2, 6],
        vec![5, 4, 3, 2, 1],
        vec![1, 2, 3, 4, 5],
        vec![42],
        vec![],
        vec![1, 1, 1, 1],
    ];
    
    for original in test_arrays {
        let sorted = quicksort(original.clone());
        
        // Property: length should be preserved
        assert_eq!(sorted.len(), original.len(), 
                  "Sorted array should have same length as original");
        
        // Property: result should be sorted
        for i in 1..sorted.len() {
            assert!(sorted[i-1] <= sorted[i], 
                   "Array should be sorted: {:?}", sorted);
        }
        
        // Property: all original elements should be present
        let mut original_sorted = original.clone();
        original_sorted.sort();
        assert_eq!(sorted, original_sorted, 
                  "Sorted result should contain all original elements");
    }
}

fn main() {
    println!("Property-based testing verifies invariants across many inputs");
}
```

Property-based testing focuses on invariants that should always hold  
rather than specific input-output pairs. Test mathematical properties,  
preservation of data, and fundamental characteristics of your algorithms.  

## Integration testing

Testing how different components work together using integration tests  
in the `tests/` directory for black-box testing.  

```rust
// This would typically be in src/lib.rs
pub mod user_service {
    use std::collections::HashMap;
    
    #[derive(Debug, Clone, PartialEq)]
    pub struct User {
        pub id: u32,
        pub username: String,
        pub email: String,
        pub active: bool,
    }
    
    pub struct UserService {
        users: HashMap<u32, User>,
        next_id: u32,
    }
    
    impl UserService {
        pub fn new() -> Self {
            Self {
                users: HashMap::new(),
                next_id: 1,
            }
        }
        
        pub fn create_user(&mut self, username: String, email: String) -> Result<User, String> {
            if username.is_empty() {
                return Err("Username cannot be empty".to_string());
            }
            
            if !email.contains('@') {
                return Err("Invalid email format".to_string());
            }
            
            let user = User {
                id: self.next_id,
                username,
                email,
                active: true,
            };
            
            self.users.insert(self.next_id, user.clone());
            self.next_id += 1;
            
            Ok(user)
        }
        
        pub fn get_user(&self, id: u32) -> Option<&User> {
            self.users.get(&id)
        }
        
        pub fn deactivate_user(&mut self, id: u32) -> Result<(), String> {
            match self.users.get_mut(&id) {
                Some(user) => {
                    user.active = false;
                    Ok(())
                }
                None => Err("User not found".to_string()),
            }
        }
        
        pub fn list_active_users(&self) -> Vec<&User> {
            self.users
                .values()
                .filter(|user| user.active)
                .collect()
        }
    }
}

// Integration tests test the full workflow
#[cfg(test)]
mod integration_tests {
    use super::user_service::*;
    
    #[test]
    fn test_complete_user_lifecycle() {
        let mut service = UserService::new();
        
        // Create multiple users
        let user1 = service.create_user(
            "alice".to_string(), 
            "alice@example.com".to_string()
        ).unwrap();
        
        let user2 = service.create_user(
            "bob".to_string(), 
            "bob@example.com".to_string()
        ).unwrap();
        
        // Verify users were created with correct IDs
        assert_eq!(user1.id, 1);
        assert_eq!(user2.id, 2);
        assert!(user1.active);
        assert!(user2.active);
        
        // Test retrieval
        let retrieved_user = service.get_user(1).unwrap();
        assert_eq!(retrieved_user, &user1);
        
        // Test listing active users
        let active_users = service.list_active_users();
        assert_eq!(active_users.len(), 2);
        
        // Test deactivation
        service.deactivate_user(1).unwrap();
        let active_users = service.list_active_users();
        assert_eq!(active_users.len(), 1);
        assert_eq!(active_users[0].id, 2);
        
        // Verify deactivated user is still retrievable but inactive
        let deactivated_user = service.get_user(1).unwrap();
        assert!(!deactivated_user.active);
    }
    
    #[test]
    fn test_error_handling_integration() {
        let mut service = UserService::new();
        
        // Test validation errors
        assert!(service.create_user("".to_string(), "test@example.com".to_string()).is_err());
        assert!(service.create_user("test".to_string(), "invalid-email".to_string()).is_err());
        
        // Test operations on non-existent users
        assert!(service.get_user(999).is_none());
        assert!(service.deactivate_user(999).is_err());
    }
}

fn main() {
    println!("Integration tests verify complete workflows");
}
```

Integration tests verify that different parts of your system work  
correctly together. They test real workflows and user scenarios,  
focusing on the public API rather than implementation details.  

## Benchmarking performance

Measuring and tracking the performance of your code using Rust's  
built-in benchmark framework for performance regression detection.  

```rust
// Benchmarks require the 'test' feature and nightly Rust
#![feature(test)]
extern crate test;

use test::Bencher;

fn fibonacci_recursive(n: u32) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci_recursive(n - 1) + fibonacci_recursive(n - 2),
    }
}

fn fibonacci_iterative(n: u32) -> u64 {
    if n == 0 {
        return 0;
    }
    
    let mut prev = 0;
    let mut curr = 1;
    
    for _ in 2..=n {
        let next = prev + curr;
        prev = curr;
        curr = next;
    }
    
    curr
}

fn fibonacci_memoized(n: u32) -> u64 {
    fn fib_memo(n: u32, memo: &mut std::collections::HashMap<u32, u64>) -> u64 {
        if let Some(&result) = memo.get(&n) {
            return result;
        }
        
        let result = match n {
            0 => 0,
            1 => 1,
            _ => fib_memo(n - 1, memo) + fib_memo(n - 2, memo),
        };
        
        memo.insert(n, result);
        result
    }
    
    let mut memo = std::collections::HashMap::new();
    fib_memo(n, &mut memo)
}

#[bench]
fn bench_fibonacci_recursive_small(b: &mut Bencher) {
    b.iter(|| {
        for i in 0..20 {
            test::black_box(fibonacci_recursive(i));
        }
    });
}

#[bench]
fn bench_fibonacci_iterative_small(b: &mut Bencher) {
    b.iter(|| {
        for i in 0..20 {
            test::black_box(fibonacci_iterative(i));
        }
    });
}

#[bench]
fn bench_fibonacci_memoized_small(b: &mut Bencher) {
    b.iter(|| {
        for i in 0..20 {
            test::black_box(fibonacci_memoized(i));
        }
    });
}

#[bench]
fn bench_fibonacci_iterative_large(b: &mut Bencher) {
    b.iter(|| {
        test::black_box(fibonacci_iterative(50));
    });
}

#[bench]
fn bench_fibonacci_memoized_large(b: &mut Bencher) {
    b.iter(|| {
        test::black_box(fibonacci_memoized(50));
    });
}

// Comparative test to verify all implementations produce same results
#[test]
fn test_fibonacci_implementations_equivalent() {
    for i in 0..20 {
        let recursive = fibonacci_recursive(i);
        let iterative = fibonacci_iterative(i);
        let memoized = fibonacci_memoized(i);
        
        assert_eq!(recursive, iterative, "Recursive and iterative differ at n={}", i);
        assert_eq!(iterative, memoized, "Iterative and memoized differ at n={}", i);
    }
}

fn main() {
    // Regular code can still be run
    println!("Fibonacci(10) = {}", fibonacci_iterative(10));
    println!("Run benchmarks with: cargo +nightly bench");
}
```

Benchmarks help identify performance bottlenecks and track performance  
over time. Use `test::black_box()` to prevent compiler optimizations  
that might skew results. Run with `cargo +nightly bench`.  

## Testing with custom test harnesses

Creating custom test frameworks and runners for specialized testing  
needs beyond the standard Rust test framework.  

```rust
// Custom test result type
#[derive(Debug)]
struct TestResult {
    name: String,
    passed: bool,
    message: Option<String>,
    duration: std::time::Duration,
}

impl TestResult {
    fn success(name: &str, duration: std::time::Duration) -> Self {
        Self {
            name: name.to_string(),
            passed: true,
            message: None,
            duration,
        }
    }
    
    fn failure(name: &str, message: &str, duration: std::time::Duration) -> Self {
        Self {
            name: name.to_string(),
            passed: false,
            message: Some(message.to_string()),
            duration,
        }
    }
}

// Custom test runner
struct TestRunner {
    tests: Vec<Box<dyn Fn() -> TestResult>>,
}

impl TestRunner {
    fn new() -> Self {
        Self { tests: Vec::new() }
    }
    
    fn add_test<F>(&mut self, test: F)
    where
        F: Fn() -> TestResult + 'static,
    {
        self.tests.push(Box::new(test));
    }
    
    fn run_all(&self) -> Vec<TestResult> {
        self.tests.iter().map(|test| test()).collect()
    }
    
    fn print_results(&self, results: &[TestResult]) {
        let mut passed = 0;
        let mut failed = 0;
        let total_duration: std::time::Duration = results.iter()
            .map(|r| r.duration)
            .sum();
        
        println!("=== Test Results ===");
        for result in results {
            if result.passed {
                println!("✓ {} ({:?})", result.name, result.duration);
                passed += 1;
            } else {
                println!("✗ {} ({:?}): {}", 
                        result.name, 
                        result.duration,
                        result.message.as_deref().unwrap_or("No message"));
                failed += 1;
            }
        }
        
        println!("\n=== Summary ===");
        println!("Total: {}, Passed: {}, Failed: {}", 
                results.len(), passed, failed);
        println!("Total duration: {:?}", total_duration);
        println!("Average duration: {:?}", 
                total_duration / results.len() as u32);
    }
}

// Example test functions
fn test_math_operations() -> TestResult {
    let start = std::time::Instant::now();
    
    let result = 2 + 2;
    if result == 4 {
        TestResult::success("test_math_operations", start.elapsed())
    } else {
        TestResult::failure(
            "test_math_operations", 
            &format!("Expected 4, got {}", result),
            start.elapsed()
        )
    }
}

fn test_string_operations() -> TestResult {
    let start = std::time::Instant::now();
    
    let s = "hello".to_uppercase();
    if s == "HELLO" {
        TestResult::success("test_string_operations", start.elapsed())
    } else {
        TestResult::failure(
            "test_string_operations",
            &format!("Expected 'HELLO', got '{}'", s),
            start.elapsed()
        )
    }
}

fn test_intentional_failure() -> TestResult {
    let start = std::time::Instant::now();
    TestResult::failure(
        "test_intentional_failure",
        "This test always fails for demonstration",
        start.elapsed()
    )
}

// Macro for easier test registration
macro_rules! add_tests {
    ($runner:expr, $($test:expr),+ $(,)?) => {
        $(
            $runner.add_test($test);
        )+
    };
}

fn main() {
    let mut runner = TestRunner::new();
    
    // Add tests to the runner
    add_tests!(runner, 
        test_math_operations,
        test_string_operations, 
        test_intentional_failure
    );
    
    // Run all tests and print results
    let results = runner.run_all();
    runner.print_results(&results);
    
    // Exit with proper code
    let has_failures = results.iter().any(|r| !r.passed);
    std::process::exit(if has_failures { 1 } else { 0 });
}
```

Custom test harnesses allow you to create specialized testing workflows,  
add custom reporting, measure performance, or integrate with external  
testing tools that the standard framework doesn't support.  

## Testing documentation examples

Ensuring that code examples in documentation comments actually compile  
and work correctly using Rust's documentation testing feature.  

```rust
/// A simple calculator for basic arithmetic operations.
/// 
/// # Examples
/// 
/// ```
/// let mut calc = Calculator::new();
/// assert_eq!(calc.add(2, 3), 5);
/// assert_eq!(calc.subtract(10, 4), 6);
/// ```
/// 
/// You can also chain operations:
/// 
/// ```
/// let mut calc = Calculator::new();
/// let result = calc.add(5, 3).multiply(2).get_result();
/// assert_eq!(result, 16); // (5 + 3) * 2 = 16
/// ```
pub struct Calculator {
    result: f64,
}

impl Calculator {
    /// Creates a new calculator with initial value of 0.
    /// 
    /// # Examples
    /// 
    /// ```
    /// let calc = Calculator::new();
    /// assert_eq!(calc.get_result(), 0.0);
    /// ```
    pub fn new() -> Self {
        Self { result: 0.0 }
    }
    
    /// Adds a value to the current result.
    /// 
    /// # Examples
    /// 
    /// ```
    /// let mut calc = Calculator::new();
    /// calc.add(5, 3);
    /// assert_eq!(calc.get_result(), 8.0);
    /// ```
    pub fn add(&mut self, a: i32, b: i32) -> &mut Self {
        self.result = (a + b) as f64;
        self
    }
    
    /// Subtracts the second value from the first.
    /// 
    /// # Examples
    /// 
    /// ```
    /// let mut calc = Calculator::new();
    /// calc.subtract(10, 3);
    /// assert_eq!(calc.get_result(), 7.0);
    /// ```
    pub fn subtract(&mut self, a: i32, b: i32) -> &mut Self {
        self.result = (a - b) as f64;
        self
    }
    
    /// Multiplies the current result by a value.
    /// 
    /// # Examples
    /// 
    /// ```
    /// let mut calc = Calculator::new();
    /// calc.add(3, 2).multiply(4);
    /// assert_eq!(calc.get_result(), 20.0); // (3 + 2) * 4
    /// ```
    pub fn multiply(&mut self, factor: i32) -> &mut Self {
        self.result *= factor as f64;
        self
    }
    
    /// Gets the current result.
    /// 
    /// # Examples
    /// 
    /// ```
    /// let mut calc = Calculator::new();
    /// calc.add(1, 2);
    /// assert_eq!(calc.get_result(), 3.0);
    /// ```
    pub fn get_result(&self) -> f64 {
        self.result
    }
    
    /// Divides the current result by a value.
    /// 
    /// # Examples
    /// 
    /// ```
    /// let mut calc = Calculator::new();
    /// calc.add(10, 5).divide(3);
    /// assert_eq!(calc.get_result(), 5.0); // (10 + 5) / 3 = 5
    /// ```
    /// 
    /// Division by zero results in infinity:
    /// 
    /// ```
    /// let mut calc = Calculator::new();
    /// calc.add(5, 0).divide(0);
    /// assert!(calc.get_result().is_infinite());
    /// ```
    pub fn divide(&mut self, divisor: i32) -> &mut Self {
        if divisor == 0 {
            self.result = f64::INFINITY;
        } else {
            self.result /= divisor as f64;
        }
        self
    }
}

// Standard unit tests
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_chained_operations() {
        let mut calc = Calculator::new();
        calc.add(2, 3).multiply(4).subtract(20, 0);
        assert_eq!(calc.get_result(), 0.0); // ((2+3)*4) then (20-0) = 20, but subtract replaces
    }
    
    #[test]
    fn test_division_by_zero() {
        let mut calc = Calculator::new();
        calc.add(5, 5).divide(0);
        assert!(calc.get_result().is_infinite());
    }
}

fn main() {
    let mut calc = Calculator::new();
    calc.add(10, 5).multiply(2);
    println!("Result: {}", calc.get_result());
    
    println!("Run documentation tests with: cargo test --doc");
}
```

Documentation tests ensure your examples stay current and correct.  
They run when you execute `cargo test --doc` and help maintain  
accurate documentation that users can trust.  

## Testing best practices and patterns

Comprehensive example demonstrating testing best practices, naming  
conventions, organization patterns, and common testing guidelines.  

```rust
/// A shopping cart system demonstrating comprehensive testing practices
pub mod shopping_cart {
    use std::collections::HashMap;
    
    #[derive(Debug, Clone, PartialEq)]
    pub struct Product {
        pub id: u32,
        pub name: String,
        pub price: f64,
    }
    
    #[derive(Debug, Clone)]
    pub struct CartItem {
        pub product: Product,
        pub quantity: u32,
    }
    
    #[derive(Debug)]
    pub struct ShoppingCart {
        items: HashMap<u32, CartItem>,
        discount_rate: f64,
    }
    
    impl ShoppingCart {
        pub fn new() -> Self {
            Self {
                items: HashMap::new(),
                discount_rate: 0.0,
            }
        }
        
        pub fn add_item(&mut self, product: Product, quantity: u32) -> Result<(), String> {
            if quantity == 0 {
                return Err("Quantity must be greater than zero".to_string());
            }
            
            if product.price < 0.0 {
                return Err("Product price cannot be negative".to_string());
            }
            
            match self.items.get_mut(&product.id) {
                Some(existing_item) => {
                    existing_item.quantity += quantity;
                }
                None => {
                    self.items.insert(product.id, CartItem { product, quantity });
                }
            }
            
            Ok(())
        }
        
        pub fn remove_item(&mut self, product_id: u32) -> Result<(), String> {
            match self.items.remove(&product_id) {
                Some(_) => Ok(()),
                None => Err("Product not found in cart".to_string()),
            }
        }
        
        pub fn update_quantity(&mut self, product_id: u32, new_quantity: u32) -> Result<(), String> {
            if new_quantity == 0 {
                return self.remove_item(product_id);
            }
            
            match self.items.get_mut(&product_id) {
                Some(item) => {
                    item.quantity = new_quantity;
                    Ok(())
                }
                None => Err("Product not found in cart".to_string()),
            }
        }
        
        pub fn apply_discount(&mut self, rate: f64) -> Result<(), String> {
            if rate < 0.0 || rate > 1.0 {
                return Err("Discount rate must be between 0.0 and 1.0".to_string());
            }
            self.discount_rate = rate;
            Ok(())
        }
        
        pub fn get_total(&self) -> f64 {
            let subtotal: f64 = self.items
                .values()
                .map(|item| item.product.price * item.quantity as f64)
                .sum();
            
            subtotal * (1.0 - self.discount_rate)
        }
        
        pub fn get_item_count(&self) -> usize {
            self.items.len()
        }
        
        pub fn is_empty(&self) -> bool {
            self.items.is_empty()
        }
        
        pub fn get_items(&self) -> Vec<&CartItem> {
            self.items.values().collect()
        }
    }
}

#[cfg(test)]
mod tests {
    use super::shopping_cart::*;
    
    // Test fixtures and helpers
    fn create_test_product(id: u32, name: &str, price: f64) -> Product {
        Product {
            id,
            name: name.to_string(),
            price,
        }
    }
    
    fn create_populated_cart() -> ShoppingCart {
        let mut cart = ShoppingCart::new();
        cart.add_item(create_test_product(1, "Apple", 1.50), 3).unwrap();
        cart.add_item(create_test_product(2, "Banana", 0.75), 5).unwrap();
        cart
    }
    
    // Group 1: Constructor and basic state tests
    mod creation_tests {
        use super::*;
        
        #[test]
        fn new_cart_should_be_empty() {
            let cart = ShoppingCart::new();
            assert!(cart.is_empty());
            assert_eq!(cart.get_item_count(), 0);
            assert_eq!(cart.get_total(), 0.0);
        }
    }
    
    // Group 2: Adding items tests
    mod adding_items_tests {
        use super::*;
        
        #[test]
        fn add_single_item_should_succeed() {
            let mut cart = ShoppingCart::new();
            let product = create_test_product(1, "Test Product", 10.0);
            
            let result = cart.add_item(product, 2);
            
            assert!(result.is_ok());
            assert_eq!(cart.get_item_count(), 1);
            assert_eq!(cart.get_total(), 20.0);
        }
        
        #[test]
        fn add_duplicate_item_should_increase_quantity() {
            let mut cart = ShoppingCart::new();
            let product = create_test_product(1, "Test Product", 5.0);
            
            cart.add_item(product.clone(), 2).unwrap();
            cart.add_item(product, 3).unwrap();
            
            assert_eq!(cart.get_item_count(), 1);
            assert_eq!(cart.get_total(), 25.0); // 5 items * $5.00
        }
        
        #[test]
        fn add_zero_quantity_should_fail() {
            let mut cart = ShoppingCart::new();
            let product = create_test_product(1, "Test Product", 10.0);
            
            let result = cart.add_item(product, 0);
            
            assert!(result.is_err());
            assert!(result.unwrap_err().contains("greater than zero"));
        }
        
        #[test]
        fn add_negative_price_product_should_fail() {
            let mut cart = ShoppingCart::new();
            let product = create_test_product(1, "Invalid Product", -5.0);
            
            let result = cart.add_item(product, 1);
            
            assert!(result.is_err());
            assert!(result.unwrap_err().contains("cannot be negative"));
        }
    }
    
    // Group 3: Removing and updating tests
    mod modification_tests {
        use super::*;
        
        #[test]
        fn remove_existing_item_should_succeed() {
            let mut cart = create_populated_cart();
            let initial_count = cart.get_item_count();
            
            let result = cart.remove_item(1);
            
            assert!(result.is_ok());
            assert_eq!(cart.get_item_count(), initial_count - 1);
        }
        
        #[test]
        fn remove_nonexistent_item_should_fail() {
            let mut cart = ShoppingCart::new();
            
            let result = cart.remove_item(999);
            
            assert!(result.is_err());
            assert!(result.unwrap_err().contains("not found"));
        }
        
        #[test]
        fn update_quantity_should_change_total() {
            let mut cart = create_populated_cart();
            let initial_total = cart.get_total();
            
            cart.update_quantity(1, 1).unwrap(); // Reduce Apple quantity from 3 to 1
            
            let new_total = cart.get_total();
            assert!(new_total < initial_total);
            assert_eq!(new_total, 1.50 + 3.75); // 1 Apple + 5 Bananas
        }
        
        #[test]
        fn update_quantity_to_zero_should_remove_item() {
            let mut cart = create_populated_cart();
            let initial_count = cart.get_item_count();
            
            cart.update_quantity(1, 0).unwrap();
            
            assert_eq!(cart.get_item_count(), initial_count - 1);
        }
    }
    
    // Group 4: Discount tests
    mod discount_tests {
        use super::*;
        
        #[test]
        fn apply_valid_discount_should_reduce_total() {
            let mut cart = create_populated_cart();
            let original_total = cart.get_total();
            
            cart.apply_discount(0.1).unwrap(); // 10% discount
            
            let discounted_total = cart.get_total();
            assert_eq!(discounted_total, original_total * 0.9);
        }
        
        #[test]
        fn apply_invalid_discount_should_fail() {
            let mut cart = ShoppingCart::new();
            
            assert!(cart.apply_discount(-0.1).is_err());
            assert!(cart.apply_discount(1.5).is_err());
        }
        
        #[test]
        fn apply_maximum_discount_should_make_total_zero() {
            let mut cart = create_populated_cart();
            
            cart.apply_discount(1.0).unwrap(); // 100% discount
            
            assert_eq!(cart.get_total(), 0.0);
        }
    }
    
    // Group 5: Integration tests
    mod integration_tests {
        use super::*;
        
        #[test]
        fn complete_shopping_workflow() {
            let mut cart = ShoppingCart::new();
            
            // Add some products
            let apple = create_test_product(1, "Apple", 1.50);
            let banana = create_test_product(2, "Banana", 0.75);
            let orange = create_test_product(3, "Orange", 2.00);
            
            cart.add_item(apple, 3).unwrap();
            cart.add_item(banana, 5).unwrap();
            cart.add_item(orange, 2).unwrap();
            
            // Verify initial state
            assert_eq!(cart.get_item_count(), 3);
            let expected_total = (1.50 * 3.0) + (0.75 * 5.0) + (2.00 * 2.0);
            assert_eq!(cart.get_total(), expected_total);
            
            // Apply discount
            cart.apply_discount(0.15).unwrap(); // 15% discount
            assert_eq!(cart.get_total(), expected_total * 0.85);
            
            // Remove one item
            cart.remove_item(2).unwrap();
            assert_eq!(cart.get_item_count(), 2);
            
            // Update quantity
            cart.update_quantity(1, 1).unwrap();
            
            // Final verification
            let final_expected = ((1.50 * 1.0) + (2.00 * 2.0)) * 0.85;
            assert_eq!(cart.get_total(), final_expected);
        }
    }
}

fn main() {
    use shopping_cart::*;
    
    let mut cart = ShoppingCart::new();
    let product = Product {
        id: 1,
        name: "Test Product".to_string(),
        price: 10.0,
    };
    
    cart.add_item(product, 2).unwrap();
    println!("Cart total: ${:.2}", cart.get_total());
    
    println!("Run tests with: cargo test");
    println!("Run tests with output: cargo test -- --nocapture");
    println!("Run specific test module: cargo test modification_tests");
}
```

This example demonstrates testing best practices: organized test modules,  
descriptive test names, test fixtures, comprehensive coverage of success  
and failure cases, and clear separation between different types of tests.  

## Testing Option types

Testing functions that return Option types requires verifying both  
Some and None cases along with proper handling of wrapped values.  

```rust
fn find_first_even(numbers: &[i32]) -> Option<i32> {
    numbers.iter().find(|&&x| x % 2 == 0).copied()
}

fn safe_get(vec: &[String], index: usize) -> Option<&String> {
    vec.get(index)
}

fn parse_positive_number(s: &str) -> Option<u32> {
    s.parse::<u32>().ok().filter(|&n| n > 0)
}

#[test]
fn test_find_first_even_some() {
    let numbers = [1, 3, 4, 7, 8];
    let result = find_first_even(&numbers);
    assert_eq!(result, Some(4));
}

#[test]
fn test_find_first_even_none() {
    let numbers = [1, 3, 5, 7, 9];
    let result = find_first_even(&numbers);
    assert_eq!(result, None);
}

#[test]
fn test_safe_get_valid_index() {
    let items = vec!["hello".to_string(), "world".to_string()];
    let result = safe_get(&items, 0);
    assert_eq!(result, Some(&"hello".to_string()));
}

#[test]
fn test_safe_get_invalid_index() {
    let items = vec!["hello".to_string()];
    let result = safe_get(&items, 5);
    assert_eq!(result, None);
}

#[test]
fn test_parse_positive_number() {
    assert_eq!(parse_positive_number("42"), Some(42));
    assert_eq!(parse_positive_number("0"), None);
    assert_eq!(parse_positive_number("-5"), None);
    assert_eq!(parse_positive_number("invalid"), None);
}

fn main() {
    println!("Testing Option type handling");
}
```

When testing Option types, verify both success and failure paths. Use  
pattern matching or helper methods like `is_some()`, `is_none()` for  
more expressive assertions when the wrapped value isn't important.  

## Testing with floating point precision

Testing floating-point operations requires careful handling of precision  
issues and proper comparison techniques for reliable results.  

```rust
use std::f64::EPSILON;

fn calculate_circle_area(radius: f64) -> f64 {
    std::f64::consts::PI * radius * radius
}

fn solve_quadratic(a: f64, b: f64, c: f64) -> Option<(f64, f64)> {
    let discriminant = b * b - 4.0 * a * c;
    if discriminant < 0.0 {
        None
    } else {
        let sqrt_discriminant = discriminant.sqrt();
        let x1 = (-b + sqrt_discriminant) / (2.0 * a);
        let x2 = (-b - sqrt_discriminant) / (2.0 * a);
        Some((x1, x2))
    }
}

fn approx_equal(a: f64, b: f64, tolerance: f64) -> bool {
    (a - b).abs() < tolerance
}

#[test]
fn test_circle_area_precision() {
    let area = calculate_circle_area(1.0);
    let expected = std::f64::consts::PI;
    
    // Use epsilon for floating point comparison
    assert!(approx_equal(area, expected, EPSILON));
    
    // Alternative approach using assert! with manual tolerance
    assert!((area - expected).abs() < 1e-10);
}

#[test]
fn test_floating_point_arithmetic() {
    let result = 0.1 + 0.2;
    
    // This would fail due to floating point precision
    // assert_eq!(result, 0.3);
    
    // Correct way to test floating point equality
    assert!(approx_equal(result, 0.3, 1e-10));
}

#[test]
fn test_quadratic_solver() {
    // Test equation: x² - 5x + 6 = 0 (solutions: x = 2, x = 3)
    let solutions = solve_quadratic(1.0, -5.0, 6.0).unwrap();
    
    assert!(approx_equal(solutions.0, 3.0, EPSILON));
    assert!(approx_equal(solutions.1, 2.0, EPSILON));
}

#[test]
fn test_quadratic_no_real_solutions() {
    // Test equation: x² + 1 = 0 (no real solutions)
    let result = solve_quadratic(1.0, 0.0, 1.0);
    assert_eq!(result, None);
}

#[test]
fn test_special_float_values() {
    assert!(f64::NAN.is_nan());
    assert!(f64::INFINITY.is_infinite());
    assert!(f64::NEG_INFINITY.is_infinite());
    assert!(!f64::INFINITY.is_finite());
    assert!(42.0_f64.is_finite());
}

fn main() {
    println!("Testing floating point precision");
}
```

Always use tolerance-based comparison for floating-point tests. Consider  
edge cases like NaN, infinity, and very small/large numbers that might  
cause precision loss or overflow.  

## Testing concurrent code

Testing concurrent and parallel code requires special techniques to  
ensure thread safety and verify race condition handling.  

```rust
use std::sync::{Arc, Mutex, mpsc};
use std::thread;
use std::time::Duration;

struct Counter {
    value: Arc<Mutex<i32>>,
}

impl Counter {
    fn new() -> Self {
        Self {
            value: Arc::new(Mutex::new(0)),
        }
    }
    
    fn increment(&self) {
        let mut val = self.value.lock().unwrap();
        *val += 1;
    }
    
    fn get(&self) -> i32 {
        *self.value.lock().unwrap()
    }
}

impl Clone for Counter {
    fn clone(&self) -> Self {
        Self {
            value: Arc::clone(&self.value),
        }
    }
}

fn producer(tx: mpsc::Sender<i32>, start: i32, count: i32) {
    for i in start..start + count {
        tx.send(i).unwrap();
        thread::sleep(Duration::from_millis(1));
    }
}

#[test]
fn test_concurrent_counter() {
    let counter = Counter::new();
    let mut handles = vec![];
    
    // Spawn multiple threads that increment the counter
    for _ in 0..10 {
        let counter_clone = counter.clone();
        let handle = thread::spawn(move || {
            for _ in 0..100 {
                counter_clone.increment();
            }
        });
        handles.push(handle);
    }
    
    // Wait for all threads to complete
    for handle in handles {
        handle.join().unwrap();
    }
    
    // Verify final count
    assert_eq!(counter.get(), 1000);
}

#[test]
fn test_channel_communication() {
    let (tx, rx) = mpsc::channel();
    let mut handles = vec![];
    
    // Spawn producer threads
    for i in 0..3 {
        let tx_clone = tx.clone();
        let handle = thread::spawn(move || {
            producer(tx_clone, i * 10, 5);
        });
        handles.push(handle);
    }
    
    // Drop the original sender
    drop(tx);
    
    // Collect all received values
    let mut received = vec![];
    while let Ok(value) = rx.recv() {
        received.push(value);
    }
    
    // Wait for all producers to finish
    for handle in handles {
        handle.join().unwrap();
    }
    
    // Verify we received all expected values
    received.sort();
    let expected: Vec<i32> = (0..15).collect();
    assert_eq!(received, expected);
}

#[test]
fn test_timeout_behavior() {
    let (tx, rx) = mpsc::channel();
    
    // Spawn a thread that sends after a delay
    thread::spawn(move || {
        thread::sleep(Duration::from_millis(100));
        tx.send(42).unwrap();
    });
    
    // Test timeout behavior
    let result = rx.recv_timeout(Duration::from_millis(50));
    assert!(result.is_err()); // Should timeout
    
    let result = rx.recv_timeout(Duration::from_millis(100));
    assert_eq!(result.unwrap(), 42); // Should succeed
}

fn main() {
    println!("Testing concurrent code");
}
```

Test concurrent code by verifying thread safety, proper synchronization,  
and absence of race conditions. Use timeouts to test asynchronous  
behavior and ensure tests don't hang indefinitely.  

## Testing macro functionality

Testing macros requires special consideration for both the macro  
expansion and the generated code behavior.  

```rust
// Define macros to test
macro_rules! create_function {
    ($name:ident, $return_type:ty, $value:expr) => {
        fn $name() -> $return_type {
            $value
        }
    };
}

macro_rules! assert_range {
    ($value:expr, $min:expr, $max:expr) => {
        assert!($value >= $min && $value <= $max, 
               "Value {} is not in range [{}, {}]", $value, $min, $max);
    };
}

macro_rules! hash_map {
    ($($key:expr => $value:expr),* $(,)?) => {
        {
            let mut map = std::collections::HashMap::new();
            $(
                map.insert($key, $value);
            )*
            map
        }
    };
}

macro_rules! vec_of_strings {
    ($($string:expr),* $(,)?) => {
        vec![$($string.to_string()),*]
    };
}

// Test the generated functions
create_function!(get_answer, i32, 42);
create_function!(get_pi, f64, 3.14159);
create_function!(get_greeting, String, "Hello, World!".to_string());

#[test]
fn test_macro_generated_functions() {
    assert_eq!(get_answer(), 42);
    assert_eq!(get_pi(), 3.14159);
    assert_eq!(get_greeting(), "Hello, World!");
}

#[test]
fn test_assert_range_macro() {
    // These should pass
    assert_range!(5, 1, 10);
    assert_range!(0, 0, 0);
    assert_range!(-5, -10, 0);
}

#[test]
#[should_panic(expected = "Value 15 is not in range [1, 10]")]
fn test_assert_range_macro_failure() {
    assert_range!(15, 1, 10);
}

#[test]
fn test_hash_map_macro() {
    let map = hash_map! {
        "name" => "Alice",
        "age" => "30",
        "city" => "Boston",
    };
    
    assert_eq!(map.len(), 3);
    assert_eq!(map.get("name"), Some(&"Alice"));
    assert_eq!(map.get("age"), Some(&"30"));
    assert_eq!(map.get("city"), Some(&"Boston"));
}

#[test]
fn test_vec_of_strings_macro() {
    let strings = vec_of_strings!["hello", "world", "rust"];
    
    assert_eq!(strings.len(), 3);
    assert_eq!(strings[0], "hello");
    assert_eq!(strings[1], "world");
    assert_eq!(strings[2], "rust");
    
    // Verify they are actually String type, not &str
    let _: Vec<String> = strings;
}

#[test]
fn test_empty_macro_invocations() {
    let empty_map = hash_map! {};
    assert!(empty_map.is_empty());
    
    let empty_strings = vec_of_strings![];
    assert!(empty_strings.is_empty());
}

fn main() {
    println!("Testing macro functionality");
}
```

When testing macros, verify both compile-time generation and runtime  
behavior. Test edge cases like empty inputs and ensure error messages  
are helpful when macro usage is incorrect.  

## Testing with temporary files

Testing file operations safely using temporary files and directories  
to avoid affecting the filesystem and ensure test isolation.  

```rust
use std::fs::{self, File};
use std::io::{self, Write, Read};
use std::path::{Path, PathBuf};

struct FileManager {
    base_dir: PathBuf,
}

impl FileManager {
    fn new(base_dir: PathBuf) -> Self {
        Self { base_dir }
    }
    
    fn create_file(&self, name: &str, content: &str) -> io::Result<()> {
        let file_path = self.base_dir.join(name);
        let mut file = File::create(file_path)?;
        file.write_all(content.as_bytes())?;
        Ok(())
    }
    
    fn read_file(&self, name: &str) -> io::Result<String> {
        let file_path = self.base_dir.join(name);
        let mut file = File::open(file_path)?;
        let mut content = String::new();
        file.read_to_string(&mut content)?;
        Ok(content)
    }
    
    fn file_exists(&self, name: &str) -> bool {
        self.base_dir.join(name).exists()
    }
    
    fn delete_file(&self, name: &str) -> io::Result<()> {
        let file_path = self.base_dir.join(name);
        fs::remove_file(file_path)
    }
    
    fn list_files(&self) -> io::Result<Vec<String>> {
        let mut files = Vec::new();
        for entry in fs::read_dir(&self.base_dir)? {
            let entry = entry?;
            if entry.file_type()?.is_file() {
                if let Some(name) = entry.file_name().to_str() {
                    files.push(name.to_string());
                }
            }
        }
        files.sort();
        Ok(files)
    }
}

// Helper function to create temporary directory
fn create_temp_dir() -> io::Result<PathBuf> {
    let temp_dir = std::env::temp_dir().join(format!("rust_test_{}", 
        std::process::id()));
    fs::create_dir_all(&temp_dir)?;
    Ok(temp_dir)
}

// Helper function to clean up temporary directory
fn cleanup_temp_dir(dir: &Path) -> io::Result<()> {
    if dir.exists() {
        fs::remove_dir_all(dir)?;
    }
    Ok(())
}

#[test]
fn test_file_create_and_read() -> io::Result<()> {
    let temp_dir = create_temp_dir()?;
    let file_manager = FileManager::new(temp_dir.clone());
    
    // Test file creation and reading
    let content = "Hello, temporary file!";
    file_manager.create_file("test.txt", content)?;
    
    assert!(file_manager.file_exists("test.txt"));
    let read_content = file_manager.read_file("test.txt")?;
    assert_eq!(read_content, content);
    
    cleanup_temp_dir(&temp_dir)?;
    Ok(())
}

#[test]
fn test_file_operations() -> io::Result<()> {
    let temp_dir = create_temp_dir()?;
    let file_manager = FileManager::new(temp_dir.clone());
    
    // Create multiple files
    file_manager.create_file("file1.txt", "Content 1")?;
    file_manager.create_file("file2.txt", "Content 2")?;
    file_manager.create_file("file3.txt", "Content 3")?;
    
    // Test listing files
    let files = file_manager.list_files()?;
    assert_eq!(files, vec!["file1.txt", "file2.txt", "file3.txt"]);
    
    // Test file deletion
    file_manager.delete_file("file2.txt")?;
    assert!(!file_manager.file_exists("file2.txt"));
    
    let files_after_delete = file_manager.list_files()?;
    assert_eq!(files_after_delete, vec!["file1.txt", "file3.txt"]);
    
    cleanup_temp_dir(&temp_dir)?;
    Ok(())
}

#[test]
fn test_file_not_found() -> io::Result<()> {
    let temp_dir = create_temp_dir()?;
    let file_manager = FileManager::new(temp_dir.clone());
    
    // Test reading non-existent file
    let result = file_manager.read_file("nonexistent.txt");
    assert!(result.is_err());
    
    // Test deleting non-existent file
    let result = file_manager.delete_file("nonexistent.txt");
    assert!(result.is_err());
    
    cleanup_temp_dir(&temp_dir)?;
    Ok(())
}

#[test]
fn test_empty_file() -> io::Result<()> {
    let temp_dir = create_temp_dir()?;
    let file_manager = FileManager::new(temp_dir.clone());
    
    // Test creating and reading empty file
    file_manager.create_file("empty.txt", "")?;
    let content = file_manager.read_file("empty.txt")?;
    assert_eq!(content, "");
    
    cleanup_temp_dir(&temp_dir)?;
    Ok(())
}

fn main() {
    println!("Testing with temporary files");
}
```

Always use temporary directories for file-based tests to ensure isolation  
and prevent interference with the actual filesystem. Clean up temporary  
resources after each test to avoid accumulating test artifacts.  

## Testing trait implementations

Testing trait implementations requires verifying that types correctly  
implement the expected behavior defined by the trait contract.  

```rust
trait Shape {
    fn area(&self) -> f64;
    fn perimeter(&self) -> f64;
    fn name(&self) -> &'static str;
}

trait Drawable {
    fn draw(&self) -> String;
}

struct Circle {
    radius: f64,
}

impl Circle {
    fn new(radius: f64) -> Self {
        Self { radius }
    }
}

impl Shape for Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * self.radius * self.radius
    }
    
    fn perimeter(&self) -> f64 {
        2.0 * std::f64::consts::PI * self.radius
    }
    
    fn name(&self) -> &'static str {
        "Circle"
    }
}

impl Drawable for Circle {
    fn draw(&self) -> String {
        format!("Drawing a circle with radius {}", self.radius)
    }
}

struct Rectangle {
    width: f64,
    height: f64,
}

impl Rectangle {
    fn new(width: f64, height: f64) -> Self {
        Self { width, height }
    }
}

impl Shape for Rectangle {
    fn area(&self) -> f64 {
        self.width * self.height
    }
    
    fn perimeter(&self) -> f64 {
        2.0 * (self.width + self.height)
    }
    
    fn name(&self) -> &'static str {
        "Rectangle"
    }
}

impl Drawable for Rectangle {
    fn draw(&self) -> String {
        format!("Drawing a rectangle {}x{}", self.width, self.height)
    }
}

// Helper function to test any shape
fn test_shape_properties<T: Shape>(shape: &T, expected_area: f64, expected_perimeter: f64) {
    let area = shape.area();
    let perimeter = shape.perimeter();
    
    assert!((area - expected_area).abs() < 1e-10, 
           "Area mismatch for {}: expected {}, got {}", 
           shape.name(), expected_area, area);
    
    assert!((perimeter - expected_perimeter).abs() < 1e-10,
           "Perimeter mismatch for {}: expected {}, got {}",
           shape.name(), expected_perimeter, perimeter);
}

#[test]
fn test_circle_trait_implementation() {
    let circle = Circle::new(5.0);
    
    // Test Shape trait
    test_shape_properties(&circle, 
                         std::f64::consts::PI * 25.0, 
                         2.0 * std::f64::consts::PI * 5.0);
    assert_eq!(circle.name(), "Circle");
    
    // Test Drawable trait
    assert_eq!(circle.draw(), "Drawing a circle with radius 5");
}

#[test]
fn test_rectangle_trait_implementation() {
    let rectangle = Rectangle::new(4.0, 6.0);
    
    // Test Shape trait
    test_shape_properties(&rectangle, 24.0, 20.0);
    assert_eq!(rectangle.name(), "Rectangle");
    
    // Test Drawable trait
    assert_eq!(rectangle.draw(), "Drawing a rectangle 4x6");
}

#[test]
fn test_polymorphic_behavior() {
    let shapes: Vec<Box<dyn Shape>> = vec![
        Box::new(Circle::new(3.0)),
        Box::new(Rectangle::new(2.0, 5.0)),
    ];
    
    let total_area: f64 = shapes.iter().map(|s| s.area()).sum();
    let expected_total = (std::f64::consts::PI * 9.0) + 10.0;
    
    assert!((total_area - expected_total).abs() < 1e-10);
}

#[test]
fn test_trait_object_collections() {
    let drawables: Vec<Box<dyn Drawable>> = vec![
        Box::new(Circle::new(2.0)),
        Box::new(Rectangle::new(3.0, 4.0)),
    ];
    
    let drawings: Vec<String> = drawables.iter().map(|d| d.draw()).collect();
    
    assert_eq!(drawings[0], "Drawing a circle with radius 2");
    assert_eq!(drawings[1], "Drawing a rectangle 3x4");
}

#[test]
fn test_multiple_trait_bounds() {
    fn process_shape<T: Shape + Drawable>(shape: &T) -> (f64, String) {
        (shape.area(), shape.draw())
    }
    
    let circle = Circle::new(1.0);
    let (area, drawing) = process_shape(&circle);
    
    assert!((area - std::f64::consts::PI).abs() < 1e-10);
    assert_eq!(drawing, "Drawing a circle with radius 1");
}

fn main() {
    println!("Testing trait implementations");
}
```

Test trait implementations by verifying the contract behavior, testing  
polymorphic usage with trait objects, and ensuring multiple traits work  
correctly together when implemented on the same type.  

## Testing with environment variables

Testing code that depends on environment variables requires careful  
setup and cleanup to avoid test interference and ensure reproducibility.  

```rust
use std::env;
use std::collections::HashMap;

struct AppConfig {
    database_url: String,
    port: u16,
    debug_mode: bool,
    api_key: Option<String>,
}

impl AppConfig {
    fn from_env() -> Result<Self, String> {
        let database_url = env::var("DATABASE_URL")
            .map_err(|_| "DATABASE_URL environment variable required")?;
        
        let port = env::var("PORT")
            .unwrap_or_else(|_| "3000".to_string())
            .parse::<u16>()
            .map_err(|_| "PORT must be a valid number")?;
        
        let debug_mode = env::var("DEBUG")
            .unwrap_or_else(|_| "false".to_string())
            .parse::<bool>()
            .unwrap_or(false);
        
        let api_key = env::var("API_KEY").ok();
        
        Ok(Self {
            database_url,
            port,
            debug_mode,
            api_key,
        })
    }
}

// Test helper to manage environment variables
struct EnvManager {
    original_vars: HashMap<String, Option<String>>,
}

impl EnvManager {
    fn new() -> Self {
        Self {
            original_vars: HashMap::new(),
        }
    }
    
    fn set_var(&mut self, key: &str, value: &str) {
        // Store original value if not already stored
        if !self.original_vars.contains_key(key) {
            self.original_vars.insert(key.to_string(), env::var(key).ok());
        }
        env::set_var(key, value);
    }
    
    fn remove_var(&mut self, key: &str) {
        // Store original value if not already stored
        if !self.original_vars.contains_key(key) {
            self.original_vars.insert(key.to_string(), env::var(key).ok());
        }
        env::remove_var(key);
    }
}

impl Drop for EnvManager {
    fn drop(&mut self) {
        // Restore original environment variables
        for (key, original_value) in &self.original_vars {
            match original_value {
                Some(value) => env::set_var(key, value),
                None => env::remove_var(key),
            }
        }
    }
}

#[test]
fn test_config_with_all_variables() {
    let mut env_manager = EnvManager::new();
    
    env_manager.set_var("DATABASE_URL", "postgres://localhost/test");
    env_manager.set_var("PORT", "8080");
    env_manager.set_var("DEBUG", "true");
    env_manager.set_var("API_KEY", "secret123");
    
    let config = AppConfig::from_env().unwrap();
    
    assert_eq!(config.database_url, "postgres://localhost/test");
    assert_eq!(config.port, 8080);
    assert_eq!(config.debug_mode, true);
    assert_eq!(config.api_key, Some("secret123".to_string()));
    
    // EnvManager will restore original values when dropped
}

#[test]
fn test_config_with_defaults() {
    let mut env_manager = EnvManager::new();
    
    env_manager.set_var("DATABASE_URL", "postgres://localhost/test");
    env_manager.remove_var("PORT");
    env_manager.remove_var("DEBUG");
    env_manager.remove_var("API_KEY");
    
    let config = AppConfig::from_env().unwrap();
    
    assert_eq!(config.database_url, "postgres://localhost/test");
    assert_eq!(config.port, 3000); // Default value
    assert_eq!(config.debug_mode, false); // Default value
    assert_eq!(config.api_key, None); // Optional variable
}

#[test]
fn test_config_missing_required_variable() {
    let mut env_manager = EnvManager::new();
    
    env_manager.remove_var("DATABASE_URL");
    
    let result = AppConfig::from_env();
    assert!(result.is_err());
    assert!(result.unwrap_err().contains("DATABASE_URL"));
}

#[test]
fn test_config_invalid_port() {
    let mut env_manager = EnvManager::new();
    
    env_manager.set_var("DATABASE_URL", "postgres://localhost/test");
    env_manager.set_var("PORT", "invalid");
    
    let result = AppConfig::from_env();
    assert!(result.is_err());
    assert!(result.unwrap_err().contains("PORT must be a valid number"));
}

#[test]
fn test_config_boolean_parsing() {
    let mut env_manager = EnvManager::new();
    
    env_manager.set_var("DATABASE_URL", "postgres://localhost/test");
    
    // Test various boolean representations
    for debug_value in &["true", "false", "1", "0", "yes", "no"] {
        env_manager.set_var("DEBUG", debug_value);
        let config = AppConfig::from_env().unwrap();
        
        // Only "true" should parse as true
        let expected = debug_value == "true";
        assert_eq!(config.debug_mode, expected,
                  "DEBUG={} should parse as {}", debug_value, expected);
    }
}

fn main() {
    println!("Testing with environment variables");
}
```

Use environment variable managers to ensure test isolation and proper  
cleanup. Test both positive and negative cases, including missing variables,  
invalid values, and different valid representations.  

## Testing serialization and deserialization

Testing serialization and deserialization ensures data integrity across  
different formats and verifies round-trip conversion reliability.  

```rust
use std::collections::HashMap;

// Simple JSON-like serialization for demonstration
trait Serializable {
    fn serialize(&self) -> String;
    fn deserialize(data: &str) -> Result<Self, String> where Self: Sized;
}

#[derive(Debug, PartialEq, Clone)]
struct Person {
    name: String,
    age: u32,
    email: String,
    active: bool,
}

impl Person {
    fn new(name: String, age: u32, email: String, active: bool) -> Self {
        Self { name, age, email, active }
    }
}

impl Serializable for Person {
    fn serialize(&self) -> String {
        format!("name:{},age:{},email:{},active:{}", 
               self.name, self.age, self.email, self.active)
    }
    
    fn deserialize(data: &str) -> Result<Self, String> {
        let mut fields = HashMap::new();
        
        for part in data.split(',') {
            let kv: Vec<&str> = part.split(':').collect();
            if kv.len() != 2 {
                return Err(format!("Invalid format: {}", part));
            }
            fields.insert(kv[0], kv[1]);
        }
        
        let name = fields.get("name")
            .ok_or("Missing name field")?
            .to_string();
        
        let age = fields.get("age")
            .ok_or("Missing age field")?
            .parse::<u32>()
            .map_err(|_| "Invalid age format")?;
        
        let email = fields.get("email")
            .ok_or("Missing email field")?
            .to_string();
        
        let active = fields.get("active")
            .ok_or("Missing active field")?
            .parse::<bool>()
            .map_err(|_| "Invalid active format")?;
        
        Ok(Person::new(name, age, email, active))
    }
}

#[derive(Debug, PartialEq)]
struct Company {
    name: String,
    employees: Vec<Person>,
    founded: u32,
}

impl Serializable for Company {
    fn serialize(&self) -> String {
        let employees_str: Vec<String> = self.employees
            .iter()
            .map(|e| format!("[{}]", e.serialize()))
            .collect();
        
        format!("company_name:{},founded:{},employees:{}",
               self.name, self.founded, employees_str.join("|"))
    }
    
    fn deserialize(data: &str) -> Result<Self, String> {
        let parts: Vec<&str> = data.split(",employees:").collect();
        if parts.len() != 2 {
            return Err("Invalid company format".to_string());
        }
        
        let header_parts: Vec<&str> = parts[0].split(',').collect();
        let mut name = None;
        let mut founded = None;
        
        for part in header_parts {
            let kv: Vec<&str> = part.split(':').collect();
            if kv.len() == 2 {
                match kv[0] {
                    "company_name" => name = Some(kv[1].to_string()),
                    "founded" => founded = Some(kv[1].parse::<u32>()
                        .map_err(|_| "Invalid founded year")?),
                    _ => {}
                }
            }
        }
        
        let name = name.ok_or("Missing company name")?;
        let founded = founded.ok_or("Missing founded year")?;
        
        let mut employees = Vec::new();
        if !parts[1].is_empty() {
            for emp_str in parts[1].split('|') {
                if emp_str.starts_with('[') && emp_str.ends_with(']') {
                    let emp_data = &emp_str[1..emp_str.len()-1];
                    employees.push(Person::deserialize(emp_data)?);
                }
            }
        }
        
        Ok(Company { name, employees, founded })
    }
}

#[test]
fn test_person_serialization_round_trip() {
    let original = Person::new(
        "Alice Johnson".to_string(),
        30,
        "alice@example.com".to_string(),
        true
    );
    
    let serialized = original.serialize();
    let deserialized = Person::deserialize(&serialized).unwrap();
    
    assert_eq!(original, deserialized);
}

#[test]
fn test_person_serialization_format() {
    let person = Person::new(
        "Bob".to_string(),
        25,
        "bob@test.com".to_string(),
        false
    );
    
    let serialized = person.serialize();
    assert_eq!(serialized, "name:Bob,age:25,email:bob@test.com,active:false");
}

#[test]
fn test_person_deserialization_errors() {
    // Test invalid format
    let result = Person::deserialize("invalid");
    assert!(result.is_err());
    
    // Test missing fields
    let result = Person::deserialize("name:Alice,age:30");
    assert!(result.is_err());
    
    // Test invalid age
    let result = Person::deserialize("name:Alice,age:invalid,email:test,active:true");
    assert!(result.is_err());
}

#[test]
fn test_company_serialization_round_trip() {
    let employees = vec![
        Person::new("Alice".to_string(), 30, "alice@example.com".to_string(), true),
        Person::new("Bob".to_string(), 25, "bob@example.com".to_string(), false),
    ];
    
    let original = Company {
        name: "Tech Corp".to_string(),
        employees,
        founded: 2020,
    };
    
    let serialized = original.serialize();
    let deserialized = Company::deserialize(&serialized).unwrap();
    
    assert_eq!(original, deserialized);
}

#[test]
fn test_empty_company_serialization() {
    let company = Company {
        name: "Empty Corp".to_string(),
        employees: vec![],
        founded: 2023,
    };
    
    let serialized = company.serialize();
    let deserialized = Company::deserialize(&serialized).unwrap();
    
    assert_eq!(company, deserialized);
}

#[test]
fn test_serialization_with_special_characters() {
    // Test edge cases with special characters
    let person = Person::new(
        "O'Connor".to_string(),
        35,
        "test@co.uk".to_string(),
        true
    );
    
    let serialized = person.serialize();
    let deserialized = Person::deserialize(&serialized).unwrap();
    
    assert_eq!(person, deserialized);
}

fn main() {
    println!("Testing serialization and deserialization");
}
```

Test serialization by verifying round-trip conversion, testing edge cases  
with special characters, validating error handling for malformed data, and  
ensuring format consistency across different data structures.  