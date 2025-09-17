# Serialization & Deserialization

## Basic struct serialization

Convert Rust structs to and from JSON format using Serde.  
This demonstrates the fundamental serialization and deserialization process  
with automatic trait derivation.  

```rust
// Add to Cargo.toml: serde = { version = "1.0", features = ["derive"] }, serde_json = "1.0"
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct User {
    id: u32,
    name: String,
    email: String,
    active: bool,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let user = User {
        id: 1,
        name: "Alice Johnson".to_string(),
        email: "alice@example.com".to_string(),
        active: true,
    };
    
    // Serialize to JSON
    let json = serde_json::to_string_pretty(&user)?;
    println!("Serialized JSON:\n{}", json);
    
    // Deserialize from JSON
    let parsed_user: User = serde_json::from_str(&json)?;
    println!("Parsed user: {:?}", parsed_user);
    
    Ok(())
}
```

The `#[derive(Serialize, Deserialize)]` attributes automatically generate  
the serialization logic. The `to_string_pretty()` function creates formatted  
JSON output, while `from_str()` parses JSON back into the struct. Error  
handling with the `?` operator ensures robust parsing of potentially  
malformed input data.  

## Handling Option and Result types

Serialize optional fields and handle missing data gracefully.  
Option types in structs become nullable JSON fields, while None values  
are omitted from the serialized output.  

```rust
use serde::{Deserialize, Serialize};
use std::collections::HashMap;

#[derive(Serialize, Deserialize, Debug)]
struct Profile {
    username: String,
    email: String,
    #[serde(skip_serializing_if = "Option::is_none")]
    phone: Option<String>,
    #[serde(skip_serializing_if = "Option::is_none")]
    bio: Option<String>,
    settings: HashMap<String, String>,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let profile = Profile {
        username: "bob_dev".to_string(),
        email: "bob@dev.com".to_string(),
        phone: None,
        bio: Some("Rust developer".to_string()),
        settings: [
            ("theme".to_string(), "dark".to_string()),
            ("notifications".to_string(), "enabled".to_string()),
        ].iter().cloned().collect(),
    };
    
    let json = serde_json::to_string_pretty(&profile)?;
    println!("Profile JSON:\n{}", json);
    
    // Deserialize with missing optional fields
    let json_input = r#"{
        "username": "alice_coder",
        "email": "alice@coder.com",
        "bio": null,
        "settings": {}
    }"#;
    
    let parsed: Profile = serde_json::from_str(json_input)?;
    println!("Parsed profile: {:?}", parsed);
    
    Ok(())
}
```

The `skip_serializing_if` attribute omits fields when they are None,  
creating cleaner JSON output. HashMap fields serialize as JSON objects.  
When deserializing, missing optional fields default to None, and explicit  
null values are also handled correctly.  

## Working with Vec and HashMap collections

Serialize complex collections including nested vectors and hash maps.  
Collections automatically serialize to appropriate JSON arrays and objects  
with proper type preservation.  

```rust
use serde::{Deserialize, Serialize};
use std::collections::HashMap;

#[derive(Serialize, Deserialize, Debug)]
struct Project {
    name: String,
    version: String,
    dependencies: Vec<String>,
    metadata: HashMap<String, serde_json::Value>,
    contributors: Vec<Contributor>,
}

#[derive(Serialize, Deserialize, Debug)]
struct Contributor {
    name: String,
    role: String,
    commits: u32,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut metadata = HashMap::new();
    metadata.insert("license".to_string(), serde_json::Value::String("MIT".to_string()));
    metadata.insert("stars".to_string(), serde_json::Value::Number(serde_json::Number::from(42)));
    metadata.insert("featured".to_string(), serde_json::Value::Bool(true));
    
    let project = Project {
        name: "awesome-rust".to_string(),
        version: "1.2.3".to_string(),
        dependencies: vec![
            "serde".to_string(),
            "tokio".to_string(),
            "clap".to_string(),
        ],
        metadata,
        contributors: vec![
            Contributor {
                name: "Alice".to_string(),
                role: "maintainer".to_string(),
                commits: 150,
            },
            Contributor {
                name: "Bob".to_string(),
                role: "contributor".to_string(),
                commits: 75,
            },
        ],
    };
    
    let json = serde_json::to_string_pretty(&project)?;
    println!("Project JSON:\n{}", json);
    
    // Parse it back
    let parsed: Project = serde_json::from_str(&json)?;
    println!("Contributors count: {}", parsed.contributors.len());
    
    Ok(())
}
```

Vec serializes to JSON arrays maintaining element order and type information.  
HashMap becomes a JSON object with string keys. The serde_json::Value enum  
allows storing arbitrary JSON values in collections, providing flexibility  
for dynamic metadata storage.  

## Nested structs and complex data structures

Handle deeply nested structures with multiple levels of composition.  
Complex hierarchies serialize naturally while maintaining referential  
integrity and type safety throughout the process.  

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Organization {
    name: String,
    departments: Vec<Department>,
    headquarters: Address,
}

#[derive(Serialize, Deserialize, Debug)]
struct Department {
    name: String,
    manager: Employee,
    employees: Vec<Employee>,
    budget: f64,
}

#[derive(Serialize, Deserialize, Debug)]
struct Employee {
    id: u32,
    personal_info: PersonalInfo,
    position: String,
    salary: f64,
}

#[derive(Serialize, Deserialize, Debug)]
struct PersonalInfo {
    first_name: String,
    last_name: String,
    contact: ContactInfo,
}

#[derive(Serialize, Deserialize, Debug)]
struct ContactInfo {
    email: String,
    phone: String,
    address: Address,
}

