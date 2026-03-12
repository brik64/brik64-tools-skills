---
name: brik64-javascript
description: "Use the brik64 npm package (v2.0.0-beta.3) to install the brikc CLI or apply Digital Circuitality in JavaScript/TypeScript projects. Covers installation, all 64 monomers, EVA composition, async patterns, Node.js and browser usage. Use when writing JS/TS code with BRIK-64 libraries."
version: 2.0.0-beta.3
---

# BRIK-64 for JavaScript / TypeScript

Apply Digital Circuitality in your JavaScript and TypeScript projects.

**Docs:** https://brik64.dev/docs
**Package:** https://www.npmjs.com/package/brik64

---

## Installation

```bash
# Install CLI + SDK (downloads brikc native binary)
npm install -g brik64        # installs brikc, brikfmt, brikcheck commands
# pnpm add -g brik64
# yarn global add brik64

# Or use as a project dependency
npm install brik64
npx brikc --version          # → brikc 2.0.0-beta.3
```

Works in **Node.js** (≥ 16). Downloads native binary for your platform on postinstall.

---

## Import

```typescript
// ESM / TypeScript
import { mc, eva } from 'brik64';

// CommonJS
const { mc, eva } = require('brik64');

// Browser (CDN)
import { mc, eva } from 'https://cdn.brik64.dev/sdk/v2/index.js';
```

---

## Arithmetic (saturating — never throws, never overflows)

```typescript
import { mc } from 'brik64';

const sum   = mc.arithmetic.add8(200, 100);    // 255 (saturating)
const diff  = mc.arithmetic.sub8(10, 20);      // 0 (saturating)
const prod  = mc.arithmetic.mul8(20, 20);      // 255 (saturating)
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
import { eva } from 'brik64';

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

Using `brik64` applies Digital Circuitality **as a methodology** in your JS/TS code:

- ✅ Saturating arithmetic (no overflow bugs in bitwise operations)
- ✅ Formally specified crypto operations
- ✅ EVA composition patterns
- ✅ Better code structure and determinism

It does **not** give you:

- ❌ CMF verification (Φ_c computation)
- ❌ Auto-generated test suites from formal proof
- ❌ Registry certification badge

For formal certification → write in PCD and compile to JS: `brikc compile src/main.pcd --target js --emit-tests`
→ https://docs.brik64.dev/pcd/tutorial
