# elasticsearch-rs: Elasticsearch Rust Client

A spacetime project to build a low level Rust client.

The project is still very much a _work in progress_; input and contributions welcome!

## Design principles

1. Generate as much of the client as feasible from the REST API specs

    The REST API specs contain information about
    - the URL parts e.g. `{index}/{type}/_search` and variants
    - accepted HTTP methods e.g. `GET`, `POST`
    - the URL query string parameters
    - whether the API accepts a body
    
2. Prefer generation methods that produce ASTs and token streams over strings. 
The [`quote`](https://docs.rs/quote/1.0.2/quote/) and [`syn`](https://docs.rs/syn/1.0.5/syn/) crates can help

3. Get it working, then refine/refactor

    - Start simple and iterate
    - Design of the API is conducive to ease of use
    - synchronous functions first, asynchronous later
    - Control API invariants through arguments on API function
    
      e.g. `client.delete_script("script_id").send()?;`
      
      An id must always be provided for a script, so the `delete_script()` function must accept
      it as a value.

### Usage

The general usage of the client is envisioned as

```rust
let settings = ConnectionSettings::new();
let connection = Connection::new(Url::parse("http://localhost:9200").unwrap());
let client = ElasticsearchClient::new(settings, connection);

let cat_response = client.cat()
                         .indices()
                         .send()?;

let search_response = client.search()
                            .index("logstash-*")
                            .body(json!({
                                "query": {
                                    "match_all": {}
                                }                                           
                            }))
                            .pretty()
                            .send()?;
```

## Development

### Running MSVC debugger in VS Code

From [Bryce Van Dyk's blog post](https://www.brycevandyk.com/debug-rust-on-windows-with-visual-studio-code-and-the-msvc-debugger/), 
if wishing to use the MSVC debugger with Rust in VS code, which may be preferred on Windows

1. Install [C/C++ VS Code extensions](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)

2. Place the following in `.vscode/launch.json` in the project root

    ```json
    {
        "version": "0.2.0",
        "configurations": [   
            {
                "name": "Debug api_generator",
                "type": "cppvsdbg",
                "request": "launch",
                "program": "${workspaceFolder}/target/debug/api_generator.exe",
                "args": [],
                "stopAtEntry": false,
                "cwd": "${workspaceFolder}",
                "environment": [],
                "externalConsole": false
            }
        ]
    }
    ```
    
3. Add `"debug.allowBreakpointsEverywhere": true` to VS code settings.json