#[derive(Serialize, Deserialize, Debug)]
struct Address {
    street: String,
    city: String,
    state: String,
    zip_code: String,
    country: String,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let organization = Organization {
        name: "TechCorp Inc.".to_string(),
        headquarters: Address {
            street: "123 Tech Street".to_string(),
            city: "San Francisco".to_string(),
            state: "CA".to_string(),
            zip_code: "94102".to_string(),
            country: "USA".to_string(),
        },
        departments: vec![
            Department {
                name: "Engineering".to_string(),
                budget: 2000000.0,
                manager: Employee {
                    id: 1,
                    position: "Engineering Manager".to_string(),
                    salary: 150000.0,
                    personal_info: PersonalInfo {
                        first_name: "Sarah".to_string(),
                        last_name: "Connor".to_string(),
                        contact: ContactInfo {
                            email: "sarah.connor@techcorp.com".to_string(),
                            phone: "+1-555-0101".to_string(),
                            address: Address {
                                street: "456 Elm Street".to_string(),
                                city: "Palo Alto".to_string(),
                                state: "CA".to_string(),
                                zip_code: "94301".to_string(),
                                country: "USA".to_string(),
                            },
                        },
                    },
                },
                employees: vec![
                    Employee {
                        id: 2,
                        position: "Senior Developer".to_string(),
                        salary: 120000.0,
                        personal_info: PersonalInfo {
                            first_name: "John".to_string(),
                            last_name: "Doe".to_string(),
                            contact: ContactInfo {
                                email: "john.doe@techcorp.com".to_string(),
                                phone: "+1-555-0102".to_string(),
                                address: Address {
                                    street: "789 Oak Avenue".to_string(),
                                    city: "Mountain View".to_string(),
                                    state: "CA".to_string(),
                                    zip_code: "94041".to_string(),
                                    country: "USA".to_string(),
                                },
                            },
                        },
                    },
                ],
            },
        ],
    };
    
    let json = serde_json::to_string_pretty(&organization)?;
    println!("Organization structure:\n{}", json);
    
    // Parse and verify structure
    let parsed: Organization = serde_json::from_str(&json)?;
    println!("Organization: {}", parsed.name);
    println!("Departments: {}", parsed.departments.len());
    
    for dept in &parsed.departments {
        println!("  - {} (Manager: {} {})", 
                 dept.name,
                 dept.manager.personal_info.first_name,
                 dept.manager.personal_info.last_name);
    }
    
    Ok(())
}
```

Nested structures serialize recursively with each level maintaining its  
own field structure. The deep hierarchy demonstrates how Serde handles  
complex object graphs while preserving all relationships and data integrity.  
Address reuse shows how the same struct can be used in multiple contexts.  

## Custom field names and renaming

Control how struct fields appear in serialized output using custom names.  
Field renaming allows mapping between Rust naming conventions and external  
API requirements or legacy data formats.  

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct ApiResponse {
    #[serde(rename = "status_code")]
    status: u16,
    #[serde(rename = "response_data")]
    data: ResponseData,
    #[serde(rename = "error_message")]
    error: Option<String>,
    #[serde(rename = "timestamp_utc")]
    timestamp: String,
}

#[derive(Serialize, Deserialize, Debug)]
struct ResponseData {
    #[serde(rename = "user_id")]
    id: u32,
    #[serde(rename = "full_name")]
    name: String,
    #[serde(rename = "email_address")]
    email: String,
    #[serde(rename = "account_type")]
    account_type: String,
    #[serde(rename = "creation_date")]
    created_at: String,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let response = ApiResponse {
        status: 200,
        data: ResponseData {
            id: 12345,
            name: "Emma Watson".to_string(),
            email: "emma.watson@example.com".to_string(),
            account_type: "premium".to_string(),
            created_at: "2023-01-15T10:30:00Z".to_string(),
        },
        error: None,
        timestamp: "2024-03-15T14:22:33Z".to_string(),
    };
    
    let json = serde_json::to_string_pretty(&response)?;
    println!("API Response JSON:\n{}", json);
    
    // Deserialize from external API format
    let api_json = r#"{
        "status_code": 404,
        "response_data": null,
        "error_message": "User not found",
        "timestamp_utc": "2024-03-15T14:25:00Z"
    }"#;
    
    let error_response: ApiResponse = serde_json::from_str(api_json)?;
    println!("Error response: {:?}", error_response);
    
    Ok(())
}
```

The `rename` attribute maps Rust field names to different JSON property names.  
This enables compatibility with external APIs that use different naming  
conventions like snake_case, camelCase, or legacy field names. The mapping  
works bidirectionally for both serialization and deserialization.  

## Skipping fields and default values

Control which fields are included in serialization and provide defaults.  
Field skipping reduces payload size while default values ensure robust  
deserialization when fields are missing from input data.  

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Config {
    name: String,
    version: String,
    #[serde(skip_serializing_if = "is_default_port")]
    #[serde(default = "default_port")]
    port: u16,
    #[serde(skip_serializing_if = "Vec::is_empty")]
    #[serde(default)]
    features: Vec<String>,
    #[serde(skip_serializing)]
    #[serde(default = "default_secret")]
    internal_secret: String,
    #[serde(skip_deserializing)]
    computed_hash: Option<String>,
    #[serde(default = "default_timeout")]
    timeout_seconds: u32,
}

fn default_port() -> u16 {
    8080
}

fn is_default_port(port: &u16) -> bool {
    *port == 8080
}

fn default_secret() -> String {
    "generated-secret-key".to_string()
}

fn default_timeout() -> u32 {
    30
}

