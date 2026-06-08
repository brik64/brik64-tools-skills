---
name: brik64-rust
description: "Historical Rust Digital Circuitality patterns for BRIK64 work. Check docs.brik64.com and the current public brik64 skill before installing crates or making SDK, certification, catalog, or extended-operation claims."
version: 0.1.0-beta.13-public-reference
---

# BRIK-64 for Rust

Apply Digital Circuitality patterns inside Rust projects. Check
https://docs.brik64.com and the current `brik64` skill before presenting crate
availability, SDK exports, or CLI behavior as current public truth.

**Docs:** https://docs.brik64.com
**Crate:** https://crates.io/crates/brik64-core

---

## Installation

```toml
[dependencies]
brik64-core = "0.1.0-beta.13"
```

Or via CLI:
```bash
brik64 --version
```

```bash
curl -fsSL https://brik64.com/cli/install.sh | bash
```

---

## Core Modules

```rust
use brik64::{mc, eva};

// mc::arithmetic — Family 0
// mc::logic      — Family 1
// mc::memory     — Family 2
// mc::control    — Family 3
// mc::io         — Family 4
// mc::string     — Family 5
// mc::crypto     — Family 6
// mc::system     — Family 7

// eva::seq       — Sequential composition (⊗)
// eva::par       — Parallel composition (∥)
// eva::cond      — Conditional composition (⊕)
// eva::pipeline  — Multi-step seq pipeline
```

---

## Historical SDK Operation Patterns

The examples below are design patterns for bounded operations. Treat package
exports and operation names as current only when docs and the installed crate
confirm them.

## Arithmetic (wrapping — never panics, wraps at 256)

```rust
use brik64_core::mc::arithmetic::*;

let sum   = add8(200, 100);      // 44 (wrapping: 300 % 256)
let diff  = sub8(10, 20);        // 246 (wrapping)
let prod  = mul8(20, 20);        // 144 (wrapping: 400 % 256)
let (q,r) = div8(17, 5);         // (3, 2) — always returns tuple
let rem   = mod8(17, 5);         // 2
let n     = neg8(1);             // 255
let p     = pow8(2, 7);          // 128 (saturating)
```

> In beta11, confirm division-by-zero behavior against the installed crate before
> using it as public documentation. Package installation does not establish
> CLI installation or workspace workflows.

---

## Logic (bitwise)

```rust
use brik64::mc::logic::*;

let a = and8(0xFF, 0x0F);    // 15
let o = or8(0xF0, 0x0F);     // 255
let x = xor8(0xAA, 0x55);    // 255
let n = not8(0xFF);          // 0
let s = shl(1, 3);           // 8
let r = shr(16, 2);          // 4
```

---

## String operations

```rust
use brik64::mc::string::*;

let joined  = concat("hello", " world");
let parts   = split("a,b,c", ",");       // Vec<String>
let sub     = substr("hello", 1, 3);     // "ell"
let n       = len("hello");              // 5
let up      = upper("hello");            // "HELLO"
let ch      = char_at("hello", 1);       // 101 (u8 of 'e')
let trimmed = trim("  hello  ");         // "hello"
let matches = match_pattern("foo", "f*"); // bool
```

---

## Crypto

```rust
use brik64::mc::crypto::*;

let hash  = sha256(b"hello world");    // [u8; 32]
let hmac  = hmac_sha256(key, b"msg"); // [u8; 32]
let enc   = aes256_enc(key, iv, data); // Vec<u8>
let dec   = aes256_dec(key, iv, enc);  // Vec<u8>
let rand  = rand_bytes(32);            // Vec<u8>

// Ed25519
let keypair = ed25519_keygen();
let sig     = sign(keypair.private, b"message");
let valid   = verify(keypair.public, b"message", &sig);  // true
```

---

## System

```rust
use brik64::mc::system::*;

let ts  = time_unix();       // u64 (seconds since epoch)
let pid = getpid();          // u32
let env = env_var("PATH");   // Option<String>
```

---

## EVA Composition

```rust
use brik64::eva;

// Sequential (⊗): output of A → input of B
let pipeline = eva::seq(
    |x: u8| mc::arithmetic::add8(x, 10),
    |x: u8| mc::arithmetic::mod8(x, 7),
);
let result = pipeline(250);   // add8(250, 10)=255, mod8(255,7)=3

// Multi-step pipeline
let process = eva::pipeline([
    |x| mc::arithmetic::add8(x, 5),
    |x| mc::arithmetic::mul8(x, 2),
    |x| mc::arithmetic::mod8(x, 100),
]);

// Parallel (∥): both must be independent (no shared mutation)
let (r1, r2) = eva::par(
    || mc::crypto::sha256(b"data_a"),
    || mc::crypto::sha256(b"data_b"),
);

// Conditional (⊕): both branches return same type
let result = eva::cond(
    score > 50,
    || "pass".to_string(),
    || "fail".to_string(),
);
```

