# karga-http
[![crates.io](https://img.shields.io/crates/v/karga_http.svg)](https://crates.io/crates/karga_http)

**karga-http** is the HTTP implementation for the [karga](https://github.com/karga-rs/karga) load testing framework.

It provides a simple and easy way to define and execute HTTP-based scenarios using `reqwest` under the hood.

---

## Installation

Add both `karga` and `karga-http` to your `Cargo.toml`:

```toml
[dependencies]
karga = "0.3"
karga-http = "0.1"
reqwest = "0.12"
```

---

## Example

A minimal example that performs GET requests to `http://localhost:3000` using `karga-http`:

```rust
use std::{sync::Arc, time::Duration};
use karga::{Scenario, Stage, StageExecutor};
use karga_http::{HttpAggregate, HttpReport, http_action};
use reqwest::{Client, Method, Request, Url};

#[tokio::main]
async fn main() {
    let client = Client::new();
    let request = Arc::new(Request::new(
        Method::GET,
        Url::parse("http://localhost:3000").unwrap(),
    ));

    let results: HttpAggregate = Scenario::builder()
        .name("Basic scenario")
        .action(move || http_action(client.clone(), request.clone()))
        .executor(
            StageExecutor::builder()
                .stages(vec![
                    Stage::new(Duration::ZERO, f64::MAX),
                    Stage::new(Duration::from_secs(1), f64::MAX),
                ])
                .build(),
        )
        .build()
        .run()
        .await
        .unwrap();

    let report = HttpReport::from(results);
    println!("{report:#?}");
}
```

---

## Features

### `http_action`

A helper function that wraps an HTTP request into a `karga` action. It measures latency, success, and bytes transferred, and feeds them into `HttpAggregate`.

### `Analysis pipeline`
Implements `HttpMetric`, `HttpAggregate` and `HttpReport`, providing the most useful data points for analysing http performance.

---

## License

Licensed under MIT. See [LICENSE](./LICENSE) for details.
