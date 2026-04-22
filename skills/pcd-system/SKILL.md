---
name: pcd-system
description: Complete PCD language reference + brik64 CLI for BRIK-64 BETA 5.0.0-beta.1. Covers syntax, all 128 monomers (64 core + 64 extended), CMF debug, policy circuits, package manager, Lifter (12 languages, 100% liftability, SSA transform), two-tier certification (CORE + CONTRACT), and 13-target compilation. Use when writing .pcd files or using the public brik64 CLI.
triggers:
  - writing PCD programs
  - using brik64 CLI
  - building AI safety policy circuits
  - debugging CMF errors
  - compiling to BIR/Rust/JS/TS/Python/C/C++/Go/COBOL/PHP/Java/Swift/WASM/native
  - lifting source code to PCD
  - managing PCD packages
version: 5.0.0-beta.1
---

# PCD System — BRIK-64 BETA 5.0.0-beta.1

Complete reference for writing PCD programs, using the public `brik64` CLI, and working with the BRIK-64 Registry platform.

`brik64` is the public CLI name. If your current alpha install still exposes
`brikc`, treat it as a temporary compatibility shim and substitute it in the
same commands below.

---

## Quick Start

```bash
# Install brik64 (recommended)
curl -fsSL https://brik64.dev/install | sh

# Interactive banner (no args)
brik64

# Full pipeline: lift → check → build → execute
brik64 lift app.ts                          # any language → PCD
brik64 check program.pcd                    # validate circuit
brik64 compile --input program.pcd --target bir --output program.bir  # build
brik64 run program.pcd                      # execute via BIR

# Lift source code to PCD (12 languages, 100% liftability)
brik64 lift app.js          # JS, TS, TSX, JSX
brik64 lift main.py         # Python
brik64 lift lib.rs          # Rust
brik64 lift main.c          # C, C++
brik64 lift main.go         # Go
brik64 lift report.cbl      # COBOL
brik64 lift index.php       # PHP
brik64 lift Main.java       # Java

# Policy circuits (5 templates)
brik64 policy new --template no_network      # no_network, no_filesystem, memory_bound, sandbox, allow_all
brik64 policy verify policy.pcd
brik64 policy list

# Package manager
brik64 pkg init                              # creates brik.toml
brik64 pkg add some-package
brik64 pkg publish                           # publish to registry.brik64.dev
brik64 pkg deps                              # show dependency tree

# Registry API
# Live at registry.brik64.dev/v1

# Or via pip
pip install brik64

# Verify
brik64 --version
```

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

## The 128 Monomers — Verified Signatures (BETA 4.0.0-beta.2)

> **128 monomers = 64 core (MC_00–MC_63) + 64 extended (MC_64–MC_127)**
> Core monomers are formally verified. Extended monomers provide higher-level operations.

### ⚠️ CRITICAL: ADD8 and MUL8 are 8-bit (U8 wrapping, max 255)

`MC_00.ADD8(250, 10)` → `4` (wraps at 256, NOT saturates)
`MC_02.MUL8(99, 3)` → `41` (297 wraps: 297 % 256 = 41)

**For 64-bit arithmetic, use PCD native operators: `+`, `-`, `*`, `/`**

### Family 0: Arithmetic (MC_00–MC_07) — 8-bit WRAPPING

| MC# | Name | Signature | Notes |
|-----|------|-----------|-------|
| MC_00 | ADD8 | (a:U8, b:U8) → U8 | **U8 WRAPPING** — use `+` for 64-bit |
| MC_01 | SUB8 | (a:U8, b:U8) → U8 | **U8 WRAPPING** — use `-` for 64-bit |
| MC_02 | MUL8 | (a:U8, b:U8) → U8 | **U8 WRAPPING** — use `*` for 64-bit |
| MC_03 | DIV8 | (a:U8, b:U8) → Tuple(U8,U8) | Returns tuple: access with `qr[0]` (quotient), `qr[1]` (remainder) |
| MC_04 | INC | (a:U8) → U8 | **1 arg only** — NOT MOD8(a,b) |
| MC_05 | NEG/DEC | (a:U8) → U8 | **1 arg only** |
| MC_06 | ABS8 | (a:I8) → U8 | Absolute value |
| MC_07 | POW8 | (base:U8, exp:U8, mod:U8) → U8 | **3 args** — NOT 2 |

