---
name: pcd-developer
description: "Expert PCD (Printed Circuit Description) developer for BRIK-64. Use when writing, reviewing, or modifying any .pcd program. Covers all 64 monomers, EVA algebra composition, CMF certification (Φ_c = 1), stdlib usage, and idiomatic patterns."
---

# PCD Developer — BRIK-64

Write, review, and optimize PCD programs with expert-level knowledge of all 64 monomers, EVA algebra, CMF certification, and idiomatic patterns.

**Documentation:** https://docs.brik64.dev

---

## What is PCD

PCD (Printed Circuit Description) is a formally verified programming language where programs are circuits. Every valid PCD program has **Φ_c = 1** — no dead branches, no unreachable code, no undefined flows. Programs that fail certification do not compile.

## Program Structure

Every PCD program is a named circuit block with a mandatory `OUTPUT`:

```pcd
PC circuit_name {
    // logic here
    OUTPUT result;
}
```

Functions are defined with `fn`:

```pcd
fn add(a, b) {
    return a + b;
}
```

## Variables and Types

All bindings are immutable (SSA). Rebinding creates a new slot:

```pcd
let x = 10;
let x = x + 1;    // new SSA slot, x is now 11
```

**Types** (inferred, no annotations needed):
- `i64` — default integer
- `bool` — `true`/`false`
- `string` — UTF-8 immutable text
- `array` — `[1, 2, 3]` (homogeneous, value semantics)
- `tuple` — `(a, b)` (heterogeneous, fixed size)
- `struct` — named fields, value semantics
- `closure` — `fn(x) { x + 1 }`

## The 64 Monomers — Quick Reference

Call monomers with `MC_XX.NAME(args)`. All return `Value::I64` except `MC_03.DIV8` which returns a tuple.

### Family 0 — Arithmetic (MC_00–MC_07)

```pcd
let sum  = MC_00.ADD8(a, b);        // a + b
let diff = MC_01.SUB8(a, b);        // a - b
let prod = MC_02.MUL8(a, b);        // a * b
let (q, r) = MC_03.DIV8(a, b);     // MUST destructure! Returns (quotient, remainder)
let rem  = MC_04.MOD8(a, b);        // a % b
let neg  = MC_05.NEG8(a);           // -a
let abs  = MC_06.ABS8(a);           // |a|
let pow  = MC_07.POW8(base, exp);   // base^exp
```

### Family 1 — Logic (MC_08–MC_15)

```pcd
let and_r = MC_08.AND8(a, b);      // bitwise AND
let or_r  = MC_09.OR8(a, b);       // bitwise OR
let xor_r = MC_10.XOR8(a, b);      // bitwise XOR
let not_r = MC_11.NOT8(a);          // bitwise NOT
let shl   = MC_12.SHL8(a, n);      // shift left
let shr   = MC_13.SHR8(a, n);      // shift right
let rotl  = MC_14.ROTL8(a, n);     // rotate left
let rotr  = MC_15.ROTR8(a, n);     // rotate right
```

### Family 2 — Memory (MC_16–MC_23)

```pcd
let arr  = MC_16.ALLOC(size);
let val  = MC_17.LOAD(arr, idx);
let arr2 = MC_18.STORE(arr, idx, val);
let len  = MC_19.LEN_MEM(arr);
```

### Family 3 — Control (MC_24–MC_31)

Control monomers are typically expressed via PCD syntax (`if`, `loop`, `return`) rather than direct calls. `MC_24.IF` requires a `bool`, not `i64`.

### Family 4 — I/O (MC_32–MC_39)

```pcd
MC_33.WRITE(1, msg, len);          // write to fd (1=stdout)
```

### Family 5 — String (MC_40–MC_47)

```pcd
let joined  = MC_40.CONCAT(a, b);
let parts   = MC_41.SPLIT(text, ",");
let sub     = MC_42.SUBSTR(text, start, length);
let slen    = MC_43.LEN(text);
let upper   = MC_44.UPPER(text);
let ch      = MC_45.CHAR_AT(text, idx);  // O(1) via as_bytes
let lower   = MC_46.LOWER(text);
let trimmed = MC_47.TRIM(text);
```

### Family 6 — Crypto (MC_48–MC_55)

```pcd
let hash = MC_48.HASH(data);        // SHA-256
let hmac = MC_49.HMAC(key, data);   // HMAC-SHA-256
let enc  = MC_50.ENCRYPT(key, data);
let dec  = MC_51.DECRYPT(key, data);
```

### Family 7 — System (MC_56–MC_63)

```pcd
let data = MC_56.READ(path);         // read file
let _    = MC_57.WRITE_FILE(path, d);// write file
let _    = MC_58.WRITE(text);        // write stdout
let line = MC_59.READ_LINE();        // read stdin line
let env  = MC_63.ENV("ARGV_1");      // CLI arg in native ELF
```

## Loops — ALWAYS Use loop(N)

