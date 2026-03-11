# brik64-tools-skills

Official AI agent skills for developing with **BRIK-64** and **PCD** (Printed Circuit Description).

These skills transform any AI agent (Claude Code, Codex, Gemini CLI, etc.) into an expert PCD developer — capable of writing formally verified programs with Φ_c = 1 certification.

🌐 [brik64.dev](https://brik64.dev) · 📖 [docs.brik64.dev](https://docs.brik64.dev)

---

## Available Skills

| Skill | Description | Use When |
|-------|-------------|----------|
| **[pcd-developer](skills/pcd-developer/)** | Complete PCD programming reference — all 64 monomers, EVA algebra, CMF certification, stdlib, patterns | Writing or reviewing any `.pcd` program |
| **[pcd-policy-circuits](skills/pcd-policy-circuits/)** | AI safety policy circuit design — ALLOW/BLOCK guardrails | Building AI safety guardrails or access control |
| **[brikc-compiler](skills/brikc-compiler/)** | brikc CLI — compile, run, check, format PCD programs | Using the `brikc` compiler or choosing targets |
| **[pcd-debugger](skills/pcd-debugger/)** | Error diagnosis — CMF failures, type mismatches, SSA bugs | Fixing compilation errors or Φ_c ≠ 1 |

## Install

### Claude Code

```bash
# Install all skills
for skill in pcd-developer pcd-policy-circuits brikc-compiler pcd-debugger; do
  mkdir -p ~/.claude/skills/$skill
  cp skills/$skill/SKILL.md ~/.claude/skills/$skill/
done
```

Or install a single skill:

```bash
mkdir -p ~/.claude/skills/pcd-developer
cp skills/pcd-developer/SKILL.md ~/.claude/skills/pcd-developer/
```

### Other AI Agents

Copy the `SKILL.md` file content into your agent's system prompt or knowledge base. The skills are plain Markdown — compatible with any agent framework.

## What These Skills Enable

After installing, an AI agent can:

- **Write idiomatic PCD programs** using all 64 monomers and 6 stdlib modules
- **Ensure Φ_c = 1 certification** by following the CMF checklist
- **Avoid known pitfalls** (WhileLoop SSA bug, DIV8 tuple, boolean return pattern)
- **Design policy circuits** for AI safety with deny-by-default patterns
- **Compile to 5 targets** (native ELF, Rust, JavaScript, Python, WASM)
- **Diagnose and fix** every CMF error type
- **Use EVA composition** correctly (⊗ sequential, ∥ parallel, ⊕ conditional)

## Quick Example

With the `pcd-developer` skill installed, an agent can generate:

```pcd
import "stdlib/string.pcd";
import "stdlib/array.pcd";

PC word_counter {
    let text = MC_56.READ(MC_63.ENV("ARGV_1"));
    let words = split(text, " ");
    let count = len(words);
    let _ = MC_58.WRITE("Words: " + from_int(count) + "\n");
    OUTPUT count;
}
```

The agent knows to:
- Use `MC_63.ENV("ARGV_1")` for CLI args in native ELF
- Import stdlib modules for `split`, `len`, `from_int`
- End with `OUTPUT` for Φ_c = 1
- Use `MC_58.WRITE` for stdout

## Documentation

Full reference at **[docs.brik64.dev](https://docs.brik64.dev)**:

- [Tutorial](https://docs.brik64.dev/pcd/tutorial) — Step-by-step first program
- [Syntax Reference](https://docs.brik64.dev/pcd/syntax) — Complete language spec
- [Monomer Reference](https://docs.brik64.dev/pcd/monomers) — All 64 monomers with signatures
- [Standard Library](https://docs.brik64.dev/pcd/stdlib) — 6 modules (math, string, array, io, fmt, json)
- [Patterns](https://docs.brik64.dev/pcd/patterns) — Idiomatic patterns and anti-patterns
- [Error Guide](https://docs.brik64.dev/pcd/errors) — CMF error diagnosis
- [Examples](https://docs.brik64.dev/pcd/examples) — 8 complete programs
- [EVA Algebra](https://docs.brik64.dev/architecture/eva-algebra) — Composition theory
- [CLI Commands](https://docs.brik64.dev/cli/commands) — Full brikc reference

## License

© 2026 BRIK-64 Inc. All rights reserved. Proprietary license.
Skills may be used freely with BRIK-64 tooling. Redistribution of modified versions prohibited.