> ⚠️ **DIV8 tuple access — verified syntax:**
> ```pcd
> let qr = MC_03.DIV8(a, b);
> let q = qr[0];   // quotient
> let r = qr[1];   // remainder
> ```
> The syntax `let (q, r) = MC_03.DIV8(a, b)` causes a **parse error** — do NOT use it.

> ⚠️ **For modulo, use native PCD operator**: `let rem = a % b;`
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
| MC_41 | SPLIT | (s:String, delim:String) → Array(String) | ✅ returns list; access with `parts[i]` |
| MC_42 | SUBSTR | (s:String, start:U64, len:U64) → String | ✅ use for CHAR_AT |
| MC_43 | LEN | (s:String|Array) → U64 | ✅ verified — works on String (byte length) AND Array (element count) |
| MC_44 | UPPER | (s:String) → String | ✅ verified |
| MC_45 | LOWER | (s:String) → String | ✅ verified — NOT CHAR_AT |
| MC_46 | TRIM | (s:String) → String | ✅ verified |
| MC_47 | MATCH | (s:String, pattern:String) → Bool | ✅ verified — uses regex; falls back to substring if invalid regex |

> ⚠️ **CHAR_AT does not exist as a monomer.** Use `MC_42.SUBSTR(s, i, 1)` to get char at index i.
>
> ✅ **MC_43.LEN works on String (byte length) and Array (element count).**
>
> ⚠️ **MC_41.SPLIT returns a list.** Access elements with `parts[i]` (square bracket indexing), NOT `.get(i)`.

### Family 6: Crypto (MC_48–MC_55)

| MC# | Name | Signature | Notes |
|-----|------|-----------|-------|
| MC_48 | HASH | (data:String) → String | ⚠️ **NOT WRITE** — SHA-256 hash; **accepts String, returns hex String** |
| MC_49 | HMAC | (key:Array(U8), data:Array(U8)) → Array(U8,32) |
| MC_50 | ENCRYPT | (key:Array(U8,32), data:Array(U8)) → Array(U8) |
| MC_51 | DECRYPT | (key:Array(U8,32), data:Array(U8)) → Array(U8) |
| MC_52 | SIGN | (key:Array(U8,64), data:Array(U8)) → Array(U8,64) |
| MC_53 | VERIFY_SIG | (key:Array(U8,32), sig:Array(U8,64), data:Array(U8)) → Bool |
| MC_54 | RAND | (len:U64) → Array(U8) |
| MC_55 | DERIVE | (seed:Array(U8), info:Array(U8)) → Array(U8,32) |

> ⚠️ **MC_48.HASH runtime behavior (verified in beta.3):**
> - Accepts: String (not Array(U8) as catalog says)
> - Returns: hex String (e.g., `"ef0dd2a0..."`) — NOT Array(U8,32)
> - Catalog signature `[Array(U8,0)]→Array(U8,32)` does NOT match runtime behavior

### Family 7: System (MC_56–MC_63)

| MC# | Name | Signature | Notes |
|-----|------|-----------|-------|
| MC_56 | GETPID | [] → U64 | ⚠️ **NOT STRLEN** — returns process id |
| MC_57 | GETTIME | [] → U8 | ⚠️ **NOT SUBSTR** — returns time byte |
| MC_58 | WRITE | (fd:U64, s:String, len:U64) → Unit | ⚠️ stdout = fd 1; catalog shows `[]→U64` but runtime accepts 3 args |
| MC_59 | READ_STR | (fd:U64, len:U64) → String |
| MC_60 | ARGS | [] → Array(String) | All CLI arguments |
| MC_61 | EXIT | (code:U8) → Unit |
| MC_62 | SLEEP | (ms:U64) → Unit |
| MC_63 | ENV | (name:String) → String | ⚠️ Always returns String, not Int; in native ELF reads argv[1] |

