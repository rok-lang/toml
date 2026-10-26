# toml

[TOML 1.1.0](https://toml.io/en/v1.1.0) for Rok: a `Value` tree with typed
date-times, a parser with `line:col` errors, a writer that lays documents out as
people write them, and typed encode / decode with `#[derive(Toml)]`. The API
follows std's `rokon` module: `parse` / `to_string` / `from_str` / `encode` /
`decode`, with `ToToml` / `FromToml` traits.

## Add it

```
rok add toml
```

## Usage

```rok
let doc = try toml::parse("title = \"demo\"\n\n[owner]\nname = \"Tom\"\n");
let owner = doc.get("owner") ?? toml::Value::Bool(false);
println(owner.get("name")?.as_str() ?? "");   // Tom
println(doc.to_string());                      // the document again
```

Derive `Toml` on your own types for typed `from_str` / `to_string`. `parse`
reads TOML 1.1.0 by default; pass `Version::V1_0` to read strictly as 1.0.

## Docs

https://docs.rok-lang.org

## License

Licensed under either of [MIT](LICENSE-MIT) or [Apache-2.0](LICENSE-APACHE), at
your option. The files under `tests/` are from toml-test, MIT License,
Copyright (c) 2018 TOML authors.