impl Default for Config {
    fn default() -> Self {
        Self {
            name: "DefaultApp".to_string(),
            version: "1.0.0".to_string(),
            port: default_port(),
            features: Vec::new(),
            internal_secret: default_secret(),
            computed_hash: None,
            timeout_seconds: default_timeout(),
        }
    }
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let config = Config {
        name: "MyApp".to_string(),
        version: "2.1.0".to_string(),
        port: 8080, // Will be skipped due to default value
        features: vec!["auth".to_string(), "logging".to_string()],
        internal_secret: "super-secret".to_string(), // Won't be serialized
        computed_hash: Some("abc123def456".to_string()), // Won't be deserialized
        timeout_seconds: 60,
    };
    
    let json = serde_json::to_string_pretty(&config)?;
    println!("Config JSON (note missing port and secret):\n{}", json);
    
    // Deserialize minimal config
    let minimal_json = r#"{
        "name": "MinimalApp",
        "version": "1.0.0"
    }"#;
    
    let minimal_config: Config = serde_json::from_str(minimal_json)?;
    println!("Minimal config with defaults: {:?}", minimal_config);
    
    // Deserialize with some fields
    let partial_json = r#"{
        "name": "PartialApp",
        "version": "1.5.0",
        "port": 9000,
        "timeout_seconds": 120
    }"#;
    
    let partial_config: Config = serde_json::from_str(partial_json)?;
    println!("Partial config: {:?}", partial_config);
    
    Ok(())
}
```

The `skip_serializing_if` attribute conditionally omits fields based on  
custom predicates. The `default` attribute provides fallback values for  
missing fields during deserialization. `skip_serializing` and  
`skip_deserializing` provide unidirectional control over field processing.  

## Enum serialization with different representations

Serialize enums using various representations including external tagging,  
internal tagging, and adjacently tagged formats to match different API  
requirements and data exchange formats.  

```rust
use serde::{Deserialize, Serialize};

// External tagging (default)
#[derive(Serialize, Deserialize, Debug)]
enum ExternalEvent {
    UserLogin { user_id: u32, timestamp: String },
    UserLogout { user_id: u32 },
    Purchase { item_id: u32, amount: f64, currency: String },
    Error { code: u16, message: String },
}

// Internal tagging
#[derive(Serialize, Deserialize, Debug)]
#[serde(tag = "type")]
enum InternalEvent {
    UserLogin { user_id: u32, timestamp: String },
    UserLogout { user_id: u32 },
    Purchase { item_id: u32, amount: f64, currency: String },
    Error { code: u16, message: String },
}

// Adjacent tagging
#[derive(Serialize, Deserialize, Debug)]
#[serde(tag = "event_type", content = "data")]
enum AdjacentEvent {
    UserLogin { user_id: u32, timestamp: String },
    UserLogout { user_id: u32 },
    Purchase { item_id: u32, amount: f64, currency: String },
    Error { code: u16, message: String },
}

// Untagged (structure-based discrimination)
#[derive(Serialize, Deserialize, Debug)]
#[serde(untagged)]
enum UntaggedEvent {
    Login { user_id: u32, login_timestamp: String },
    Purchase { item_id: u32, amount: f64 },
    Error { error_code: u16 },
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let purchase = ExternalEvent::Purchase {
        item_id: 12345,
        amount: 99.99,
        currency: "USD".to_string(),
    };
    
    println!("=== External Tagging ===");
    let external_json = serde_json::to_string_pretty(&purchase)?;
    println!("External: {}", external_json);
    
    let internal_purchase = InternalEvent::Purchase {
        item_id: 12345,
        amount: 99.99,
        currency: "USD".to_string(),
    };
    
    println!("\n=== Internal Tagging ===");
    let internal_json = serde_json::to_string_pretty(&internal_purchase)?;
    println!("Internal: {}", internal_json);
    
    let adjacent_purchase = AdjacentEvent::Purchase {
        item_id: 12345,
        amount: 99.99,
        currency: "USD".to_string(),
    };
    
    println!("\n=== Adjacent Tagging ===");
    let adjacent_json = serde_json::to_string_pretty(&adjacent_purchase)?;
    println!("Adjacent: {}", adjacent_json);
    
    let untagged_login = UntaggedEvent::Login {
        user_id: 789,
        login_timestamp: "2024-03-15T14:30:00Z".to_string(),
    };
    
    println!("\n=== Untagged ===");
    let untagged_json = serde_json::to_string_pretty(&untagged_login)?;
    println!("Untagged: {}", untagged_json);
    
    // Parse different formats
    let external_parsed: ExternalEvent = serde_json::from_str(&external_json)?;
    let internal_parsed: InternalEvent = serde_json::from_str(&internal_json)?;
    let adjacent_parsed: AdjacentEvent = serde_json::from_str(&adjacent_json)?;
    let untagged_parsed: UntaggedEvent = serde_json::from_str(&untagged_json)?;
    
    println!("\nParsed results:");
    println!("External: {:?}", external_parsed);
    println!("Internal: {:?}", internal_parsed);
    println!("Adjacent: {:?}", adjacent_parsed);
    println!("Untagged: {:?}", untagged_parsed);
    
    Ok(())
}
```

External tagging wraps variant data in an object with the variant name as  
key. Internal tagging adds a discriminator field within the variant data.  
Adjacent tagging separates the variant name and data into different fields.  
Untagged enums rely on structure differences for variant identification.  

## DateTime and timestamp handling

Serialize and deserialize date and time values using chrono integration.  
DateTime handling supports multiple formats, timezones, and custom  
serialization patterns for different API requirements.  

```rust
// Add to Cargo.toml: chrono = { version = "0.4", features = ["serde"] }
use serde::{Deserialize, Serialize};
use chrono::{DateTime, Utc, NaiveDateTime, NaiveDate};

