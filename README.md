# brik64-tools-skills

Official AI agent skills for developing with **BRIK-64** and **Digital Circuitality**.

These skills give any AI agent (Claude Code, Codex, Gemini CLI, etc.) expert knowledge of circuit-based programming — from pure methodology to full PCD certification.

🌐 [brik64.dev](https://brik64.dev) · 📖 [docs.brik64.dev](https://docs.brik64.dev)

---

## Available Skills

| Skill | Description | Use When |
|-------|-------------|----------|
| **[digital-circuitality](skills/digital-circuitality/)** | Circuit thinking methodology for ANY language — Rust, Python, JS, Go... No PCD required | Designing programs with circuit principles, using brik64 libraries, or applying methodology without changing language |
| **[pcd-developer](skills/pcd-developer/)** | Complete PCD reference — 64 monomers, EVA algebra, CMF certification, patterns, test generation | Writing or reviewing any `.pcd` program |
| **[brik64-rust](skills/brik64-rust/)** | brik64-core Rust crate — arithmetic, crypto, EVA composition in Rust | Adding BRIK-64 to a Rust project |
| **[brik64-javascript](skills/brik64-javascript/)** | @brik64/core npm package — full JS/TS reference with async patterns | Adding BRIK-64 to a JS/TS project |
| **[brik64-python](skills/brik64-python/)** | brik64 Python package — monomers, EVA pipeline, integration patterns | Adding BRIK-64 to a Python project |
| **[brikc-compiler](skills/brikc-compiler/)** | brikc CLI — compile, run, check, format, emit tests | Using the `brikc` compiler or choosing targets |
| **[pcd-debugger](skills/pcd-debugger/)** | CMF error diagnosis — Φ_c failures, type mismatches, SSA bugs | Fixing compilation errors or Φ_c ≠ 1 |
| **[pcd-policy-circuits](skills/pcd-policy-circuits/)** | AI safety policy circuit design — ALLOW/BLOCK guardrails | Building AI safety guardrails or access control |

---

## Install

### Claude Code

```bash
# Install all skills
for skill in digital-circuitality pcd-developer brik64-rust brik64-javascript brik64-python brikc-compiler pcd-debugger pcd-policy-circuits; do
  mkdir -p ~/.claude/skills/$skill
  cp skills/$skill/SKILL.md ~/.claude/skills/$skill/
done
```

Or install a single skill:

```bash
mkdir -p ~/.claude/skills/digital-circuitality
cp skills/digital-circuitality/SKILL.md ~/.claude/skills/digital-circuitality/
```

### Codex CLI

```bash
for skill in digital-circuitality pcd-developer brik64-rust brik64-javascript brik64-python brikc-compiler pcd-debugger pcd-policy-circuits; do
  mkdir -p ~/.codex/skills/$skill
  cp skills/$skill/SKILL.md ~/.codex/skills/$skill/
done
```

### Gemini CLI

```bash
for skill in digital-circuitality pcd-developer brik64-rust brik64-javascript brik64-python brikc-compiler pcd-debugger pcd-policy-circuits; do
  mkdir -p ~/.gemini/skills/$skill
  cp skills/$skill/SKILL.md ~/.gemini/skills/$skill/
done
```

### Other AI Agents

Copy the `SKILL.md` file content into your agent's system prompt or knowledge base. The skills are plain Markdown — compatible with any agent framework that supports system prompt injection.

---

## Three Levels of Engagement

```
Level 1: Methodology (any language, no install)
         → digital-circuitality skill
         Apply circuit thinking in Rust, Python, JS without any library.
         Better code structure. Fewer bugs. No new toolchain.

Level 2: Libraries (your language, install brik64 package)
         → brik64-rust / brik64-javascript / brik64-python skills
         cargo add brik64-core  |  npm install @brik64/core  |  pip install brik64
         Formally verified operations in your existing codebase.
         No new language. No certification.

Level 3: Full Certification (PCD language + brikc compiler)
         → pcd-developer + brikc-compiler skills
         curl -fsSL https://brik64.dev/install | sh
         Φ_c = 1 proof. Auto-generated test suites. Registry badge.
         Generated code in Rust/JS/Python inherits the proof.
```

> **Use the principles everywhere. Certify through PCD.**

---

## What These Skills Enable

After installing, an AI agent can:

- **Apply circuit thinking** in any language — design functions as closed circuits, eliminate open branches, reason about composition
- **Use verified operations** — saturating arithmetic, typed crypto, deterministic string ops
- **Write idiomatic PCD programs** using all 64 monomers and 6 stdlib modules
- **Ensure Φ_c = 1 certification** by following the CMF checklist
- **Generate code + tests together** — `--emit-tests` produces the suite from the formal proof
- **Avoid known pitfalls** — WhileLoop SSA bug, DIV8 tuple, boolean return pattern
- **Compile to 5 targets** — native ELF, Rust, JavaScript, Python, WASM
- **Diagnose and fix** every CMF error type

---

## License

Apache License 2.0 — see [LICENSE](LICENSE) for details.

© 2026 BRIK-64 Inc.
