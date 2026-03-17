---
name: pcd-system
description: Complete PCD language reference + brikc CLI for BRIK-64 BETA 4.1.0-beta.1. Covers syntax, all 128 monomers (64 core + 64 extended, verified signatures), CMF debug, policy circuits, multi-target compilation, and the Registry/MCP platform layer (2-tool minimalist architecture). Use when writing .pcd files, using the brikc compiler, building policy circuits, or working with the BRIK-64 Registry/MCP.
triggers:
  - writing PCD programs
  - using brikc CLI
  - building AI safety policy circuits
  - debugging CMF errors
  - compiling to BIR/Rust/JS/Python/native/wasm32
  - using BRIK-64 Registry or MCP
  - discovering certified circuits
  - composing circuit assets
version: 4.0.0
---

# PCD System — BRIK-64 BETA 4.1.0-beta.1

Complete reference for writing PCD programs, using the brikc compiler, and working with the BRIK-64 Registry platform.

---

## Quick Start

```bash
# Install brikc (recommended)
curl -fsSL https://brik64.dev/install | sh

# Or via npm
npm install -g brik64

# Or via pip
pip install brik64

# Verify
brikc --version
```

---

## PCD Syntax

PCD (Printed Circuit Description) — programs are circuit schematics, not imperative scripts.

### Functions

```pcd
fn policy_budget(spent: i64, limit: i64) -> i64 {
    let remaining = sub8(limit, spent);
    let allowed = if (remaining > 0) { 1 } else { 0 };
    return allowed;
}
```

### Types

- `i64` — 64-bit signed integer (primary numeric type)
- `bool` — boolean
- `String` — UTF-8 string
- `Array(T)` — array of T
- `Tuple(T...)` — fixed-size tuple

### Control Flow

