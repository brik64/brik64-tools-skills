---
name: pcd-system
description: Complete PCD language reference + brikc CLI for BRIK-64 BETA 2.0.0. Covers syntax, all 64 monomers (verified signatures), CMF debug, policy circuits, and multi-target compilation. Use when writing .pcd files or using the brikc compiler.
triggers:
  - writing PCD programs
  - using brikc CLI
  - building AI safety policy circuits
  - debugging CMF errors
  - compiling to BIR/Rust/JS/Python/native
version: 2.0.0-beta.1
---

# PCD System — BRIK-64 BETA 2.0.0

> **BETA**: Core BIR interpreter is production-ready. Codegen backends (Rust/JS/Python/native) are WIP.

## Quick Start

```bash
# Check a program
brikc check --imports program.pcd

# Compile to BIR (recommended, fully functional)
brikc compile --input program.pcd --target bir --output program.bir

# Run via BIR interpreter (fully functional)
TEXT="hello" brikc run program.pcd

# Compile to Rust/JS/Python (WIP — basic programs work)
brikc compile --input program.pcd --target rust --output program.rs
brikc compile --input program.pcd --target js   --output program.js
brikc compile --input program.pcd --target python --output program.py

# Format
brikc fmt program.pcd

# Interactive REPL
brikc repl
```

> **CLI flags — verified against BETA 2.0.0:**
> - `compile` requires `--input FILE` (NOT positional)
> - `run FILE` IS positional (no --input)
> - `check FILE` is positional
> - `--target rust` (NOT `--target rs`)
> - `--target python` (NOT `--target py`)
> - `native` and `wasm32` targets exist in --help but are NOT YET IMPLEMENTED

---

## PCD Syntax

```pcd
// Circuit definition
PC my_circuit {
    // Variables
    let x = 42;
    let name = "hello";
    let arr = [1, 2, 3];

    // Function definition
    fn add(a, b) {
        return a + b;
    }

    // Conditional
    if (x > 10) {
        let msg = "big";
    } else {
        let msg = "small";
    }

    // Counted loop (PREFERRED — while loops have SSA bug in native ELF)
    loop(10) as i {
        let item = arr[i];
    }

    // While loop (works in BIR interpreter, broken in native ELF)
    while (x > 0) {
        let x = x - 1;
    }

    // Output
    OUTPUT x;
    return x;
}
```

### Key syntax rules
- `loop(N) as i` — N in parentheses, REQUIRED (formatter preserves them)
- `loop(N)` without counter also valid
- Variables are immutable bindings — rebind with another `let name = ...`
- `return` exits current function or circuit
- `OUTPUT expr;` declares the circuit output (sets Φ_c metadata)
- Native arithmetic: `+`, `-`, `*`, `/`, `%` — 64-bit operations
- **NEVER** use `while` in programs targeting native ELF — SSA bug. Use `loop(N) { if(cond) { ... } }` instead.

---

## Stdout pattern (VERIFIED)

There is NO simple print monomer. Use this pattern (from stdlib io.pcd):

```pcd
fn println(s) {
    let n = MC_43.LEN(s);
    MC_58.WRITE(1, s, n);
    MC_58.WRITE(1, "\n", 1);
    return 0;
}

// Usage:
let _ = println("Hello, World!");
let _ = println(MC_40.CONCAT("Value: ", some_string));
```

Or import stdlib directly:
```pcd
import "stdlib/io.pcd";
// then use println(s) directly
```

---

## The 64 Monomers — Verified Signatures (ECO BETA 2.0.0)

### Family 0: Arithmetic (MC_00–MC_07) — 8-bit saturating

| MC# | Name | Signature | Notes |
|-----|------|-----------|-------|
| MC_00 | ADD8 | (a:U8, b:U8) → U8 | Saturating add |
| MC_01 | SUB8 | (a:U8, b:U8) → U8 | Saturating sub |
| MC_02 | MUL8 | (a:U8, b:U8) → U8 | Saturating mul |
| MC_03 | DIV8 | (a:U8, b:U8) → Tuple(U8,U8) | Returns (quotient, remainder) |
| MC_04 | INC/NEG | (a:U8) → U8 | **1 arg only** — NOT MOD8(a,b) |
| MC_05 | NEG | (a:U8) → U8 | **1 arg only** |
| MC_06 | ABS8 | (a:I8) → U8 | Absolute value |
| MC_07 | POW8 | (base:U8, exp:U8, mod:U8) → U8 | **3 args** — NOT 2 |

