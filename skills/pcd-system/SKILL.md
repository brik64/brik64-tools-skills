---
name: pcd-system
description: "Complete reference for the BRIK-64 system: write PCD programs, compile with brikc, certify circuits (Φ_c = 1), debug CMF errors, and build AI safety policy circuits. Use when writing .pcd programs, using the brikc CLI, fixing certification errors, or designing ALLOW/BLOCK guardrails."
---

# PCD System — Complete Reference

Write, compile, certify, debug, and deploy formally verified programs in PCD.
Every valid PCD program has Φ_c = 1: structurally impossible to have logic errors, type errors, or undefined behavior.

**Docs:** https://docs.brik64.dev/pcd/tutorial

---

## Install brikc

```bash
curl -fsSL https://brik64.dev/install | sh
brikc --version          # brikc v2.0.0
brikc check --self       # verify authentic binary
```

---

## Program Structure

```pcd
PC circuit_name {
    // logic here
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

// Match — wildcard mandatory for Φ_c = 1
let label = match cmd {
    "add" => do_add(a, b),
    "sub" => do_sub(a, b),
    _     => "unknown"
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
| MC_00.ADD8 | (a: i64, b: i64) → i64 | min(a+b, 255) | Saturating |
| MC_01.SUB8 | (a: i64, b: i64) → i64 | max(a-b, 0) | Saturating |
| MC_02.MUL8 | (a: i64, b: i64) → i64 | min(a*b, 255) | Saturating |
| MC_03.DIV8 | (a: i64, b: i64) → **Tuple(i64, i64)** | (quotient, remainder) | **Always destructure!** |
| MC_04.MOD8 | (a: i64, b: i64) → i64 | a % b | Guard b≠0 |
| MC_05.NEG8 | (a: i64) → i64 | 256-a | Two's complement |
| MC_06.ABS8 | (a: i64) → i64 | identity | |
| MC_07.POW8 | (base: i64, exp: i64) → i64 | min(base^exp, 255) | Saturating |

```pcd
let (q, r) = MC_03.DIV8(17, 5);   // q=3, r=2 — ALWAYS destructure DIV8
```

### Family 1 — Logic (bitwise)

| Monomer | Op |
|---------|----|
| MC_08.AND8 | bitwise AND |
| MC_09.OR8 | bitwise OR |
| MC_10.XOR8 | bitwise XOR |
| MC_11.NOT8 | bitwise NOT |
| MC_12.SHL | left shift |
| MC_13.SHR | right shift |
| MC_14.ROTL | rotate left |
| MC_15.ROTR | rotate right |

### Family 2 — Memory

| Monomer | Purpose |
|---------|---------|
| MC_16.LOAD | Load value |
| MC_17.STORE | Store value |
| MC_18.ALLOC | Allocate N bytes |
| MC_19.FREE | Free memory |
| MC_20.COPY | Copy N bytes |
| MC_21.SWAP | Swap two values |
| MC_22.CAS | Compare-and-swap (atomic) |
| MC_23.FENCE | Memory barrier |

### Family 3 — Control

| Monomer | Purpose | Note |
|---------|---------|------|
| MC_24.IF | Conditional branch | Requires Bool |
| MC_25.JUMP | Unconditional goto | |
| MC_26.CALL | Function call | |
| MC_27.RET | Return | |
| MC_28.LOOP | Bounded iteration | |
| MC_29.BREAK | Exit loop | |
| MC_30.CONT | Continue loop | |
| MC_31.HALT | Stop execution | |

### Family 4 — I/O

| Monomer | Purpose |
|---------|---------|
| MC_32.READ | Read from fd |
| MC_33.WRITE | Write to fd (stdout = fd 1) |
| MC_34.OPEN | Open file → fd |
| MC_35.CLOSE | Close fd |
| MC_36.SEEK | Seek in file |
| MC_37.STAT | File metadata |
| MC_38.POLL | Poll fd readiness |
| MC_39.FLUSH | Flush buffer |

```pcd
// Write to stdout
let msg = "Hello\n";
let n = MC_43.LEN(msg);
let _ = MC_33.WRITE(1, msg, n);
// OR (simpler):
let _ = MC_58.WRITE("Hello\n");
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

| Monomer | Purpose | Note |
|---------|---------|------|
| MC_56.READ | Read file → String | |
| MC_57.WRITE | Write String → file | |
| MC_58.WRITE | Write to stdout | Most common I/O |
| MC_59.EXIT | Exit with code | |
| MC_60.PID | Get process ID | |
| MC_61.SIGNAL | Signal handling | |
| MC_62.MMAP | Memory-map file | |
| MC_63.ENV | Get env var / CLI arg | `MC_63.ENV("ARGV_1")` = first CLI arg |

---

## CMF Metrics — What Φ_c = 1 Requires

