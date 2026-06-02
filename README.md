# raven-uuid

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
"github.com/martian56/raven-uuid" = "0.1"
```

## Usage

```raven
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

```raven
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
| `u.version()` | Version number (1, 4, 7, ...) |
| `u.variant()` | Variant field (2 is the RFC 9562 variant) |
| `u.is_nil()` | True for the all-zero UUID |
| `u.equals(other)` | Byte-wise equality |

## Requirements

The no-argument generators (`v4`, `v7`, `v1`) need **raven >= 2.0.1**, which
makes `Rng.from_entropy()` return a distinct seed on every call. On older
toolchains, prefer the `*_with` forms and hold a single long-lived `Rng`.

## License

MIT
