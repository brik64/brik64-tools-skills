---
name: brik64-python
description: "Historical Python Digital Circuitality patterns for BRIK64 work. Check docs.brik64.com and the current public brik64 skill before installing packages or making SDK, certification, catalog, or extended-operation claims."
version: 0.1.0-beta.11-public-reference
---

# BRIK-64 for Python

Apply Digital Circuitality patterns in Python projects. Check
https://docs.brik64.com and the current `brik64` skill before presenting package
availability, SDK exports, or CLI behavior as current public truth.

**Docs:** https://docs.brik64.com
**Package:** https://pypi.org/project/brik64/0.1.0b11/

---

## Installation

```bash
# Current public CLI beta
curl -fsSL https://brik64.com/cli/install.sh | bash
brik64 --version

# Current public Python SDK beta
pip install brik64==0.1.0b11
```

Do not treat package installation as CLI installation or a workspace workflow.

---

## Import

```python
from brik64 import mc, eva
# or:
from brik64.mc import arithmetic, logic, string, crypto, system
from brik64.eva import seq, par, cond, pipeline
```

---

## Historical SDK Operation Patterns

The examples below are design patterns for bounded operations. Check current
docs and package exports before using them in public instructions.

## Arithmetic (wrapping — never raises, wraps at 256)

```python
from brik64.mc import arithmetic

total     = arithmetic.add8(200, 100)      # 44 (wrapping: 300 % 256)
diff      = arithmetic.sub8(10, 20)        # 246 (wrapping)
product   = arithmetic.mul8(20, 20)        # 144 (wrapping: 400 % 256)
q, r      = arithmetic.div8(17, 5)         # (3, 2) — always tuple
remainder = arithmetic.mod8(17, 5)         # 2
neg       = arithmetic.neg8(1)             # 255
power     = arithmetic.pow8(2, 7)          # 128 (saturating)
```

> Check the installed beta11 package for exact division-by-zero behavior before
> publishing behavior-sensitive examples.

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
    return "F"                 # wildcard branch, explicit fallback

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

- ✅ Bounded arithmetic examples
- ✅ Explicit crypto operation boundaries
- ✅ EVA composition patterns

It does **not** give you:

- formal verification claims
- auto-generated proof/test claims
- catalog or certification badge claims

For PCD guidance, use the current `brik64` skill and docs.brik64.com.

---


## Closure Domains

Every monomer declares its domain: the bounded set of valid inputs and outputs.
This is methodology guidance for keeping examples explicit and reviewable.

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
- **U8/I64-style examples:** exact integer arithmetic, no rounding
- **F64-style examples:** IEEE 754 floats, with normal floating-point rounding
- **Fixed-point pattern:** scale to integers (3.14 → 3140), compute exactly, scale back

Choose the right type for each calculation. If the result exceeds the range, the circuit doesn't close.

## Historical Extended Operation Notes

Older drafts referenced extended operation families. Treat those notes as
roadmap or historical material unless a current public release and docs page
publish the exact SDK surface.

### Float64 & Math

```python
from brik64.mc import float64, math as bmath

# Float64 (F8)
total = float64.fadd(1.5, 2.3)        # 3.8
root  = float64.fsqrt(16.0)           # 4.0
fabs  = float64.fabs(-3.14)           # 3.14

# Math (F9)
import math
sine   = bmath.sin(math.pi / 2)       # 1.0
cosine = bmath.cos(0.0)               # 1.0
power  = bmath.pow(2.0, 10.0)         # 1024.0
log    = bmath.ln(math.e)             # 1.0
ceil   = bmath.ceil(3.2)              # 4.0
```

### Other Extended Families

```python
from brik64.mc import network, fs, interop

# Network (F10)
resp = network.http_req("GET", "https://api.example.com/data", "")

# Filesystem+ (F13)
exists = fs.fs_exists("/tmp/data.json")
files  = fs.fs_list("/var/log")

# Interop/FFI (F15)
json_str = interop.json_encode(value)
obj      = interop.json_decode('{"key": 42}')
```

> Current public agent guidance lives in the `brik64` skill and
> https://docs.brik64.com.
