---
name: pcd-developer
description: "Complete PCD programming reference — write formally verified programs in Printed Circuit Description (BRIK-64's language). Covers all 64 monomers, EVA algebra, CMF certification, stdlib, patterns, anti-patterns, and auto-generated test suites. Use when writing or reviewing any .pcd program."
---

# PCD Developer — Complete Reference

Write formally verified programs in PCD. Every valid PCD program has Φ_c = 1: structurally impossible to have logic errors, type errors, or undefined behavior.

**Full docs:** https://docs.brik64.dev/pcd/tutorial

---

## Program Structure

```pcd
PC circuit_name {
    // functions and logic
    OUTPUT result;   // mandatory — closes the circuit
}
```

Every circuit must end with `OUTPUT`. Without it, Φ_c < 1 and the program does not compile.

---

## Variables & Types

```pcd
let x = 42;           // i64
let s = "hello";      // string
let b = true;         // bool
let arr = [1, 2, 3];  // array

// Loop-carried: rebind with SAME name to mutate across iterations
let count = 0;
loop(10) as i {
    let count = count + 1;   // ✅ same name = loop-carried
}
```

---

## Functions

```pcd
fn add(a, b) { return a + b; }

// Closure
let double = fn(x) { x * 2 };

// Multiple return via tuple
fn divmod(a, b) {
    let (q, r) = MC_03.DIV8(a, b);
    return (q, r);
}
```

---

## Control Flow

```pcd
// Conditional
if (x > 0) { return "pos"; } else { return "non-pos"; }

// Match
let label = match cmd {
    "add" => do_add(a, b),
    "sub" => do_sub(a, b),
    _ => "unknown"          // wildcard mandatory for Φ_c = 1
};

// Loop — ALWAYS prefer over while
loop(10) as i { ... }

// Safe while-equivalent (avoids SSA bug)
let pos = 0;
loop(max_len) as _i {
    if (pos < max) {
        let ch = MC_45.CHAR_AT(text, pos);
        let pos = pos + 1;
    }
}
```

---

## Structs

```pcd
struct Point { x: i64, y: i64 }
let p = Point { x: 10, y: 20 };
let px = p.x;
```

---

## Try / Catch

```pcd
fn safe_read(path) {
    try {
        let data = MC_56.READ(path);
        return data;
    } catch (err) {
        return "ERROR: " + err;
    }
}
```

---

## Imports & Stdlib

```pcd
import "stdlib/string.pcd";
import "stdlib/array.pcd";
import "stdlib/math.pcd";
import "stdlib/io.pcd";
import "stdlib/fmt.pcd";
import "stdlib/json.pcd";
```

---

## The 64 Core Monomers

### Family 0 — Arithmetic (saturating, never overflow)

| Monomer | Signature | Returns | Notes |
|---------|-----------|---------|-------|
| MC_00.ADD8 | (a: u8, b: u8) → i64 | min(a+b, 255) | Saturating |
| MC_01.SUB8 | (a: u8, b: u8) → i64 | max(a-b, 0) | Saturating |
| MC_02.MUL8 | (a: u8, b: u8) → i64 | min(a*b, 255) | Saturating |
| MC_03.DIV8 | (a: u8, b: u8) → **Tuple(i64, i64)** | (quotient, remainder) | **Always destructure!** |
| MC_04.MOD8 | (a: u8, b: u8) → i64 | a % b | Guard b≠0 |
| MC_05.NEG8 | (a: u8) → i64 | 256-a | Two's complement |
| MC_06.ABS8 | (a: u8) → i64 | identity (u8 ≥ 0) | |
| MC_07.POW8 | (base: u8, exp: u8) → i64 | min(base^exp, 255) | Saturating |

```pcd
let (q, r) = MC_03.DIV8(17, 5);   // q=3, r=2 — ALWAYS destructure DIV8
```

### Family 1 — Logic (bitwise)

| Monomer | Op | Example |
|---------|----|---------|
| MC_08.AND8 | bitwise AND | MC_08.AND8(0xFF, 0x0F) → 15 |
| MC_09.OR8 | bitwise OR | MC_09.OR8(0xF0, 0x0F) → 255 |
| MC_10.XOR8 | bitwise XOR | MC_10.XOR8(0xFF, 0xFF) → 0 |
| MC_11.NOT8 | bitwise NOT | MC_11.NOT8(0xFF) → 0 |
| MC_12.SHL | left shift | MC_12.SHL(1, 3) → 8 |
| MC_13.SHR | right shift | MC_13.SHR(16, 2) → 4 |
| MC_14.ROTL | rotate left | MC_14.ROTL(0x80, 1) → 1 |
| MC_15.ROTR | rotate right | MC_15.ROTR(1, 1) → 0x80 |

### Family 2 — Memory

| Monomer | Purpose |
|---------|---------|
| MC_16.LOAD | Load value from address |
| MC_17.STORE | Store value to address |
| MC_18.ALLOC | Allocate N bytes, returns pointer |
| MC_19.FREE | Free allocated memory |
| MC_20.COPY | Copy N bytes src→dst |
| MC_21.SWAP | Swap two values |
| MC_22.CAS | Compare-and-swap (atomic) |
| MC_23.FENCE | Memory barrier |

### Family 3 — Control

| Monomer | Purpose | Note |
|---------|---------|------|
| MC_24.IF | Conditional branch | Requires Bool, not I64 |
| MC_25.JUMP | Unconditional goto | |
| MC_26.CALL | Function call | |
| MC_27.RET | Return | |
| MC_28.LOOP | Bounded iteration | |
| MC_29.BREAK | Exit loop | |
| MC_30.CONT | Continue loop | |
| MC_31.HALT | Stop execution | |

### Family 4 — I/O