---

## Integration Patterns

### Drop-in safe arithmetic

```rust
// Replace: let total = a + b;  (can overflow)
// With:
let total = mc::arithmetic::add8(a, b);  // saturating, verified
```

### Verified crypto pipeline

```rust
fn hash_and_sign(data: &[u8], key: &PrivateKey) -> (Hash32, Signature) {
    let hash = mc::crypto::sha256(data);
    let sig  = mc::crypto::sign(key, data);
    (hash, sig)   // ∥ composition: both independent
}
```

### Circuit-closed function (practice without library)

```rust
// Apply circuit thinking without the library
fn process(input: &str) -> Result<String, String> {
    if input.is_empty() {           // guard domain
        return Err("empty".into()); // all branches return
    }
    let trimmed = input.trim();     // bounded operation
    if trimmed.len() > 256 {        // bounded output
        return Err("too long".into());
    }
    Ok(trimmed.to_uppercase())      // all paths covered with explicit fallback above
}
```

---

## Important Distinction

Using `brik64` applies Digital Circuitality **as a practice** inside your Rust code. This gives you:

- ✅ Bounded arithmetic examples
- ✅ explicit crypto operation boundaries
- ✅ composition patterns
- ✅ Better code structure and determinism

It does **not** give you:

- formal verification claims
- auto-generated proof/test claims
- catalog or certification badge claims
- compiler-enforced closure claims

For PCD guidance, use the current `brik64` skill and docs.brik64.com.

---


## Closure Domains

Every monomer declares its domain: the bounded set of valid inputs and outputs.
This is practice guidance for keeping examples explicit and reviewable.

- **Range**: `[0, 255]` for u8 operations
- **Set**: `{true, false}` for boolean operations  
- **Bounded**: predicate on finite domain (e.g., even numbers in [0,100])
- **Product**: cartesian product for multi-input operations

Without bounded domains, the design remains open-ended. Treat domain notes here
as practice guidance, not as a public certification claim.

### You Are the Circuit Designer

The programmer defines domain bounds based on their problem context:
- Flight computer: velocity `[0, 900]` km/h, altitude `[0, 15000]` m
- Banking: transaction amount `[0.01, 1000000]`, account balance `[0, MAX_I64]`
- Temperature sensor: reading `[-273, 1000]` °C (absolute zero to furnace)

If a result falls outside the intended domain, the design needs a tighter
boundary or an explicit fallback.

**Normal software:** calculates velocity = 100,000 km/s, stores it, crashes later.
**Digital Circuitality:** the program does not compile. The circuit is open.

### Precision Engineering

Domains are numeric ranges, not physical units. Precision depends on monomer choice:
- **U8/I64-style examples:** exact integer arithmetic, no rounding
- **F64-style examples:** IEEE 754 floats, with normal floating-point rounding
- **Fixed-point pattern:** scale to integers (3.14 → 3140), compute exactly, scale back

Choose the right type for each calculation. If the result exceeds the range, the circuit doesn't close.

## Historical Extended Operation Notes

Older drafts referenced extended operation families. Treat those notes as
roadmap or historical material unless a current public release and docs page
publish the exact SDK surface.

### Float64 & Math

```rust
use brik64::mc::float64;
use brik64::mc::math;

// Float64 (F8)
let sum  = float64::fadd(1.5, 2.3);      // 3.8
let root = float64::fsqrt(16.0);          // 4.0
let abs  = float64::fabs(-3.14);          // 3.14

// Math (F9)
let sine   = math::sin(std::f64::consts::FRAC_PI_2);  // 1.0
let cosine = math::cos(0.0);                           // 1.0
let power  = math::pow(2.0, 10.0);                     // 1024.0
let log    = math::ln(std::f64::consts::E);             // 1.0
let ceil   = math::ceil(3.2);                           // 4.0
```

### Other Extended Families

```rust
use brik64::mc::{network, fs, interop};

// Network (F10)
let resp = network::http_req("GET", "https://api.example.com/data", "");

// Filesystem+ (F13)
let exists = fs::fs_exists("/tmp/data.bin");
let files  = fs::fs_list("/var/log");

// Interop/FFI (F15)
let json = interop::json_encode(value);
let obj  = interop::json_decode(r#"{"key": 42}"#);
```

> Current public agent guidance lives in the `brik64` skill and
> https://docs.brik64.com.
