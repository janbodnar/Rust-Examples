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

## Complex nested structures in YAML

Handle deeply nested YAML structures with references, anchors, and  
complex relationships. YAML excels at representing hierarchical data  
with support for references that reduce duplication.  

```rust
use serde::{Deserialize, Serialize};
use std::collections::HashMap;

#[derive(Serialize, Deserialize, Debug)]
struct InfrastructureConfig {
    metadata: Metadata,
    environments: HashMap<String, Environment>,
    shared_resources: SharedResources,
    applications: Vec<Application>,
}

#[derive(Serialize, Deserialize, Debug)]
struct Metadata {
    version: String,
    description: String,
    maintainer: Contact,
    tags: Vec<String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct Contact {
    name: String,
    email: String,
    team: String,
}

#[derive(Serialize, Deserialize, Debug)]
struct Environment {
    region: String,
    cluster: ClusterConfig,
    networking: NetworkConfig,
    security: SecurityPolicy,
    monitoring: MonitoringConfig,
}

#[derive(Serialize, Deserialize, Debug)]
struct ClusterConfig {
    name: String,
    node_count: u32,
    machine_type: String,
    auto_scaling: AutoScalingConfig,
    maintenance_window: MaintenanceWindow,
}

#[derive(Serialize, Deserialize, Debug)]
struct AutoScalingConfig {
    enabled: bool,
    min_nodes: u32,
    max_nodes: u32,
    target_cpu_utilization: f64,
}

#[derive(Serialize, Deserialize, Debug)]
struct MaintenanceWindow {
    day: String,
    start_time: String,
    duration_hours: u32,
}

#[derive(Serialize, Deserialize, Debug)]
struct NetworkConfig {
    vpc_cidr: String,
    subnets: Vec<Subnet>,
    load_balancers: Vec<LoadBalancer>,
}

#[derive(Serialize, Deserialize, Debug)]
struct Subnet {
    name: String,
    cidr: String,
    availability_zone: String,
    subnet_type: String,
}

#[derive(Serialize, Deserialize, Debug)]
struct LoadBalancer {
    name: String,
    lb_type: String,
    listeners: Vec<Listener>,
}

#[derive(Serialize, Deserialize, Debug)]
struct Listener {
    port: u16,
    protocol: String,
    target_group: String,
}

#[derive(Serialize, Deserialize, Debug)]
struct SecurityPolicy {
    encryption: EncryptionConfig,
    access_control: AccessControl,
    audit_logging: bool,
}

#[derive(Serialize, Deserialize, Debug)]
struct EncryptionConfig {
    at_rest: bool,
    in_transit: bool,
    key_management: String,
}

#[derive(Serialize, Deserialize, Debug)]
struct AccessControl {
    rbac_enabled: bool,
    network_policies: Vec<NetworkPolicy>,
    pod_security_policies: Vec<String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct NetworkPolicy {
    name: String,
    namespace: String,
    ingress_rules: Vec<String>,
    egress_rules: Vec<String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct MonitoringConfig {
    metrics: MetricsConfig,
    logging: LoggingConfig,
    alerting: AlertingConfig,
}

#[derive(Serialize, Deserialize, Debug)]
struct MetricsConfig {
    enabled: bool,
    retention_days: u32,
    exporters: Vec<String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct LoggingConfig {
    level: String,
    centralized: bool,
    retention_days: u32,
}

#[derive(Serialize, Deserialize, Debug)]
struct AlertingConfig {
    enabled: bool,
    notification_channels: Vec<String>,
    rules: Vec<AlertRule>,
}

#[derive(Serialize, Deserialize, Debug)]
struct AlertRule {
    name: String,
    condition: String,
    severity: String,
    threshold: f64,
}

#[derive(Serialize, Deserialize, Debug)]
struct SharedResources {
    databases: Vec<DatabaseInstance>,
    cache_clusters: Vec<CacheCluster>,
    storage: StorageConfig,
}

#[derive(Serialize, Deserialize, Debug)]
struct DatabaseInstance {
    name: String,
    engine: String,
    version: String,
    instance_class: String,
    storage_gb: u32,
    backup_retention_days: u32,
}

#[derive(Serialize, Deserialize, Debug)]
struct CacheCluster {
    name: String,
    engine: String,
    node_type: String,
    num_nodes: u32,
}

#[derive(Serialize, Deserialize, Debug)]
struct StorageConfig {
    default_storage_class: String,
    backup_enabled: bool,
    encryption_enabled: bool,
}

#[derive(Serialize, Deserialize, Debug)]
struct Application {
    name: String,
    namespace: String,
    deployment: DeploymentSpec,
    service: ServiceSpec,
    ingress: Option<IngressSpec>,
}

#[derive(Serialize, Deserialize, Debug)]
struct DeploymentSpec {
    replicas: u32,
    container: ContainerSpec,
    resources: ResourceRequirements,
}

#[derive(Serialize, Deserialize, Debug)]
struct ContainerSpec {
    image: String,
    tag: String,
    ports: Vec<u16>,
    environment: HashMap<String, String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct ResourceRequirements {
    requests: ResourceLimits,
    limits: ResourceLimits,
}

#[derive(Serialize, Deserialize, Debug)]
struct ResourceLimits {
    cpu: String,
    memory: String,
}

#[derive(Serialize, Deserialize, Debug)]
struct ServiceSpec {
    service_type: String,
    ports: Vec<ServicePort>,
}

#[derive(Serialize, Deserialize, Debug)]
struct ServicePort {
    name: String,
    port: u16,
    target_port: u16,
}

#[derive(Serialize, Deserialize, Debug)]
struct IngressSpec {
    host: String,
    paths: Vec<String>,
    tls_enabled: bool,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let config = InfrastructureConfig {
        metadata: Metadata {
            version: "2.0.0".to_string(),
            description: "Production infrastructure configuration".to_string(),
            maintainer: Contact {
                name: "Platform Team".to_string(),
                email: "platform@company.com".to_string(),
                team: "Infrastructure".to_string(),
            },
            tags: vec!["production".to_string(), "kubernetes".to_string(), "microservices".to_string()],
        },
        environments: {
            let mut envs = HashMap::new();
            envs.insert("production".to_string(), Environment {
                region: "us-west-2".to_string(),
                cluster: ClusterConfig {
                    name: "prod-cluster".to_string(),
                    node_count: 6,
                    machine_type: "n1-standard-4".to_string(),
                    auto_scaling: AutoScalingConfig {
                        enabled: true,
                        min_nodes: 3,
                        max_nodes: 12,
                        target_cpu_utilization: 70.0,
                    },
                    maintenance_window: MaintenanceWindow {
                        day: "Sunday".to_string(),
                        start_time: "02:00".to_string(),
                        duration_hours: 4,
                    },
                },
                networking: NetworkConfig {
                    vpc_cidr: "10.0.0.0/16".to_string(),
                    subnets: vec![
                        Subnet {
                            name: "public-subnet-1".to_string(),
                            cidr: "10.0.1.0/24".to_string(),
                            availability_zone: "us-west-2a".to_string(),
                            subnet_type: "public".to_string(),
                        },
                        Subnet {
                            name: "private-subnet-1".to_string(),
                            cidr: "10.0.2.0/24".to_string(),
                            availability_zone: "us-west-2a".to_string(),
                            subnet_type: "private".to_string(),
                        },
                    ],
                    load_balancers: vec![
                        LoadBalancer {
                            name: "api-lb".to_string(),
                            lb_type: "application".to_string(),
                            listeners: vec![
                                Listener {
                                    port: 80,
                                    protocol: "HTTP".to_string(),
                                    target_group: "api-targets".to_string(),
                                },
                                Listener {
                                    port: 443,
                                    protocol: "HTTPS".to_string(),
                                    target_group: "api-targets".to_string(),
                                },
                            ],
                        },
                    ],
                },
                security: SecurityPolicy {
                    encryption: EncryptionConfig {
                        at_rest: true,
                        in_transit: true,
                        key_management: "aws-kms".to_string(),
                    },
                    access_control: AccessControl {
                        rbac_enabled: true,
                        network_policies: vec![
                            NetworkPolicy {
                                name: "default-deny".to_string(),
                                namespace: "default".to_string(),
                                ingress_rules: vec!["deny-all".to_string()],
                                egress_rules: vec!["allow-dns".to_string(), "allow-egress".to_string()],
                            },
                        ],
                        pod_security_policies: vec!["restricted".to_string()],
                    },
                    audit_logging: true,
                },
                monitoring: MonitoringConfig {
                    metrics: MetricsConfig {
                        enabled: true,
                        retention_days: 90,
                        exporters: vec!["prometheus".to_string(), "cloudwatch".to_string()],
                    },
                    logging: LoggingConfig {
                        level: "info".to_string(),
                        centralized: true,
                        retention_days: 30,
                    },
                    alerting: AlertingConfig {
                        enabled: true,
                        notification_channels: vec!["slack".to_string(), "email".to_string()],
                        rules: vec![
                            AlertRule {
                                name: "high-cpu".to_string(),
                                condition: "cpu > threshold".to_string(),
                                severity: "warning".to_string(),
                                threshold: 80.0,
                            },
                        ],
                    },
                },
            });
            envs
        },
        shared_resources: SharedResources {
            databases: vec![
                DatabaseInstance {
                    name: "primary-db".to_string(),
                    engine: "postgresql".to_string(),
                    version: "13.7".to_string(),
                    instance_class: "db.r5.xlarge".to_string(),
                    storage_gb: 500,
                    backup_retention_days: 7,
                },
            ],
            cache_clusters: vec![
                CacheCluster {
                    name: "redis-cluster".to_string(),
                    engine: "redis".to_string(),
                    node_type: "cache.r5.large".to_string(),
                    num_nodes: 3,
                },
            ],
            storage: StorageConfig {
                default_storage_class: "gp2".to_string(),
                backup_enabled: true,
                encryption_enabled: true,
            },
        },
        applications: vec![
            Application {
                name: "api-service".to_string(),
                namespace: "production".to_string(),
                deployment: DeploymentSpec {
                    replicas: 3,
                    container: ContainerSpec {
                        image: "mycompany/api".to_string(),
                        tag: "v2.1.0".to_string(),
                        ports: vec![8080],
                        environment: {
                            let mut env = HashMap::new();
                            env.insert("ENV".to_string(), "production".to_string());
                            env.insert("LOG_LEVEL".to_string(), "info".to_string());
                            env
                        },
                    },
                    resources: ResourceRequirements {
                        requests: ResourceLimits {
                            cpu: "250m".to_string(),
                            memory: "512Mi".to_string(),
                        },
                        limits: ResourceLimits {
                            cpu: "500m".to_string(),
                            memory: "1Gi".to_string(),
                        },
                    },
                },
                service: ServiceSpec {
                    service_type: "ClusterIP".to_string(),
                    ports: vec![
                        ServicePort {
                            name: "http".to_string(),
                            port: 80,
                            target_port: 8080,
                        },
                    ],
                },
                ingress: Some(IngressSpec {
                    host: "api.company.com".to_string(),
                    paths: vec!["/api/*".to_string()],
                    tls_enabled: true,
                }),
            },
        ],
    };
    
    let yaml_output = serde_yaml::to_string(&config)?;
    println!("Infrastructure Configuration YAML:\n{}", yaml_output);
    
    // Parse a subset for validation
    let parsed: InfrastructureConfig = serde_yaml::from_str(&yaml_output)?;
    println!("Configuration version: {}", parsed.metadata.version);
    println!("Environments: {}", parsed.environments.len());
    println!("Applications: {}", parsed.applications.len());
    
    Ok(())
}
```

Deep YAML structures maintain clear hierarchy through indentation levels.  
Complex nested relationships serialize naturally while preserving all  
structural information. Multi-level configurations demonstrate YAML's  
capability to handle enterprise-scale infrastructure definitions with  
proper organization and readability.  

## Sequences and mappings in YAML

Work with YAML sequences (arrays) and mappings (objects) including  
mixed types, flow syntax, and block syntax for different data  
representation needs and formatting preferences.  

