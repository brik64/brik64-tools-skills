---
name: digital-circuitality
description: "Teaches the Digital Circuitality practice — how to think and design programs as bounded, inspectable circuits in any language. No PCD or public CLI required. Check the current brik64 skill and docs.brik64.com before making CLI, SDK, catalog, certification, or formal-proof claims."
---

# Digital Circuitality — Circuit Thinking for Any Language

Apply the engineering discipline of Digital Circuitality to your code regardless
of language. This skill is practice guidance: it helps agents design bounded,
inspectable logic. It is not itself a certification surface.

**Full reference:** https://docs.brik64.com/theory/digital-circuitality

---

## The Core Idea

Modern software fails because it runs as **open circuits** — no boundaries, no formal closure, unbounded failure modes. A null pointer opens a path that was never designed. An integer overflow routes execution through undefined behavior. Tests check some paths; the undefined ones remain.

**Digital Circuitality** is the discipline of writing software the way hardware engineers design circuits: every input domain bounded, every operation finite, every output range defined, every path closed.

```
Hardware engineer designs:      You design:
─────────────────────────       ─────────────────────────
Logic gate (NAND, AND, OR)  →   Monomer (add8, hash, concat)
Circuit board               →   Function (polymer)
Signal integrity            →   Φ_c = 1 (circuit closed)
Spec-driven review          →   Deterministic, bounded, inspectable
```

---

## The Three Principles

### 1. Finite Operations (Monomers)

Every operation in your program should have a **defined domain, defined range, and no undefined behavior**. No silent saturation without a contract, no implicit coercions, no "it works on my machine."

In practice:
- Use saturating arithmetic instead of wrapping or panicking
- Every function has a defined return type for every input
- No nullable returns without explicit handling
- No unbounded recursion — loops have explicit bounds

**Example: arithmetic that cannot overflow**

```rust
// Open circuit — undefined on overflow in debug, wrapping in release
let total = a + b;

// Closed circuit — saturating, always defined
let total = a.saturating_add(b);  // or: use brik64-core::mc::add8(a, b)
```

```python
# Open circuit — silent int overflow not an issue in Python, but float is
total = float('inf') + float('nan')  # NaN — undefined

# Closed circuit — guard the domain
if math.isfinite(a) and math.isfinite(b):
    total = a + b
else:
    total = 0.0  # defined fallback
```

### 2. Compositional Determinism (EVA Algebra)

Programs are compositions of operations. The EVA algebra defines three valid composition forms:

| Operator | Symbol | Meaning | Rule |
|----------|--------|---------|------|
| Sequential | `⊗` | A then B (B uses A's output) | Output type of A must match input type of B |
| Parallel | `∥` | A and B independently | A and B must not share mutable state |
| Conditional | `⊕` | A or B based on condition | Both branches must produce the same type |

**In any language, you already use these operators.** The discipline is making them explicit:

```rust
// Sequential (⊗): trim → hash → encode
let result = encode(hash(trim(input)));

// Parallel (∥): independent operations
let (hash_a, hash_b) = rayon::join(|| hash(a), || hash(b)); // isolated!

// Conditional (⊕): both branches return String
let label = if score > 50 { "pass".to_string() } else { "fail".to_string() };
```

```python
# Sequential: parse → validate → transform
result = transform(validate(parse(raw_input)))

# Parallel: truly independent, no shared mutation
import concurrent.futures
with concurrent.futures.ThreadPoolExecutor() as ex:
    f1 = ex.submit(process_a, data_a)
    f2 = ex.submit(process_b, data_b)   # data_b ≠ data_a
    result_a, result_b = f1.result(), f2.result()

# Conditional: same output type in both branches
status = "ok" if response.status_code == 200 else "error"
```

```javascript
// Sequential: fetch → parse → validate
const result = validate(parse(await fetch(url).then(r => r.json())));

// Conditional: both branches produce the same type
const message = isError ? `Error: ${err.message}` : `Success: ${data.id}`;
```

**Key rule:** A conditional (`if/else`) is only closed if **both branches produce a value of the same type**. Missing `else` = open circuit.

### 3. Circuit Closure (Φ_c = 1)

A function is a **closed circuit** when:
- Every input is consumed (no unused parameters)
- Every branch returns a value (no implicit `undefined`/`None`/`null` returns)
- All loops terminate (no unbounded iteration)
- No data is silently discarded

**Closed circuit checklist for any function:**

```
□ Every parameter is used — or explicitly ignored (prefixed with _)
□ Every if/else has an else branch that returns a value
□ Every match/switch handles all cases (including default/wildcard)
□ Every loop has a bound or an exit condition that is provably reachable
□ The function returns a value on ALL code paths
□ No side effects that escape the function boundary without documentation
```

---

## Public Beta Boundary

Current public agent guidance lives in the `brik64` skill and
https://docs.brik64.com. Do not present lifter, multi-target compilation,
distribution catalog, formal certification, or SDK package behavior as current public
capabilities unless current release evidence supports the exact statement.

Use this skill for design review and code-structure discipline. Use the
`brik64` skill for current CLI workflows.

## Historical Lifter Notes

Older drafts described a source-to-PCD lifter workflow:

```bash
brik lift app.ts
brik lift main.py
brik lift lib.rs
```

Treat these as roadmap/reference notes until the current public CLI and docs
publish the exact commands.

### Historical Closure Language

| Tier | Condition | Meaning |
|------|-----------|---------|
| **Core closure** | bounded local logic | deterministic review target |
| **Contract boundary** | external dependency | explicit runtime assumption |

## Operation Families As Design Vocabulary

Use operation families as a vocabulary for design review. Do not treat this
table as the current public SDK catalog or a formal-proof claim.

| Family | Operations | Circuit Design Principle |
|--------|-----------|--------------------------|
| F0: Arithmetic | add8, sub8, mul8, div8, mod8, neg8, abs8, pow8 | Saturating (never overflow), always defined |
| F1: Logic | and8, or8, xor8, not8, shl, shr, rotl, rotr | Bitwise — exact, no surprises |
| F2: Memory | load, store, alloc, free, copy, swap, cas, fence | Explicit lifecycle — no implicit aliasing |
| F3: Control | if, jump, call, ret, loop, break, cont, halt | Bounded branching, explicit termination |
| F4: I/O | read, write, open, close, seek, stat, poll, flush | Explicit handles, paired open/close |
| F5: String | concat, split, substr, len, upper, char_at, trim, match | Immutable, always returns valid string |
| F6: Crypto | hash, hmac, aes_enc, aes_dec, sha256, rand, sign, verify | Typed inputs/outputs, no silent failures |
| F7: System | time, sleep, env, exit, pid, signal, mmap, sysinfo | Explicit system calls, typed results |

### External Operation Families

External operations such as filesystem, network, graphics, audio, concurrency,
and interop need explicit contracts. In public writing, describe them as
contract-bound design areas unless a current standard or release publishes a
stronger statement.

The same circuit thinking applies: bound your domains, pair your resources, handle all branches.

---

## Practical Application by Language

### Rust

```rust
use brik64_core::{mc, eva};

// ✅ Closed: saturating arithmetic, never panics
let sum = mc::arithmetic::add8(200, 100);   // 255
let (q, r) = mc::arithmetic::div8(17, 5);  // (3, 2) — no division-by-zero panic

// ✅ Closed: EVA sequential pipeline
let pipeline = eva::seq(
    |x| mc::arithmetic::add8(x, 10),
    |x| mc::arithmetic::mod8(x, 7),
);

// ✅ Closed without library: explicit bounds, all paths return
fn process(data: &[u8]) -> Result<String, String> {
    if data.is_empty() {
        return Err("empty input".to_string());  // all paths return
    }
    let trimmed = data.iter().take(256).copied().collect::<Vec<_>>();  // bounded
    Ok(format!("{:?}", trimmed))
}
```

### Python

```python
from brik64 import mc, eva

# ✅ Saturating arithmetic
total = mc.arithmetic.add8(200, 100)    # 255

# ✅ Pipeline composition
pipeline = eva.pipeline(
    lambda x: mc.arithmetic.add8(x, 10),
    lambda x: mc.arithmetic.mod8(x, 7),
)

# ✅ Closed without library: every branch returns
def classify(score: int) -> str:
    if score >= 90:
        return "A"
    elif score >= 80:
        return "B"
    elif score >= 70:
        return "C"
    else:
        return "F"          # wildcard — circuit closed
```

### JavaScript / TypeScript

```typescript
import { mc, eva } from '@brik64/core';

// ✅ Saturating arithmetic
const sum = mc.arithmetic.add8(200, 100);   // 255

// ✅ Pipeline
const pipeline = eva.seq(
    (x: number) => mc.arithmetic.add8(x, 10),
    (x: number) => mc.arithmetic.mod8(x, 7),
);

// ✅ Closed without library: ternary always produces a value
const status: string = response.ok ? "success" : "error";

// ✅ No implicit undefined — explicit exhaustive match via Map
const handlers = new Map<string, () => string>([
    ["add", () => String(a + b)],
    ["sub", () => String(a - b)],
]);
const result = handlers.get(cmd)?.() ?? "unknown command";  // fallback explicit
```

---

## Design Checklist: Is Your Function a Closed Circuit?

Before finalizing any function, verify:

```
□ Every branch has a return value (no implicit void in typed contexts)
□ Conditional branches produce the same type
□ No unbounded loops (while true without proven exit)
□ No null/undefined returns without the caller handling them
□ No operations that silently discard data (lost errors, dropped values)
□ No global mutable state mutated inside the function without explicit contract
□ Division by zero guarded (check divisor ≠ 0 before div)
□ Array bounds guarded (check index < length before access)
□ Error cases handled (not silently swallowed)
```

---


## Closure Domains

Every monomer declares its domain — the bounded set of valid inputs and outputs.
This is what makes Φ_c = 1 possible.

- **Range**: `[0, 255]` for u8 operations
- **Set**: `{true, false}` for boolean operations  
- **Bounded**: predicate on finite domain (e.g., even numbers in [0,100])
- **Product**: cartesian product for multi-input operations

Without bounded domains, the design remains open-ended. The domain is the
design boundary.

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
- **Integer arithmetic:** exact within declared bounds
- **Float arithmetic:** requires explicit rounding and NaN/Inf handling
- **Fixed-point pattern:** scale to integers (3.14 → 3140), compute exactly, scale back

Choose the right type for each calculation. If the result exceeds the range, the circuit doesn't close.

### Fixed-Point Math Pattern

Mathematical functions can often be represented with fixed-point patterns when
precision is declared explicitly.

The designer declares the precision:
```
domain pi_3:  Range [3141, 3142];           // π with 3 decimals
domain pi_6:  Range [3141592, 3141593];     // π with 6 decimals
domain pi_15: Range [3141592653589793, ...]; // π with 15 decimals
```

Like choosing a 12-bit vs 16-bit ADC — the engineer decides the resolution. The "error" is not accidental IEEE 754 rounding. It's the precision you declared in your domain.

Result: precision becomes a visible engineering choice instead of an accidental
runtime side effect.

## Practice vs. Product Evidence

| | Digital Circuitality (this skill) | BRIK64 product evidence |
|---|---|---|
| **Language** | Any (Rust, Python, JS, Go...) | Current docs and release artifacts |
| **Verification** | discipline + code review + tests | bounded CLI reports or evidence packs when available |
| **Public claims** | practice only | exact artifact/version scope |
| **Value** | clearer boundaries and fewer hidden assumptions | traceable operational evidence |

> Use circuit thinking everywhere. Use current BRIK64 docs and release evidence
> when a public claim needs product support.
