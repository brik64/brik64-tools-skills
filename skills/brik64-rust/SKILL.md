---
name: brik64-rust
description: "Use the brik64 Rust crate (v4.0.0-beta.1) to apply Digital Circuitality in Rust projects. Covers installation, all 128 monomers (64 core + 64 extended), EVA composition, integration patterns, and the methodology vs. certification distinction. Use when writing Rust code with BRIK-64 libraries."
version: 4.0.0-beta.1
---

# BRIK-64 for Rust

Apply Digital Circuitality inside your existing Rust projects using the `brik64` crate.

**Docs:** https://brik64.dev/docs
**Crate:** https://crates.io/crates/brik64

---

## Installation

```toml
[dependencies]
brik64 = "4.0.0-beta.1"
```

Or via CLI:
```bash
cargo add brik64
```

```bash
cargo add brik64
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

## Arithmetic (saturating — never panics, never overflows)

```rust
use brik64::mc::arithmetic::*;

let sum   = add8(200, 100);      // 255 (saturating)
let diff  = sub8(10, 20);        // 0 (saturating)
let prod  = mul8(20, 20);        // 255 (saturating)
let (q,r) = div8(17, 5);         // (3, 2) — always returns tuple
let rem   = mod8(17, 5);         // 2
let n     = neg8(1);             // 255
let p     = pow8(2, 7);          // 128 (saturating)
```

> `div8` never panics — returns (0, 0) for division by zero.

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

### Circuit-closed function (methodology without library)

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
    Ok(trimmed.to_uppercase())      // all paths covered — Φ_c = 1
}
```

---

## Important Distinction

Using `brik64` applies Digital Circuitality **as a methodology** inside your Rust code. This gives you:

- ✅ Saturating arithmetic (no panics, no overflow)
- ✅ Formally specified crypto operations
- ✅ EVA composition patterns
- ✅ Better code structure and determinism

It does **not** give you:

- ❌ CMF verification (Φ_c computation)
- ❌ Auto-generated test suites from formal proof
- ❌ Registry certification badge
- ❌ Φ_c = 1 compiler enforcement

For formal certification → write in PCD: https://docs.brik64.dev/pcd/tutorial

---


## Closure Domains

Every monomer declares its domain — the bounded set of valid inputs and outputs.
This is what makes Φ_c = 1 possible.

- **Range**: `[0, 255]` for u8 operations
- **Set**: `{true, false}` for boolean operations  
- **Bounded**: predicate on finite domain (e.g., even numbers in [0,100])
- **Product**: cartesian product for multi-input operations

Without bounded domains, the circuit is open and cannot be certified.
The domain IS the circuit boundary.

## Extended Monomers (MC_64–MC_127) — v4.0.0-beta.1+

Extended monomers add 64 new operations across 8 families: Float64, Math, Network, Graphics, Audio, Filesystem+, Concurrency, and Interop/FFI. They operate under **CONTRACT closure (Φ_c = CONTRACT)** rather than static Coq proofs.

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

> **Note:** Core monomers (MC_00–MC_63) have Φ_c = 1 (Coq-proven). Extended monomers (MC_64–MC_127) use Φ_c = CONTRACT — runtime contracts enforce correctness for external-facing operations.