| Metric | Symbol | Requirement | Fix if broken |
|--------|--------|-------------|---------------|
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
    OUTPUT 1;
}
```

### Accumulator with loop-carried variable

```pcd
let total = 0;
loop(n) as i {
    let total = total + arr[i];
}
```

### Boolean-returning function (explicit 0/1)

```pcd
fn is_valid(x) {
    if (x > 0) { return 1; }
    return 0;
    // NEVER: return (x > 0); — comparison not guaranteed 0/1
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
| No binary literals | Use hex (0xFF) or decimal |

---

## brikc CLI Reference

```bash
# Compile
brikc compile src/main.pcd                  # Native x86-64 ELF
brikc compile src/main.pcd --target rs      # Rust
brikc compile src/main.pcd --target js      # JavaScript ES2022
brikc compile src/main.pcd --target py      # Python 3.10+
brikc compile src/main.pcd --target wasm32  # WebAssembly
brikc compile src/main.pcd --emit-tests     # Code + auto-generated test suite

# Verify without compiling
brikc check src/main.pcd                    # CMF check only
brikc check src/main.pcd --json             # Machine-readable (CI)

# Other
brikc fmt src/main.pcd                      # Format in place
brikc run src/main.pcd                      # Compile + execute
brikc repl                                  # Interactive session
brikc catalog                               # List all 64 monomers
brikc lsp                                   # Start LSP server
```

**Exit codes:** 0 = success · 1 = parse error · 2 = Φ_c < 1 · 3 = codegen error

---

## Auto-Generated Test Suite

```bash
brikc compile src/main.pcd --emit-tests
# build/main.rs          ← certified implementation
# build/main_spec.rs     ← property tests + boundary tests + regression anchors
```

The test suite is derived from the formal proof: covers all input/output ranges, boundary conditions, and EVA composition invariants. These tests serve as a specification — if the generated code is modified manually, the tests will catch any divergence from the certified behavior.

---

## Debugging CMF Errors

### Reading CMF output

```bash
brikc check program.pcd --json
```
```json
{
  "cmf": { "e": 4, "h": 0.44, "s": 0.000, "c": 1, "t": 0, "δ": 0.000, "Φ_c": 1.000 },
  "omega": 1
}
```

**Red flags:** `δ > 0` → unused params · `Φ_c < 1.000` → open circuit

### Common errors

| Error | Cause | Fix |
|-------|-------|-----|
| `Φ_c < 1.000` | Missing OUTPUT on some path | Add fallback `OUTPUT` at end |
| `δ > 0` | Unused function parameter | Remove it or prefix with `_` |
| Type error at monomer | Wrong argument type | Check monomer signature above |
| `Tuple` type mismatch | DIV8 not destructured | `let (q, r) = MC_03.DIV8(a, b);` |
| MAX_DEPTH | Nesting too deep | Extract into functions |
| While loop vars stale | WhileLoop SSA bug | Replace with `loop(N)` + `if` |
| Circular import | A→B→A | Extract shared code into third module |
| Boolean compare broken | `return (X == Y)` | Use `if (X==Y) { return 1; } return 0;` |

---

## Policy Circuits — AI Safety Guardrails

A Policy Circuit is a PCD program that intercepts AI agent actions and returns `ALLOW` or `BLOCK: reason`.
Φ_c = 1 guarantees: **every possible input produces a definitive result — no undefined behavior, no crash, no ambiguity.**

### Template (deny-by-default)

```pcd
import "stdlib/string.pcd";
import "stdlib/json.pcd";

PC policy {
    let action_json = MC_56.READ("/dev/stdin");
    let action      = parse(action_json);
    let op          = get(action, "operation");
    let path        = get(action, "path");
    let agent_id    = get(action, "agent_id");

    // Block dangerous paths
    if (starts_with(path, "/etc/")) { OUTPUT "BLOCK: /etc/ forbidden"; }
    if (contains(path, ".env"))     { OUTPUT "BLOCK: .env forbidden"; }
    if (contains(path, "/."))       { OUTPUT "BLOCK: dotfiles forbidden"; }

    // Block shell
    if (op == "shell") { OUTPUT "BLOCK: shell execution forbidden"; }

    // Write restrictions
    if (op == "write") {
        if (!starts_with(path, "/tmp/")) {
            OUTPUT "BLOCK: writes only in /tmp/";
        }
    }

    OUTPUT "ALLOW";
}
```

### Design rules

1. **BLOCK before ALLOW** — check dangerous conditions first, `OUTPUT "ALLOW"` only at the end
2. **Always include reason** — `BLOCK: <human-readable reason>` for auditability
3. **Wildcard fallback** — if using `match`, always include `_ =>` arm
4. **Validate agent identity** — maintain an allowlist of known agent IDs

### Compile and deploy

```bash
brikc compile policy.pcd -o policy

# Test
echo '{"operation":"write","path":"/etc/passwd","agent_id":"claude"}' | ./policy
# BLOCK: /etc/ forbidden

# Cross-compile for integration
brikc compile policy.pcd --target js -o policy.mjs    # Node.js middleware
brikc compile policy.pcd --target py -o policy.py     # Python wrapper
brikc compile policy.pcd --target wasm32 -o policy.wasm
```

### Policy circuit checklist

- [ ] Every rule ends with `OUTPUT "BLOCK: reason"`
- [ ] Last line is `OUTPUT "ALLOW"` (deny-by-default)
- [ ] Agent identity validated against allowlist
- [ ] `.env`, `/etc/`, `credentials` paths are blocked
- [ ] `brikc check policy.pcd` returns `Φ_c = 1.000`
- [ ] Tested with both ALLOW and BLOCK scenarios
