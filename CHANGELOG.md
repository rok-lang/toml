# Changelog

All notable changes to this stone. Versions follow semver, independent of the Rok toolchain.

## 0.2.0 (unreleased)

Needs Rok 0.2.

### Changed

- `Value::Table`, `Value::as_table`, `read::parse`, `write::document` and derived `ToToml` /
  `FromToml` use `Dict`, which keeps insertion order.
- `Value::to_string` and `toml::to_string` are now `to_str`.
- `toml::encode` and `toml::item` are now `to_value`, which panics on `none` (no TOML form
  outside a table); `toml::decode(v, seg)` is now `from_value(v)`.
- `TomlError` fields are private: read them with `message()`, `line()`, `col()` and `path()`.

### Removed

- The `OrderedDict` and `OrderedSet` impls of `ToToml` / `FromToml`; the `Dict` and `Set` ones
  cover them.
- The derive helpers `Fields`, `pack`, `unpack`, `variant_of`, `payload`, `no_payload`,
  `unknown_variant`, `seg` and `put` are internal; derived code still reaches them.

## 0.1.0

First release.
