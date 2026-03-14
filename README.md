# BRIK-64 Agent Skills

Five skills covering Digital Circuitality from theory to production, including the Registry platform and minimalist MCP interface.

---

## Which skills do I need?

```
Are you learning the theory / designing with circuit thinking?
  └─ YES → install: digital-circuitality

Are you using brik64 libraries in an existing codebase?
  ├─ Rust               → install: brik64-rust
  ├─ JavaScript / TypeScript → install: brik64-javascript
  └─ Python             → install: brik64-python

Are you writing PCD programs, using brikc CLI, or building policy circuits?
  └─ YES → install: pcd-system

Are you working with the BRIK-64 Registry, MCP, or Marketplace?
  └─ YES → install: pcd-system (includes Registry/MCP section)
```

Install all 5 for full coverage. Each skill is independent.

---

## Skills

### `digital-circuitality`
**The circuit thinking methodology — language agnostic.**

Teaches you to think in terms of closed circuits: bounded inputs, verified operations, proven output ranges. No undefined behavior by construction. Use this skill to understand WHY Digital Circuitality exists and how to apply circuit design principles to any language or system.

Install when: you are designing a system, reviewing architecture, or want to understand the formal model.

---

### `pcd-system`
**The complete PCD + brikc + Registry environment.**

Full reference for writing PCD programs, compiling with `brikc`, certifying circuits (Φ_c = 1), debugging CMF errors, building AI safety policy circuits, and targeting multiple backends (native ELF, Rust, JS, Python, WASM).

**NEW in v4.0:** Includes the BRIK-64 Registry & MCP platform layer — the 2-tool minimalist MCP architecture (`brik64.discover` + `brik64.execute`), inherited certification, domain packs, the Reuse Before Create workflow, and the API/Marketplace reference.

Covers: PCD syntax, all 64 monomers with signatures, CMF metrics, known bugs and workarounds, brikc CLI commands, auto-generated test suites, policy circuit templates, Registry MCP tools, domain packs, composition API.

Install when: you are writing `.pcd` files, using the brikc compiler, building ALLOW/BLOCK guardrails for AI agents, or working with the BRIK-64 Registry/MCP/Marketplace.

---

### `brik64-rust`
**Use brik64-core in Rust — no brikc needed.**

The `brik64-core` crate brings all 64 monomers and EVA algebra operators to native Rust. Use saturating arithmetic, verified crypto, and composable pipelines directly in your Rust code. No PCD files, no compiler invocation.

Install when: you are working in a Rust project and want verified, formally-proven operations without adopting PCD.

---

### `brik64-javascript`
**Use @brik64/core in JavaScript / TypeScript — no brikc needed.**

The `@brik64/core` npm package brings the 64 monomers to Node.js and browsers. Works with Web Crypto API. Full EVA algebra pipeline composition in TypeScript.

Install when: you are working in a JS/TS project and want Digital Circuitality properties without adopting PCD.

---

### `brik64-python`
**Use brik64 in Python — no brikc needed.**

The `brik64` PyPI package brings the 64 monomers to Python 3.10+. Identical semantics to the Rust and JS versions. Pipeline composition via `eva.pipeline()`.

Install when: you are working in a Python project and want verified, formally-proven operations without adopting PCD.

---

## Skill combinations

| Goal | Skills to install |
|------|------------------|
| Understand Digital Circuitality | `digital-circuitality` |
| Write PCD programs | `pcd-system` |
| Build AI safety policy circuits | `pcd-system` |
| Use BRIK-64 Registry / MCP | `pcd-system` |
| BRIK-64 operations in Rust | `brik64-rust` |
| BRIK-64 operations in JS/TS | `brik64-javascript` |
| BRIK-64 operations in Python | `brik64-python` |
| Full stack: theory + PCD + all languages | all 5 |
| AI agent with Registry access | `pcd-system` (Registry + MCP section) |

---

## MCP Architecture (Minimalist)

The BRIK-64 MCP server follows 2026 best practices: **2 tools, not 10.**

| Tool | Type | Purpose |
|------|------|--------|
| `brik64.discover` | Read-only | Search, compare, compatibility, lineage — all server-side |
| `brik64.execute` | Write | Compose, register, publish, acquire, export compliance |

**~800 tokens** total schema (vs 12K+ for 10 tools). Preserves **98% of context window** for reasoning.

Procedural knowledge lives in **skills** loaded on-demand, not in tool schemas.

---

## Repository

`brik64/brik64-tools-skills` — maintained by BRIK-64 Inc.

Documentation: https://docs.brik64.dev
Contact: info@brik64.com
