# sparqling

Minimal async [SPARQL](https://www.w3.org/TR/sparql11-query/)-over-HTTP client for Rust.

Sends queries to a SPARQL endpoint over HTTP POST and parses the SPARQL 1.1
JSON results format. Point it at [Wikidata](https://query.wikidata.org/sparql),
a local [Oxigraph](https://github.com/oxigraph/oxigraph), or any other endpoint.

```toml
[dependencies]
sparqling = "0.1"
```

Requires Rust 1.75 or later; toolchains older than the newest dependencies
resolve with `CARGO_RESOLVER_INCOMPATIBLE_RUST_VERSIONS=fallback`.

The crate is published as `sparqling`; the import path is `sparql_client`:

```rust
use sparql_client::SparqlClient;

let client = SparqlClient::new("https://query.wikidata.org/sparql");
let rows = client
    .sparql_query("SELECT ?item WHERE { ?item wdt:P31 wd:Q5 } LIMIT 5")
    .await?;
```

## Features

- **SELECT** (`sparql_query`) and **ASK** (`sparql_ask`) queries.
- **Full response** — `query_response()` keeps `head.vars` and the raw
  bindings; `rows()` yields each row in projection order, and the response
  serializes back to the wire shape.
- **Typed rows** — `query_into::<T>()` deserializes each binding into your own
  struct, coercing `xsd:` datatypes to numbers/booleans.
- **Typed accessors** on `SparqlValue` (`as_i64`, `as_bool`, `is_uri`, …;
  `as_datetime` behind the `chrono` feature).
- **Configurable client** via `SparqlClient::builder()` — user agent, timeout,
  or a shared `reqwest::Client`.
- **Retries** with exponential backoff that honor `Retry-After`
  (`.max_retries(n)`).
- **Pacing** — `.min_interval(d)` keeps consecutive requests at least `d`
  apart.
- **Body cap** — `.max_body_bytes(n)` fails on an oversized response instead
  of buffering it.
- `escape_literal` for safely embedding strings in SPARQL literals.

Queries are sent in the request body, so long queries don't hit URL-length
limits. Many public endpoints (Wikidata especially) require a meaningful user
agent — set one with the builder or `with_user_agent`.

## License

Licensed under either of [Apache-2.0](LICENSE-APACHE) or [MIT](LICENSE-MIT) at
your option.