```pcd
// Conditional
let x = if (cond) { a } else { b };

// Bounded loop
loop(10) {
    // body executes 10 times
}

// While loop
while (condition) {
    // body
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

## The 128 Monomers (64 Core + 64 Extended)

### F0 — Arithmetic (MC_00–MC_07)

| MC | Name | Signature | Notes |
|----|------|-----------|-------|
| 00 | ADD8 | (i64, i64) → i64 | Wrapping 8-bit addition |
| 01 | SUB8 | (i64, i64) → i64 | Wrapping 8-bit subtraction |
| 02 | MUL8 | (i64, i64) → i64 | Wrapping 8-bit multiplication |
| 03 | DIV8 | (i64, i64) → Tuple(i64, i64) | Returns (quotient, remainder) |
| 04 | INC | i64 → i64 | Increment (wrapping) |
| 05 | DEC | i64 → i64 | Decrement (wrapping) |
| 06 | ABS | i64 → i64 | Absolute value |
| 07 | CLAMP | (i64, i64, i64) → i64 | Clamp to range |

### F1 — Logic (MC_08–MC_15)

| MC | Name | Signature | Notes |
|----|------|-----------|-------|
| 08 | AND8 | (i64, i64) → i64 | Bitwise AND |
| 09 | OR8 | (i64, i64) → i64 | Bitwise OR |
| 10 | XOR8 | (i64, i64) → i64 | Bitwise XOR |
| 11 | NOT8 | i64 → i64 | Bitwise NOT |
| 12 | SHL | (i64, i64) → i64 | Shift left |
| 13 | SHR | (i64, i64) → i64 | Shift right |
| 14 | ROL | (i64, i64) → i64 | Rotate left |
| 15 | ROR | (i64, i64) → i64 | Rotate right |

### F2 — Memory (MC_16–MC_23)

| MC | Name | Signature | Notes |
|----|------|-----------|-------|
| 16 | LOAD | i64 → i64 | Read from address |
| 17 | STORE | (i64, i64) → () | Write to address |
| 18 | PUSH | i64 → () | Push to stack |
| 19 | POP | () → i64 | Pop from stack |
| 20 | PEEK | () → i64 | Peek top |
| 21 | SWAP | () → () | Swap top 2 |
| 22 | DUP | () → i64 | Duplicate top |
| 23 | DROP | () → () | Discard top |

### F3 — Control (MC_24–MC_31)

| MC | Name | Signature | Notes |
|----|------|-----------|-------|
| 24 | IF | (bool, T, T) → T | Conditional select |
| 25 | LOOP | (i64, T) → Array(T) | Replicate N times |
| 26 | JUMP | i64 → i64 | Jump target |
| 27 | CALL | (i64, ...) → Array | Frame call |
| 28 | RET | T → T | Return value |
| 29 | BREAK | () → bool | Break signal |
| 30 | CONTINUE | () → bool | Continue signal |
| 31 | NOP | () → () | No operation |

### F4 — I/O (MC_32–MC_39)

| MC | Name | Signature | Notes |
|----|------|-----------|-------|
| 32 | READ | i64 → i64 | Read byte |
| 33 | WRITE | (i64, i64) → bool | Write byte |
| 34 | SEEK | (i64, i64) → bool | Seek position |
| 35 | FLUSH | () → bool | Flush stream |
| 36 | OPEN | String → i64 | Open handle |
| 37 | CLOSE | i64 → bool | Close handle |
| 38 | EOF | i64 → bool | End of file |
| 39 | ERR | i64 → i64 | Error code |

### F5 — String (MC_40–MC_47)

| MC | Name | Signature | Notes |
|----|------|-----------|-------|
| 40 | CONCAT | (String, String) → String | Concatenate |
| 41 | SPLIT | (String, String) → Array(String) | Split by delimiter |
| 42 | SUBSTR | (String, i64, i64) → String | Substring (UTF-8 safe) |
| 43 | LEN | String → i64 | Length (bytes for String) |
| 44 | UPPER | String → String | Uppercase |
| 45 | LOWER | String → String | Lowercase |
| 46 | TRIM | String → String | Trim whitespace |
| 47 | MATCH | (String, String) → bool | Regex match or contains |

### F6 — Crypto (MC_48–MC_55)

| MC | Name | Signature | Notes |
|----|------|-----------|-------|
| 48 | HASH | String → String | SHA-256 hex |
| 49 | ENCRYPT | (String, String, String) → String | AES-256-GCM |
| 50 | DECRYPT | (String, String, String) → String | AES-256-GCM |
| 51 | SIGN | (String, String) → String | Ed25519 |
| 52 | VERIFY | (String, String, String) → bool | Ed25519 |
| 53 | RNG | () → i64 | Deterministic ChaCha20 |
| 54 | KDF | (String, String, String) → String | HKDF |
| 55 | MAC | (String, String) → String | HMAC-SHA256 |

### F7 — System (MC_56–MC_63)

| MC | Name | Signature | Notes |
|----|------|-----------|-------|
| 56 | TIME | () → i64 | Unix timestamp |
| 57 | CPU | () → i64 | CPU usage % |
| 58 | MEM | () → i64 | Free memory bytes |
| 59 | DISK | () → i64 | Free disk bytes |
| 60 | NET | () → bool | Network available |
| 61 | PID | () → i64 | Process ID |
| 62 | UID | () → i64 | User ID |
| 63 | ENV | String → String | Environment variable |

### Extended Monomers (Φ_c = CONTRACT)

Extended monomers interact with external systems (network, filesystem, hardware). They operate under **CONTRACT closure** — runtime contracts enforce correctness rather than static Coq proofs. Use `MC_NN.NAME(args)` syntax in PCD.

### F8 — Float64 (MC_64–MC_71)

| MC | Name | Signature | Notes |
|----|------|-----------|-------|
| 64 | FADD | (f64, f64) → f64 | Floating-point addition |
| 65 | FSUB | (f64, f64) → f64 | Floating-point subtraction |
| 66 | FMUL | (f64, f64) → f64 | Floating-point multiplication |
| 67 | FDIV | (f64, f64) → f64 | Floating-point division (returns NaN on /0) |
| 68 | FABS | f64 → f64 | Absolute value |
| 69 | FNEG | f64 → f64 | Negate |
| 70 | FSQRT | f64 → f64 | Square root |
| 71 | FMOD | (f64, f64) → f64 | Floating-point modulo |

### F9 — Math (MC_72–MC_79)

| MC | Name | Signature | Notes |
|----|------|-----------|-------|
| 72 | SIN | f64 → f64 | Sine (radians) |
| 73 | COS | f64 → f64 | Cosine (radians) |
| 74 | TAN | f64 → f64 | Tangent (radians) |
| 75 | EXP | f64 → f64 | e^x |
| 76 | LN | f64 → f64 | Natural logarithm |
| 77 | LOG2 | f64 → f64 | Base-2 logarithm |
| 78 | POW | (f64, f64) → f64 | Power |
| 79 | CEIL | f64 → f64 | Ceiling |

### F10 — Network (MC_80–MC_87)

| MC | Name | Signature | Notes |
|----|------|-----------|-------|
| 80 | TCP_CONN | (String, i64) → i64 | Connect to host:port, returns handle |
| 81 | TCP_SEND | (i64, String) → bool | Send data on handle |
| 82 | TCP_RECV | (i64, i64) → String | Receive up to N bytes |
| 83 | TCP_CLOSE | i64 → bool | Close connection |
| 84 | UDP_SEND | (String, i64, String) → bool | Send UDP datagram |
| 85 | UDP_RECV | (i64, i64) → String | Receive UDP datagram |
| 86 | DNS_RESOLVE | String → String | Resolve hostname to IP |
| 87 | HTTP_REQ | (String, String, String) → String | HTTP request (method, url, body) |

### F11 — Graphics (MC_88–MC_95)

| MC | Name | Signature | Notes |
|----|------|-----------|-------|
| 88 | FB_CREATE | (i64, i64) → i64 | Create framebuffer (w, h) |
| 89 | FB_SET_PX | (i64, i64, i64, i64) → bool | Set pixel (fb, x, y, color) |
| 90 | FB_GET_PX | (i64, i64, i64) → i64 | Get pixel (fb, x, y) |
| 91 | FB_CLEAR | (i64, i64) → bool | Clear framebuffer to color |
| 92 | FB_BLIT | (i64, i64, i64, i64) → bool | Blit region |
| 93 | FB_LINE | (i64, i64, i64, i64, i64, i64) → bool | Draw line |
| 94 | FB_RECT | (i64, i64, i64, i64, i64, i64) → bool | Draw rectangle |
| 95 | FB_DIMS | i64 → Tuple(i64, i64) | Get framebuffer dimensions |

### F12 — Audio (MC_96–MC_103)

| MC | Name | Signature | Notes |
|----|------|-----------|-------|
| 96 | AUD_CREATE | (i64, i64) → i64 | Create audio buffer (rate, channels) |
| 97 | AUD_WRITE | (i64, Array(f64)) → bool | Write samples |
| 98 | AUD_READ | (i64, i64) → Array(f64) | Read N samples |
| 99 | AUD_MIX | (i64, i64) → i64 | Mix two buffers |
| 100 | AUD_GAIN | (i64, f64) → bool | Apply gain |
| 101 | AUD_LEN | i64 → i64 | Buffer length in samples |
| 102 | AUD_RATE | i64 → i64 | Sample rate |
| 103 | AUD_CHANS | i64 → i64 | Channel count |

### F13 — Filesystem+ (MC_104–MC_111)

| MC | Name | Signature | Notes |
|----|------|-----------|-------|
| 104 | FS_STAT | String → Tuple(i64, i64, i64) | File stat (size, mtime, mode) |
| 105 | FS_MKDIR | String → bool | Create directory |
| 106 | FS_RMDIR | String → bool | Remove directory |
| 107 | FS_DELETE | String → bool | Delete file |
| 108 | FS_RENAME | (String, String) → bool | Rename file |
| 109 | FS_LIST | String → Array(String) | List directory entries |
| 110 | FS_EXISTS | String → bool | Check existence |
| 111 | FS_COPY | (String, String) → bool | Copy file |

### F14 — Concurrency (MC_112–MC_119)

| MC | Name | Signature | Notes |
|----|------|-----------|-------|
| 112 | SPAWN | i64 → i64 | Spawn task, returns handle |
| 113 | JOIN | i64 → i64 | Join task, returns result |
| 114 | CHAN_NEW | () → i64 | Create channel |
| 115 | CHAN_SEND | (i64, i64) → bool | Send value on channel |
| 116 | CHAN_RECV | i64 → i64 | Receive value from channel |
| 117 | MUTEX_NEW | () → i64 | Create mutex |
| 118 | MUTEX_LOCK | i64 → bool | Lock mutex |
| 119 | MUTEX_UNLOCK | i64 → bool | Unlock mutex |

### F15 — Interop/FFI (MC_120–MC_127)

| MC | Name | Signature | Notes |
|----|------|-----------|-------|
| 120 | JSON_ENCODE | i64 → String | Encode value to JSON |
| 121 | JSON_DECODE | String → i64 | Decode JSON to value |
| 122 | FFI_CALL | (i64, String, Array(i64)) → i64 | Call foreign function |
| 123 | FFI_LOAD | String → i64 | Load shared library |
| 124 | FFI_FREE | i64 → bool | Free library handle |
| 125 | WASM_LOAD | String → i64 | Load WASM module |
| 126 | WASM_CALL | (i64, String, Array(i64)) → i64 | Call WASM export |
| 127 | WASM_FREE | i64 → bool | Free WASM module |

### PCD Syntax for Extended Monomers

```pcd
// Float64 operations
let sum = MC_64.FADD(1.0, 2.0);        // 3.0
let root = MC_70.FSQRT(16.0);          // 4.0