```rust
use serde::{Deserialize, Serialize};
use std::collections::{HashMap, BTreeMap};

#[derive(Serialize, Deserialize, Debug)]
struct DataStructures {
    simple_sequences: SimpleSequences,
    complex_sequences: Vec<ComplexItem>,
    mappings: Mappings,
    mixed_structures: MixedStructures,
    ordered_data: BTreeMap<String, OrderedItem>,
}

#[derive(Serialize, Deserialize, Debug)]
struct SimpleSequences {
    numbers: Vec<i32>,
    strings: Vec<String>,
    booleans: Vec<bool>,
    mixed_primitives: Vec<serde_yaml::Value>,
}

#[derive(Serialize, Deserialize, Debug)]
struct ComplexItem {
    id: u32,
    name: String,
    properties: HashMap<String, String>,
    nested_data: NestedData,
    tags: Vec<String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct NestedData {
    category: String,
    scores: Vec<f64>,
    metadata: Option<HashMap<String, serde_yaml::Value>>,
}

#[derive(Serialize, Deserialize, Debug)]
struct Mappings {
    string_to_string: HashMap<String, String>,
    string_to_number: HashMap<String, i32>,
    string_to_object: HashMap<String, ConfigItem>,
    nested_mappings: HashMap<String, HashMap<String, String>>,
}

#[derive(Serialize, Deserialize, Debug)]
struct ConfigItem {
    enabled: bool,
    value: String,
    priority: u32,
}

#[derive(Serialize, Deserialize, Debug)]
struct MixedStructures {
    matrix: Vec<Vec<i32>>,
    records: Vec<Record>,
    lookup_tables: HashMap<String, Vec<String>>,
    hierarchical: Hierarchical,
}

#[derive(Serialize, Deserialize, Debug)]
struct Record {
    timestamp: String,
    event: String,
    data: serde_yaml::Value,
    severity: Option<String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct Hierarchical {
    root: Node,
}

#[derive(Serialize, Deserialize, Debug)]
struct Node {
    name: String,
    value: Option<String>,
    children: Vec<Node>,
    attributes: HashMap<String, String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct OrderedItem {
    sequence: u32,
    description: String,
    depends_on: Vec<String>,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let data = DataStructures {
        simple_sequences: SimpleSequences {
            numbers: vec![1, 1, 2, 3, 5, 8, 13, 21, 34],
            strings: vec![
                "alpha".to_string(),
                "beta".to_string(),
                "gamma".to_string(),
                "delta".to_string(),
            ],
            booleans: vec![true, false, true, true, false],
            mixed_primitives: vec![
                serde_yaml::Value::Number(serde_yaml::Number::from(42)),
                serde_yaml::Value::String("hello".to_string()),
                serde_yaml::Value::Bool(true),
                serde_yaml::Value::Null,
            ],
        },
        complex_sequences: vec![
            ComplexItem {
                id: 1,
                name: "Database Server".to_string(),
                properties: {
                    let mut props = HashMap::new();
                    props.insert("type".to_string(), "postgresql".to_string());
                    props.insert("version".to_string(), "13.7".to_string());
                    props.insert("port".to_string(), "5432".to_string());
                    props
                },
                nested_data: NestedData {
                    category: "infrastructure".to_string(),
                    scores: vec![95.5, 87.2, 91.8, 93.1],
                    metadata: Some({
                        let mut meta = HashMap::new();
                        meta.insert("location".to_string(), serde_yaml::Value::String("us-west-2".to_string()));
                        meta.insert("replicas".to_string(), serde_yaml::Value::Number(serde_yaml::Number::from(3)));
                        meta
                    }),
                },
                tags: vec!["database".to_string(), "critical".to_string(), "production".to_string()],
            },
            ComplexItem {
                id: 2,
                name: "Cache Layer".to_string(),
                properties: {
                    let mut props = HashMap::new();
                    props.insert("type".to_string(), "redis".to_string());
                    props.insert("version".to_string(), "6.2".to_string());
                    props.insert("port".to_string(), "6379".to_string());
                    props
                },
                nested_data: NestedData {
                    category: "cache".to_string(),
                    scores: vec![98.1, 94.7, 96.3],
                    metadata: None,
                },
                tags: vec!["cache".to_string(), "performance".to_string()],
            },
        ],
        mappings: Mappings {
            string_to_string: {
                let mut map = HashMap::new();
                map.insert("database_host".to_string(), "db.example.com".to_string());
                map.insert("api_endpoint".to_string(), "https://api.example.com".to_string());
                map.insert("cdn_url".to_string(), "https://cdn.example.com".to_string());
                map
            },
            string_to_number: {
                let mut map = HashMap::new();
                map.insert("max_connections".to_string(), 100);
                map.insert("timeout_seconds".to_string(), 30);
                map.insert("retry_attempts".to_string(), 3);
                map
            },
            string_to_object: {
                let mut map = HashMap::new();
                map.insert("auth".to_string(), ConfigItem {
                    enabled: true,
                    value: "jwt".to_string(),
                    priority: 1,
                });
                map.insert("logging".to_string(), ConfigItem {
                    enabled: true,
                    value: "structured".to_string(),
                    priority: 2,
                });
                map
            },
            nested_mappings: {
                let mut outer = HashMap::new();
                let mut dev_config = HashMap::new();
                dev_config.insert("debug".to_string(), "true".to_string());
                dev_config.insert("log_level".to_string(), "debug".to_string());
                outer.insert("development".to_string(), dev_config);
                
                let mut prod_config = HashMap::new();
                prod_config.insert("debug".to_string(), "false".to_string());
                prod_config.insert("log_level".to_string(), "info".to_string());
                outer.insert("production".to_string(), prod_config);
                outer
            },
        },
        mixed_structures: MixedStructures {
            matrix: vec![
                vec![1, 2, 3, 4],
                vec![5, 6, 7, 8],
                vec![9, 10, 11, 12],
            ],
            records: vec![
                Record {
                    timestamp: "2024-03-15T10:30:00Z".to_string(),
                    event: "user_login".to_string(),
                    data: serde_yaml::Value::Mapping({
                        let mut map = serde_yaml::Mapping::new();
                        map.insert(
                            serde_yaml::Value::String("user_id".to_string()),
                            serde_yaml::Value::Number(serde_yaml::Number::from(12345))
                        );
                        map.insert(
                            serde_yaml::Value::String("ip_address".to_string()),
                            serde_yaml::Value::String("192.168.1.100".to_string())
                        );
                        map
                    }),
                    severity: Some("info".to_string()),
                },
                Record {
                    timestamp: "2024-03-15T10:31:00Z".to_string(),
                    event: "error_occurred".to_string(),
                    data: serde_yaml::Value::String("Database connection timeout".to_string()),
                    severity: Some("error".to_string()),
                },
            ],
            lookup_tables: {
                let mut tables = HashMap::new();
                tables.insert("http_codes".to_string(), vec![
                    "200".to_string(), "201".to_string(), "400".to_string(), "404".to_string(), "500".to_string()
                ]);
                tables.insert("log_levels".to_string(), vec![
                    "trace".to_string(), "debug".to_string(), "info".to_string(), "warn".to_string(), "error".to_string()
                ]);
                tables
            },
            hierarchical: Hierarchical {
                root: Node {
                    name: "application".to_string(),
                    value: None,
                    attributes: {
                        let mut attrs = HashMap::new();
                        attrs.insert("version".to_string(), "2.0.0".to_string());
                        attrs.insert("environment".to_string(), "production".to_string());
                        attrs
                    },
                    children: vec![
                        Node {
                            name: "database".to_string(),
                            value: Some("postgresql://db:5432/app".to_string()),
                            attributes: HashMap::new(),
                            children: vec![],
                        },
                        Node {
                            name: "services".to_string(),
                            value: None,
                            attributes: HashMap::new(),
                            children: vec![
                                Node {
                                    name: "api".to_string(),
                                    value: Some("http://api:8080".to_string()),
                                    attributes: {
                                        let mut attrs = HashMap::new();
                                        attrs.insert("replicas".to_string(), "3".to_string());
                                        attrs
                                    },
                                    children: vec![],
                                },
                                Node {
                                    name: "worker".to_string(),
                                    value: Some("worker:9000".to_string()),
                                    attributes: {
                                        let mut attrs = HashMap::new();
                                        attrs.insert("replicas".to_string(), "2".to_string());
                                        attrs
                                    },
                                    children: vec![],
                                },
                            ],
                        },
                    ],
                },
            },
        },
        ordered_data: {
            let mut ordered = BTreeMap::new();
            ordered.insert("step_001".to_string(), OrderedItem {
                sequence: 1,
                description: "Initialize system".to_string(),
                depends_on: vec![],
            });
            ordered.insert("step_002".to_string(), OrderedItem {
                sequence: 2,
                description: "Load configuration".to_string(),
                depends_on: vec!["step_001".to_string()],
            });
            ordered.insert("step_003".to_string(), OrderedItem {
                sequence: 3,
                description: "Start services".to_string(),
                depends_on: vec!["step_002".to_string()],
            });
            ordered
        },
    };
    
    let yaml_output = serde_yaml::to_string(&data)?;
    println!("Data Structures YAML:\n{}", yaml_output);
    
    // Parse and validate structure
    let parsed: DataStructures = serde_yaml::from_str(&yaml_output)?;
    println!("Simple sequences - numbers count: {}", parsed.simple_sequences.numbers.len());
    println!("Complex sequences count: {}", parsed.complex_sequences.len());
    println!("Matrix dimensions: {}x{}", parsed.mixed_structures.matrix.len(), 
             parsed.mixed_structures.matrix[0].len());
    println!("Ordered items: {}", parsed.ordered_data.len());
    
    Ok(())
}
```

YAML sequences render as hyphen-prefixed lists with consistent indentation.  
Mappings create key-value pairs with colon separation and proper nesting.  
Mixed data types within sequences and mappings maintain type information  
through YAML's natural syntax. BTreeMap ensures deterministic key ordering  
in serialized output for consistent file generation.  

## Multi-document YAML

Handle multiple YAML documents in a single file using document separators.  
Multi-document YAML enables batch processing of related configurations  
and supports streaming scenarios with document boundaries.  

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Document {
    document_type: String,
    version: String,
    content: DocumentContent,
}

#[derive(Serialize, Deserialize, Debug)]
#[serde(tag = "type")]
enum DocumentContent {
    #[serde(rename = "config")]
    Config {
        name: String,
        settings: std::collections::HashMap<String, String>,
    },
    #[serde(rename = "deployment")]
    Deployment {
        app_name: String,
        replicas: u32,
        image: String,
        resources: ResourceSpec,
    },
    #[serde(rename = "service")]
    Service {
        name: String,
        ports: Vec<PortSpec>,
        selector: std::collections::HashMap<String, String>,
    },
    #[serde(rename = "secret")]
    Secret {
        name: String,
        data_type: String,
        encoded: bool,
    },
}

#[derive(Serialize, Deserialize, Debug)]
struct ResourceSpec {
    cpu: String,
    memory: String,
    storage: Option<String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct PortSpec {
    name: String,
    port: u16,
    target_port: u16,
    protocol: String,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let documents = vec![
        Document {
            document_type: "ConfigMap".to_string(),
            version: "v1".to_string(),
            content: DocumentContent::Config {
                name: "app-config".to_string(),
                settings: {
                    let mut settings = std::collections::HashMap::new();
                    settings.insert("database_url".to_string(), "postgres://localhost:5432/app".to_string());
                    settings.insert("log_level".to_string(), "info".to_string());
                    settings.insert("debug_mode".to_string(), "false".to_string());
                    settings
                },
            },
        },
        Document {
            document_type: "Deployment".to_string(),
            version: "apps/v1".to_string(),
            content: DocumentContent::Deployment {
                app_name: "web-app".to_string(),
                replicas: 3,
                image: "mycompany/web-app:v2.1.0".to_string(),
                resources: ResourceSpec {
                    cpu: "500m".to_string(),
                    memory: "1Gi".to_string(),
                    storage: Some("10Gi".to_string()),
                },
            },
        },
        Document {
            document_type: "Service".to_string(),
            version: "v1".to_string(),
            content: DocumentContent::Service {
                name: "web-app-service".to_string(),
                ports: vec![
                    PortSpec {
                        name: "http".to_string(),
                        port: 80,
                        target_port: 8080,
                        protocol: "TCP".to_string(),
                    },
                    PortSpec {
                        name: "https".to_string(),
                        port: 443,
                        target_port: 8443,
                        protocol: "TCP".to_string(),
                    },
                ],
                selector: {
                    let mut selector = std::collections::HashMap::new();
                    selector.insert("app".to_string(), "web-app".to_string());
                    selector.insert("version".to_string(), "v2.1.0".to_string());
                    selector
                },
            },
        },
        Document {
            document_type: "Secret".to_string(),
            version: "v1".to_string(),
            content: DocumentContent::Secret {
                name: "database-credentials".to_string(),
                data_type: "Opaque".to_string(),
                encoded: true,
            },
        },
    ];
    
    // Serialize multiple documents
    let mut yaml_documents = String::new();
    for (i, doc) in documents.iter().enumerate() {
        if i > 0 {
            yaml_documents.push_str("---\n");
        }
        yaml_documents.push_str(&serde_yaml::to_string(doc)?);
    }
    
    println!("Multi-document YAML:\n{}", yaml_documents);
    
    // Parse multi-document YAML string
    let multi_doc_yaml = r#"---
document_type: "ConfigMap"
version: "v1"
content:
  type: "config"
  name: "test-config"
  settings:
    env: "test"
    debug: "true"
---
document_type: "Deployment"
version: "apps/v1"
content:
  type: "deployment"
  app_name: "test-app"
  replicas: 1
  image: "test-app:latest"
  resources:
    cpu: "100m"
    memory: "256Mi"
---
document_type: "Service"
version: "v1"
content:
  type: "service"
  name: "test-service"
  ports:
    - name: "http"
      port: 80
      target_port: 8080
      protocol: "TCP"
  selector:
    app: "test-app"
"#;
    
    // Parse each document separately
    let yaml_docs: Vec<&str> = multi_doc_yaml.split("---").collect();
    let mut parsed_documents = Vec::new();
    
    for doc_str in yaml_docs {
        let trimmed = doc_str.trim();
        if !trimmed.is_empty() {
            let doc: Document = serde_yaml::from_str(trimmed)?;
            parsed_documents.push(doc);
        }
    }
    
    println!("\nParsed {} documents", parsed_documents.len());
    for (i, doc) in parsed_documents.iter().enumerate() {
        println!("Document {}: {} ({})", i + 1, doc.document_type, doc.version);
        match &doc.content {
            DocumentContent::Config { name, settings } => {
                println!("  Config '{}' with {} settings", name, settings.len());
            },
            DocumentContent::Deployment { app_name, replicas, .. } => {
                println!("  Deployment '{}' with {} replicas", app_name, replicas);
            },
            DocumentContent::Service { name, ports, .. } => {
                println!("  Service '{}' with {} ports", name, ports.len());
            },
            DocumentContent::Secret { name, data_type, .. } => {
                println!("  Secret '{}' of type '{}'", name, data_type);
            },
        }
    }
    
    Ok(())
}
```

Multi-document YAML uses `---` separators to distinguish between documents  
in a single file. Each document maintains independent structure and type  
information. This format is commonly used in Kubernetes manifests and  
configuration management systems where related resources are grouped  
together for batch processing and deployment workflows.  

## Custom serializers and deserializers

Implement custom serialization logic for specific data types or formats.  
Custom serializers provide fine-grained control over the serialization  
process, enabling specialized formats and data transformations.  

```rust
use serde::{Deserialize, Deserializer, Serialize, Serializer};
use serde::de::{self, Visitor};
use std::fmt;

#[derive(Debug)]
struct CustomData {
    timestamp: u64,
    coordinates: Coordinates,
    encoded_data: Vec<u8>,
    flags: BitFlags,
    version: Version,
}

#[derive(Debug)]
struct Coordinates {
    latitude: f64,
    longitude: f64,
}

#[derive(Debug)]
struct BitFlags(u32);

#[derive(Debug)]
struct Version {
    major: u16,
    minor: u16,
    patch: u16,
}

// Custom serialization for Coordinates as "lat,lng" string
impl Serialize for Coordinates {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        let coord_string = format!("{:.6},{:.6}", self.latitude, self.longitude);
        serializer.serialize_str(&coord_string)
    }
}