**NEVER use `while`** in production PCD. The `while` loop has a known SSA bug.

```pcd
// CORRECT: bounded, deterministic
let sum = 0;
loop(100) as i {
    let sum = sum + i;    // same name = loop-carried variable
}

// While-equivalent pattern:
let pos = 0;
loop(max) as _i {
    if (pos < max) {
        let pos = pos + 1;
    }
}
```

**Critical:** Loop-carried variables MUST use the exact same name as the outer binding.

## Conditionals and Pattern Matching

```pcd
// if/else — all branches must terminate coherently
if (x > 0) {
    return "positive";
} else {
    return "non-positive";
}

// match — ALWAYS include wildcard _
let result = match cmd {
    "add" => do_add(),
    "sub" => do_sub(),
    _ => "unknown"          // required for Φ_c = 1
};
```

## Structs

```pcd
struct Point { x: i64, y: i64 }

let p = Point { x: 10, y: 20 };
let px = p.x;

// "Update" = construct new struct (value semantics)
let p2 = Point { x: p.x + 1, y: p.y };
```

## Error Handling

```pcd
// try/catch is a STATEMENT, not an expression
try {
    let data = MC_56.READ("file.txt");
    OUTPUT data;
} catch (err) {
    OUTPUT "Error: " + err;
}
```

## Imports and Stdlib

```pcd
import "stdlib/math.pcd";       // abs, min, max, clamp, pow, sqrt, gcd, lcm
import "stdlib/string.pcd";     // len, upper, lower, trim, split, join, find, contains, starts_with, ends_with, replace, to_int, from_int
import "stdlib/array.pcd";      // push, pop, sort, reverse, map, filter, reduce, contains, find_index, slice, concat, zip, flatten
import "stdlib/io.pcd";         // read_file, write_file, read_line, write_line, exists
import "stdlib/fmt.pcd";        // format, to_string, pad_left, pad_right, truncate, repeat
import "stdlib/json.pcd";       // parse, stringify, get, set, keys, values, has, merge

// Selective import
import "stdlib/math.pcd" { abs, max };

// Aliased import
import "utils.pcd" as utils;
```

## EVA Composition (Automatic)

The planner infers EVA operators from data dependencies — never write them explicitly:

1. **Data dependency → Sequential (⊗):** B uses A's output → `A ⊗ B`
2. **No dependency → Parallel (∥):** Independent computations → `A ∥ B`
3. **if/else → Conditional (⊕):** Branching → `pred ⊕ (A | B)`

## CMF Certification Checklist

For Φ_c = 1, ensure:
- [ ] All function parameters are used (or prefixed with `_`)
- [ ] All control paths end with `OUTPUT` or `return`
- [ ] All `match` expressions have a `_` wildcard
- [ ] `MC_03.DIV8` results are destructured as `let (q, r) = ...`
- [ ] No dead code after `return` or `OUTPUT`
- [ ] No circular imports
- [ ] Loop nesting depth < 256

## Critical Anti-Patterns — NEVER Do These

1. **NEVER use `while` loops** — use `loop(N)` with `if` guard
2. **NEVER use `return (X == Y)` for boolean functions** — use explicit `if/return`:
   ```pcd
   // BAD:  return (a == b);
   // GOOD:
   if (a == b) { return 1; }
   return 0;
   ```
3. **NEVER forget to destructure DIV8** — it returns a tuple, not i64
4. **NEVER build strings in tight loops** — use array.push + join
5. **NEVER use recursion for large data** — MAX_DEPTH=256, use loop(N)
6. **NEVER omit fallback in match** — always include `_ => default`

## CLI Dispatch Pattern (Canonical)

```pcd
PC my_tool {
    let cmd = MC_63.ENV("ARGV_1");

    if (cmd == "build") {
        // build logic
        OUTPUT "OK";
    }
    if (cmd == "help") {
        let _ = MC_58.WRITE("Usage: my_tool [build|help]\n");
        OUTPUT 0;
    }

    // Fallback — ALWAYS present for Φ_c = 1
    let _ = MC_58.WRITE("Unknown command: " + cmd + "\n");
    OUTPUT 1;
}
```

## Compilation Targets

```bash
brikc compile src/main.pcd                  # native x86-64 ELF
brikc compile src/main.pcd --target rs      # Rust
brikc compile src/main.pcd --target js      # JavaScript ES2022
brikc compile src/main.pcd --target py      # Python 3.10+
brikc compile src/main.pcd --target wasm32  # WebAssembly
```

## Performance Tips

1. Pre-compute `MC_43.LEN()` outside loops
2. Use tight `loop(N)` bounds — `loop(1000)` when you need 10 wastes cycles
3. Prefer iterative over recursive — each call saves ~3592 vars
4. Use `array.push + join` over string concat — O(N) vs O(N²)
5. Use multi-level division for number→string: 10000→1000→100→10 peeling