> ⚠️ **For modulo, use native PCD operator**: `let rem = (a + b) % range;`
> The `%` operator works natively for 64-bit integers.

### Family 1: Logic (MC_08–MC_15)

| MC# | Name | Signature |
|-----|------|-----------|
| MC_08 | AND | (a:U8, b:U8) → U8 |
| MC_09 | OR | (a:U8, b:U8) → U8 |
| MC_10 | XOR | (a:U8, b:U8) → U8 |
| MC_11 | NOT | (a:U8) → U8 |
| MC_12 | NAND | (a:U8, b:U8) → U8 |
| MC_13 | NOR | (a:U8, b:U8) → U8 |
| MC_14 | XNOR | (a:U8, b:U8) → U8 |
| MC_15 | BUF | (a:U8) → U8 |

### Family 2: Memory (MC_16–MC_23)

| MC# | Name | Signature |
|-----|------|-----------|
| MC_16 | ALLOC | (size:U64) → U64 (ptr) |
| MC_17 | FREE | (ptr:U64) → Unit |
| MC_18 | READ | (ptr:U64, offset:U64) → U8 |
| MC_19 | WRITE_MEM | (ptr:U64, offset:U64, val:U8) → Unit |
| MC_20 | COPY | (dst:U64, src:U64, len:U64) → Unit |
| MC_21 | ZERO | (ptr:U64, len:U64) → Unit |
| MC_22 | CMP_MEM | (a:U64, b:U64, len:U64) → I8 |
| MC_23 | BOUNDS | (ptr:U64, size:U64, len:U64) → Bool |

### Family 3: Control (MC_24–MC_31)

> ⚠️ MC_24 IF is NOT a ternary function. Use PCD `if/else` syntax instead.

| MC# | Name | Signature | Notes |
|-----|------|-----------|-------|
| MC_24 | IF | [Bool] → Unit | Low-level branch signal — use `if(cond){...}` syntax |
| MC_25 | JUMP | [U64] → Unit | Low-level jump |
| MC_26 | CALL | [U64] → Unit | Low-level call |
| MC_27 | RET | [] → Unit | Low-level return |
| MC_28 | HALT | [] → Unit | Halt execution |
| MC_29 | NOP | [] → Unit | No-op |
| MC_30 | CMP | (a:I64, b:I64) → I8 | Compare: -1/0/1 |
| MC_31 | LOOP_CTRL | [Bool] → Unit | Loop control signal |

### Family 4: I/O (MC_32–MC_39)

| MC# | Name | Signature | Notes |
|-----|------|-----------|-------|
| MC_32 | READ_FD | (fd:U64) → U8 | Read byte from file descriptor |
| MC_33 | WRITE_FD | (fd:U64, byte:U8) → Unit | Write byte to fd |
| MC_34 | OPEN | (path:String, flags:U64) → U64 | Returns fd |
| MC_35 | CLOSE | (fd:U64) → Unit | Close fd |
| MC_36 | SEEK | (fd:U64, offset:I64, whence:U8) → I64 |
| MC_37 | STAT | (path:String) → Array | File metadata |
| MC_38 | MKDIR | (path:String) → Bool |
| MC_39 | DELETE | (path:String) → Bool |

### Family 5: String (MC_40–MC_47)

| MC# | Name | Signature | Notes |
|-----|------|-----------|-------|
| MC_40 | CONCAT | (a:String, b:String) → String | ✅ verified |
| MC_41 | SPLIT | (s:String, delim:String) → Array | ✅ verified |
| MC_42 | SUBSTR | (s:String, start:U64, len:U64) → String | ✅ use for CHAR_AT |
| MC_43 | LEN | (s:String) → U64 | ✅ verified |
| MC_44 | UPPER | (s:String) → String | ✅ verified |
| MC_45 | LOWER | (s:String) → String | ⚠️ **NOT CHAR_AT** — takes 1 arg |
| MC_46 | TRIM | (s:String) → String | ✅ verified |
| MC_47 | MATCH | (s:String, pattern:String) → Bool | ✅ verified |

> ⚠️ **CHAR_AT does not exist as a monomer.** Use `MC_42.SUBSTR(s, i, 1)` to get char at index i.

### Family 6: Crypto (MC_48–MC_55)

