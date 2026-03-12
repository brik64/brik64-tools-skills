---
name: brik64-python
description: "Use the brik64 Python package (v2.0.0) to apply Digital Circuitality in Python projects. Covers installation, all 64 monomers, EVA composition, and integration patterns. Use when writing Python code with BRIK-64 libraries."
version: 2.0.0
---

# BRIK-64 for Python

Apply Digital Circuitality in your Python projects.

**Docs:** https://docs.brik64.dev/installation
**Package:** https://pypi.org/project/brik64/

---

## Installation

```bash
pip install brik64
# uv add brik64
# poetry add brik64
```

Python 3.10+. Zero dependencies.

---

## Import

```python
from brik64 import mc, eva
# or:
from brik64.mc import arithmetic, logic, string, crypto, system
from brik64.eva import seq, par, cond, pipeline
```

---

## Arithmetic (saturating — never raises, never overflows)

```python
from brik64.mc import arithmetic

total     = arithmetic.add8(200, 100)      # 255 (saturating)
diff      = arithmetic.sub8(10, 20)        # 0 (saturating)
product   = arithmetic.mul8(20, 20)        # 255 (saturating)
q, r      = arithmetic.div8(17, 5)         # (3, 2) — always tuple
remainder = arithmetic.mod8(17, 5)         # 2
neg       = arithmetic.neg8(1)             # 255
power     = arithmetic.pow8(2, 7)          # 128 (saturating)
```

> `div8` never raises — returns `(0, 0)` for division by zero.

---

## Logic (bitwise)

```python
from brik64.mc import logic

a = logic.and8(0xFF, 0x0F)    # 15
o = logic.or8(0xF0, 0x0F)     # 255
x = logic.xor8(0xAA, 0x55)    # 255
n = logic.not8(0xFF)          # 0
s = logic.shl(1, 3)           # 8
r = logic.shr(16, 2)          # 4
```

---

## String

```python
from brik64.mc import string as s

joined  = s.concat("hello", " world")
parts   = s.split("a,b,c", ",")           # list[str]
sub     = s.substr("hello", 1, 3)         # "ell"
n       = s.len("hello")                  # 5
up      = s.upper("hello")               # "HELLO"
ch      = s.char_at("hello", 1)           # 101 (ord of 'e')
trimmed = s.trim("  hello  ")             # "hello"
```

---

## Crypto

```python
from brik64.mc import crypto

digest = crypto.sha256(b"hello world")      # bytes (32)
hmac   = crypto.hmac_sha256(key, b"msg")    # bytes (32)
enc    = crypto.aes256_enc(key, iv, data)   # bytes
dec    = crypto.aes256_dec(key, iv, enc)    # bytes
rand   = crypto.rand_bytes(32)              # bytes (32)

# Ed25519
private_key, public_key = crypto.ed25519_keygen()
sig   = crypto.sign(private_key, b"message")
valid = crypto.verify(public_key, b"message", sig)  # True
```

---

## EVA Composition

```python
from brik64 import eva
from brik64.mc import arithmetic

# Sequential (⊗)
process = eva.seq(
    lambda x: arithmetic.add8(x, 10),
    lambda x: arithmetic.mod8(x, 7),
)
result = process(250)   # 3

# Multi-step pipeline
transform = eva.pipeline([
    lambda x: arithmetic.add8(x, 5),
    lambda x: arithmetic.mul8(x, 2),
    lambda x: arithmetic.mod8(x, 100),
])

# Parallel (∥) — both must be independent
import concurrent.futures
r1, r2 = eva.par(
    lambda: crypto.sha256(data_a),
    lambda: crypto.sha256(data_b),
)

# Conditional (⊕)
label = eva.cond(
    score > 50,
    lambda: "pass",
    lambda: "fail",
)
```

---

## Integration Patterns

### Circuit-closed function (methodology without library)

```python
# Apply circuit thinking — every branch returns, domain guarded
def classify(score: int) -> str:
    if not (0 <= score <= 100):
        return "invalid"       # guard domain
    if score >= 90: return "A"
    if score >= 80: return "B"
    if score >= 70: return "C"
    return "F"                 # wildcard — Φ_c = 1

# No silent None returns — explicit Result pattern
from typing import Union

def safe_divide(a: int, b: int) -> Union[int, str]:
    if b == 0:
        return "division by zero"   # all branches return same type family
    q, _ = arithmetic.div8(a, b)
    return q
```

---

## Important Distinction

Using `brik64` applies Digital Circuitality **as a methodology** in your Python code:

- ✅ Saturating arithmetic (no overflow surprises)
- ✅ Formally specified crypto operations
- ✅ EVA composition patterns

It does **not** give you:

- ❌ CMF verification (Φ_c)
- ❌ Auto-generated test suites
- ❌ Registry certification

For formal certification → write in PCD and compile to Python: `brikc compile src/main.pcd --target py --emit-tests`
→ https://docs.brik64.dev/pcd/tutorial
