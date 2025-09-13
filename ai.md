# AI



## Simple chat with genai

```rust
use genai::chat::ChatRequest;
use genai::Client;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Note: Set the DEEPSEEK_API_KEY environment variable before running
    // export DEEPSEEK_API_KEY=your_api_key_here

    // Create a new genai client
    let client = Client::default();

    let prompt = "What is NetBSD in a sentence?";
    // Create a chat request with a user message
    let request = ChatRequest::from_user(prompt);

    // Send the request to DeepSeek AI
    let response = client.exec_chat("deepseek-chat", request, None).await?;

    response.content.map_or_else(
    || println!("No content in response"),
    |content| {
        println!("Response: {:?}", content);
        match content.text_into_string() {
            Some(resp) => println!("{}", resp),
            None => println!("Failed to convert content to string"),
        }
    },
);

    // Print the response content
    // if let Some(content) = response.content {
    //     println!("Response: {:?}", content);

    //     if let Some(resp) = content.text_into_string() {
    //         println!("{}", resp);
    //     } else {
    //         println!("Failed to convert content to string");
    //     }   

    // } else {
    //     println!("No content in response");
    // }

    Ok(())
}
```

## Toll call with API

```rust
use reqwest::Client;
use serde::{Deserialize, Serialize};
use std::collections::HashMap;
use std::env;

#[derive(Serialize, Deserialize)]
struct ChatRequest {
    model: String,
    messages: Vec<Message>,
    #[serde(skip_serializing_if = "Option::is_none")]
    tools: Option<Vec<Tool>>,
}

#[derive(Serialize, Deserialize)]
struct Message {
    role: String,
    content: String,
    #[serde(skip_serializing_if = "Option::is_none")]
    tool_calls: Option<Vec<ToolCall>>,
}

#[derive(Serialize, Deserialize)]
struct ChatResponse {
    choices: Vec<Choice>,
}

#[derive(Serialize, Deserialize)]
struct Choice {
    message: Message,
}

#[derive(Serialize, Deserialize)]
struct Tool {
    #[serde(rename = "type")]
    tool_type: String,
    function: Function,
}

#[derive(Serialize, Deserialize)]
struct Function {
    name: String,
    #[serde(skip_serializing_if = "Option::is_none")]
    description: Option<String>,
    #[serde(skip_serializing_if = "Option::is_none")]
    parameters: Option<HashMap<String, serde_json::Value>>,
}

#[derive(Serialize, Deserialize)]
struct ToolCall {
    id: String,
    #[serde(rename = "type")]
    call_type: String,
    function: Function,
}

fn get_current_datetime() -> String {
    use chrono::Utc;
    Utc::now().format("%Y-%m-%d %H:%M:%S").to_string()
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let api_key = env::var("DEEPSEEK_API_KEY")
        .unwrap_or_else(|_| {
            println!("Please set the DEEPSEEK_API_KEY environment variable");
            std::process::exit(1);
        });

    let prompt = "Who was Napolen in one paragraph. What time is it.";
    let tool = Tool {
        tool_type: "function".to_string(),
        function: Function {
            name: "get_current_datetime".to_string(),
            description: Some("Get the current date and time".to_string()),
            parameters: Some({
                let mut params = HashMap::new();
                params.insert("type".to_string(), serde_json::Value::String("object".to_string()));
                params.insert("properties".to_string(), serde_json::Value::Object(serde_json::Map::new()));
                params
            }),
        },
    };

    let request = ChatRequest {
        model: "deepseek-chat".to_string(),
        messages: vec![Message {
            role: "user".to_string(),
            content: prompt.to_string(),
            tool_calls: None,
        }],
        tools: Some(vec![tool]),
    };

    let client = Client::new();
    let response = client
        .post("https://api.deepseek.com/v1/chat/completions")
        .header("Content-Type", "application/json")
        .header("Authorization", format!("Bearer {}", api_key))
        .json(&request)
        .send()
        .await?;

    let chat_resp: ChatResponse = response.json().await?;

    if let Some(choice) = chat_resp.choices.first() {
        let message = &choice.message;
        if !message.content.is_empty() {
            println!("Response: {}", message.content);
        }
        if let Some(tool_calls) = &message.tool_calls {
            for tool_call in tool_calls {
                if tool_call.function.name == "get_current_datetime" {
                    let result = get_current_datetime();
                    println!("Tool call result: {}", result);
                }
            }
        }
    } else {
        println!("No content in response");
    }

    println!("Request completed at: {}", chrono::Utc::now().format("%a, %d %b %Y %H:%M:%S GMT"));

    Ok(())
}
```


