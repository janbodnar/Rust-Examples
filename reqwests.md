# Requests


## Async fetch title

```rust
use reqwest::get;
use scraper::{Html, Selector};

#[tokio::main]  // Add tokio runtime for async main
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // URL to fetch
    let url = "https://webcode.me";

    // Send GET request (await the async call)
    let response = get(url).await?.text().await?;

    // Parse the HTML
    let document = Html::parse_document(&response);
    let selector = Selector::parse("title").unwrap();

    // Extract and print the title
    if let Some(element) = document.select(&selector).next() {
        let title = element.text().collect::<Vec<_>>().join("");
        println!("Page title: {}", title);
    } else {
        println!("No title found");
    }

    Ok(())
}
```