impl<'de> Deserialize<'de> for Coordinates {
    fn deserialize<D>(deserializer: D) -> Result<Self, D::Error>
    where
        D: Deserializer<'de>,
    {
        struct CoordinatesVisitor;
        
        impl<'de> Visitor<'de> for CoordinatesVisitor {
            type Value = Coordinates;
            
            fn expecting(&self, formatter: &mut fmt::Formatter) -> fmt::Result {
                formatter.write_str("a coordinate string in format 'lat,lng'")
            }
            
            fn visit_str<E>(self, value: &str) -> Result<Coordinates, E>
            where
                E: de::Error,
            {
                let parts: Vec<&str> = value.split(',').collect();
                if parts.len() != 2 {
                    return Err(E::custom("expected 'lat,lng' format"));
                }
                
                let latitude = parts[0].parse::<f64>()
                    .map_err(|_| E::custom("invalid latitude"))?;
                let longitude = parts[1].parse::<f64>()
                    .map_err(|_| E::custom("invalid longitude"))?;
                
                Ok(Coordinates { latitude, longitude })
            }
        }
        
        deserializer.deserialize_str(CoordinatesVisitor)
    }
}

// Custom serialization for BitFlags as hex string
impl Serialize for BitFlags {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        let hex_string = format!("0x{:08X}", self.0);
        serializer.serialize_str(&hex_string)
    }
}

impl<'de> Deserialize<'de> for BitFlags {
    fn deserialize<D>(deserializer: D) -> Result<Self, D::Error>
    where
        D: Deserializer<'de>,
    {
        struct BitFlagsVisitor;
        
        impl<'de> Visitor<'de> for BitFlagsVisitor {
            type Value = BitFlags;
            
            fn expecting(&self, formatter: &mut fmt::Formatter) -> fmt::Result {
                formatter.write_str("a hex string starting with '0x'")
            }
            
            fn visit_str<E>(self, value: &str) -> Result<BitFlags, E>
            where
                E: de::Error,
            {
                if !value.starts_with("0x") {
                    return Err(E::custom("expected hex string to start with '0x'"));
                }
                
                let hex_part = &value[2..];
                let flags = u32::from_str_radix(hex_part, 16)
                    .map_err(|_| E::custom("invalid hex number"))?;
                
                Ok(BitFlags(flags))
            }
        }
        
        deserializer.deserialize_str(BitFlagsVisitor)
    }
}

// Custom serialization for Version as "major.minor.patch" string
impl Serialize for Version {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        let version_string = format!("{}.{}.{}", self.major, self.minor, self.patch);
        serializer.serialize_str(&version_string)
    }
}

impl<'de> Deserialize<'de> for Version {
    fn deserialize<D>(deserializer: D) -> Result<Self, D::Error>
    where
        D: Deserializer<'de>,
    {
        struct VersionVisitor;
        
        impl<'de> Visitor<'de> for VersionVisitor {
            type Value = Version;
            
            fn expecting(&self, formatter: &mut fmt::Formatter) -> fmt::Result {
                formatter.write_str("a version string in format 'major.minor.patch'")
            }
            
            fn visit_str<E>(self, value: &str) -> Result<Version, E>
            where
                E: de::Error,
            {
                let parts: Vec<&str> = value.split('.').collect();
                if parts.len() != 3 {
                    return Err(E::custom("expected 'major.minor.patch' format"));
                }
                
                let major = parts[0].parse::<u16>()
                    .map_err(|_| E::custom("invalid major version"))?;
                let minor = parts[1].parse::<u16>()
                    .map_err(|_| E::custom("invalid minor version"))?;
                let patch = parts[2].parse::<u16>()
                    .map_err(|_| E::custom("invalid patch version"))?;
                
                Ok(Version { major, minor, patch })
            }
        }
        
        deserializer.deserialize_str(VersionVisitor)
    }
}

// Custom serialization for the main struct
impl Serialize for CustomData {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        use serde::ser::SerializeStruct;
        
        let mut state = serializer.serialize_struct("CustomData", 5)?;
        state.serialize_field("timestamp", &self.timestamp)?;
        state.serialize_field("location", &self.coordinates)?;
        
        // Encode binary data as base64
        let encoded = base64::encode(&self.encoded_data);
        state.serialize_field("data", &encoded)?;
        
        state.serialize_field("flags", &self.flags)?;
        state.serialize_field("version", &self.version)?;
        state.end()
    }
}

impl<'de> Deserialize<'de> for CustomData {
    fn deserialize<D>(deserializer: D) -> Result<Self, D::Error>
    where
        D: Deserializer<'de>,
    {
        #[derive(Deserialize)]
        #[serde(field_identifier, rename_all = "lowercase")]
        enum Field { Timestamp, Location, Data, Flags, Version }
        
        struct CustomDataVisitor;
        
        impl<'de> Visitor<'de> for CustomDataVisitor {
            type Value = CustomData;
            
            fn expecting(&self, formatter: &mut fmt::Formatter) -> fmt::Result {
                formatter.write_str("struct CustomData")
            }
            
            fn visit_map<V>(self, mut map: V) -> Result<CustomData, V::Error>
            where
                V: de::MapAccess<'de>,
            {
                let mut timestamp = None;
                let mut coordinates = None;
                let mut encoded_data = None;
                let mut flags = None;
                let mut version = None;
                
                while let Some(key) = map.next_key()? {
                    match key {
                        Field::Timestamp => {
                            if timestamp.is_some() {
                                return Err(de::Error::duplicate_field("timestamp"));
                            }
                            timestamp = Some(map.next_value()?);
                        }
                        Field::Location => {
                            if coordinates.is_some() {
                                return Err(de::Error::duplicate_field("location"));
                            }
                            coordinates = Some(map.next_value()?);
                        }
                        Field::Data => {
                            if encoded_data.is_some() {
                                return Err(de::Error::duplicate_field("data"));
                            }
                            let encoded_string: String = map.next_value()?;
                            let decoded = base64::decode(&encoded_string)
                                .map_err(|_| de::Error::custom("invalid base64 data"))?;
                            encoded_data = Some(decoded);
                        }
                        Field::Flags => {
                            if flags.is_some() {
                                return Err(de::Error::duplicate_field("flags"));
                            }
                            flags = Some(map.next_value()?);
                        }
                        Field::Version => {
                            if version.is_some() {
                                return Err(de::Error::duplicate_field("version"));
                            }
                            version = Some(map.next_value()?);
                        }
                    }
                }
                
                let timestamp = timestamp.ok_or_else(|| de::Error::missing_field("timestamp"))?;
                let coordinates = coordinates.ok_or_else(|| de::Error::missing_field("location"))?;
                let encoded_data = encoded_data.ok_or_else(|| de::Error::missing_field("data"))?;
                let flags = flags.ok_or_else(|| de::Error::missing_field("flags"))?;
                let version = version.ok_or_else(|| de::Error::missing_field("version"))?;
                
                Ok(CustomData {
                    timestamp,
                    coordinates,
                    encoded_data,
                    flags,
                    version,
                })
            }
        }
        
        const FIELDS: &[&str] = &["timestamp", "location", "data", "flags", "version"];
        deserializer.deserialize_struct("CustomData", FIELDS, CustomDataVisitor)
    }
}

// Add base64 support
mod base64 {
    pub fn encode(data: &[u8]) -> String {
        const CHARS: &[u8] = b"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
        let mut result = String::new();
        
        for chunk in data.chunks(3) {
            let mut buf = [0u8; 3];
            for (i, &b) in chunk.iter().enumerate() {
                buf[i] = b;
            }
            
            let b = (buf[0] as u32) << 16 | (buf[1] as u32) << 8 | (buf[2] as u32);
            result.push(CHARS[((b >> 18) & 63) as usize] as char);
            result.push(CHARS[((b >> 12) & 63) as usize] as char);
            result.push(if chunk.len() > 1 { CHARS[((b >> 6) & 63) as usize] as char } else { '=' });
            result.push(if chunk.len() > 2 { CHARS[(b & 63) as usize] as char } else { '=' });
        }
        result
    }
    
    pub fn decode(s: &str) -> Result<Vec<u8>, &'static str> {
        if s.len() % 4 != 0 {
            return Err("Invalid base64 length");
        }
        
        let mut result = Vec::new();
        for chunk in s.as_bytes().chunks(4) {
            let mut buf = [0u8; 4];
            for (i, &b) in chunk.iter().enumerate() {
                buf[i] = match b {
                    b'A'..=b'Z' => b - b'A',
                    b'a'..=b'z' => b - b'a' + 26,
                    b'0'..=b'9' => b - b'0' + 52,
                    b'+' => 62,
                    b'/' => 63,
                    b'=' => 0,
                    _ => return Err("Invalid base64 character"),
                };
            }
            
            let combined = (buf[0] as u32) << 18 | (buf[1] as u32) << 12 | (buf[2] as u32) << 6 | (buf[3] as u32);
            result.push((combined >> 16) as u8);
            if chunk[2] != b'=' {
                result.push((combined >> 8) as u8);
            }
            if chunk[3] != b'=' {
                result.push(combined as u8);
            }
        }
        Ok(result)
    }
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let data = CustomData {
        timestamp: 1710515400,
        coordinates: Coordinates {
            latitude: 37.7749,
            longitude: -122.4194,
        },
        encoded_data: vec![0x48, 0x65, 0x6C, 0x6C, 0x6F, 0x20, 0x52, 0x75, 0x73, 0x74],
        flags: BitFlags(0x12345678),
        version: Version {
            major: 2,
            minor: 1,
            patch: 3,
        },
    };
    
    // Serialize with custom logic
    let json = serde_json::to_string_pretty(&data)?;
    println!("Custom serialized JSON:\n{}", json);
    
    // Deserialize from JSON
    let json_input = r#"{
        "timestamp": 1710515400,
        "location": "40.7128,-74.0060",
        "data": "SGVsbG8gV29ybGQ=",
        "flags": "0xABCDEF00",
        "version": "3.2.1"
    }"#;
    
    let parsed: CustomData = serde_json::from_str(json_input)?;
    println!("Parsed custom data: {:?}", parsed);
    
    Ok(())
}
```

Custom serializers enable specialized data representations like coordinate  
strings, hex-encoded flags, and semantic version strings. The Visitor  
pattern provides fine-grained control over deserialization with proper  
error handling and validation. Custom implementations can transform data  
formats while maintaining type safety and Serde integration.  

## Flattening and untagged enums

Use flattening to merge struct fields and untagged enums for flexible  
data structures. These features enable more natural JSON representations  
and support for polymorphic data without explicit type tags.  

```rust
use serde::{Deserialize, Serialize};
use std::collections::HashMap;

#[derive(Serialize, Deserialize, Debug)]
struct Event {
    id: String,
    timestamp: u64,
    #[serde(flatten)]
    metadata: EventMetadata,
    #[serde(flatten)]
    payload: EventPayload,
}

#[derive(Serialize, Deserialize, Debug)]
struct EventMetadata {
    source: String,
    version: String,
    trace_id: Option<String>,
    user_id: Option<u32>,
}