| MC# | Name | Signature | Notes |
|-----|------|-----------|-------|
| MC_48 | HASH | (data:Array(U8)) → Array(U8,32) | ⚠️ **NOT WRITE** — SHA-256 hash |
| MC_49 | HMAC | (key:Array(U8), data:Array(U8)) → Array(U8,32) |
| MC_50 | ENCRYPT | (key:Array(U8,32), data:Array(U8)) → Array(U8) |
| MC_51 | DECRYPT | (key:Array(U8,32), data:Array(U8)) → Array(U8) |
| MC_52 | SIGN | (key:Array(U8,64), data:Array(U8)) → Array(U8,64) |
| MC_53 | VERIFY_SIG | (key:Array(U8,32), sig:Array(U8,64), data:Array(U8)) → Bool |
| MC_54 | RAND | (len:U64) → Array(U8) |
| MC_55 | DERIVE | (seed:Array(U8), info:Array(U8)) → Array(U8,32) |

### Family 7: System (MC_56–MC_63)

| MC# | Name | Signature | Notes |
|-----|------|-----------|-------|
| MC_56 | GETPID | [] → U64 | ⚠️ **NOT STRLEN** — returns process id |
| MC_57 | GETTIME | [] → U8 | ⚠️ **NOT SUBSTR** — returns time byte |
| MC_58 | WRITE | (fd:U64, s:String, len:U64) → Unit | ⚠️ stdout = fd 1 |
| MC_59 | READ_STR | (fd:U64, len:U64) → String |
| MC_60 | ARGS | [] → Array(String) | All CLI arguments |
| MC_61 | EXIT | (code:U8) → Unit |
| MC_62 | SLEEP | (ms:U64) → Unit |
| MC_63 | ENV | (name:String) → String | ⚠️ Always returns String, not Int |

> ⚠️ **MC_58.WRITE verified pattern:**
> ```pcd
> MC_58.WRITE(1, "hello\n", 6);  // fd=1 is stdout
> ```
>
> ⚠️ **MC_63.ENV always returns String.** `MC_63.ENV("PORT")` returns `"8080"`, not `8080`.
> To convert: use `to_int()` from `stdlib/string.pcd` (if available) or hardcode numeric values.

---

## Imports and stdlib

```pcd
import "stdlib/io.pcd";              // println, print, read_line
import "stdlib/string.pcd";          // join, trim, replace, starts_with, char_at
import "stdlib/math.pcd";            // min, max, abs, pow, sqrt, clamp, gcd, factorial
import "stdlib/array.pcd";           // map, filter, reduce, sort, contains
import "stdlib/fmt.pcd";             // format, pad_left, pad_right
import "stdlib/json.pcd";            // parse, stringify
import { println, print } from "stdlib/io.pcd";   // selective import
import "mylib.pcd" as lib;            // alias import
```

Stdlib location on ECO: `~/brik64/software/examples/stdlib/`

---

## CMF Error Reference

CMF (Circuit Metric Failure) errors appear during `brikc check` or `brikc run`:

| Code | Meaning | Fix |
|------|---------|-----|
| CMF-001x | Type mismatch | Check monomer signature — ensure types match |
| CMF-002x | Arity error | Wrong number of arguments to monomer |
| CMF-003x | Undefined variable | Declare with `let` before use |
| CMF-004x | Φ_c < 1 | Circuit not closed — add `OUTPUT` statement |
| CMF-005x | Import not found | Check path relative to .pcd file location |
| CMF-006x | Circular import | Remove circular dependency |

> `brikc check` validates syntax and circuit structure but does NOT validate monomer arity.
> Arity errors only appear at `brikc run` (BIR interpreter) or compile time.

---

## Policy Circuit Pattern (AI Safety)

```pcd
// ai-safety-policy.pcd — ALLOW/BLOCK guardrail
PC file_write_policy {
    let action  = MC_63.ENV("ACTION");      // "write_file"
    let path    = MC_63.ENV("TARGET_PATH"); // "/etc/passwd"
    let agent   = MC_63.ENV("AGENT_ID");   // "gpt-4"

    fn is_protected_path(p) {
        let is_etc   = MC_42.SUBSTR(p, 0, 4) == "/etc";
        let is_sys   = MC_42.SUBSTR(p, 0, 4) == "/sys";
        let is_root  = p == "/";
        return is_etc + is_sys + is_root;
    }

    fn is_allowed_action(act) {
        // Allowlist approach: DENY by default
        let allowed = 0;
        if (act == "read_file")    { let allowed = 1; }
        if (act == "write_temp")   { let allowed = 1; }
        return allowed;
    }

    let protected = is_protected_path(path);
    let allowed   = is_allowed_action(action);

    let verdict = "BLOCK";
    if ((protected == 0) && (allowed == 1)) {
        let verdict = "ALLOW";
    }

    let _ = MC_58.WRITE(1, verdict, MC_43.LEN(verdict));
    let _ = MC_58.WRITE(1, "\n", 1);

    OUTPUT verdict;
    return verdict;
}
```