// Math operations
let angle = MC_72.SIN(3.14159);        // ≈ 0.0
let power = MC_78.POW(2.0, 10.0);      // 1024.0

// Network
let conn = MC_80.TCP_CONN("example.com", 443);
let resp = MC_87.HTTP_REQ("GET", "https://api.example.com/data", "");

// Filesystem+
let exists = MC_110.FS_EXISTS("/tmp/data.bin");
let files = MC_109.FS_LIST("/var/log");
```

> **Note:** Core monomers (MC_00–MC_63) are verified with Φ_c = 1 (Coq proofs). Extended monomers (MC_64–MC_127) operate under Φ_c = CONTRACT — runtime contracts enforce correctness for operations that interact with external systems.

---

## brikc CLI Commands

```bash
# Compile to target
brikc compile --target bir program.pcd      # BIR bytecode (default)
brikc compile --target rust program.pcd     # Rust source
brikc compile --target js program.pcd       # JavaScript
brikc compile --target python program.pcd   # Python
brikc compile --target native program.pcd   # ELF x86-64

# Run directly
brikc run program.pcd                       # Compile + execute BIR

# Check without compiling
brikc check program.pcd                     # Type check + CMF

# Format
brikc fmt program.pcd                       # Format PCD source

# REPL
brikc repl                                  # Interactive mode