#[derive(Serialize, Deserialize, Debug)]
#[serde(untagged)]
enum EventPayload {
    UserAction {
        action: String,
        target: String,
        details: HashMap<String, String>,
    },
    SystemEvent {
        component: String,
        severity: String,
        message: String,
        error_code: Option<u32>,
    },
    MetricData {
        metric_name: String,
        value: f64,
        unit: String,
        tags: Vec<String>,
    },
    CustomData {
        data_type: String,
        content: serde_json::Value,
    },
}

#[derive(Serialize, Deserialize, Debug)]
struct ApiResponse {
    success: bool,
    #[serde(flatten)]
    result: ApiResult,
}

#[derive(Serialize, Deserialize, Debug)]
#[serde(untagged)]
enum ApiResult {
    Success {
        data: serde_json::Value,
        metadata: Option<ResponseMetadata>,
    },
    Error {
        error: ErrorInfo,
        request_id: String,
    },
    Partial {
        partial_data: serde_json::Value,
        warnings: Vec<String>,
        next_page: Option<String>,
    },
}

#[derive(Serialize, Deserialize, Debug)]
struct ResponseMetadata {
    total_count: u32,
    page_size: u32,
    has_more: bool,
}

#[derive(Serialize, Deserialize, Debug)]
struct ErrorInfo {
    code: u32,
    message: String,
    details: Option<HashMap<String, String>>,
}

#[derive(Serialize, Deserialize, Debug)]
struct Config {
    #[serde(flatten)]
    app_config: AppConfig,
    #[serde(flatten)]
    env_vars: HashMap<String, String>,
    services: Vec<ServiceConfig>,
}

#[derive(Serialize, Deserialize, Debug)]
struct AppConfig {
    name: String,
    version: String,
    debug: bool,
}

#[derive(Serialize, Deserialize, Debug)]
struct ServiceConfig {
    name: String,
    #[serde(flatten)]
    connection: ConnectionType,
}

#[derive(Serialize, Deserialize, Debug)]
#[serde(untagged)]
enum ConnectionType {
    Http {
        url: String,
        timeout: u32,
        retries: u32,
    },
    Database {
        host: String,
        port: u16,
        database: String,
        pool_size: u32,
    },
    MessageQueue {
        broker_url: String,
        queue_name: String,
        prefetch_count: u32,
    },
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Event with flattened metadata and untagged payload
    let user_event = Event {
        id: "evt_12345".to_string(),
        timestamp: 1710515400,
        metadata: EventMetadata {
            source: "web_app".to_string(),
            version: "2.1.0".to_string(),
            trace_id: Some("trace_abc123".to_string()),
            user_id: Some(98765),
        },
        payload: EventPayload::UserAction {
            action: "login".to_string(),
            target: "dashboard".to_string(),
            details: {
                let mut details = HashMap::new();
                details.insert("ip_address".to_string(), "192.168.1.100".to_string());
                details.insert("user_agent".to_string(), "Mozilla/5.0".to_string());
                details
            },
        },
    };
    
    let json = serde_json::to_string_pretty(&user_event)?;
    println!("Flattened event JSON:\n{}", json);
    
    // System event with different payload structure
    let system_event = Event {
        id: "evt_67890".to_string(),
        timestamp: 1710515500,
        metadata: EventMetadata {
            source: "monitoring".to_string(),
            version: "1.0.0".to_string(),
            trace_id: None,
            user_id: None,
        },
        payload: EventPayload::SystemEvent {
            component: "database".to_string(),
            severity: "warning".to_string(),
            message: "Connection pool near capacity".to_string(),
            error_code: Some(1001),
        },
    };
    
    let system_json = serde_json::to_string_pretty(&system_event)?;
    println!("\nSystem event JSON:\n{}", system_json);
    
    // API Response with flattened result
    let success_response = ApiResponse {
        success: true,
        result: ApiResult::Success {
            data: serde_json::json!({
                "users": [
                    {"id": 1, "name": "Alice"},
                    {"id": 2, "name": "Bob"}
                ]
            }),
            metadata: Some(ResponseMetadata {
                total_count: 2,
                page_size: 10,
                has_more: false,
            }),
        },
    };
    
    let response_json = serde_json::to_string_pretty(&success_response)?;
    println!("\nAPI response JSON:\n{}", response_json);
    
    // Config with flattened app config and environment variables
    let config = Config {
        app_config: AppConfig {
            name: "MyApp".to_string(),
            version: "1.2.3".to_string(),
            debug: false,
        },
        env_vars: {
            let mut env = HashMap::new();
            env.insert("LOG_LEVEL".to_string(), "info".to_string());
            env.insert("MAX_CONNECTIONS".to_string(), "100".to_string());
            env.insert("TIMEOUT".to_string(), "30".to_string());
            env
        },
        services: vec![
            ServiceConfig {
                name: "database".to_string(),
                connection: ConnectionType::Database {
                    host: "db.example.com".to_string(),
                    port: 5432,
                    database: "myapp".to_string(),
                    pool_size: 20,
                },
            },
            ServiceConfig {
                name: "api_gateway".to_string(),
                connection: ConnectionType::Http {
                    url: "https://api.example.com".to_string(),
                    timeout: 30,
                    retries: 3,
                },
            },
        ],
    };
    
    let config_json = serde_json::to_string_pretty(&config)?;
    println!("\nFlattened config JSON:\n{}", config_json);
    
    // Parse flattened JSON back
    let parsed_json = r#"{
        "id": "evt_test",
        "timestamp": 1710515600,
        "source": "test_app",
        "version": "1.0.0",
        "trace_id": null,
        "user_id": null,
        "metric_name": "response_time",
        "value": 245.5,
        "unit": "ms",
        "tags": ["api", "production", "web"]
    }"#;
    
    let parsed_event: Event = serde_json::from_str(parsed_json)?;
    println!("\nParsed flattened event: {:?}", parsed_event);
    
    Ok(())
}
```

Flattening merges nested struct fields into the parent structure,  
creating cleaner JSON representations without unnecessary nesting.  
Untagged enums enable polymorphic serialization based on structure  
rather than explicit type tags, supporting flexible data schemas  
and external API integration with varying response formats.  

## Borrowing and zero-copy deserialization

Leverage borrowing for efficient deserialization without data copying.  
Zero-copy deserialization improves performance when working with large  
datasets by referencing original string data instead of allocating new memory.  

```rust
use serde::{Deserialize, Serialize};
use std::borrow::Cow;

// Zero-copy struct with borrowed string slices
#[derive(Deserialize, Debug)]
struct LogEntry<'a> {
    #[serde(borrow)]
    timestamp: &'a str,
    #[serde(borrow)]
    level: &'a str,
    #[serde(borrow)]
    message: &'a str,
    #[serde(borrow)]
    module: &'a str,
    line_number: u32,
    #[serde(borrow)]
    thread_id: Option<&'a str>,
}

// Cow (Clone on Write) for flexible borrowing
#[derive(Serialize, Deserialize, Debug)]
struct FlexibleData<'a> {
    #[serde(borrow)]
    id: Cow<'a, str>,
    #[serde(borrow)]
    name: Cow<'a, str>,
    age: u32,
    #[serde(borrow)]
    tags: Vec<Cow<'a, str>>,
    #[serde(borrow)]
    metadata: std::collections::HashMap<Cow<'a, str>, Cow<'a, str>>,
}

// Borrowed bytes for binary data
#[derive(Deserialize, Debug)]
struct BinaryData<'a> {
    #[serde(borrow)]
    header: &'a [u8],
    #[serde(borrow)]
    payload: &'a [u8],
    checksum: u32,
}

// Mixed borrowing and owned data
#[derive(Deserialize, Debug)]
struct MixedData<'a> {
    #[serde(borrow)]
    static_fields: StaticFields<'a>,
    dynamic_fields: DynamicFields,
    computed_values: Vec<f64>,
}

#[derive(Deserialize, Debug)]
struct StaticFields<'a> {
    #[serde(borrow)]
    category: &'a str,
    #[serde(borrow)]
    subcategory: &'a str,
    #[serde(borrow)]
    tags: Vec<&'a str>,
}

#[derive(Deserialize, Debug)]
struct DynamicFields {
    created_at: String,
    updated_at: String,
    processing_time_ms: u64,
}

// Custom deserializer for borrowed slices
#[derive(Debug)]
struct BorrowedSlice<'a> {
    data: &'a [u8],
}

impl<'de> Deserialize<'de> for BorrowedSlice<'de> {
    fn deserialize<D>(deserializer: D) -> Result<Self, D::Error>
    where
        D: serde::Deserializer<'de>,
    {
        struct BorrowedSliceVisitor;
        
        impl<'de> serde::de::Visitor<'de> for BorrowedSliceVisitor {
            type Value = BorrowedSlice<'de>;
            
            fn expecting(&self, formatter: &mut std::fmt::Formatter) -> std::fmt::Result {
                formatter.write_str("a borrowed byte slice")
            }
            
            fn visit_borrowed_bytes<E>(self, v: &'de [u8]) -> Result<Self::Value, E>
            where
                E: serde::de::Error,
            {
                Ok(BorrowedSlice { data: v })
            }
        }
        
        deserializer.deserialize_bytes(BorrowedSliceVisitor)
    }
}

// Streaming processor for large datasets
fn process_log_entries(json_data: &str) -> Result<Vec<LogEntry>, Box<dyn std::error::Error>> {
    // Split by lines and parse each entry
    let mut entries = Vec::new();
    
    for line in json_data.lines() {
        if line.trim().is_empty() {
            continue;
        }
        
        let entry: LogEntry = serde_json::from_str(line)?;
        entries.push(entry);
    }
    
    Ok(entries)
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Zero-copy log parsing
    let log_data = r#"{"timestamp": "2024-03-15T10:30:00Z", "level": "INFO", "message": "Application started", "module": "main", "line_number": 42, "thread_id": "main-thread"}
{"timestamp": "2024-03-15T10:30:01Z", "level": "DEBUG", "message": "Loading configuration", "module": "config", "line_number": 156, "thread_id": "worker-1"}
{"timestamp": "2024-03-15T10:30:02Z", "level": "WARN", "message": "Connection retry", "module": "database", "line_number": 89, "thread_id": "db-pool"}"#;
    
    let log_entries = process_log_entries(log_data)?;
    println!("Parsed {} log entries without copying strings:", log_entries.len());
    for entry in &log_entries {
        println!("  {} [{}] {}", entry.timestamp, entry.level, entry.message);
    }
    
    // Flexible data with Cow
    let flexible_json = r#"{
        "id": "user_12345",
        "name": "Alice Johnson",
        "age": 30,
        "tags": ["developer", "rust", "senior"],
        "metadata": {
            "department": "engineering",
            "location": "remote",
            "start_date": "2020-01-15"
        }
    }"#;
    
    let flexible_data: FlexibleData = serde_json::from_str(flexible_json)?;
    println!("\nFlexible data with Cow:");
    println!("  ID: {:?} (borrowed: {})", flexible_data.id, flexible_data.id.is_borrowed());
    println!("  Name: {:?} (borrowed: {})", flexible_data.name, flexible_data.name.is_borrowed());
    println!("  Tags: {} items", flexible_data.tags.len());
    
    // Mixed borrowed and owned data
    let mixed_json = r#"{
        "static_fields": {
            "category": "sensor_data",
            "subcategory": "temperature",
            "tags": ["indoor", "celsius", "hourly"]
        },
        "dynamic_fields": {
            "created_at": "2024-03-15T10:30:00Z",
            "updated_at": "2024-03-15T10:35:00Z", 
            "processing_time_ms": 125
        },
        "computed_values": [20.5, 21.0, 20.8, 21.2, 20.9]
    }"#;
    
    let mixed_data: MixedData = serde_json::from_str(mixed_json)?;
    println!("\nMixed data structure:");
    println!("  Category: {} (borrowed)", mixed_data.static_fields.category);
    println!("  Created: {} (owned)", mixed_data.dynamic_fields.created_at);
    println!("  Values: {} computed values", mixed_data.computed_values.len());
    
    // Performance comparison example
    let large_json = r#"{"id": "performance_test_with_very_long_identifier_string", "name": "Performance Test Object With Very Long Name String", "age": 25, "tags": ["performance", "benchmark", "test", "very_long_tag_name"], "metadata": {"description": "This is a very long description string for performance testing", "category": "benchmark_category_with_long_name"}}"#;
    
    // Measure zero-copy parsing
    let start = std::time::Instant::now();
    for _ in 0..1000 {
        let _: FlexibleData = serde_json::from_str(large_json)?;
    }
    let zero_copy_time = start.elapsed();
    
    println!("\nPerformance test (1000 iterations):");
    println!("  Zero-copy parsing: {:?}", zero_copy_time);
    
    Ok(())
}
```

Zero-copy deserialization uses `&str` and `&[u8]` references to avoid  
string allocation during parsing. The `#[serde(borrow)]` attribute enables  
borrowing from the input data. Cow<str> provides flexibility between  
borrowed and owned data based on parsing context. This approach  
significantly improves performance for large datasets and streaming scenarios.  

## Streaming large datasets

Process large datasets incrementally using streaming deserialization.  
Streaming enables memory-efficient processing of data that doesn't fit  
in memory by processing one record at a time instead of loading everything.  

