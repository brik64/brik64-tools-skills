---
name: pcd-debugger
description: "Diagnose and fix PCD compilation errors, CMF certification failures (Φ_c ≠ 1), type mismatches, and runtime issues. Use when a .pcd program fails to compile or produces unexpected results."
---

# PCD Debugger — Error Diagnosis & Fixes

Diagnose and fix all PCD compilation and runtime errors with systematic approaches.

**Documentation:** https://docs.brik64.dev/pcd/errors

---

## Diagnostic Flow

When a PCD program fails, follow this sequence:

```
1. Run `brikc check file.pcd --json` → read CMF metrics
2. Check δ (unused inputs) → remove or prefix with _
3. Check Φ_c → ensure all paths have OUTPUT/return
4. Check type errors → verify monomer signatures
5. Check runtime errors → guard with if or try/catch
```

## Error: Φ_c < 1.000 (Circuit Not Closed)

**Most common causes:**

### Cause 1: Missing OUTPUT on some paths

```pcd
// BAD: what if x == 0?
PC classify {
    if (x > 0) { OUTPUT "positive"; }
    if (x < 0) { OUTPUT "negative"; }
    // Falls through when x == 0!
}

// FIX: add fallback
PC classify {
    if (x > 0) { OUTPUT "positive"; }
    if (x < 0) { OUTPUT "negative"; }
    OUTPUT "zero";    // catches x == 0
}
```

### Cause 2: Unused function parameters (δ > 0)

```pcd
// BAD: b is never used → δ = 0.5
fn process(a, b) { return a + 1; }

// FIX: use it or remove it
fn process(a) { return a + 1; }
// OR prefix with _ to exclude from δ:
fn process(a, _b) { return a + 1; }
```

### Cause 3: Dead code after return/OUTPUT

```pcd
// BAD: unreachable code
fn check(x) {
    return x;
    let dead = "never runs";  // Φ_c < 1
}
```

### Cause 4: Match without wildcard

```pcd
// BAD: what if cmd is "delete"?
let r = match cmd { "add" => 1, "sub" => 2 };

// FIX: always include _
let r = match cmd { "add" => 1, "sub" => 2, _ => 0 };
```

## Error: Type Mismatch at Monomer Boundary

```
Type error: MC_00.ADD8 expects (u8, u8), got (string, i64)
```

**Fix:** Check the monomer signature. All arithmetic/logic monomers expect numeric types. String monomers expect strings.

**Quick reference — common type confusion:**
| Monomer | Input | Output | Gotcha |
|---------|-------|--------|--------|
| MC_00.ADD8 | (i64, i64) | i64 | — |
| MC_03.DIV8 | (i64, i64) | **Tuple(i64, i64)** | MUST destructure |
| MC_24.IF | **(Bool**, Block, Block) | T | Requires Bool, not i64 |
| MC_43.LEN | string | i64 | — |
| MC_45.CHAR_AT | (string, i64) | i64 | Returns byte value |

## Error: DIV8 Not Destructured

```pcd
// BAD
let result = MC_03.DIV8(10, 3);
let doubled = result * 2;    // Error: can't multiply a Tuple

// FIX
let (q, r) = MC_03.DIV8(10, 3);
let doubled = q * 2;
```

This is the **#1 beginner mistake** in PCD. MC_03.DIV8 is the only monomer that returns a Tuple.

## Error: WhileLoop SSA Bug

```pcd
// BUGGY: variables don't update across iterations
let pos = 0;
while (pos < max) {
    let pos = pos + 1;    // may read stale value
}

// FIX: replace with loop(N) + if
let pos = 0;
loop(max) as _i {
    if (pos < max) {
        let pos = pos + 1;    // works correctly
    }
}
```

**Root cause:** The planner captures variable versions before the loop body instead of after. This is a known bug — always use `loop(N)`.

## Error: Boolean Return Pattern Bug

```pcd
// SUBTLE BUG: comparison result may not work as expected
fn is_match(a, b) {
    return (a == b);    // BIR may not produce 0/1 correctly
}
if (is_match(x, y) == 0) { ... }  // UNRELIABLE

// FIX: explicit if/return
fn is_match(a, b) {
    if (a == b) { return 1; }
    return 0;
}
```

## Error: Division by Zero

```pcd
// Runtime error
let (q, r) = MC_03.DIV8(10, 0);

// FIX: guard before dividing
if (b == 0) {
    return (0, 0);
}
return MC_03.DIV8(a, b);
```

## Error: MAX_DEPTH (256) Exceeded

**Fix:** Extract nested logic into functions to reduce nesting depth.

## Error: Circular Import

```
a.pcd imports b.pcd
b.pcd imports a.pcd → circular!
```

**Fix:** Extract shared code into a third module `common.pcd`.

## Error: Out of Bounds (Array/String)

```pcd
// FIX: check length before accessing
let slen = MC_43.LEN(text);
if (idx < slen) {
    let ch = MC_45.CHAR_AT(text, idx);
}

// Or use try/catch
try {
    let ch = MC_45.CHAR_AT(text, idx);
} catch (err) {
    let ch = "";
}
```

## Reading CMF Output

```bash
brikc check program.pcd --json
```

```json
{
  "cmf": {
    "e": 4,         "h": 0.44,      "s": 0.000,
    "c": 1,         "t": 0,         "δ": 0.000,
    "Φ_c": 1.000
  },
  "omega": 1
}
```

**Red flags:**
- `δ > 0` → unused parameters
- `Φ_c < 1.000` → open circuit (check δ + missing OUTPUT paths)
- Very high `s` → simplify branching
- Very high `t` → reduce loop nesting depth

## Quick Troubleshooting Table

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| `Φ_c < 1.000` | Unused inputs or missing OUTPUT | Remove unused params, add fallback OUTPUT |
| `δ > 0` | Unused function parameters | Remove or prefix with `_` |
| Type error at monomer | Wrong argument types | Check monomer signatures |
| `Tuple` type mismatch | Forgot to destructure DIV8 | `let (q, r) = MC_03.DIV8(...)` |
| Parser MAX_DEPTH | Too many nested blocks | Extract into functions |
| While loop vars stale | WhileLoop SSA bug | Use `loop(N)` instead |
| Circular import | A→B→A | Extract shared code to third module |
| Runtime crash | Div by zero or OOB | Guard with `if` or `try/catch` |
| Boolean compare broken | `return (X == Y)` pattern | Use explicit `if/return` |
| String building slow | O(N²) concat in loop | Use `push` + `join` |