#[derive(Serialize, Deserialize, Debug)]
struct Event {
    id: u32,
    name: String,
    #[serde(with = "chrono::serde::ts_seconds")]
    created_at: DateTime<Utc>,
    #[serde(with = "chrono::serde::ts_milliseconds")]
    updated_at: DateTime<Utc>,
    start_date: NaiveDate,
    #[serde(with = "date_format")]
    end_time: NaiveDateTime,
    timezone: String,
}

mod date_format {
    use chrono::NaiveDateTime;
    use serde::{self, Deserialize, Deserializer, Serializer};

    const FORMAT: &str = "%Y-%m-%d %H:%M:%S";

    pub fn serialize<S>(date: &NaiveDateTime, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        let s = format!("{}", date.format(FORMAT));
        serializer.serialize_str(&s)
    }

    pub fn deserialize<'de, D>(deserializer: D) -> Result<NaiveDateTime, D::Error>
    where
        D: Deserializer<'de>,
    {
        let s = String::deserialize(deserializer)?;
        NaiveDateTime::parse_from_str(&s, FORMAT).map_err(serde::de::Error::custom)
    }
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    use chrono::TimeZone;
    
    let event = Event {
        id: 1,
        name: "Rust Conference 2024".to_string(),
        created_at: Utc::now(),
        updated_at: Utc.timestamp_millis_opt(1710515400000).unwrap(),
        start_date: NaiveDate::from_ymd_opt(2024, 6, 15).unwrap(),
        end_time: NaiveDateTime::parse_from_str(
            "2024-06-17 18:00:00", 
            "%Y-%m-%d %H:%M:%S"
        )?,
        timezone: "UTC".to_string(),
    };
    
    let json = serde_json::to_string_pretty(&event)?;
    println!("Event with timestamps:\n{}", json);
    
    // Parse event with different timestamp formats
    let json_input = r#"{
        "id": 2,
        "name": "Workshop",
        "created_at": 1710515400,
        "updated_at": 1710515400000,
        "start_date": "2024-07-01",
        "end_time": "2024-07-01 14:30:00",
        "timezone": "UTC"
    }"#;
    
    let parsed: Event = serde_json::from_str(json_input)?;
    println!("Parsed event: {:?}", parsed);
    
    Ok(())
}
```

Chrono provides serialization modules for common timestamp formats including  
Unix timestamps in seconds and milliseconds. Custom date format modules  
enable specific string representations. NaiveDate and NaiveDateTime handle  
timezone-agnostic dates and times with automatic ISO format support.  

## Configuration file serialization with TOML

Serialize configuration data to and from TOML format for human-readable  
config files. TOML provides excellent support for nested structures,  
arrays, and comments while maintaining readability.  

```rust
// Add to Cargo.toml: toml = "0.8", serde = { version = "1.0", features = ["derive"] }
use serde::{Deserialize, Serialize};
use std::collections::HashMap;

#[derive(Serialize, Deserialize, Debug)]
struct AppConfig {
    app: AppSettings,
    database: DatabaseConfig,
    server: ServerConfig,
    logging: LoggingConfig,
    features: HashMap<String, bool>,
    environments: Vec<Environment>,
}

#[derive(Serialize, Deserialize, Debug)]
struct AppSettings {
    name: String,
    version: String,
    debug: bool,
    max_workers: u32,
}

#[derive(Serialize, Deserialize, Debug)]
struct DatabaseConfig {
    host: String,
    port: u16,
    database: String,
    username: String,
    #[serde(skip_serializing_if = "Option::is_none")]
    password: Option<String>,
    pool_size: u32,
    timeout_seconds: u64,
}

#[derive(Serialize, Deserialize, Debug)]
struct ServerConfig {
    bind_address: String,
    port: u16,
    tls_enabled: bool,
    #[serde(skip_serializing_if = "Option::is_none")]
    cert_path: Option<String>,
    #[serde(skip_serializing_if = "Option::is_none")]
    key_path: Option<String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct LoggingConfig {
    level: String,
    output: String,
    format: String,
    rotate_logs: bool,
    max_file_size_mb: u32,
}

#[derive(Serialize, Deserialize, Debug)]
struct Environment {
    name: String,
    url: String,
    timeout: u32,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let config = AppConfig {
        app: AppSettings {
            name: "MyWebApp".to_string(),
            version: "2.1.0".to_string(),
            debug: false,
            max_workers: 4,
        },
        database: DatabaseConfig {
            host: "localhost".to_string(),
            port: 5432,
            database: "myapp_prod".to_string(),
            username: "app_user".to_string(),
            password: None, // Will be skipped in output
            pool_size: 10,
            timeout_seconds: 30,
        },
        server: ServerConfig {
            bind_address: "0.0.0.0".to_string(),
            port: 8080,
            tls_enabled: true,
            cert_path: Some("/etc/ssl/cert.pem".to_string()),
            key_path: Some("/etc/ssl/key.pem".to_string()),
        },
        logging: LoggingConfig {
            level: "info".to_string(),
            output: "file".to_string(),
            format: "json".to_string(),
            rotate_logs: true,
            max_file_size_mb: 100,
        },
        features: [
            ("authentication".to_string(), true),
            ("rate_limiting".to_string(), true),
            ("metrics".to_string(), false),
            ("debug_mode".to_string(), false),
        ].iter().cloned().collect(),
        environments: vec![
            Environment {
                name: "production".to_string(),
                url: "https://api.example.com".to_string(),
                timeout: 30,
            },
            Environment {
                name: "staging".to_string(),
                url: "https://staging-api.example.com".to_string(),
                timeout: 60,
            },
        ],
    };
    