```rust
use serde::{Deserialize, Serialize};
use std::io::{BufRead, BufReader, Write};

#[derive(Serialize, Deserialize, Debug)]
struct DataRecord {
    id: u64,
    timestamp: String,
    sensor_id: String,
    value: f64,
    quality: String,
    metadata: std::collections::HashMap<String, String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct ProcessingResult {
    total_records: usize,
    valid_records: usize,
    invalid_records: usize,
    min_value: f64,
    max_value: f64,
    average_value: f64,
    processing_time_ms: u64,
}

// Streaming JSON Lines processor
struct StreamProcessor {
    valid_count: usize,
    invalid_count: usize,
    sum: f64,
    min_value: f64,
    max_value: f64,
    start_time: std::time::Instant,
}

impl StreamProcessor {
    fn new() -> Self {
        Self {
            valid_count: 0,
            invalid_count: 0,
            sum: 0.0,
            min_value: f64::INFINITY,
            max_value: f64::NEG_INFINITY,
            start_time: std::time::Instant::now(),
        }
    }
    
    fn process_line(&mut self, line: &str) -> Result<(), Box<dyn std::error::Error>> {
        match serde_json::from_str::<DataRecord>(line) {
            Ok(record) => {
                self.valid_count += 1;
                self.sum += record.value;
                self.min_value = self.min_value.min(record.value);
                self.max_value = self.max_value.max(record.value);
                
                // Optional: Process record immediately
                if self.valid_count % 1000 == 0 {
                    println!("Processed {} valid records", self.valid_count);
                }
            }
            Err(_) => {
                self.invalid_count += 1;
            }
        }
        Ok(())
    }
    
    fn finalize(self) -> ProcessingResult {
        let total = self.valid_count + self.invalid_count;
        let average = if self.valid_count > 0 {
            self.sum / self.valid_count as f64
        } else {
            0.0
        };
        
        ProcessingResult {
            total_records: total,
            valid_records: self.valid_count,
            invalid_records: self.invalid_count,
            min_value: if self.min_value.is_finite() { self.min_value } else { 0.0 },
            max_value: if self.max_value.is_finite() { self.max_value } else { 0.0 },
            average_value: average,
            processing_time_ms: self.start_time.elapsed().as_millis() as u64,
        }
    }
}

// Streaming JSON array processor using serde_json::Deserializer
fn process_json_array_stream<R: BufRead>(
    reader: R,
) -> Result<ProcessingResult, Box<dyn std::error::Error>> {
    let mut processor = StreamProcessor::new();
    let mut de = serde_json::Deserializer::from_reader(reader);
    
    // Expect array start
    use serde::de::{SeqAccess, Visitor};
    
    struct RecordArrayVisitor<'a> {
        processor: &'a mut StreamProcessor,
    }
    
    impl<'de, 'a> Visitor<'de> for RecordArrayVisitor<'a> {
        type Value = ();
        
        fn expecting(&self, formatter: &mut std::fmt::Formatter) -> std::fmt::Result {
            formatter.write_str("an array of records")
        }
        
        fn visit_seq<A>(self, mut seq: A) -> Result<Self::Value, A::Error>
        where
            A: SeqAccess<'de>,
        {
            while let Some(record) = seq.next_element::<DataRecord>()? {
                self.processor.valid_count += 1;
                self.processor.sum += record.value;
                self.processor.min_value = self.processor.min_value.min(record.value);
                self.processor.max_value = self.processor.max_value.max(record.value);
                
                if self.processor.valid_count % 500 == 0 {
                    println!("Streamed {} records", self.processor.valid_count);
                }
            }
            Ok(())
        }
    }
    
    de.deserialize_seq(RecordArrayVisitor {
        processor: &mut processor,
    })?;
    
    Ok(processor.finalize())
}

// Batch processing with controlled memory usage
fn process_in_batches<R: BufRead>(
    reader: R,
    batch_size: usize,
) -> Result<ProcessingResult, Box<dyn std::error::Error>> {
    let mut processor = StreamProcessor::new();
    let mut batch = Vec::with_capacity(batch_size);
    
    for line_result in reader.lines() {
        let line = line_result?;
        if line.trim().is_empty() {
            continue;
        }
        
        batch.push(line);
        
        if batch.len() >= batch_size {
            process_batch(&mut processor, &batch)?;
            batch.clear();
        }
    }
    
    // Process remaining records
    if !batch.is_empty() {
        process_batch(&mut processor, &batch)?;
    }
    
    Ok(processor.finalize())
}

fn process_batch(
    processor: &mut StreamProcessor,
    batch: &[String],
) -> Result<(), Box<dyn std::error::Error>> {
    for line in batch {
        processor.process_line(line)?;
    }
    Ok(())
}

// Generate sample streaming data
fn generate_sample_data() -> String {
    let mut data = String::new();
    
    for i in 1..=5000 {
        let record = DataRecord {
            id: i,
            timestamp: format!("2024-03-15T{:02}:{:02}:{:02}Z", 
                              (i / 3600) % 24, (i / 60) % 60, i % 60),
            sensor_id: format!("sensor_{:03}", (i % 100) + 1),
            value: 20.0 + (i as f64 * 0.1) % 10.0 + 
                   ((i as f64 * 0.017).sin() * 5.0),
            quality: if i % 100 != 0 { "good".to_string() } else { "fair".to_string() },
            metadata: {
                let mut meta = std::collections::HashMap::new();
                meta.insert("location".to_string(), format!("zone_{}", (i % 10) + 1));
                meta.insert("calibrated".to_string(), "true".to_string());
                meta
            },
        };
        
        data.push_str(&serde_json::to_string(&record).unwrap());
        data.push('\n');
    }
    
    data
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    println!("Generating sample data...");
    let sample_data = generate_sample_data();
    println!("Generated {} bytes of sample data", sample_data.len());
    
    // Stream processing line by line
    println!("\n=== Line-by-line streaming ===");
    let cursor = std::io::Cursor::new(&sample_data);
    let reader = BufReader::new(cursor);
    let mut processor = StreamProcessor::new();
    
    for line_result in reader.lines() {
        let line = line_result?;
        if !line.trim().is_empty() {
            processor.process_line(&line)?;
        }
    }
    
    let result = processor.finalize();
    println!("Streaming result: {:?}", result);
    
    // Batch processing
    println!("\n=== Batch processing (batches of 1000) ===");
    let cursor = std::io::Cursor::new(&sample_data);
    let reader = BufReader::new(cursor);
    let batch_result = process_in_batches(reader, 1000)?;
    println!("Batch result: {:?}", batch_result);
    
    // Memory usage demonstration
    println!("\n=== Memory usage comparison ===");
    
    // Simulate large dataset processing
    let start_memory = get_memory_usage();
    
    // All-at-once approach (memory intensive)
    let all_lines: Vec<String> = sample_data.lines().map(|s| s.to_string()).collect();
    let after_load_memory = get_memory_usage();
    
    drop(all_lines);
    let after_drop_memory = get_memory_usage();
    
    println!("Memory usage:");
    println!("  Start: {} KB", start_memory);
    println!("  After loading all data: {} KB (+{} KB)", 
             after_load_memory, after_load_memory - start_memory);
    println!("  After dropping: {} KB", after_drop_memory);
    
    // Streaming approach uses minimal additional memory
    let cursor = std::io::Cursor::new(&sample_data);
    let reader = BufReader::new(cursor);
    let stream_start = get_memory_usage();
    let _stream_result = process_in_batches(reader, 100)?;
    let stream_end = get_memory_usage();
    
    println!("  Streaming processing: {} KB -> {} KB (+{} KB)",
             stream_start, stream_end, stream_end - stream_start);
    
    Ok(())
}

fn get_memory_usage() -> usize {
    // Simplified memory usage estimation
    // In real applications, use proper memory profiling tools
    std::hint::black_box(std::process::id() as usize * 1024)
}
```

Streaming processing enables handling datasets larger than available memory  
by processing records incrementally. Line-by-line processing provides the  
lowest memory footprint, while batch processing balances memory usage  
with processing efficiency. The streaming approach scales to arbitrarily  
large datasets without memory limitations.  

## Error handling and validation

Implement comprehensive error handling and validation for serialization.  
Robust error handling ensures data integrity and provides meaningful  
feedback when deserialization fails or data validation errors occur.  