> ⚠️ **MC_58.WRITE verified pattern:**
> ```pcd
> MC_58.WRITE(1, "hello\n", 6);  // fd=1 is stdout
> ```
>
> ⚠️ **MC_63.ENV always returns String.** `MC_63.ENV("PORT")` returns `"8080"`, not `8080`.
> To convert: use `to_int()` from `stdlib/string.pcd` or hardcode numeric values.
>
> ⚠️ **MC_63.ENV in native ELF reads argv[1]** (command-line argument), NOT environment variable.
> In BIR interpreter, reads actual environment variable.

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

CMF (Circuit Metric Failure) errors appear during `brik64 check` or `brik64 run`:

| Code | Meaning | Fix |
|------|---------|-----|
| CMF-001x | Type mismatch | Check monomer signature — ensure types match |
| CMF-002x | Arity error | Wrong number of arguments to monomer |
| CMF-003x | Undefined variable | Declare with `let` before use |
| CMF-004x | Φ_c < 1 | Circuit not closed — add `OUTPUT` statement |
| CMF-005x | Import not found | Check path relative to .pcd file location |
| CMF-006x | Circular import | Remove circular dependency |

> `brik64 check` validates syntax and circuit structure but does NOT validate monomer arity.
> Arity errors only appear at `brik64 run` (BIR interpreter) or compile time.

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

### EVA Composition

```pcd
// Sequential: A ⊗ B — output of A feeds input of B
let h = hash_sha256_int(data);
let keyed = xor_bytes_int(h, key);

// Parallel: A ∥ B — independent execution
let (a, b) = (compute_x(input), compute_y(input));

// Conditional: A ⊕ B — select based on predicate
let result = if (check) { path_a(x) } else { path_b(x) };
```

---

## brik64 CLI Reference — Verified Flags (4.0.0-beta.2)

```bash
# Compile to target
brik64 compile --target bir program.pcd      # BIR bytecode (default)
brik64 compile --target rust program.pcd     # Rust source
brik64 compile --target js program.pcd       # JavaScript
brik64 compile --target typescript program.pcd # TypeScript
brik64 compile --target python program.pcd   # Python
brik64 compile --target c program.pcd        # C
brik64 compile --target cpp program.pcd      # C++
brik64 compile --target go program.pcd       # Go
brik64 compile --target cobol program.pcd    # COBOL
brik64 compile --target php program.pcd      # PHP
brik64 compile --target java program.pcd     # Java
brik64 compile --target swift program.pcd    # Swift
brik64 compile --target wasm program.pcd     # WASM
brik64 compile --target native program.pcd   # ELF x86-64

# Run directly
brik64 run program.pcd                       # Compile + execute BIR

# Check without compiling
brik64 check program.pcd                     # Type check + CMF

# Format
brik64 fmt program.pcd                       # Format PCD source

# REPL
brik64 repl                                  # Interactive mode

# Self-verify
brik64 self-verify                           # Verify all 128 monomers

# catalog — list monomers
brik64 catalog list
brik64 catalog show <N>

# lift — reverse compile source to PCD (12 languages, SSA transform, 100% liftability)
brik64 lift <file>            # JS, TS, TSX, JSX, Python, Rust, C, C++, Go, COBOL, PHP, Java
brik64 lift app.tsx           # TSX/JSX supported
# SSA transform: variable reassignment (total = total + x) → SSA form automatically
# Two-tier certification: CORE (Phi_c=1) for core monomers, CONTRACT for extended

# Full roundtrip: lift → check → build → execute
brik64 lift calcPrice.js -o calcPrice.pcd
brik64 check calcPrice.pcd
brik64 build calcPrice.pcd -t python -o dist/
python3 dist/calcPrice.py

# policy — AI safety policy circuits
brik64 policy new --template <TEMPLATE>   # no_network, no_filesystem, memory_bound, sandbox, allow_all
brik64 policy verify <file.pcd>
brik64 policy list

# pkg — package manager
brik64 pkg init               # creates brik.toml
brik64 pkg add <package>
brik64 pkg publish            # publishes to registry.brik64.dev
brik64 pkg deps               # dependency tree

# self-hosting
brik64 self-host-status
brik64 self-certify-pcd-cli   # verify PCD CLI fixed-point (gen1==gen2)
```

---

