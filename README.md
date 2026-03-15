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

Covers: PCD syntax, all 128 monomers (64 core + 64 extended) with signatures, CMF metrics, known bugs and workarounds, brikc CLI commands, auto-generated test suites, policy circuit templates, Registry MCP tools, domain packs, composition API.

Install when: you are writing `.pcd` files, using the brikc compiler, building ALLOW/BLOCK guardrails for AI agents, or working with the BRIK-64 Registry/MCP/Marketplace.

---

### `brik64-rust`
**Use brik64-core in Rust — no brikc needed.**

The `brik64-core` crate brings all 128 monomers (64 core + 64 extended) and EVA algebra operators to native Rust. Use saturating arithmetic, verified crypto, and composable pipelines directly in your Rust code. No PCD files, no compiler invocation.

Install when: you are working in a Rust project and want verified, formally-proven operations without adopting PCD.

---

### `brik64-javascript`
**Use @brik64/core in JavaScript / TypeScript — no brikc needed.**

The `@brik64/core` npm package brings 128 monomers (64 core + 64 extended) to Node.js and browsers. Works with Web Crypto API. Full EVA algebra pipeline composition in TypeScript.

Install when: you are working in a JS/TS project and want Digital Circuitality properties without adopting PCD.

---

### `brik64-python`
**Use brik64 in Python — no brikc needed.**

The `brik64` PyPI package brings 128 monomers (64 core + 64 extended) to Python 3.10+. Identical semantics to the Rust and JS versions. Pipeline composition via `eva.pipeline()`.

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

## Extended Monomer Families (v3.0.0-beta.1+)

In addition to the 64 core monomers (F0–F7, Φ_c = 1), BRIK-64 now includes 64 extended monomers across 8 new families. Extended monomers operate under **CONTRACT closure (Φ_c = CONTRACT)** — they interact with external systems and require runtime contracts rather than static proofs.

| Family | Range | Operations |
|--------|-------|------------|
| F8: Float64 | MC_64–MC_71 | FADD, FSUB, FMUL, FDIV, FABS, FNEG, FSQRT, FMOD |
| F9: Math | MC_72–MC_79 | SIN, COS, TAN, EXP, LN, LOG2, POW, CEIL |
| F10: Network | MC_80–MC_87 | TCP_CONN, TCP_SEND, TCP_RECV, TCP_CLOSE, UDP_SEND, UDP_RECV, DNS_RESOLVE, HTTP_REQ |
| F11: Graphics | MC_88–MC_95 | FB_CREATE, FB_SET_PX, FB_GET_PX, FB_CLEAR, FB_BLIT, FB_LINE, FB_RECT, FB_DIMS |
| F12: Audio | MC_96–MC_103 | AUD_CREATE, AUD_WRITE, AUD_READ, AUD_MIX, AUD_GAIN, AUD_LEN, AUD_RATE, AUD_CHANS |
| F13: Filesystem+ | MC_104–MC_111 | FS_STAT, FS_MKDIR, FS_RMDIR, FS_DELETE, FS_RENAME, FS_LIST, FS_EXISTS, FS_COPY |
| F14: Concurrency | MC_112–MC_119 | SPAWN, JOIN, CHAN_NEW, CHAN_SEND, CHAN_RECV, MUTEX_NEW, MUTEX_LOCK, MUTEX_UNLOCK |
| F15: Interop/FFI | MC_120–MC_127 | JSON_ENCODE, JSON_DECODE, FFI_CALL, FFI_LOAD, FFI_FREE, WASM_LOAD, WASM_CALL, WASM_FREE |

All language SDKs (Rust, JavaScript, Python) expose extended monomers in v3.0.0-beta.1+.

---

## Repository

`brik64/brik64-tools-skills` — maintained by BRIK-64 Inc.

Documentation: https://docs.brik64.dev
Contact: info@brik64.com