```rust
use serde::{Deserialize, Serialize, Deserializer};
use serde::de::{self, Visitor};
use std::fmt;

#[derive(Debug)]
enum ValidationError {
    InvalidEmail(String),
    InvalidPhoneNumber(String),
    AgeOutOfRange(u32),
    InvalidDateFormat(String),
    MissingRequiredField(String),
    InvalidEnumValue(String),
    CustomValidation(String),
}

impl fmt::Display for ValidationError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            ValidationError::InvalidEmail(email) => write!(f, "Invalid email format: {}", email),
            ValidationError::InvalidPhoneNumber(phone) => write!(f, "Invalid phone number: {}", phone),
            ValidationError::AgeOutOfRange(age) => write!(f, "Age {} is out of valid range (0-150)", age),
            ValidationError::InvalidDateFormat(date) => write!(f, "Invalid date format: {}", date),
            ValidationError::MissingRequiredField(field) => write!(f, "Missing required field: {}", field),
            ValidationError::InvalidEnumValue(value) => write!(f, "Invalid enum value: {}", value),
            ValidationError::CustomValidation(msg) => write!(f, "Validation error: {}", msg),
        }
    }
}

impl std::error::Error for ValidationError {}

// Validated email type
#[derive(Debug, Clone)]
struct Email(String);

impl Email {
    fn new(email: String) -> Result<Self, ValidationError> {
        if email.contains('@') && email.contains('.') && email.len() > 5 {
            Ok(Email(email))
        } else {
            Err(ValidationError::InvalidEmail(email))
        }
    }
    
    fn as_str(&self) -> &str {
        &self.0
    }
}

impl Serialize for Email {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: serde::Serializer,
    {
        serializer.serialize_str(&self.0)
    }
}

impl<'de> Deserialize<'de> for Email {
    fn deserialize<D>(deserializer: D) -> Result<Self, D::Error>
    where
        D: Deserializer<'de>,
    {
        struct EmailVisitor;
        
        impl<'de> Visitor<'de> for EmailVisitor {
            type Value = Email;
            
            fn expecting(&self, formatter: &mut fmt::Formatter) -> fmt::Result {
                formatter.write_str("a valid email address")
            }
            
            fn visit_str<E>(self, value: &str) -> Result<Email, E>
            where
                E: de::Error,
            {
                Email::new(value.to_string()).map_err(|e| E::custom(e.to_string()))
            }
        }
        
        deserializer.deserialize_str(EmailVisitor)
    }
}

// Validated age type
#[derive(Debug, Clone)]
struct Age(u32);

impl Age {
    fn new(age: u32) -> Result<Self, ValidationError> {
        if age <= 150 {
            Ok(Age(age))
        } else {
            Err(ValidationError::AgeOutOfRange(age))
        }
    }
}

impl Serialize for Age {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: serde::Serializer,
    {
        serializer.serialize_u32(self.0)
    }
}

impl<'de> Deserialize<'de> for Age {
    fn deserialize<D>(deserializer: D) -> Result<Self, D::Error>
    where
        D: Deserializer<'de>,
    {
        struct AgeVisitor;
        
        impl<'de> Visitor<'de> for AgeVisitor {
            type Value = Age;
            
            fn expecting(&self, formatter: &mut fmt::Formatter) -> fmt::Result {
                formatter.write_str("an age between 0 and 150")
            }
            
            fn visit_u32<E>(self, value: u32) -> Result<Age, E>
            where
                E: de::Error,
            {
                Age::new(value).map_err(|e| E::custom(e.to_string()))
            }
            
            fn visit_u64<E>(self, value: u64) -> Result<Age, E>
            where
                E: de::Error,
            {
                if value <= u32::MAX as u64 {
                    self.visit_u32(value as u32)
                } else {
                    Err(E::custom("age value too large"))
                }
            }
        }
        
        deserializer.deserialize_u32(AgeVisitor)
    }
}

// Validated phone number
#[derive(Debug, Clone)]
struct PhoneNumber(String);

impl PhoneNumber {
    fn new(phone: String) -> Result<Self, ValidationError> {
        let cleaned = phone.chars().filter(|c| c.is_ascii_digit()).collect::<String>();
        if cleaned.len() >= 10 && cleaned.len() <= 15 {
            Ok(PhoneNumber(phone))
        } else {
            Err(ValidationError::InvalidPhoneNumber(phone))
        }
    }
}

impl Serialize for PhoneNumber {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: serde::Serializer,
    {
        serializer.serialize_str(&self.0)
    }
}

impl<'de> Deserialize<'de> for PhoneNumber {
    fn deserialize<D>(deserializer: D) -> Result<Self, D::Error>
    where
        D: Deserializer<'de>,
    {
        struct PhoneVisitor;
        
        impl<'de> Visitor<'de> for PhoneVisitor {
            type Value = PhoneNumber;
            
            fn expecting(&self, formatter: &mut fmt::Formatter) -> fmt::Result {
                formatter.write_str("a valid phone number")
            }
            
            fn visit_str<E>(self, value: &str) -> Result<PhoneNumber, E>
            where
                E: de::Error,
            {
                PhoneNumber::new(value.to_string()).map_err(|e| E::custom(e.to_string()))
            }
        }
        
        deserializer.deserialize_str(PhoneVisitor)
    }
}

// Main validated struct
#[derive(Serialize, Deserialize, Debug)]
struct ValidatedUser {
    id: u32,
    name: String,
    email: Email,
    age: Age,
    phone: Option<PhoneNumber>,
    #[serde(default)]
    preferences: UserPreferences,
    #[serde(deserialize_with = "validate_tags")]
    tags: Vec<String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct UserPreferences {
    #[serde(default = "default_language")]
    language: String,
    #[serde(default)]
    notifications: bool,
    #[serde(default = "default_theme")]
    theme: String,
}

impl Default for UserPreferences {
    fn default() -> Self {
        Self {
            language: default_language(),
            notifications: false,
            theme: default_theme(),
        }
    }
}

fn default_language() -> String {
    "en".to_string()
}

fn default_theme() -> String {
    "light".to_string()
}

fn validate_tags<'de, D>(deserializer: D) -> Result<Vec<String>, D::Error>
where
    D: Deserializer<'de>,
{
    let tags: Vec<String> = Vec::deserialize(deserializer)?;
    
    for tag in &tags {
        if tag.is_empty() {
            return Err(de::Error::custom("tags cannot be empty"));
        }
        if tag.len() > 50 {
            return Err(de::Error::custom("tags cannot exceed 50 characters"));
        }
    }
    
    if tags.len() > 10 {
        return Err(de::Error::custom("maximum 10 tags allowed"));
    }
    
    Ok(tags)
}

// Error handling utilities
fn parse_user_with_detailed_errors(json: &str) -> Result<ValidatedUser, Vec<String>> {
    match serde_json::from_str::<ValidatedUser>(json) {
        Ok(user) => Ok(user),
        Err(e) => {
            let mut errors = Vec::new();
            
            // Parse the error message to provide detailed feedback
            let error_msg = e.to_string();
            
            if error_msg.contains("invalid type") {
                errors.push("Type mismatch in one or more fields".to_string());
            }
            
            if error_msg.contains("missing field") {
                errors.push("One or more required fields are missing".to_string());
            }
            
            if error_msg.contains("email") {
                errors.push("Invalid email format".to_string());
            }
            
            if error_msg.contains("age") {
                errors.push("Age must be between 0 and 150".to_string());
            }
            
            if error_msg.contains("phone") {
                errors.push("Invalid phone number format".to_string());
            }
            
            if error_msg.contains("tags") {
                errors.push("Invalid tags (check length and count limits)".to_string());
            }
            
            if errors.is_empty() {
                errors.push(format!("Unknown parsing error: {}", error_msg));
            }
            
            Err(errors)
        }
    }
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Valid user data
    let valid_json = r#"{
        "id": 1,
        "name": "Alice Johnson",
        "email": "alice@example.com",
        "age": 30,
        "phone": "+1-555-123-4567",
        "preferences": {
            "language": "en",
            "notifications": true,
            "theme": "dark"
        },
        "tags": ["developer", "rust", "senior"]
    }"#;
    
    match parse_user_with_detailed_errors(valid_json) {
        Ok(user) => {
            println!("Successfully parsed valid user: {:?}", user);
        }
        Err(errors) => {
            println!("Validation errors: {:?}", errors);
        }
    }
    
    // Test various invalid data scenarios
    let test_cases = vec![
        (
            "Invalid email",
            r#"{"id": 1, "name": "Test", "email": "invalid-email", "age": 25, "tags": []}"#,
        ),
        (
            "Age out of range",
            r#"{"id": 1, "name": "Test", "email": "test@example.com", "age": 200, "tags": []}"#,
        ),
        (
            "Invalid phone number",
            r#"{"id": 1, "name": "Test", "email": "test@example.com", "age": 25, "phone": "123", "tags": []}"#,
        ),
        (
            "Too many tags",
            r#"{"id": 1, "name": "Test", "email": "test@example.com", "age": 25, "tags": ["a", "b", "c", "d", "e", "f", "g", "h", "i", "j", "k"]}"#,
        ),
        (
            "Empty tag",
            r#"{"id": 1, "name": "Test", "email": "test@example.com", "age": 25, "tags": ["valid", ""]}"#,
        ),
        (
            "Missing required field",
            r#"{"name": "Test", "email": "test@example.com", "age": 25, "tags": []}"#,
        ),
    ];
    
    println!("\n=== Testing validation errors ===");
    for (description, json) in test_cases {
        println!("\nTesting: {}", description);
        match parse_user_with_detailed_errors(json) {
            Ok(_) => println!("  Unexpected success!"),
            Err(errors) => {
                println!("  Validation errors:");
                for error in errors {
                    println!("    - {}", error);
                }
            }
        }
    }
    
    // Demonstrate partial parsing with default values
    let partial_json = r#"{"id": 2, "name": "Bob", "email": "bob@test.com", "age": 35, "tags": ["manager"]}"#;
    
    match serde_json::from_str::<ValidatedUser>(partial_json) {
        Ok(user) => {
            println!("\nPartial user with defaults: {:?}", user);
            println!("Default language: {}", user.preferences.language);
            println!("Default notifications: {}", user.preferences.notifications);
        }
        Err(e) => println!("Error parsing partial user: {}", e),
    }
    
    Ok(())
}
```

Validation during deserialization ensures data integrity at parse time.  
Custom types with validation logic provide compile-time safety guarantees.  
Detailed error messages help identify specific validation failures.  
Default value support enables robust parsing of incomplete data while  
maintaining required field validation for critical information.  

## Binary formats with bincode

Serialize to compact binary format using bincode for efficient storage  
and network transmission. Binary serialization provides significant  
space savings and faster parsing compared to text formats.  

```rust
// Add to Cargo.toml: bincode = "1.3"
use serde::{Deserialize, Serialize};
use std::collections::HashMap;

#[derive(Serialize, Deserialize, Debug, PartialEq)]
struct BinaryData {
    header: Header,
    payload: Payload,
    checksum: u32,
}

#[derive(Serialize, Deserialize, Debug, PartialEq)]
struct Header {
    version: u16,
    message_type: MessageType,
    timestamp: u64,
    flags: u32,
    source_id: u32,
    sequence_number: u64,
}

#[derive(Serialize, Deserialize, Debug, PartialEq)]
enum MessageType {
    Heartbeat,
    Data,
    Control,
    Error,
    Custom(u16),
}

#[derive(Serialize, Deserialize, Debug, PartialEq)]
enum Payload {
    Heartbeat {
        status: String,
        uptime_seconds: u64,
    },
    SensorData {
        sensors: Vec<SensorReading>,
        location: GeoLocation,
        metadata: HashMap<String, String>,
    },
    ControlMessage {
        command: String,
        parameters: Vec<Parameter>,
        target_devices: Vec<u32>,
    },
    ErrorReport {
        error_code: u32,
        message: String,
        stack_trace: Option<String>,
        context: HashMap<String, String>,
    },
}

#[derive(Serialize, Deserialize, Debug, PartialEq)]
struct SensorReading {
    sensor_id: u16,
    value: f64,
    quality: QualityIndicator,
    timestamp: u64,
}

#[derive(Serialize, Deserialize, Debug, PartialEq)]
enum QualityIndicator {
    Good,
    Uncertain,
    Bad,
    Calibration,
}

#[derive(Serialize, Deserialize, Debug, PartialEq)]
struct GeoLocation {
    latitude: f64,
    longitude: f64,
    altitude: Option<f32>,
    accuracy: f32,
}

#[derive(Serialize, Deserialize, Debug, PartialEq)]
struct Parameter {
    name: String,
    value: ParameterValue,
}

#[derive(Serialize, Deserialize, Debug, PartialEq)]
enum ParameterValue {
    String(String),
    Integer(i64),
    Float(f64),
    Boolean(bool),
    Binary(Vec<u8>),
}

// Binary protocol utilities
struct BinaryProtocol;

impl BinaryProtocol {
    fn serialize_with_compression(data: &BinaryData) -> Result<Vec<u8>, Box<dyn std::error::Error>> {
        let serialized = bincode::serialize(data)?;
        
        // Simple compression simulation (in practice use flate2 or similar)
        let compressed = Self::simple_compress(&serialized);
        
        // Add compression header
        let mut result = Vec::new();
        result.extend_from_slice(&(compressed.len() as u32).to_le_bytes());
        result.extend_from_slice(&(serialized.len() as u32).to_le_bytes());
        result.extend_from_slice(&compressed);
        
        Ok(result)
    }
    
    fn deserialize_with_decompression(data: &[u8]) -> Result<BinaryData, Box<dyn std::error::Error>> {
        if data.len() < 8 {
            return Err("Invalid compressed data header".into());
        }
        
        let compressed_len = u32::from_le_bytes([data[0], data[1], data[2], data[3]]) as usize;
        let original_len = u32::from_le_bytes([data[4], data[5], data[6], data[7]]) as usize;
        
        if data.len() < 8 + compressed_len {
            return Err("Incomplete compressed data".into());
        }
        
        let compressed_data = &data[8..8 + compressed_len];
        let decompressed = Self::simple_decompress(compressed_data, original_len)?;
        let deserialized = bincode::deserialize(&decompressed)?;
        
        Ok(deserialized)
    }
    
    fn simple_compress(data: &[u8]) -> Vec<u8> {
        // Placeholder compression - use run-length encoding for demonstration
        let mut compressed = Vec::new();
        let mut i = 0;
        
        while i < data.len() {
            let current_byte = data[i];
            let mut count = 1u8;
            
            while i + count as usize < data.len() && 
                  data[i + count as usize] == current_byte && 
                  count < 255 {
                count += 1;
            }
            
            compressed.push(count);
            compressed.push(current_byte);
            i += count as usize;
        }
        
        compressed
    }
    
    fn simple_decompress(data: &[u8], expected_len: usize) -> Result<Vec<u8>, Box<dyn std::error::Error>> {
        let mut decompressed = Vec::with_capacity(expected_len);
        let mut i = 0;
        
        while i + 1 < data.len() {
            let count = data[i];
            let byte = data[i + 1];
            
            for _ in 0..count {
                decompressed.push(byte);
            }
            
            i += 2;
        }
        
        if decompressed.len() != expected_len {
            return Err("Decompression size mismatch".into());
        }
        
        Ok(decompressed)
    }
    
    fn calculate_checksum(data: &[u8]) -> u32 {
        data.iter().fold(0u32, |acc, &byte| acc.wrapping_add(byte as u32))
    }
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Create sample binary data
    let binary_data = BinaryData {
        header: Header {
            version: 1,
            message_type: MessageType::Data,
            timestamp: 1710515400,
            flags: 0x12345678,
            source_id: 42,
            sequence_number: 1001,
        },
        payload: Payload::SensorData {
            sensors: vec![
                SensorReading {
                    sensor_id: 1,
                    value: 23.5,
                    quality: QualityIndicator::Good,
                    timestamp: 1710515400,
                },
                SensorReading {
                    sensor_id: 2,
                    value: 45.2,
                    quality: QualityIndicator::Uncertain,
                    timestamp: 1710515401,
                },
                SensorReading {
                    sensor_id: 3,
                    value: 67.8,
                    quality: QualityIndicator::Good,
                    timestamp: 1710515402,
                },
            ],
            location: GeoLocation {
                latitude: 37.7749,
                longitude: -122.4194,
                altitude: Some(100.0),
                accuracy: 5.0,
            },
            metadata: {
                let mut meta = HashMap::new();
                meta.insert("device_id".to_string(), "sensor_array_01".to_string());
                meta.insert("firmware_version".to_string(), "2.1.3".to_string());
                meta.insert("calibration_date".to_string(), "2024-01-15".to_string());
                meta
            },
        },
        checksum: 0, // Will be calculated
    };
    
    // Serialize to binary
    let serialized = bincode::serialize(&binary_data)?;
    println!("Binary serialization:");
    println!("  Size: {} bytes", serialized.len());
    println!("  First 20 bytes: {:?}", &serialized[..20.min(serialized.len())]);
    
    // Compare with JSON size
    let json_data = serde_json::to_string(&binary_data)?;
    println!("  JSON size: {} bytes", json_data.len());
    println!("  Binary vs JSON: {:.1}% size", 
             (serialized.len() as f64 / json_data.len() as f64) * 100.0);
    
    // Deserialize from binary
    let deserialized: BinaryData = bincode::deserialize(&serialized)?;
    println!("  Round-trip successful: {}", binary_data == deserialized);
    
    // Test compressed binary protocol
    println!("\nCompressed binary protocol:");
    let compressed = BinaryProtocol::serialize_with_compression(&binary_data)?;
    println!("  Compressed size: {} bytes", compressed.len());
    println!("  Compression ratio: {:.1}%", 
             (compressed.len() as f64 / serialized.len() as f64) * 100.0);
    
    let decompressed = BinaryProtocol::deserialize_with_decompression(&compressed)?;
    println!("  Decompression successful: {}", binary_data == decompressed);
    
    // Performance comparison
    println!("\nPerformance comparison (1000 iterations):");
    
    // Binary serialization
    let start = std::time::Instant::now();
    for _ in 0..1000 {
        let _serialized = bincode::serialize(&binary_data)?;
    }
    let binary_serialize_time = start.elapsed();
    
    // JSON serialization
    let start = std::time::Instant::now();
    for _ in 0..1000 {
        let _serialized = serde_json::to_string(&binary_data)?;
    }
    let json_serialize_time = start.elapsed();
    
    // Binary deserialization
    let start = std::time::Instant::now();
    for _ in 0..1000 {
        let _: BinaryData = bincode::deserialize(&serialized)?;
    }
    let binary_deserialize_time = start.elapsed();
    
    // JSON deserialization
    let start = std::time::Instant::now();
    for _ in 0..1000 {
        let _: BinaryData = serde_json::from_str(&json_data)?;
    }
    let json_deserialize_time = start.elapsed();
    
    println!("  Binary serialize: {:?}", binary_serialize_time);
    println!("  JSON serialize: {:?}", json_serialize_time);
    println!("  Binary deserialize: {:?}", binary_deserialize_time);
    println!("  JSON deserialize: {:?}", json_deserialize_time);
    
    // Test different message types
    println!("\nTesting different message types:");
    
    let messages = vec![
        BinaryData {
            header: Header {
                version: 1,
                message_type: MessageType::Heartbeat,
                timestamp: 1710515500,
                flags: 0,
                source_id: 1,
                sequence_number: 1002,
            },
            payload: Payload::Heartbeat {
                status: "healthy".to_string(),
                uptime_seconds: 86400,
            },
            checksum: 0,
        },
        BinaryData {
            header: Header {
                version: 1,
                message_type: MessageType::Control,
                timestamp: 1710515600,
                flags: 0x1,
                source_id: 5,
                sequence_number: 1003,
            },
            payload: Payload::ControlMessage {
                command: "calibrate".to_string(),
                parameters: vec![
                    Parameter {
                        name: "sensor_id".to_string(),
                        value: ParameterValue::Integer(42),
                    },
                    Parameter {
                        name: "reference_value".to_string(),
                        value: ParameterValue::Float(23.5),
                    },
                    Parameter {
                        name: "auto_adjust".to_string(),
                        value: ParameterValue::Boolean(true),
                    },
                ],
                target_devices: vec![1, 2, 3],
            },
            checksum: 0,
        },
    ];
    
    for (i, message) in messages.iter().enumerate() {
        let serialized = bincode::serialize(message)?;
        let deserialized: BinaryData = bincode::deserialize(&serialized)?;
        println!("  Message {}: {} bytes, round-trip OK: {}", 
                 i + 1, serialized.len(), message == &deserialized);
    }
    
    Ok(())
}
```

