# toml

[TOML 1.0.0](https://toml.io/en/v1.0.0) for rok: a `Value` tree with typed date-times, a parser with `line:col` errors, a writer that lays documents out as people write them, and typed encode / decode of your own types with `#[derive(Toml)]`. The API follows std's `rokon` module: `parse` / `to_string` / `from_str` / `encode` / `decode`, and `ToToml` / `FromToml` traits in place of `ToRokon` / `FromRokon`.

## Usage

```rokon
Package {
    deps: { "toml": "0.1" },
}
```

```rok
let doc = try toml::parse("title = \"demo\"\n\n[owner]\nname = \"Tom\"\nborn = 1979-05-27T07:32:00-08:00\n");
let owner = doc.get("owner") ?? toml::Value::Bool(false);
println(owner.get("name")?.as_str() ?? "");               // Tom
let born = owner.get("born")?.as_datetime();             // offset date-time
println(doc.to_string());                                // the document again
```

### Typed

```rok
#[derive(Toml)]
struct Server { host: str; port: u16 = 80; tls: bool | none; }

#[derive(Toml)]
enum Mode { Fast, Limit(u32), Custom { level: i32; } }

#[derive(Toml)]
struct Config { name: str; day: toml::Date; servers: List<Server>; mode: Mode; }

fn load(text: str) throws (toml::TomlError) -> Config {
    try toml::from_str(text)   // errors are "line:col: msg"
}

let cfg = try load("name = \"x\"\nday = 2024-01-02\nmode = { Limit = 3 }\n\n[[servers]]\nhost = \"a\"\n");
print(toml::to_string(cfg));
// name = "x"
// day = 2024-01-02
//
// [[servers]]
// host = "a"
// port = 80
//
// [mode]
// Limit = 3
```

| rok                    | TOML                                                          |
|------------------------|---------------------------------------------------------------|
| struct `S { a: .. }`   | table; a missing key takes `= default`, `none` for `T \| none`, else is an error; unknown keys are an error |
| tuple struct `S(a, b)` | array (one field: the field itself)                           |
| unit struct `S`        | `{}`                                                          |
| enum variant           | `"V"`, `{ V = x }`, `{ V = [a, b] }`, `{ V = { .. } }`        |
| `T \| none`            | the value, or no key at all (TOML has no null)                |
| integer types, `int`   | integer (range-checked; TOML integers are 64-bit)             |
| `f32`, `f64`, `float`  | float (an integer is taken too)                               |
| `bool`, `str`, `char`  | boolean, string, one-character string                         |
| `Datetime`, `Date`, `Time` | any date-time, local date, local time                     |
| `List`, `Set`, `OrderedSet`, `[T; N]`, tuples (1-6) | array                            |
| `Dict<K, V>`, `OrderedDict<K, V>` | table; keys are strings, integers or bools written as strings |
| `Value`                | any value                                                     |

A decode error carries a `path` (`.servers[0].port`, `."a b"`); `from_str` turns it into the `line:col` of that value (or of the key, for an unknown key).

## API

| Item | Description |
|------|-------------|
| `parse(s: str) throws (TomlError) -> Value` | Read a document into a `Value::Table` |
| `to_string<T: ToToml>(x: T) -> str` | A document; panics unless `x` is written as a table |
| `from_str<T: FromToml>(s: str) throws (TomlError) -> T` | Read a document as a `T` |
| `encode<T: ToToml>(x: T) -> Value \| none` / `decode<T: FromToml>(v: Value, seg: str) throws (TomlError) -> T` | Typed conversion of a `Value` |
| `Value` | `Str(str)` `Int(i64)` `Float(f64)` `Bool(bool)` `Datetime(Datetime)` `Array(List<Value>)` `Table(OrderedDict<str, Value>)` |
| `Value::to_string()` | A table as a document, other values inline; also `Display` |
| `Value::get(key)` / `at(i)` / `kind()` / `as_str()` / `as_int()` / `as_f64()` / `as_bool()` / `as_datetime()` / `as_array()` / `as_table()` | Inspection |
| `Datetime { date: Date \| none, time: Time \| none, offset: i16 \| none }` | Offset date-time, local date-time, local date or local time; `offset` in minutes (`Z` = 0). `kind()`, `Datetime::parse(s)`, `Display` (RFC 3339) |
| `Date { year, month, day }`, `Time { hour, minute, second, nanosecond }` | Parts |
| `TomlError { msg, line, col, path }` | `Display`: `line:col: msg`, or `at <path>: msg` without a position |

### Details

- Tables keep insertion order. Header, dotted-key and inline-table rules of TOML 1.0.0 are enforced: a table is defined once, inline tables and static arrays cannot be extended, dotted keys cannot add to a table defined by a header.
- Fractional seconds are kept to the nanosecond; further digits are truncated.
- Integers outside 64 bits and floats outside `f64` are errors. Floats are written with the shortest digits that read back to the same value, always with a `.` or an exponent (`1.0`, `1e300`), and `inf` / `-inf` / `nan`.
- Line endings in multi-line strings read as `\n`. A leading byte order mark is skipped.
- The writer puts a table's plain keys first, then each sub-table as a `[a.b]` section and each array of tables as `[[a]]` sections; a table holding only tables gets no header of its own. Arrays go on one line when they fit in 100 columns, else one item per line. Strings are written as basic strings. Comments and the original key order across sections are not kept.

## Conformance

[toml-test](https://github.com/toml-lang/toml-test) TOML 1.0.0 cases (`tests/valid.txt`, `tests/invalid.txt`): all 208 valid documents read to the expected values, and their written form reads back to the same values; all 501 invalid documents are rejected (invalid UTF-8 already when making the `str`).

## Development

```bash
rok test        # tests
rok fmt --check
```

## License

Licensed under either of [MIT](LICENSE-MIT) or [Apache-2.0](LICENSE-APACHE), at your option. The files under `tests/` are from toml-test, MIT License, Copyright (c) 2018 TOML authors.