    // Serialize to TOML
    let toml_string = toml::to_string_pretty(&config)?;
    println!("Configuration TOML:\n{}", toml_string);
    
    // Deserialize from TOML
    let toml_input = r#"
[app]
name = "TestApp"
version = "1.0.0"
debug = true
max_workers = 2

[database]
host = "db.test.local"
port = 5432
database = "testdb"
username = "test_user"
pool_size = 5
timeout_seconds = 15

[server]
bind_address = "127.0.0.1"
port = 3000
tls_enabled = false

[logging]
level = "debug"
output = "console"
format = "plain"
rotate_logs = false
max_file_size_mb = 50

[features]
authentication = false
rate_limiting = true
metrics = true

[[environments]]
name = "local"
url = "http://localhost:8000"
timeout = 10
"#;
    
    let parsed_config: AppConfig = toml::from_str(toml_input)?;
    println!("Parsed config app name: {}", parsed_config.app.name);
    println!("Features enabled: {}", parsed_config.features.len());
    println!("Environments: {}", parsed_config.environments.len());
    
    Ok(())
}
```

TOML serialization creates human-readable configuration files with clear  
section headers and nested tables. Arrays of tables use double brackets  
for definition. Missing optional fields are handled gracefully, and the  
format supports comments for documentation purposes.  

## Tables and nested structures in TOML

Handle complex nested TOML structures including tables of tables,  
inline tables, and mixed data types. TOML excels at representing  
hierarchical configuration data with clear section organization.  

```rust
use serde::{Deserialize, Serialize};
use std::collections::HashMap;

#[derive(Serialize, Deserialize, Debug)]
struct NetworkConfig {
    global: GlobalSettings,
    servers: HashMap<String, ServerDef>,
    load_balancer: LoadBalancerConfig,
    monitoring: MonitoringConfig,
    security: SecurityConfig,
}

#[derive(Serialize, Deserialize, Debug)]
struct GlobalSettings {
    timeout: u32,
    retries: u32,
    keepalive: bool,
    user_agent: String,
}

#[derive(Serialize, Deserialize, Debug)]
struct ServerDef {
    host: String,
    port: u16,
    weight: u32,
    health_check: HealthCheck,
    metadata: HashMap<String, String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct HealthCheck {
    endpoint: String,
    interval_seconds: u32,
    timeout_seconds: u32,
    healthy_threshold: u32,
    unhealthy_threshold: u32,
}

#[derive(Serialize, Deserialize, Debug)]
struct LoadBalancerConfig {
    algorithm: String,
    session_affinity: bool,
    health_check_interval: u32,
    backend_pools: Vec<BackendPool>,
}

#[derive(Serialize, Deserialize, Debug)]
struct BackendPool {
    name: String,
    servers: Vec<String>,
    backup_servers: Vec<String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct MonitoringConfig {
    enabled: bool,
    metrics: MetricsConfig,
    alerts: Vec<AlertRule>,
}

#[derive(Serialize, Deserialize, Debug)]
struct MetricsConfig {
    endpoint: String,
    interval: u32,
    tags: HashMap<String, String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct AlertRule {
    name: String,
    condition: String,
    threshold: f64,
    severity: String,
}

#[derive(Serialize, Deserialize, Debug)]
struct SecurityConfig {
    tls: TlsConfig,
    authentication: AuthConfig,
    rate_limits: Vec<RateLimit>,
}

#[derive(Serialize, Deserialize, Debug)]
struct TlsConfig {
    enabled: bool,
    min_version: String,
    cipher_suites: Vec<String>,
    cert_path: String,
    key_path: String,
}

#[derive(Serialize, Deserialize, Debug)]
struct AuthConfig {
    method: String,
    token_lifetime: u32,
    refresh_enabled: bool,
}

#[derive(Serialize, Deserialize, Debug)]
struct RateLimit {
    path: String,
    requests_per_minute: u32,
    burst: u32,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut servers = HashMap::new();
    servers.insert("web1".to_string(), ServerDef {
        host: "web1.example.com".to_string(),
        port: 80,
        weight: 100,
        health_check: HealthCheck {
            endpoint: "/health".to_string(),
            interval_seconds: 30,
            timeout_seconds: 5,
            healthy_threshold: 2,
            unhealthy_threshold: 3,
        },
        metadata: [
            ("region".to_string(), "us-west".to_string()),
            ("environment".to_string(), "production".to_string()),
        ].iter().cloned().collect(),
    });
    
    servers.insert("web2".to_string(), ServerDef {
        host: "web2.example.com".to_string(),
        port: 80,
        weight: 100,
        health_check: HealthCheck {
            endpoint: "/health".to_string(),
            interval_seconds: 30,
            timeout_seconds: 5,
            healthy_threshold: 2,
            unhealthy_threshold: 3,
        },
        metadata: [
            ("region".to_string(), "us-east".to_string()),
            ("environment".to_string(), "production".to_string()),
        ].iter().cloned().collect(),
    });
    
