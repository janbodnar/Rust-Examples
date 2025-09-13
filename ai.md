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