# Self-verify
brikc self-verify                           # Verify all 128 monomers

# LSP
brikc lsp                                   # Start Language Server
```

---

## CMF Metrics (Coherence Metric Framework)

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
brikc compile --target js guardrails.pcd    # Node.js / LangChain
brikc compile --target python guardrails.pcd # Python agents
brikc compile --target rust guardrails.pcd   # High-performance
```

---

## Installation

```bash
# Recommended — install script
curl -fsSL https://brik64.dev/install | sh

# npm
npm install -g brik64

# pip
pip install brik64

# Manual — Linux x86-64
curl -L https://github.com/brik64/brik64-dist-releases/releases/download/beta-4.1.0-beta.1/brikc-beta-linux-x86_64 -o brikc && chmod +x brikc

# Manual — macOS Apple Silicon (M1/M2/M3/M4)
curl -L .../brikc-beta-macos-arm64 -o brikc && chmod +x brikc
xattr -d com.apple.quarantine brikc   # Remove macOS quarantine

# Manual — macOS Intel
curl -L .../brikc-beta-macos-intel -o brikc && chmod +x brikc
xattr -d com.apple.quarantine brikc
```

**Platform support in BETA 4.1.0-beta.1:**
- ✅ Linux x86-64 (fully tested — ECO server)
- ✅ macOS Apple Silicon (ARM64)
- ✅ macOS Intel (x86-64)
- ✅ Linux ARM64

---

## Known BETA Limitations (beta.3)

1. **ADD8/MUL8 are 8-bit wrapping** — Use native `+`, `*` for accumulation >255
2. **DIV8 returns Tuple** — Access with `qr[0]`/`qr[1]`, NOT `let (q,r) = ...`
3. **MC_43.LEN works on String and Array** — returns byte count for String, element count for Array
4. **MC_48.HASH takes String** — NOT Array(U8); returns hex String NOT Array(U8,32)
5. **native target generates ELF x86-64** — Not usable directly on macOS arm64
6. **wasm32 target generates WAT text** — Needs wat2wasm to convert to binary
7. **while loops broken in native ELF** — SSA bug; use `loop(N) { if(cond){...} }` instead
8. **LSP protocol incomplete** — `brikc lsp` starts but doesn't respond to JSON-RPC
9. **ENV in native ELF reads argv[1]** — In BIR interpreter reads actual env var
10. **Python backend**: variable `hex` renamed to `_brik_hex`; list indexing uses `[i]` not `.get(i)` (fixed beta.3)
11. **JS backend**: generated code requires `node file.js` to produce output (fixed in beta.3 — adds `execute()` call)

---

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