## SDK Libraries (4.0.0-beta.1)

The CMF computes 7 metrics producing Φ_c (circuit closure):

| Metric | What it measures |
|--------|------------------|
| E_c | Computational energy (efficiency) |
| H_d | Hamiltonian distance (optimality) |
| S_d | Structural entropy (complexity) |
| C_s | Cyclomatic stability (robustness) |
| ETC | Entropic transfer coefficient (information flow) |
| ΔN | Complexity deviation (overhead) |
| Φ_c | Circuit closure — **1.0 = circuit is closed and correct** |

**Φ_c = 1 means:** all inputs consumed, all outputs produced, all paths verified.

---

## Policy Circuits (AI Guardrails)

Policy circuits are PCD programs that verify AI agent actions and return ALLOW/BLOCK:

```pcd
fn policy_rate_limit(count: i64, window_ms: i64, max: i64) -> i64 {
    let rate = div8(count, window_ms);
    let q = rate[0];
    return if (q > max) { 0 } else { 1 };
}

fn policy_budget(spent: i64, limit: i64) -> i64 {
    let remaining = sub8(limit, spent);
    return if (remaining > 0) { 1 } else { 0 };
}

fn policy_data_class(sensitivity: i64, dest_trust: i64) -> i64 {
    return if (dest_trust < sensitivity) { 0 } else { 1 };
}
```

Deploy as middleware:
```bash
# Python
pip install brik64         # v4.0.0-beta.1 — 128 monomers (64 core + 64 extended)
python3 -c "import brik64; print(dir(brik64.mc))"

# npm / Node.js
npm install brik64          # v4.0.0-beta.1 — 128 monomers, wrapping arithmetic
npx brik64 --version

# Rust
cargo add brik64            # v4.0.0-beta.1 — 128 monomers, wrapping arithmetic
```

> All SDKs use **wrapping arithmetic** (not saturating) as of v4.0.0.

---

## Installation (BETA 4.0.0-beta.2)

```bash
# Recommended — install script
curl -fsSL https://brik64.dev/install | sh

# npm
npm install -g brik64

# pip
pip install brik64

# Manual — Linux x86-64 (legacy artifact name during alpha transition)
curl -L https://github.com/brik64/brik64-dist-releases/releases/download/beta-4.0.0-beta.2/brikc-beta-linux-x86_64 -o brik64 && chmod +x brik64

# Manual — macOS Apple Silicon (M1/M2/M3/M4), legacy artifact name during alpha transition
curl -L .../brikc-beta-macos-arm64 -o brik64 && chmod +x brik64
xattr -d com.apple.quarantine brik64   # Remove macOS quarantine

# Manual — macOS Intel, legacy artifact name during alpha transition
curl -L .../brikc-beta-macos-intel -o brik64 && chmod +x brik64
xattr -d com.apple.quarantine brik64
```

**Platform support in BETA 4.0.0-beta.2:**
- ✅ Linux x86-64 (fully tested — ECO server)
- ✅ macOS Apple Silicon (ARM64)
- ✅ macOS Intel (x86-64)
- ✅ Linux ARM64

---

## Known BETA Limitations (beta.2)

1. **ADD8/MUL8 are 8-bit wrapping** — Use native `+`, `*` for accumulation >255
2. **DIV8 returns Tuple** — Access with `qr[0]`/`qr[1]`, NOT `let (q,r) = ...`
3. **MC_43.LEN works on String and Array** — returns byte count for String, element count for Array
4. **MC_48.HASH takes String** — NOT Array(U8); returns hex String NOT Array(U8,32)
5. **native target generates ELF x86-64** — Not usable directly on macOS arm64
6. **wasm32 target generates WAT text** — Needs wat2wasm to convert to binary
7. **while loops broken in native ELF** — SSA bug; use `loop(N) { if(cond){...} }` instead
8. **LSP protocol incomplete** — `brik64 lsp` starts but doesn't respond to JSON-RPC
9. **ENV in native ELF reads argv[1]** — In BIR interpreter reads actual env var
10. **Python backend**: variable `hex` renamed to `_brik_hex`; list indexing uses `[i]` not `.get(i)` (fixed beta.3)
11. **JS backend**: generated code requires `node file.js` to produce output (fixed in beta.3 — adds `execute()` call)

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

