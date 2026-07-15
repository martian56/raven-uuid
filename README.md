# raven-uuid

[![CI](https://github.com/martian56/raven-uuid/actions/workflows/ci.yml/badge.svg)](https://github.com/martian56/raven-uuid/actions/workflows/ci.yml)

UUID generation and parsing for Raven, in pure Raven with no external dependencies.

Supported versions:

- **v4**: random
- **v7**: Unix-time ordered (sortable by creation time)
- **v1**: time plus a random node

Name-based **v3** (MD5) and **v5** (SHA-1) need a hash implementation and are
deferred to a future crypto package.

## Install

```toml
[dependencies]
"github.com/martian56/raven-uuid" = "v0.2.1"
```

## Usage

```rust
import "github.com/martian56/raven-uuid" { Uuid }

fun main() {
    let id = Uuid.v4()
    print(id.to_string())        // e.g. 4a81a9ca-f983-4ed2-aad7-df5f73db6ef9
    print(id.version())          // 4

    let ordered = Uuid.v7()      // time-ordered, uses the system clock
    print(ordered.to_string())

    match Uuid.parse("4a81a9ca-f983-4ed2-aad7-df5f73db6ef9") {
        Ok(u) -> print(u.is_nil()),
        Err(e) -> print(e),
    }
}
```

For reproducible output (tests), use the `*_with` forms and seed the `Rng`
yourself:

```rust
import std/random { Rng }

let rng = Rng.new(42)
let id = Uuid.v4_with(rng)       // deterministic given the seed
```

## API

| Function | Description |
|----------|-------------|
| `Uuid.v4()` | Random UUID |
| `Uuid.v7()` | Time-ordered UUID from the system clock |
| `Uuid.v1()` | Time-based UUID with a random node |
| `Uuid.v4_with(rng)` | Random UUID from an explicit `Rng` (deterministic) |
| `Uuid.v7_with(rng, unix_ms)` | Time-ordered UUID from an explicit `Rng` and timestamp |
| `Uuid.v1_with(rng, unix_ms)` | Time-based UUID from an explicit `Rng` and timestamp |
| `Uuid.nil()` | All-zero UUID |
| `Uuid.max()` | All-ones UUID |
| `Uuid.parse(s)` | Parse a hyphenated or bare 32-hex string, returns `Result<Uuid, String>` |
| `u.to_string()` | Canonical `8-4-4-4-12` lowercase hex |
| `u.simple()` | The 32-hex form with no hyphens |
| `u.urn()` | The `urn:uuid:...` form |
| `u.version()` | Version number (1, 4, 7, ...) |
| `u.variant()` | Variant field (2 is the RFC 9562 variant) |
| `u.is_nil()` | True for the all-zero UUID |
| `u.is_max()` | True for the all-ones UUID |
| `u.equals(other)` | Byte-wise equality (or use `==`) |
| `u.compare(other)` | Lexicographic order, `-1` / `0` / `1` |
| `u.hash()` | A stable hash, so a `Uuid` works as a `Map`/`Set` key |

## Traits

`Uuid` implements the core traits, so it behaves like a built-in value:

- **`Eq`**: compare with `==` and `!=`.
- **`ToString`**: print directly and use in `"${...}"` interpolation.
- **`Ord`**: order with `compare`, and sort with [std/cmp](https://martian56.github.io/raven/v2/guide/stdlib/cmp/).
- **`Hash`**: use a `Uuid` as a `Map` or `Set` key.

```rust
import std/collections
import "github.com/martian56/raven-uuid" { Uuid }

fun main() {
    let a = Uuid.v4()
    let b = Uuid.v4()
    print(a == b)                       // false

    let names: Map<Uuid, String> = Map.new()
    names.set(a, "alpha")               // Uuid as a key
    print(names.get_or(a, "missing"))   // alpha
}
```

## Requirements

The no-argument generators (`v4`, `v7`, `v1`) need **raven >= 2.0.1**, which
makes `Rng.from_entropy()` return a distinct seed on every call. On older
toolchains, prefer the `*_with` forms and hold a single long-lived `Rng`.

## License

MIT