    let config = NetworkConfig {
        global: GlobalSettings {
            timeout: 30,
            retries: 3,
            keepalive: true,
            user_agent: "LoadBalancer/1.0".to_string(),
        },
        servers,
        load_balancer: LoadBalancerConfig {
            algorithm: "round_robin".to_string(),
            session_affinity: false,
            health_check_interval: 10,
            backend_pools: vec![
                BackendPool {
                    name: "primary".to_string(),
                    servers: vec!["web1".to_string(), "web2".to_string()],
                    backup_servers: vec!["backup1".to_string()],
                },
            ],
        },
        monitoring: MonitoringConfig {
            enabled: true,
            metrics: MetricsConfig {
                endpoint: "http://metrics.example.com:8086".to_string(),
                interval: 60,
                tags: [
                    ("service".to_string(), "load_balancer".to_string()),
                    ("version".to_string(), "1.0".to_string()),
                ].iter().cloned().collect(),
            },
            alerts: vec![
                AlertRule {
                    name: "high_error_rate".to_string(),
                    condition: "error_rate > threshold".to_string(),
                    threshold: 0.05,
                    severity: "warning".to_string(),
                },
            ],
        },
        security: SecurityConfig {
            tls: TlsConfig {
                enabled: true,
                min_version: "1.2".to_string(),
                cipher_suites: vec![
                    "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384".to_string(),
                    "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256".to_string(),
                ],
                cert_path: "/etc/ssl/certs/server.crt".to_string(),
                key_path: "/etc/ssl/private/server.key".to_string(),
            },
            authentication: AuthConfig {
                method: "jwt".to_string(),
                token_lifetime: 3600,
                refresh_enabled: true,
            },
            rate_limits: vec![
                RateLimit {
                    path: "/api/*".to_string(),
                    requests_per_minute: 1000,
                    burst: 100,
                },
                RateLimit {
                    path: "/auth/*".to_string(),
                    requests_per_minute: 100,
                    burst: 10,
                },
            ],
        },
    };
    
    let toml_output = toml::to_string_pretty(&config)?;
    println!("Network Configuration TOML:\n{}", toml_output);
    
    // Parse it back
    let parsed: NetworkConfig = toml::from_str(&toml_output)?;
    println!("Parsed servers: {}", parsed.servers.len());
    println!("TLS enabled: {}", parsed.security.tls.enabled);
    
