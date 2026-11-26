# pickle-uuid

RFC 9562 UUIDs for [PickleScript](https://picklescript.dpdns.org): version 4
(random), version 7 (time-ordered), parsing, formatting, and byte-order
comparison. No dependencies beyond the standard library.

- **Package:** `vilnisgrinbergs2000/pickle-uuid`
- **Version:** 1.0.0
- **License:** MIT

## Install

```sh
pickle add vilnisgrinbergs2000/pickle-uuid
```

## Quick start

```pickle
import { newV4, newV7, parse } from pickle_uuid

fn main() {
    if (let some(id) = newV7()) {
        println(id.toString())        // 019ab5c4-7e2a-7c31-9d0e-...
        println(id.version())         // 7
        println(id.timestampMs())     // 1795...
    }

    if (let some(id) = newV4()) {
        println(id.toString())        // 5f0c2d1e-...
    }

    if (let some(id) = parse("6ba7b810-9dad-11d1-80b4-00c04fd430c8")) {
        println(id.toUrn())           // urn:uuid:6ba7b810-...
    }
}
```

`newV4` and `newV7` return `Uuid?` because they read the operating system
CSPRNG (`std.crypto.randomBytes`), which can fail. `parse` and `fromBytes`
return `Uuid?` for invalid input.

## API

### Constructors

| Function | Result |
| --- | --- |
| `newV4() -> Uuid?` | A version 4 UUID with 122 random bits. |
| `newV7() -> Uuid?` | A version 7 UUID with the current Unix millisecond timestamp and 74 random bits. |
| `parse(text: string) -> Uuid?` | Parse canonical, compact, `urn:uuid:`, or braced text. |
| `fromBytes(data: List<byte>) -> Uuid?` | Build from exactly 16 bytes in network order. |
| `nilUuid() -> Uuid` | The all-zero UUID. |
| `maxUuid() -> Uuid` | The all-ones UUID. |
| `isValid(text: string) -> bool` | Whether `text` parses. |

### `Uuid`

| Method | Result |
| --- | --- |
| `toString() -> string` | Lowercase canonical `8-4-4-4-12` text. |
| `toHex() -> string` | Lowercase 32-digit hex, no hyphens. |
| `toUrn() -> string` | `urn:uuid:` plus the canonical text. |
| `bytes() -> List<byte>` | A fresh copy of the 16 bytes. |
| `version() -> int` | The version nibble (0 for nil, 1-8 for versioned UUIDs). |
| `variant() -> string` | `ncs`, `rfc9562`, `microsoft`, or `future`. |
| `timestampMs() -> int?` | The 48-bit timestamp of a v7 UUID; `none` otherwise. |
| `isNil() -> bool` | Whether every byte is zero. |
| `compare(other: Uuid) -> int` | Byte-order comparison (`-1`, `0`, `1`). |
| `equals(other: Uuid) -> bool` | Byte-for-byte equality. |

## Parsing

`parse` accepts:

```text
6ba7b810-9dad-11d1-80b4-00c04fd430c8
6BA7B8109DAD11D180B400C04FD430C8
urn:uuid:6ba7b810-9dad-11d1-80b4-00c04fd430c8
{6ba7b810-9dad-11d1-80b4-00c04fd430c8}
```

Surrounding whitespace is ignored. The canonical hyphenated form requires
hyphens at the standard positions; the compact form is exactly 32 hex digits.
Output is always lowercase canonical text.

## Ordering

`compare` compares the 16 bytes from most significant to least significant.
For v7 values that follows the timestamp prefix, so IDs generated in different
milliseconds sort chronologically. The 74 random bits are not monotonic: two
IDs generated in the same millisecond sort by their random tails, not by
creation order. The library keeps no process-wide counter so it has no shared
mutable state.

## Development

Run the example from the repository root:

```sh
pklc run examples/demo.pkl
```

Run the tests with the toolchain test runner:

```sh
pklc test tests/pickle_uuid_test.pkl
```

The test suite covers parsing, formatting, variant and version extraction,
byte-order comparison, input copying, CSPRNG output shape, v7 timestamps, and
uniqueness across repeated generation.

## License

MIT. See [LICENSE](LICENSE).