Compile and run as middleware:
```bash
# Compile to any backend
brikc compile --input ai-safety-policy.pcd --target bir --output policy.bir

# Use in pipeline
ACTION="write_file" TARGET_PATH="/tmp/safe" AGENT_ID="gpt-4" brikc run ai-safety-policy.pcd
# → ALLOW

ACTION="write_file" TARGET_PATH="/etc/passwd" AGENT_ID="gpt-4" brikc run ai-safety-policy.pcd
# → BLOCK
```

---

## TCE Metrics

Every compiled circuit produces TCE metrics:

| Metric | Meaning | Target |
|--------|---------|--------|
| Φ_c | Circuit closure | = 1.0 (must be exactly 1) |
| E_c | Energy coherence | > 0.7 |
| H_d | Hamming distance | < 0.3 |
| S_d | State diversity | > 0.5 |
| C_s | Coupling strength | < 0.8 |
| ETC | Energy-time coherence | > 0.6 |
| ΔN | Noise differential | < 0.1 |

```bash
brikc tce --input program.pcd --report    # Full TCE report
brikc check program.pcd                   # Quick Φ_c check
```

---

## brikc CLI Reference — Verified Flags

```bash
# check — validate circuit
brikc check <file.pcd> [--imports]
# Note: --json flag does NOT exist

# fmt — format in place
brikc fmt <file.pcd> [--check]
# Idempotent. loop(N) parentheses are preserved.

# compile — generate target
brikc compile --input <file.pcd> --target <TARGET> --output <outfile>
# Targets available: bir ✅ | rust ⚠️ | js ⚠️ | python ⚠️
# Targets NOT implemented: native ❌ | wasm32 ❌ | x86-64 ❌ | elf ❌

# run — BIR interpreter (fully functional)
brikc run <file.pcd> [--inputs a,b,c] [--args a,b,c]
# Note: positional, NOT --input

# repl — interactive
brikc repl

# lsp — language server (WIP)
brikc lsp
# Starts but JSON-RPC protocol not fully implemented in BETA

# catalog — list monomers
brikc catalog list
brikc catalog show <N>
# Note: catalog show does not display monomer name, only family+signature+TCE

# self-hosting status
brikc self-host-status
```

---

## Installation (BETA 2.0.0)

```bash
# Official installer (auto-detects platform)
curl -fsSL https://github.com/brik64/brik64-dist-releases/releases/latest/download/install.sh | sh

# Manual — Linux x86-64
curl -L .../brikc-beta-linux-x86_64 -o brikc && chmod +x brikc

# Manual — macOS Apple Silicon (M1/M2/M3/M4)
curl -L .../brikc-beta-macos-arm64 -o brikc && chmod +x brikc
xattr -d com.apple.quarantine brikc   # Remove macOS quarantine

# Manual — macOS Intel
curl -L .../brikc-beta-macos-intel -o brikc && chmod +x brikc
xattr -d com.apple.quarantine brikc

# Manual — Windows (PowerShell)
Invoke-WebRequest .../brikc-beta-windows-x64.exe -OutFile brikc.exe
```

**Platform support in BETA 2.0.0:**
- ✅ Linux x86-64 (fully tested)
- ✅ macOS Apple Silicon (ARM64) — binary available
- ✅ macOS Intel (x86-64) — binary available
- ✅ Windows x64 — binary available
- ✅ Linux ARM64 — binary available

---

## Known BETA Limitations

1. **Codegen backends WIP**: `--target rust/js/python` generate scaffolding but may fail for programs with functions, loops, or native operators. Use `--target bir` + `brikc run` for production use.
2. **native/wasm32 not yet public**: The native ELF pipeline exists internally (used for self-hosting bootstrap) but is not yet connected to the public `--target` API.
3. **ENV always returns String**: `MC_63.ENV("N")` returns `"42"` not `42`. Hardcode numeric values or parse manually.
4. **while loops broken in native ELF**: SSA bug. Use `loop(N) { if(cond) {...} }` pattern.
5. **LSP protocol incomplete**: `brikc lsp` starts but does not respond to JSON-RPC. Not usable in editors yet.
6. **check does not validate monomer arity**: Arity errors only surface at runtime.
