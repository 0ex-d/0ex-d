---
title: "RPC Connection with Rust for ARB Stats"
summary: "A guide to connecting Rust with blockchain providers to retrieve Arbitrum statistics via RPC."
date: 2023-01-08
layout: default
---

## Introduction

Interacting with blockchain networks often requires efficient and reliable access to their Remote Procedure Call (RPC) interfaces. In this article, I’ll walk through how I used Rust to establish an RPC connection with a blockchain provider to fetch real-time Arbitrum (ARB) statistics.

---

## Prerequisites

Before getting started, ensure you have:

-   **Rust** installed (get it [here](https://www.rust-lang.org/tools/install)).
-   Familiarity with the basics of blockchain RPCs.
-   Access to an Arbitrum blockchain provider's RPC endpoint (e.g., Infura, Alchemy, or a self-hosted node).

Additionally, include the following dependencies in your `Cargo.toml` file:

```toml
[dependencies]
reqwest = "0.11"
tokio = { version = "1", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

Setting Up the RPC Client
Here’s how you can make an RPC call to an Arbitrum provider using the reqwest crate:

```rust
use reqwest::Client;
use serde_json::json;

#[tokio::main]
async fn main() {
    // Blockchain provider's RPC endpoint
    let rpc_url = "https://arb-mainnet.g.alchemy.com/v2/[your-api-key]";

    // JSON-RPC request payload
    let payload = json!({
        "jsonrpc": "2.0",
        "method": "eth_blockNumber",
        "params": [],
        "id": 1
    });

    // Create an HTTP client
    let client = Client::new();

    // Send the RPC request
    let response = client
        .post(rpc_url)
        .json(&payload)
        .send()
        .await
        .expect("Failed to send RPC request");

    // Parse the response
    if let Ok(body) = response.json::<serde_json::Value>().await {
        println!("Current block number: {}", body["result"]);
    } else {
        eprintln!("Failed to parse RPC response");
    }
}
```

This example fetches the latest block number from the Arbitrum network using the `eth_blockNumber` method.

Fetching ARB Statistics
You can retrieve more specific ARB stats, such as gas prices and account balances, by calling additional RPC methods. Below is an example of fetching the gas price:

Example: Fetching Gas Price

```rust
use reqwest::Client;
use serde_json::json;

#[tokio::main]
async fn main() {
    let rpc_url = "https://arb-mainnet.g.alchemy.com/v2/[your-api-key]";

    let payload = json!({
        "jsonrpc": "2.0",
        "method": "eth_gasPrice",
        "params": [],
        "id": 1
    });

    let client = Client::new();

    let response = client
        .post(rpc_url)
        .json(&payload)
        .send()
        .await
        .expect("Failed to send RPC request");

    if let Ok(body) = response.json::<serde_json::Value>().await {
        let gas_price_wei = body["result"]
            .as_str()
            .expect("Invalid gas price format");
        println!("Current gas price (in wei): {}", gas_price_wei);
    } else {
        eprintln!("Failed to fetch gas price");
    }
}
```

This snippet queries the `eth_gasPrice` method to get the current gas price in wei.

Combining Stats: Block Number and Gas Price
You can combine multiple RPC requests to fetch different statistics and process them together. For instance:

```rust
use reqwest::Client;
use serde_json::json;

#[tokio::main]
async fn main() {
    let rpc_url = "https://arb-mainnet.g.alchemy.com/v2/[your-api-key]";
    let client = Client::new();

    let block_number_payload = json!({
        "jsonrpc": "2.0",
        "method": "eth_blockNumber",
        "params": [],
        "id": 1
    });

    let gas_price_payload = json!({
        "jsonrpc": "2.0",
        "method": "eth_gasPrice",
        "params": [],
        "id": 2
    });

    let block_number_response = client
        .post(rpc_url)
        .json(&block_number_payload)
        .send()
        .await
        .expect("Failed to fetch block number");

    let gas_price_response = client
        .post(rpc_url)
        .json(&gas_price_payload)
        .send()
        .await
        .expect("Failed to fetch gas price");

    if let (Ok(block_body), Ok(gas_body)) = (
        block_number_response.json::<serde_json::Value>().await,
        gas_price_response.json::<serde_json::Value>().await,
    ) {
        let block_number = block_body["result"].as_str().unwrap();
        let gas_price = gas_body["result"].as_str().unwrap();
        println!("Current block number: {}", block_number);
        println!("Current gas price (in wei): {}", gas_price);
    } else {
        eprintln!("Failed to fetch one or more statistics");
    }
}
```

This program makes two simultaneous requests to fetch the block number and gas price.

Conclusion
Using Rust to interact with blockchain RPC endpoints is efficient and scalable for applications requiring real-time blockchain data. With crates like `reqwest`, `serde`, and `tokio`, you can easily fetch and process blockchain statistics such as block numbers, gas prices, or account balances.

For further exploration, consider extending this implementation to monitor ARB transactions, events, or smart contract interactions.