Bincode provides compact binary serialization with automatic handling  
of complex data structures including enums and nested types. Binary  
formats offer significant size and performance advantages over text  
formats, making them ideal for network protocols, file storage,  
and high-performance applications requiring efficient data exchange.  

## URL-encoded data

Serialize data as URL-encoded query parameters for web APIs and forms.  
URL encoding enables structured data transmission through HTTP query  
strings and form submissions with proper escaping and formatting.  

```rust
// Add to Cargo.toml: serde_urlencoded = "0.7"
use serde::{Deserialize, Serialize};
use std::collections::HashMap;

#[derive(Serialize, Deserialize, Debug, PartialEq)]
struct SearchQuery {
    q: String,
    category: Option<String>,
    sort: SortOrder,
    limit: u32,
    offset: u32,
    filters: Vec<String>,
    #[serde(flatten)]
    advanced: AdvancedOptions,
}

#[derive(Serialize, Deserialize, Debug, PartialEq)]
#[serde(rename_all = "lowercase")]
enum SortOrder {
    Relevance,
    Date,
    Price,
    Rating,
    Name,
}

#[derive(Serialize, Deserialize, Debug, PartialEq)]
struct AdvancedOptions {
    include_archived: bool,
    min_price: Option<f64>,
    max_price: Option<f64>,
    location: Option<String>,
    radius_km: Option<u32>,
}

#[derive(Serialize, Deserialize, Debug, PartialEq)]
struct FormData {
    username: String,
    email: String,
    age: u32,
    newsletter: bool,
    interests: Vec<String>,
    contact_method: ContactMethod,
    #[serde(skip_serializing_if = "Option::is_none")]
    referrer: Option<String>,
    #[serde(skip_serializing_if = "HashMap::is_empty")]
    custom_fields: HashMap<String, String>,
}

#[derive(Serialize, Deserialize, Debug, PartialEq)]
#[serde(rename_all = "snake_case")]
enum ContactMethod {
    Email,
    Phone,
    Mail,
    DoNotContact,
}

#[derive(Serialize, Deserialize, Debug, PartialEq)]
struct ApiRequest {
    action: String,
    #[serde(flatten)]
    params: RequestParams,
    #[serde(default)]
    debug: bool,
}

#[derive(Serialize, Deserialize, Debug, PartialEq)]
#[serde(untagged)]
enum RequestParams {
    UserQuery {
        user_id: u32,
        include_profile: bool,
        include_history: bool,
    },
    SearchQuery {
        search_term: String,
        page: u32,
        per_page: u32,
        facets: Vec<String>,
    },
    BulkOperation {
        operation: String,
        target_ids: Vec<u32>,
        options: HashMap<String, String>,
    },
}

// URL encoding utilities
struct UrlEncoder;

impl UrlEncoder {
    fn encode_with_arrays<T: Serialize>(data: &T) -> Result<String, Box<dyn std::error::Error>> {
        let encoded = serde_urlencoded::to_string(data)?;
        Ok(encoded)
    }
    
    fn decode_with_arrays<T: for<'de> Deserialize<'de>>(input: &str) -> Result<T, Box<dyn std::error::Error>> {
        let decoded = serde_urlencoded::from_str(input)?;
        Ok(decoded)
    }
    
    fn build_query_string(
        base_url: &str, 
        params: &HashMap<String, String>
    ) -> String {
        if params.is_empty() {
            return base_url.to_string();
        }
        
        let query_string: Vec<String> = params
            .iter()
            .map(|(k, v)| format!("{}={}", 
                                  Self::url_encode(k), 
                                  Self::url_encode(v)))
            .collect();
        
        format!("{}?{}", base_url, query_string.join("&"))
    }
    
    fn url_encode(input: &str) -> String {
        input
            .chars()
            .map(|c| match c {
                'A'..='Z' | 'a'..='z' | '0'..='9' | '-' | '_' | '.' | '~' => c.to_string(),
                ' ' => "+".to_string(),
                _ => format!("%{:02X}", c as u8),
            })
            .collect()
    }
    
    fn parse_query_string(url: &str) -> HashMap<String, String> {
        let mut params = HashMap::new();
        
        if let Some(query_start) = url.find('?') {
            let query = &url[query_start + 1..];
            
            for pair in query.split('&') {
                if let Some(eq_pos) = pair.find('=') {
                    let key = Self::url_decode(&pair[..eq_pos]);
                    let value = Self::url_decode(&pair[eq_pos + 1..]);
                    params.insert(key, value);
                }
            }
        }
        
        params
    }
    
    fn url_decode(input: &str) -> String {
        let mut result = String::new();
        let mut chars = input.chars().peekable();
        
        while let Some(c) = chars.next() {
            match c {
                '+' => result.push(' '),
                '%' => {
                    if let (Some(h1), Some(h2)) = (chars.next(), chars.next()) {
                        if let (Some(d1), Some(d2)) = (h1.to_digit(16), h2.to_digit(16)) {
                            result.push((d1 * 16 + d2) as u8 as char);
                        }
                    }
                },
                _ => result.push(c),
            }
        }
        
        result
    }
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Search query example
    let search_query = SearchQuery {
        q: "rust programming".to_string(),
        category: Some("books".to_string()),
        sort: SortOrder::Relevance,
        limit: 20,
        offset: 0,
        filters: vec!["available".to_string(), "new".to_string()],
        advanced: AdvancedOptions {
            include_archived: false,
            min_price: Some(10.0),
            max_price: Some(100.0),
            location: Some("San Francisco".to_string()),
            radius_km: Some(50),
        },
    };
    
    let query_string = UrlEncoder::encode_with_arrays(&search_query)?;
    println!("Search query URL-encoded:");
    println!("  {}", query_string);
    
    let decoded_query: SearchQuery = UrlEncoder::decode_with_arrays(&query_string)?;
    println!("  Round-trip successful: {}", search_query == decoded_query);
    
    // Form data example
    let form_data = FormData {
        username: "alice_dev".to_string(),
        email: "alice@example.com".to_string(),
        age: 28,
        newsletter: true,
        interests: vec!["programming".to_string(), "rust".to_string(), "web dev".to_string()],
        contact_method: ContactMethod::Email,
        referrer: Some("https://google.com".to_string()),
        custom_fields: {
            let mut fields = HashMap::new();
            fields.insert("company".to_string(), "Tech Corp".to_string());
            fields.insert("experience".to_string(), "5 years".to_string());
            fields
        },
    };
    
    let form_encoded = UrlEncoder::encode_with_arrays(&form_data)?;
    println!("\nForm data URL-encoded:");
    println!("  {}", form_encoded);
    
    let decoded_form: FormData = UrlEncoder::decode_with_arrays(&form_encoded)?;
    println!("  Round-trip successful: {}", form_data == decoded_form);
    
    // API request with flattened parameters
    let api_request = ApiRequest {
        action: "search_users".to_string(),
        params: RequestParams::SearchQuery {
            search_term: "john doe".to_string(),
            page: 1,
            per_page: 25,
            facets: vec!["department".to_string(), "location".to_string()],
        },
        debug: false,
    };
    
    let api_encoded = UrlEncoder::encode_with_arrays(&api_request)?;
    println!("\nAPI request URL-encoded:");
    println!("  {}", api_encoded);
    
    // Build complete URLs
    let base_url = "https://api.example.com/search";
    let manual_params = {
        let mut params = HashMap::new();
        params.insert("q".to_string(), "rust tutorials".to_string());
        params.insert("type".to_string(), "video".to_string());
        params.insert("duration".to_string(), "10-30 min".to_string());
        params
    };
    
    let complete_url = UrlEncoder::build_query_string(base_url, &manual_params);
    println!("\nComplete URL with manual parameters:");
    println!("  {}", complete_url);
    
    // Parse URL back to parameters
    let parsed_params = UrlEncoder::parse_query_string(&complete_url);
    println!("  Parsed parameters: {:?}", parsed_params);
    
    // Demonstrate special character handling
    let special_form = FormData {
        username: "user@domain.com".to_string(),
        email: "test+tag@example.org".to_string(),
        age: 25,
        newsletter: false,
        interests: vec!["C++".to_string(), "C#".to_string(), "F#".to_string()],
        contact_method: ContactMethod::DoNotContact,
        referrer: Some("https://example.com/path?param=value&other=test".to_string()),
        custom_fields: HashMap::new(),
    };
    
    let special_encoded = UrlEncoder::encode_with_arrays(&special_form)?;
    println!("\nSpecial characters URL-encoded:");
    println!("  {}", special_encoded);
    
    let special_decoded: FormData = UrlEncoder::decode_with_arrays(&special_encoded)?;
    println!("  Special chars preserved: {}", special_form == special_decoded);
    
    // Performance comparison with JSON
    println!("\nSize comparison:");
    let json_size = serde_json::to_string(&form_data)?.len();
    let url_size = form_encoded.len();
    println!("  JSON: {} bytes", json_size);
    println!("  URL-encoded: {} bytes", url_size);
    println!("  Ratio: {:.1}%", (url_size as f64 / json_size as f64) * 100.0);
    
    Ok(())
}
```

URL encoding enables structured data transmission through HTTP query  
parameters and form submissions. The format handles arrays, nested  
objects through flattening, and special character escaping. URL-encoded  
data is widely supported by web frameworks and APIs, making it ideal  
for RESTful API parameters and HTML form processing.  

## Custom text format

Implement a custom text-based serialization format for domain-specific  
requirements. Custom formats enable optimized representations for  
specific use cases while maintaining human readability and simplicity.  

