---
name: brik64-javascript
description: "Historical JavaScript/TypeScript Digital Circuitality patterns for BRIK64 work. Check docs.brik64.com and the current public brik64 skill before installing packages or making SDK, certification, catalog, or extended-operation claims."
version: 0.1.0-beta.11-public-reference
---

# BRIK-64 for JavaScript / TypeScript

Apply Digital Circuitality patterns in JavaScript and TypeScript projects.
Check https://docs.brik64.com and the current `brik64` skill before presenting
package availability, SDK exports, or CLI behavior as current public truth.

**Docs:** https://docs.brik64.com
**Package:** https://www.npmjs.com/package/@brik64/core

---

## Installation

```bash
# Current public CLI beta
npm install @brik64/core@0.1.0-beta.11
node --version

# Historical SDK examples below may reference older packages or APIs.
# Verify current package availability before using them in public instructions.
```

Do not assume native binary download behavior unless the current package proves
it.

---

## Import

```typescript
// ESM / TypeScript
import { mc, eva } from '@brik64/core';

// CommonJS
const { mc, eva } = require('@brik64/core');

// Browser (CDN)
import { mc, eva } from 'https://cdn.brik64.dev/sdk/v2/index.js';
```

---

## Historical SDK Operation Patterns

The examples below are design patterns for bounded operations. Treat package
exports and operation names as historical reference unless current docs confirm
them.

## Arithmetic (wrapping — never throws, wraps at 256)

```typescript
import { mc } from '@brik64/core';

const sum   = mc.arithmetic.add8(200, 100);    // 44 (wrapping: 300 % 256)
const diff  = mc.arithmetic.sub8(10, 20);      // 246 (wrapping)
const prod  = mc.arithmetic.mul8(20, 20);      // 144 (wrapping: 400 % 256)
const [q,r] = mc.arithmetic.div8(17, 5);       // [3, 2] — always array
const rem   = mc.arithmetic.mod8(17, 5);       // 2
const neg   = mc.arithmetic.neg8(1);           // 255
const pow   = mc.arithmetic.pow8(2, 7);        // 128 (saturating)
```

> `div8` never throws — returns `[0, 0]` for division by zero.

---

## Logic (bitwise)

```typescript
const a = mc.logic.and8(0xFF, 0x0F);    // 15
const o = mc.logic.or8(0xF0, 0x0F);     // 255
const x = mc.logic.xor8(0xAA, 0x55);    // 255
const n = mc.logic.not8(0xFF);          // 0
const s = mc.logic.shl(1, 3);           // 8
const r = mc.logic.shr(16, 2);          // 4
```

---

## String

```typescript
const joined  = mc.string.concat("hello", " world");
const parts   = mc.string.split("a,b,c", ",");        // string[]
const sub     = mc.string.substr("hello", 1, 3);       // "ell"
const n       = mc.string.len("hello");                // 5
const up      = mc.string.upper("hello");              // "HELLO"
const ch      = mc.string.charAt("hello", 1);          // 101 (code of 'e')
const trimmed = mc.string.trim("  hello  ");           // "hello"
```

---

## Crypto (async)

```typescript
const hash  = await mc.crypto.sha256(new TextEncoder().encode("hello"));
// Uint8Array(32)

const hmac  = await mc.crypto.hmacSha256(key, new TextEncoder().encode("msg"));
const enc   = await mc.crypto.aes256Enc(key, iv, data);
const dec   = await mc.crypto.aes256Dec(key, iv, enc);
const rand  = await mc.crypto.randBytes(32);   // Uint8Array(32)

// Ed25519
const keypair = await mc.crypto.ed25519Keygen();
const sig     = await mc.crypto.sign(keypair.privateKey, data);
const valid   = await mc.crypto.verify(keypair.publicKey, data, sig); // true
```

---

## EVA Composition

