---
name: brikc-compiler
description: "Use the brikc compiler to compile, run, check, and format PCD programs. Covers all CLI commands, compilation targets, CMF verification, and self-compilation fixpoint. Use when compiling .pcd files or working with brikc CLI."
---

# brikc Compiler — CLI Reference

Compile, run, verify, and format PCD programs using the `brikc` compiler.

**Documentation:** https://docs.brik64.dev/cli/commands

---

## Installation

```bash
curl -fsSL https://brik64.dev/install | sh
brikc --version          # brikc v2.0.0 (fixpoint: 7229cfcd...)
brikc check --self       # verify authentic compiler binary
```

## Core Commands

### compile — Compile PCD to target

```bash
# Native x86-64 ELF (standalone, no dependencies)
brikc compile src/main.pcd
brikc compile src/main.pcd -o myprogram

# Cross-compile to other targets
brikc compile src/main.pcd --target rs       # Rust source
brikc compile src/main.pcd --target js       # JavaScript ES2022
brikc compile src/main.pcd --target py       # Python 3.10+
brikc compile src/main.pcd --target wasm32   # WebAssembly
brikc compile src/main.pcd --target bir      # BIR bytecode
```

**Exit codes:**
| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Parse/syntax error |
| 2 | CMF rejects circuit (Φ_c ≠ 1) |
| 3 | Backend code generation error |

### run — Compile and execute

```bash
brikc run program.pcd
brikc run program.pcd -- arg1 arg2    # pass arguments
```

### check — CMF verification without compiling

```bash
brikc check circuit.pcd
brikc check circuit.pcd --json    # machine-readable output
```

Output:
```
[CMF] Parsing circuit...              ✓
[CMF] EVA composition valid:          ✓
[CMF] Circuit closedness (Φ_c):       1.000
[CMF] Determinism:                    ✓
[CMF] Termination:                    ✓
[CMF] CIRCUIT CLOSED ✓   Φ_c = 1
```

**CMF metrics:**
| Metric | Description | Required |
|--------|-------------|----------|
| e | Operational complexity (monomer count) | — |
| h | Signature distance (type transformation) | — |
| s | Structural entropy (branching complexity) | — |
| c | Cyclomatic complexity (independent paths) | — |
| t | Termination depth (loop nesting) | < 256 |
| δ | Unused input ratio | = 0 |
| Φ_c | Circuit closure | = 1.000 |

### fmt — Format PCD source

```bash
brikc fmt src/main.pcd           # format in place
brikc fmt --check src/main.pcd   # check without modifying
```

### repl — Interactive PCD session

```bash
brikc repl
>>> let x = MC_00.ADD8(10, 20);
30
>>> :check    # run CMF on session definitions
```

### catalog — List all 64 monomers

```bash
brikc catalog
brikc catalog --family 0    # arithmetic only
brikc catalog --json
```

### verify — Self-verification

```bash
brikc verify               # self-compilation fixpoint: Ω = 1
brikc check --self         # verify binary hash
```

### lsp — Language Server Protocol

```bash
brikc lsp                  # start LSP server for IDE integration
```

### cert — Certification (planned)

```bash
brikc cert publish circuit.pcd    # publish to registry
brikc cert verify <hash>          # verify certificate
brikc cert badge <hash>           # get live badge URL
```

## Compilation Pipeline

```
.pcd → hand_parser → AST → planner (SSA + type_check) → CPF → backend
                                                           ├── native (x86-64 ELF)
                                                           ├── Rust
                                                           ├── JavaScript
                                                           ├── Python
                                                           ├── BIR bytecode
                                                           └── WASM
```

The CMF runs during the planner phase. If Φ_c ≠ 1, compilation aborts before any backend code generation.

**Certification scope:** Cross-compiled output (Rust, JS, Python, WASM) inherits the Φ_c = 1 certification from the PCD source — the proof happens at compile time. However, formal registry badges and certificates require BIR compilation and registration in the BRIK-64 registry. Code written directly in Rust/JS/Python without PCD cannot be CMF-certified.

## Self-Compilation Fixpoint

The `brikc` compiler compiles its own source (`brikc.pcd`) and produces an identical hash across 4 generations:

```
Gen1 == Gen2 == Gen3 == Gen4
SHA-256: 7229cfcde9613de42eda4dd207da3bac80d2bf2b5f778f3270c2321e8e489e95
```

This is the mathematical proof that the compiler is correct.

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `Φ_c < 1.000` | Remove unused params or add fallback OUTPUT |
| `δ > 0` | Remove unused function parameters or prefix with `_` |
| Type error at monomer | Check monomer signatures at https://docs.brik64.dev/pcd/monomers |
| Parser MAX_DEPTH | Extract deeply nested logic into functions |
| While loop vars stale | Replace `while` with `loop(N)` + `if` guard |