```rust
use serde::{Deserialize, Serialize, Serializer, Deserializer};
use serde::de::{self, Visitor};
use std::fmt;
use std::collections::HashMap;

// Custom configuration format: KEY=VALUE with sections
#[derive(Serialize, Deserialize, Debug, PartialEq)]
struct ConfigFile {
    sections: HashMap<String, ConfigSection>,
}

#[derive(Serialize, Deserialize, Debug, PartialEq)]
struct ConfigSection {
    values: HashMap<String, ConfigValue>,
    #[serde(skip_serializing_if = "Option::is_none")]
    comment: Option<String>,
}

#[derive(Debug, PartialEq, Clone)]
enum ConfigValue {
    String(String),
    Integer(i64),
    Float(f64),
    Boolean(bool),
    List(Vec<String>),
}

// Custom serialization for ConfigValue
impl Serialize for ConfigValue {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        match self {
            ConfigValue::String(s) => serializer.serialize_str(s),
            ConfigValue::Integer(i) => serializer.serialize_i64(*i),
            ConfigValue::Float(f) => serializer.serialize_f64(*f),
            ConfigValue::Boolean(b) => serializer.serialize_bool(*b),
            ConfigValue::List(list) => {
                let joined = list.join(",");
                serializer.serialize_str(&joined)
            }
        }
    }
}

impl<'de> Deserialize<'de> for ConfigValue {
    fn deserialize<D>(deserializer: D) -> Result<Self, D::Error>
    where
        D: Deserializer<'de>,
    {
        struct ConfigValueVisitor;
        
        impl<'de> Visitor<'de> for ConfigValueVisitor {
            type Value = ConfigValue;
            
            fn expecting(&self, formatter: &mut fmt::Formatter) -> fmt::Result {
                formatter.write_str("a configuration value")
            }
            
            fn visit_str<E>(self, value: &str) -> Result<ConfigValue, E>
            where
                E: de::Error,
            {
                // Try to parse as different types
                if let Ok(int_val) = value.parse::<i64>() {
                    return Ok(ConfigValue::Integer(int_val));
                }
                
                if let Ok(float_val) = value.parse::<f64>() {
                    return Ok(ConfigValue::Float(float_val));
                }
                
                if let Ok(bool_val) = value.parse::<bool>() {
                    return Ok(ConfigValue::Boolean(bool_val));
                }
                
                // Check if it's a comma-separated list
                if value.contains(',') {
                    let list: Vec<String> = value
                        .split(',')
                        .map(|s| s.trim().to_string())
                        .collect();
                    return Ok(ConfigValue::List(list));
                }
                
                Ok(ConfigValue::String(value.to_string()))
            }
        }
        
        deserializer.deserialize_str(ConfigValueVisitor)
    }
}

// Custom text format serializer
struct CustomTextFormat;

impl CustomTextFormat {
    fn serialize(config: &ConfigFile) -> String {
        let mut output = String::new();
        
        for (section_name, section) in &config.sections {
            // Add section header
            output.push_str(&format!("[{}]\n", section_name));
            
            // Add section comment if present
            if let Some(comment) = &section.comment {
                output.push_str(&format!("# {}\n", comment));
            }
            
            // Add key-value pairs
            for (key, value) in &section.values {
                let value_str = match value {
                    ConfigValue::String(s) => {
                        if s.contains(' ') || s.contains('\n') {
                            format!("\"{}\"", s.replace('"', "\\\""))
                        } else {
                            s.clone()
                        }
                    }
                    ConfigValue::Integer(i) => i.to_string(),
                    ConfigValue::Float(f) => f.to_string(),
                    ConfigValue::Boolean(b) => b.to_string(),
                    ConfigValue::List(list) => list.join(", "),
                };
                
                output.push_str(&format!("{}={}\n", key, value_str));
            }
            
            output.push('\n');
        }
        
        output
    }
    
    fn deserialize(input: &str) -> Result<ConfigFile, Box<dyn std::error::Error>> {
        let mut sections = HashMap::new();
        let mut current_section = String::new();
        let mut current_values = HashMap::new();
        let mut current_comment = None;
        
        for line in input.lines() {
            let line = line.trim();
            
            // Skip empty lines
            if line.is_empty() {
                continue;
            }
            
            // Handle comments
            if line.starts_with('#') {
                current_comment = Some(line[1..].trim().to_string());
                continue;
            }
            
            // Handle section headers
            if line.starts_with('[') && line.ends_with(']') {
                // Save previous section
                if !current_section.is_empty() {
                    sections.insert(current_section.clone(), ConfigSection {
                        values: current_values.clone(),
                        comment: current_comment.clone(),
                    });
                }
                
                // Start new section
                current_section = line[1..line.len()-1].to_string();
                current_values.clear();
                current_comment = None;
                continue;
            }
            
            // Handle key-value pairs
            if let Some(eq_pos) = line.find('=') {
                let key = line[..eq_pos].trim().to_string();
                let value_str = line[eq_pos + 1..].trim();
                
                // Parse value
                let value = Self::parse_value(value_str)?;
                current_values.insert(key, value);
            }
        }
        
        // Save final section
        if !current_section.is_empty() {
            sections.insert(current_section, ConfigSection {
                values: current_values,
                comment: current_comment,
            });
        }
        
        Ok(ConfigFile { sections })
    }
    
    fn parse_value(value_str: &str) -> Result<ConfigValue, Box<dyn std::error::Error>> {
        let value_str = value_str.trim();
        
        // Handle quoted strings
        if value_str.starts_with('"') && value_str.ends_with('"') {
            let unquoted = &value_str[1..value_str.len()-1];
            let unescaped = unquoted.replace("\\\"", "\"");
            return Ok(ConfigValue::String(unescaped));
        }
        
        // Try to parse as integer
        if let Ok(int_val) = value_str.parse::<i64>() {
            return Ok(ConfigValue::Integer(int_val));
        }
        
        // Try to parse as float
        if let Ok(float_val) = value_str.parse::<f64>() {
            return Ok(ConfigValue::Float(float_val));
        }
        
        // Try to parse as boolean
        if let Ok(bool_val) = value_str.parse::<bool>() {
            return Ok(ConfigValue::Boolean(bool_val));
        }
        
        // Check for comma-separated list
        if value_str.contains(", ") {
            let list: Vec<String> = value_str
                .split(", ")
                .map(|s| s.trim().to_string())
                .collect();
            return Ok(ConfigValue::List(list));
        }
        
        // Default to string
        Ok(ConfigValue::String(value_str.to_string()))
    }
}

// Protocol buffer style format
#[derive(Debug, PartialEq)]
struct Message {
    fields: HashMap<u32, Field>,
}

#[derive(Debug, PartialEq, Clone)]
enum Field {
    VarInt(u64),
    Fixed64(u64),
    LengthDelimited(Vec<u8>),
    Fixed32(u32),
}

struct ProtoBufFormat;

impl ProtoBufFormat {
    fn serialize_message(msg: &Message) -> Vec<u8> {
        let mut output = Vec::new();
        
        for (&field_number, field) in &msg.fields {
            match field {
                Field::VarInt(value) => {
                    Self::write_tag(&mut output, field_number, 0); // VarInt wire type
                    Self::write_varint(&mut output, *value);
                }
                Field::Fixed64(value) => {
                    Self::write_tag(&mut output, field_number, 1); // Fixed64 wire type
                    output.extend_from_slice(&value.to_le_bytes());
                }
                Field::LengthDelimited(data) => {
                    Self::write_tag(&mut output, field_number, 2); // Length-delimited wire type
                    Self::write_varint(&mut output, data.len() as u64);
                    output.extend_from_slice(data);
                }
                Field::Fixed32(value) => {
                    Self::write_tag(&mut output, field_number, 5); // Fixed32 wire type
                    output.extend_from_slice(&value.to_le_bytes());
                }
            }
        }
        
        output
    }
    
    fn deserialize_message(data: &[u8]) -> Result<Message, Box<dyn std::error::Error>> {
        let mut fields = HashMap::new();
        let mut cursor = 0;
        
        while cursor < data.len() {
            let (tag, new_cursor) = Self::read_varint(data, cursor)?;
            cursor = new_cursor;
            
            let field_number = (tag >> 3) as u32;
            let wire_type = tag & 0x7;
            
            match wire_type {
                0 => { // VarInt
                    let (value, new_cursor) = Self::read_varint(data, cursor)?;
                    cursor = new_cursor;
                    fields.insert(field_number, Field::VarInt(value));
                }
                1 => { // Fixed64
                    if cursor + 8 > data.len() {
                        return Err("Insufficient data for Fixed64".into());
                    }
                    let value = u64::from_le_bytes([
                        data[cursor], data[cursor+1], data[cursor+2], data[cursor+3],
                        data[cursor+4], data[cursor+5], data[cursor+6], data[cursor+7]
                    ]);
                    cursor += 8;
                    fields.insert(field_number, Field::Fixed64(value));
                }
                2 => { // Length-delimited
                    let (length, new_cursor) = Self::read_varint(data, cursor)?;
                    cursor = new_cursor;
                    let length = length as usize;
                    
                    if cursor + length > data.len() {
                        return Err("Insufficient data for length-delimited field".into());
                    }
                    
                    let field_data = data[cursor..cursor + length].to_vec();
                    cursor += length;
                    fields.insert(field_number, Field::LengthDelimited(field_data));
                }
                5 => { // Fixed32
                    if cursor + 4 > data.len() {
                        return Err("Insufficient data for Fixed32".into());
                    }
                    let value = u32::from_le_bytes([
                        data[cursor], data[cursor+1], data[cursor+2], data[cursor+3]
                    ]);
                    cursor += 4;
                    fields.insert(field_number, Field::Fixed32(value));
                }
                _ => return Err(format!("Unknown wire type: {}", wire_type).into()),
            }
        }
        
        Ok(Message { fields })
    }
    
    fn write_tag(output: &mut Vec<u8>, field_number: u32, wire_type: u32) {
        let tag = (field_number << 3) | wire_type;
        Self::write_varint(output, tag as u64);
    }
    
    fn write_varint(output: &mut Vec<u8>, mut value: u64) {
        while value >= 0x80 {
            output.push(((value & 0x7F) | 0x80) as u8);
            value >>= 7;
        }
        output.push(value as u8);
    }
    
    fn read_varint(data: &[u8], mut cursor: usize) -> Result<(u64, usize), Box<dyn std::error::Error>> {
        let mut result = 0u64;
        let mut shift = 0;
        
        loop {
            if cursor >= data.len() {
                return Err("Incomplete varint".into());
            }
            
            let byte = data[cursor];
            cursor += 1;
            
            result |= ((byte & 0x7F) as u64) << shift;
            
            if byte & 0x80 == 0 {
                break;
            }
            
            shift += 7;
            if shift >= 64 {
                return Err("Varint overflow".into());
            }
        }
        
        Ok((result, cursor))
    }
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Custom configuration format example
    let config = ConfigFile {
        sections: {
            let mut sections = HashMap::new();
            
            sections.insert("database".to_string(), ConfigSection {
                comment: Some("Database configuration".to_string()),
                values: {
                    let mut values = HashMap::new();
                    values.insert("host".to_string(), ConfigValue::String("localhost".to_string()));
                    values.insert("port".to_string(), ConfigValue::Integer(5432));
                    values.insert("database".to_string(), ConfigValue::String("myapp".to_string()));
                    values.insert("ssl_enabled".to_string(), ConfigValue::Boolean(true));
                    values.insert("connection_timeout".to_string(), ConfigValue::Float(30.5));
                    values
                },
            });
            
            sections.insert("features".to_string(), ConfigSection {
                comment: Some("Feature flags and settings".to_string()),
                values: {
                    let mut values = HashMap::new();
                    values.insert("enabled_features".to_string(), 
                                ConfigValue::List(vec!["auth".to_string(), "logging".to_string(), "metrics".to_string()]));
                    values.insert("beta_features".to_string(), 
                                ConfigValue::List(vec!["new_ui".to_string(), "experimental_api".to_string()]));
                    values.insert("debug_mode".to_string(), ConfigValue::Boolean(false));
                    values
                },
            });
            
            sections
        },
    };
    
    // Serialize to custom format
    let custom_text = CustomTextFormat::serialize(&config);
    println!("Custom configuration format:");
    println!("{}", custom_text);
    
    // Deserialize from custom format
    let parsed_config = CustomTextFormat::deserialize(&custom_text)?;
    println!("Round-trip successful: {}", config == parsed_config);
    
    // Protocol buffer style format
    let proto_message = Message {
        fields: {
            let mut fields = HashMap::new();
            fields.insert(1, Field::VarInt(150)); // user_id
            fields.insert(2, Field::LengthDelimited("Alice Johnson".as_bytes().to_vec())); // name
            fields.insert(3, Field::Fixed32(30)); // age
            fields.insert(4, Field::Fixed64(1710515400)); // timestamp
            fields.insert(5, Field::LengthDelimited("alice@example.com".as_bytes().to_vec())); // email
            fields
        },
    };
    
    println!("\nProtocol buffer style format:");
    let proto_binary = ProtoBufFormat::serialize_message(&proto_message);
    println!("Serialized size: {} bytes", proto_binary.len());
    println!("Binary data: {:?}", &proto_binary[..20.min(proto_binary.len())]);
    
    let parsed_proto = ProtoBufFormat::deserialize_message(&proto_binary)?;
    println!("Proto round-trip successful: {}", proto_message == parsed_proto);
    
    // Display parsed protocol buffer fields
    println!("Parsed protocol buffer fields:");
    for (field_num, field) in &parsed_proto.fields {
        match field {
            Field::VarInt(val) => println!("  Field {}: VarInt({})", field_num, val),
            Field::Fixed64(val) => println!("  Field {}: Fixed64({})", field_num, val),
            Field::LengthDelimited(data) => {
                if let Ok(s) = String::from_utf8(data.clone()) {
                    println!("  Field {}: String(\"{}\")", field_num, s);
                } else {
                    println!("  Field {}: Binary({} bytes)", field_num, data.len());
                }
            }
            Field::Fixed32(val) => println!("  Field {}: Fixed32({})", field_num, val),
        }
    }
    
    Ok(())
}
```

Custom text formats enable domain-specific optimizations and human-readable  
representations tailored to specific requirements. The configuration format  
demonstrates section-based organization with type inference, while the  
protocol buffer style format shows efficient binary encoding with field  
numbering and wire type discrimination for compact data transmission.  