```typescript
import { eva } from '@brik64/core';

// Sequential (⊗): output of A → input of B
const pipeline = eva.seq(
    (x: number) => mc.arithmetic.add8(x, 10),
    (x: number) => mc.arithmetic.mod8(x, 7),
);
const result = pipeline(250);   // 3

// Multi-step pipeline
const process = eva.pipeline([
    (x: number) => mc.arithmetic.add8(x, 5),
    (x: number) => mc.arithmetic.mul8(x, 2),
    (x: number) => mc.arithmetic.mod8(x, 100),
]);

// Parallel (∥): both must be independent
const [r1, r2] = await eva.par(
    () => mc.crypto.sha256(dataA),
    () => mc.crypto.sha256(dataB),  // independent → parallel
);

// Conditional (⊕): both branches must return same type
const label = eva.cond(
    score > 50,
    () => "pass",
    () => "fail",
);
```

---

## Integration Patterns

### Safe arithmetic in calculations

```typescript
// Replace: const total = a + b;  (can silently overflow in bitwise ops)
// With:
const total = mc.arithmetic.add8(a, b);  // saturating, always safe

// Hashing pipeline
const digest = mc.string.concat(
    Buffer.from(await mc.crypto.sha256(data)).toString('hex'),
    "-v2"
);
```

### Circuit-closed function (methodology without library)

```typescript
// Apply circuit thinking without the library
function classify(score: number): string {
    if (score < 0 || score > 100) {
        return "invalid";           // guard domain
    }
    if (score >= 90) return "A";    // all branches return
    if (score >= 80) return "B";
    if (score >= 70) return "C";
    return "F";                     // wildcard — Φ_c = 1
}

// No implicit undefined returns: use explicit Result pattern
function safeParse(json: string): { ok: true; data: unknown } | { ok: false; error: string } {
    try {
        return { ok: true, data: JSON.parse(json) };
    } catch (e) {
        return { ok: false, error: String(e) };  // both branches return same shape
    }
}
```

---

## Important Distinction

Using these patterns applies Digital Circuitality **as a methodology** in your
JS/TS code:

- ✅ Saturating arithmetic (no overflow bugs in bitwise operations)
- ✅ explicit crypto operation boundaries
- ✅ composition patterns
- ✅ Better code structure and determinism

It does **not** give you:

- CMF verification claims
- auto-generated proof/test claims
- catalog or certification badge claims

For PCD guidance, use the current `brik64` skill and docs.brik64.com.

---


## Closure Domains

Every monomer declares its domain — the bounded set of valid inputs and outputs.
This is what makes Φ_c = 1 possible.

- **Range**: `[0, 255]` for u8 operations
- **Set**: `{true, false}` for boolean operations  
- **Bounded**: predicate on finite domain (e.g., even numbers in [0,100])
- **Product**: cartesian product for multi-input operations

Without bounded domains, the design remains open-ended. Treat domain notes here
as methodology guidance, not as a public certification claim.

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
- **U8/I64 (core):** exact integer arithmetic, no rounding, Φ_c = 1
- **F64 (extended):** IEEE 754 floats, has rounding errors, Φ_c = CONTRACT
- **Fixed-point pattern:** scale to integers (3.14 → 3140), compute exactly, scale back

Choose the right type for each calculation. If the result exceeds the range, the circuit doesn't close.

## Historical Extended Operation Notes

Older drafts referenced extended operation families. Treat those notes as
roadmap or historical material unless a current public release and docs page
publish the exact SDK surface.

### Float64 & Math

```typescript
import { mc } from '@brik64/core';

// Float64 (F8)
const sum  = mc.float64.fadd(1.5, 2.3);     // 3.8
const root = mc.float64.fsqrt(16.0);         // 4.0
const abs  = mc.float64.fabs(-3.14);         // 3.14

// Math (F9)
const sine   = mc.math.sin(Math.PI / 2);    // 1.0
const cosine = mc.math.cos(0.0);            // 1.0
const power  = mc.math.pow(2.0, 10.0);      // 1024.0
const log    = mc.math.ln(Math.E);           // 1.0
const ceil   = mc.math.ceil(3.2);            // 4.0
```

### Other Extended Families

```typescript
// Network (F10)
const resp = await mc.network.httpReq("GET", "https://api.example.com/data", "");

// Filesystem+ (F13)
const exists = mc.fs.fsExists("/tmp/data.json");
const files  = mc.fs.fsList("/var/log");

// Interop/FFI (F15)
const json = mc.interop.jsonEncode(value);
const obj  = mc.interop.jsonDecode('{"key": 42}');
```

> Current public agent guidance lives in the `brik64` skill and
> https://docs.brik64.com.