    Ok(())
}
```

Complex TOML structures use table notation with dotted keys for deep nesting.  
HashMap serializes as inline tables or separate sections depending on  
complexity. Arrays of complex structures become arrays of tables with  
double bracket notation. The hierarchical structure remains clear and  
human-readable even with deep nesting levels.  

## Arrays and mixed types in TOML

Handle arrays of different types and mixed-type structures in TOML.  
TOML supports arrays of primitives, arrays of tables, and nested  
array structures while maintaining type safety and readability.  

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct DataConfig {
    simple_arrays: SimpleArrays,
    mixed_data: Vec<DataEntry>,
    matrix: Vec<Vec<i32>>,
    key_value_pairs: Vec<KeyValue>,
    processors: Vec<Processor>,
}

#[derive(Serialize, Deserialize, Debug)]
struct SimpleArrays {
    numbers: Vec<i32>,
    strings: Vec<String>,
    booleans: Vec<bool>,
    floats: Vec<f64>,
}

#[derive(Serialize, Deserialize, Debug)]
struct DataEntry {
    id: u32,
    name: String,
    values: Vec<f64>,
    metadata: Option<String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct KeyValue {
    key: String,
    value: String,
    priority: i32,
}

#[derive(Serialize, Deserialize, Debug)]
#[serde(tag = "type")]
enum Processor {
    #[serde(rename = "filter")]
    Filter { field: String, operation: String, value: String },
    #[serde(rename = "transform")]
    Transform { input_field: String, output_field: String, function: String },
    #[serde(rename = "aggregate")]
    Aggregate { fields: Vec<String>, operation: String, group_by: Option<String> },
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let config = DataConfig {
        simple_arrays: SimpleArrays {
            numbers: vec![1, 2, 3, 5, 8, 13, 21],
            strings: vec![
                "alpha".to_string(),
                "beta".to_string(),
                "gamma".to_string(),
                "delta".to_string(),
            ],
            booleans: vec![true, false, true, true, false],
            floats: vec![3.14159, 2.71828, 1.41421, 0.57721],
        },
        mixed_data: vec![
            DataEntry {
                id: 1,
                name: "Temperature Sensor".to_string(),
                values: vec![20.5, 21.0, 20.8, 21.2],
                metadata: Some("Celsius".to_string()),
            },
            DataEntry {
                id: 2,
                name: "Humidity Sensor".to_string(),
                values: vec![45.2, 46.1, 44.8, 45.9],
                metadata: Some("Percentage".to_string()),
            },
            DataEntry {
                id: 3,
                name: "Pressure Sensor".to_string(),
                values: vec![1013.25, 1012.8, 1013.1],
                metadata: None,
            },
        ],
        matrix: vec![
            vec![1, 2, 3],
            vec![4, 5, 6],
            vec![7, 8, 9],
        ],
        key_value_pairs: vec![
            KeyValue {
                key: "database.host".to_string(),
                value: "localhost".to_string(),
                priority: 1,
            },
            KeyValue {
                key: "database.port".to_string(),
                value: "5432".to_string(),
                priority: 2,
            },
            KeyValue {
                key: "cache.enabled".to_string(),
                value: "true".to_string(),
                priority: 3,
            },
        ],
        processors: vec![
            Processor::Filter {
                field: "status".to_string(),
                operation: "equals".to_string(),
                value: "active".to_string(),
            },
            Processor::Transform {
                input_field: "timestamp".to_string(),
                output_field: "date".to_string(),
                function: "format_date".to_string(),
            },
            Processor::Aggregate {
                fields: vec!["sales".to_string(), "quantity".to_string()],
                operation: "sum".to_string(),
                group_by: Some("region".to_string()),
            },
        ],
    };
    
    let toml_output = toml::to_string_pretty(&config)?;
    println!("Data Configuration TOML:\n{}", toml_output);
    
    // Parse the configuration back
    let parsed: DataConfig = toml::from_str(&toml_output)?;
    println!("Parsed data entries: {}", parsed.mixed_data.len());
    println!("Matrix dimensions: {}x{}", parsed.matrix.len(), parsed.matrix[0].len());
    println!("Number of processors: {}", parsed.processors.len());
    
    Ok(())
}
```

TOML arrays maintain homogeneous types within each array while supporting  
complex nested structures. Arrays of tables create repeated sections with  
consistent structure. Matrix data serializes as nested arrays preserving  
dimensional relationships. Enum variants with internal tagging create  
clear processor definitions with type discrimination.  

## Comments and metadata preservation

Work with TOML files that include comments and preserve formatting.  
While serde doesn't preserve comments during round-trip serialization,  
you can structure data to include metadata and documentation fields.  

```rust
use serde::{Deserialize, Serialize};
use std::collections::HashMap;

#[derive(Serialize, Deserialize, Debug)]
struct DocumentedConfig {
    #[serde(rename = "_metadata")]
    metadata: ConfigMetadata,
    application: ApplicationConfig,
    services: HashMap<String, ServiceConfig>,
    deployment: DeploymentConfig,
}

#[derive(Serialize, Deserialize, Debug)]
struct ConfigMetadata {
    version: String,
    description: String,
    last_modified: String,
    author: String,
    notes: Vec<String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct ApplicationConfig {
    #[serde(rename = "_description")]
    description: String,
    name: String,
    version: String,
    environment: String,
    settings: AppSettings,
}

#[derive(Serialize, Deserialize, Debug)]
struct AppSettings {
    debug_mode: bool,
    log_level: String,
    max_connections: u32,
    timeout_seconds: u32,
}

#[derive(Serialize, Deserialize, Debug)]
struct ServiceConfig {
    #[serde(rename = "_description")]
    description: String,
    enabled: bool,
    host: String,
    port: u16,
    options: HashMap<String, String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct DeploymentConfig {
    #[serde(rename = "_description")]
    description: String,
    strategy: String,
    replicas: u32,
    resources: ResourceLimits,
    health_check: HealthCheckConfig,
}

#[derive(Serialize, Deserialize, Debug)]
struct ResourceLimits {
    cpu_limit: String,
    memory_limit: String,
    storage_limit: String,
}

#[derive(Serialize, Deserialize, Debug)]
struct HealthCheckConfig {
    enabled: bool,
    path: String,
    interval_seconds: u32,
    timeout_seconds: u32,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let config = DocumentedConfig {
        metadata: ConfigMetadata {
            version: "2.1".to_string(),
            description: "Production configuration for web application".to_string(),
            last_modified: "2024-03-15".to_string(),
            author: "DevOps Team".to_string(),
            notes: vec![
                "This configuration is deployed to production environment".to_string(),
                "All changes must be reviewed and approved".to_string(),
                "Contact devops@company.com for questions".to_string(),
            ],
        },
        application: ApplicationConfig {
            description: "Main application configuration settings".to_string(),
            name: "WebAPI".to_string(),
            version: "3.2.1".to_string(),
            environment: "production".to_string(),
            settings: AppSettings {
                debug_mode: false,
                log_level: "info".to_string(),
                max_connections: 1000,
                timeout_seconds: 30,
            },
        },
        services: {
            let mut services = HashMap::new();
            services.insert("database".to_string(), ServiceConfig {
                description: "Primary PostgreSQL database connection".to_string(),
                enabled: true,
                host: "db.prod.internal".to_string(),
                port: 5432,
                options: {
                    let mut opts = HashMap::new();
                    opts.insert("pool_size".to_string(), "20".to_string());
                    opts.insert("ssl_mode".to_string(), "require".to_string());
                    opts.insert("connect_timeout".to_string(), "10".to_string());
                    opts
                },
            });
            services.insert("redis".to_string(), ServiceConfig {
                description: "Redis cache for session storage and caching".to_string(),
                enabled: true,
                host: "cache.prod.internal".to_string(),
                port: 6379,
                options: {
                    let mut opts = HashMap::new();
                    opts.insert("db".to_string(), "0".to_string());
                    opts.insert("password_required".to_string(), "true".to_string());
                    opts
                },
            });
            services
        },
        deployment: DeploymentConfig {
            description: "Kubernetes deployment configuration".to_string(),
            strategy: "rolling_update".to_string(),
            replicas: 3,
            resources: ResourceLimits {
                cpu_limit: "1000m".to_string(),
                memory_limit: "2Gi".to_string(),
                storage_limit: "10Gi".to_string(),
            },
            health_check: HealthCheckConfig {
                enabled: true,
                path: "/health".to_string(),
                interval_seconds: 30,
                timeout_seconds: 5,
            },
        },
    };
    
    let toml_output = toml::to_string_pretty(&config)?;
    println!("Documented Configuration TOML:\n{}", toml_output);
    
    // Parse and validate metadata
    let parsed: DocumentedConfig = toml::from_str(&toml_output)?;
    println!("Configuration version: {}", parsed.metadata.version);
    println!("Application: {} v{}", parsed.application.name, parsed.application.version);
    println!("Services configured: {}", parsed.services.len());
    println!("Deployment replicas: {}", parsed.deployment.replicas);
    
    Ok(())
}
```

Documentation fields with underscore prefixes create self-documenting  
configurations. Metadata sections provide version control and authorship  
information. Structured descriptions replace traditional comments while  
remaining part of the serialized data. This approach maintains  
documentation through serialization round-trips.  

## Basic YAML operations

Serialize and deserialize data using YAML format for configuration files  
and data exchange. YAML provides excellent human readability with support  
for complex nested structures and reference relationships.  

```rust
// Add to Cargo.toml: serde_yaml = "0.9"
use serde::{Deserialize, Serialize};
use std::collections::HashMap;

#[derive(Serialize, Deserialize, Debug)]
struct SystemConfig {
    name: String,
    version: String,
    components: Vec<Component>,
    settings: Settings,
    connections: HashMap<String, Connection>,
}

#[derive(Serialize, Deserialize, Debug)]
struct Component {
    name: String,
    enabled: bool,
    config: ComponentConfig,
    dependencies: Vec<String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct ComponentConfig {
    memory_limit: String,
    cpu_limit: f64,
    environment: HashMap<String, String>,
    volumes: Option<Vec<Volume>>,
}

#[derive(Serialize, Deserialize, Debug)]
struct Volume {
    source: String,
    target: String,
    read_only: bool,
}

#[derive(Serialize, Deserialize, Debug)]
struct Settings {
    log_level: String,
    max_retries: u32,
    timeout_duration: String,
    security: SecuritySettings,
}

#[derive(Serialize, Deserialize, Debug)]
struct SecuritySettings {
    encryption_enabled: bool,
    key_rotation_days: u32,
    allowed_origins: Vec<String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct Connection {
    host: String,
    port: u16,
    protocol: String,
    authentication: AuthMethod,
}

#[derive(Serialize, Deserialize, Debug)]
#[serde(tag = "type")]
enum AuthMethod {
    #[serde(rename = "basic")]
    Basic { username: String, password: String },
    #[serde(rename = "token")]
    Token { token: String, expires_at: Option<String> },
    #[serde(rename = "certificate")]
    Certificate { cert_path: String, key_path: String },
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let config = SystemConfig {
        name: "Microservices Platform".to_string(),
        version: "1.3.2".to_string(),
        components: vec![
            Component {
                name: "api-gateway".to_string(),
                enabled: true,
                config: ComponentConfig {
                    memory_limit: "512Mi".to_string(),
                    cpu_limit: 0.5,
                    environment: {
                        let mut env = HashMap::new();
                        env.insert("PORT".to_string(), "8080".to_string());
                        env.insert("LOG_LEVEL".to_string(), "info".to_string());
                        env
                    },
                    volumes: Some(vec![
                        Volume {
                            source: "/app/config".to_string(),
                            target: "/etc/config".to_string(),
                            read_only: true,
                        },
                    ]),
                },
                dependencies: vec!["database".to_string(), "cache".to_string()],
            },
            Component {
                name: "user-service".to_string(),
                enabled: true,
                config: ComponentConfig {
                    memory_limit: "256Mi".to_string(),
                    cpu_limit: 0.3,
                    environment: {
                        let mut env = HashMap::new();
                        env.insert("DATABASE_URL".to_string(), "postgres://...".to_string());
                        env
                    },
                    volumes: None,
                },
                dependencies: vec!["database".to_string()],
            },
        ],
        settings: Settings {
            log_level: "info".to_string(),
            max_retries: 3,
            timeout_duration: "30s".to_string(),
            security: SecuritySettings {
                encryption_enabled: true,
                key_rotation_days: 90,
                allowed_origins: vec![
                    "https://app.example.com".to_string(),
                    "https://admin.example.com".to_string(),
                ],
            },
        },
        connections: {
            let mut connections = HashMap::new();
            connections.insert("primary_db".to_string(), Connection {
                host: "db.cluster.local".to_string(),
                port: 5432,
                protocol: "postgresql".to_string(),
                authentication: AuthMethod::Basic {
                    username: "app_user".to_string(),
                    password: "${DATABASE_PASSWORD}".to_string(),
                },
            });
            connections.insert("cache_server".to_string(), Connection {
                host: "redis.cluster.local".to_string(),
                port: 6379,
                protocol: "redis".to_string(),
                authentication: AuthMethod::Token {
                    token: "${REDIS_TOKEN}".to_string(),
                    expires_at: None,
                },
            });
            connections
        },
    };
    
    // Serialize to YAML
    let yaml_output = serde_yaml::to_string(&config)?;
    println!("System Configuration YAML:\n{}", yaml_output);
    
    // Deserialize from YAML string
    let yaml_input = r#"
name: "Test System"
version: "0.1.0"
components:
  - name: "test-component"
    enabled: false
    config:
      memory_limit: "128Mi"
      cpu_limit: 0.1
      environment:
        DEBUG: "true"
    dependencies: []
settings:
  log_level: "debug"
  max_retries: 5
  timeout_duration: "10s"
  security:
    encryption_enabled: false
    key_rotation_days: 30
    allowed_origins:
      - "http://localhost:3000"
connections:
  test_db:
    host: "localhost"
    port: 5432
    protocol: "postgresql"
    authentication:
      type: "basic"
      username: "test"
      password: "test123"
"#;
    
    let parsed_config: SystemConfig = serde_yaml::from_str(yaml_input)?;
    println!("Parsed system: {}", parsed_config.name);
    println!("Components: {}", parsed_config.components.len());
    println!("Connections: {}", parsed_config.connections.len());
    
    Ok(())
}
```

YAML serialization creates highly readable configuration files with  
automatic indentation and flow control. Complex nested structures render  
clearly with proper hierarchy visualization. Enum variants serialize  
with clear type discrimination. YAML's natural syntax makes it ideal  
for human-edited configuration files and documentation.  