| Monomer | Purpose | Common Usage |
|---------|---------|--------------|
| MC_32.READ | Read from fd | Read file/stdin |
| MC_33.WRITE | Write to fd | MC_33.WRITE(1, data, len) → stdout |
| MC_34.OPEN | Open file | Returns fd |
| MC_35.CLOSE | Close fd | |
| MC_36.SEEK | Seek in file | |
| MC_37.STAT | File metadata | |
| MC_38.POLL | Poll fd readiness | |
| MC_39.FLUSH | Flush buffer | |

**Write to stdout (common pattern):**
```pcd
let _ = MC_58.WRITE("Hello\n");   // stdlib wraps MC_33
// or directly:
let msg = "Hello\n";
let n = MC_43.LEN(msg);
let _ = MC_33.WRITE(1, msg, n);
```

### Family 5 — String

| Monomer | Signature | Result |
|---------|-----------|--------|
| MC_40.CONCAT | (a: str, b: str) → str | a + b |
| MC_41.SPLIT | (s: str, delim: str) → array | Split by delimiter |
| MC_42.SUBSTR | (s: str, start: i64, len: i64) → str | Substring |
| MC_43.LEN | (s: str\|array) → i64 | Length |
| MC_44.UPPER | (s: str) → str | Uppercase |
| MC_45.CHAR_AT | (s: str, i: i64) → i64 | Char as int |
| MC_46.TRIM | (s: str) → str | Strip whitespace |
| MC_47.MATCH | (s: str, pattern: str) → bool | Pattern match |

### Family 6 — Crypto

| Monomer | Purpose | Output |
|---------|---------|--------|
| MC_48.HASH | Generic hash | Hash32 |
| MC_49.HMAC | HMAC-SHA256 | Hash32 |
| MC_50.AES_ENC | AES-256 encrypt | Bytes |
| MC_51.AES_DEC | AES-256 decrypt | Bytes |
| MC_52.SHA256 | SHA-256 | Hash32 |
| MC_53.RAND | Secure random N bytes | Bytes |
| MC_54.SIGN | Ed25519 sign | Sig |
| MC_55.VERIFY | Ed25519 verify | Bool |

### Family 7 — System

| Monomer | Purpose | Native ELF Note |
|---------|---------|-----------------|
| MC_56.READ | Read file → String | |
| MC_57.WRITE | Write String → file | |
| MC_58.WRITE | Write to stdout | Most common I/O |
| MC_59.EXIT | Exit with code | |
| MC_60.PID | Get process ID | |
| MC_61.SIGNAL | Signal handling | |
| MC_62.MMAP | Memory-map file | |
| MC_63.ENV | Get env var / CLI arg | MC_63.ENV("ARGV_1") = first CLI arg |

---

## CMF Metrics — What Φ_c = 1 Requires

| Metric | Symbol | Requirement | Common Fix |
|--------|--------|-------------|------------|
| Operational complexity | e | — | No constraint |
| Signature distance | h | — | No constraint |
| Structural entropy | s | — | No constraint |
| Cyclomatic complexity | c | — | No constraint |
| Termination depth | t | < 256 | Extract deep nesting into functions |
| Unused input ratio | δ | = 0 | Remove unused params or prefix with `_` |
| Circuit closure | **Φ_c** | **= 1.000** | All branches must reach OUTPUT |

---

## Common Patterns

### CLI Dispatch
```pcd
PC my_tool {
    let cmd = MC_63.ENV("ARGV_1");
    if (cmd == "build") { /* ... */ OUTPUT 0; }
    if (cmd == "check") { /* ... */ OUTPUT 0; }
    let _ = MC_58.WRITE("Unknown: " + cmd + "\n");
    OUTPUT 1;   // fallback
}
```

### Accumulator with loop-carried variable
```pcd
let total = 0;
loop(n) as i {
    let total = total + arr[i];   // same name = carried
}
```

### Boolean-returning function (explicit 0/1)
```pcd
fn is_valid(x) {
    if (x > 0) { return 1; }   // ✅ explicit
    return 0;
    // ❌ NEVER: return (x > 0);  — BIR comparison not guaranteed 0/1
}
```

---

## Known Bugs & Workarounds

| Issue | Workaround |
|-------|-----------|
| `while` SSA bug — loop vars stale | Use `loop(N) as i { if (cond) { ... } }` |
| `return (X == Y)` comparison | Use `if (X==Y) { return 1; } return 0;` |
| DIV8 used as integer | Always destructure: `let (q, r) = MC_03.DIV8(a, b);` |
| MAX_DEPTH=256 exceeded | Extract deeply nested logic into functions |
| No binary literals | Use hex (0xFF) or decimal instead of 0b11111111 |

---

## Auto-Generated Test Suite

```bash
brikc compile src/main.pcd --target rs --emit-tests
# build/main.rs          ← certified implementation
# build/main_spec.rs     ← property tests + boundary tests + regression anchors
```

The test suite is generated from the formal proof. It covers all input/output ranges, boundary conditions, and EVA composition invariants. Tests always pass on the certified code — and fail if the generated code is later modified manually.

→ Full testing reference: https://docs.brik64.dev/pcd/testing

---

## Quick Compile Reference

```bash
brikc compile src/main.pcd                # Native x86-64 ELF
brikc compile src/main.pcd --target rs   # Rust
brikc compile src/main.pcd --target js   # JavaScript ES2022
brikc compile src/main.pcd --target py   # Python 3.10+
brikc compile src/main.pcd --target wasm32  # WebAssembly
brikc compile src/main.pcd --emit-tests  # + auto-generated test suite
brikc check src/main.pcd                 # CMF verify only (no compile)
brikc check src/main.pcd --json          # Machine-readable for CI
brikc fmt src/main.pcd                   # Format in place
```