### You Are the Circuit Designer

The programmer defines domain bounds based on their problem context:
- Flight computer: velocity `[0, 900]` km/h, altitude `[0, 15000]` m
- Banking: transaction amount `[0.01, 1000000]`, account balance `[0, MAX_I64]`
- Temperature sensor: reading `[-273, 1000]` °C (absolute zero to furnace)

If a result falls outside the declared domain, the circuit does not close (Φ_c ≠ 1) and the program does not compile. This is not a bug — it is physics.

**Normal software:** calculates velocity = 100,000 km/s, stores it, crashes later.
**Digital Circuitality:** the program does not compile. The circuit is open.

### Precision Engineering

Domains are numeric ranges, not physical units. Precision depends on monomer choice:
- **U8/I64 (core):** exact integer arithmetic, no rounding, Φ_c = 1
- **F64 (extended):** IEEE 754 floats, has rounding errors, Φ_c = CONTRACT
- **Fixed-point pattern:** scale to integers (3.14 → 3140), compute exactly, scale back

Choose the right type for each calculation. If the result exceeds the range, the circuit doesn't close.

## BRIK-64 Registry & MCP (Platform Layer)

### What is the Registry?

The Registry transforms BRIK-64 from a verification technology into a **platform of certified reusable software circuits**. Certified PCD polymers become discoverable, composable, licensable, and governable assets.

**The monomers are the physics. The polymers are the product. The Registry is the market.**

### MCP Server: 2-Tool Minimalist Architecture

Following 2026 best practices (30 tools = confusion threshold, 100+ = guaranteed failure), the BRIK-64 MCP exposes **exactly 2 semantic tools** (~800 tokens) preserving 98% of context window.

#### Tool 1: `brik64.discover` (read-only)

All discovery operations in a single call. Server performs search, ranking, comparison, and compatibility check server-side.

```json
{
  "intent": "Find a rate-limiting policy circuit for financial transactions",
  "filters": {
    "domain": "finance",
    "criticality": "safety",
    "license": "commercial"
  },
  "operations": ["search", "compare", "check_compatibility", "get_lineage"]
}
```

#### Tool 2: `brik64.execute` (write operations)

All state-changing operations in a single call.

```json
{
  "action": "compose_and_register",
  "inputs": ["asset://brik64/policy_rate_limit@1.2.0", "asset://brik64/policy_budget@1.0.3"],
  "composition": "sequential",
  "metadata": { "name": "financial_guardrail", "domain": "finance" }
}
```

Actions: `compose_and_register`, `publish`, `acquire_license`, `export_compliance`, `deprecate`, `version`

### Reuse Before Create Workflow

```
1. Interpret objective
2. brik64.discover with intent + filters
3. Evaluate ranked results (server-side)
4. EXISTS? → brik64.execute { action: "acquire_license" }
5. COMPOSE? → brik64.execute { action: "compose_and_register" }
6. NOTHING? → Create from monomers, then register
```

### Inherited Certification

If A (Φ_c=1) and B (Φ_c=1) are registered, then A ⊗ B is automatically certified — no new proofs needed. 1,000 circuits → millions of certified compositions.

### Domain Packs

1. AI Guardrails — rate limiting, budget, data classification, privilege
2. Identity & Integrity — hashing, HMAC, signatures, provenance
3. Enterprise Workflow — approvals, segregation of duties, SLA
4. Financial Risk — spending controls, position limits, fraud rules
5. DevSecOps — artifact verification, release gates, dependency approval
6. Robotics — speed/force limits, geofencing, safe stop
7. Medical Safety — dosage limits, contraindication checks
8. Telecom — provisioning, routing policy, failover

### API Endpoints

| Domain | Key Endpoints |
|--------|---------------|
| Registry | `GET /assets`, `POST /search/advanced`, `GET /assets/{id}/lineage` |
| Composition | `POST /compose`, `POST /compose/register` |
| Marketplace | `GET /marketplace/listings`, `POST /marketplace/purchase` |
| Audit | `GET /audit/events`, `GET /compliance/export` |
