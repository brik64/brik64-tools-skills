# BRIK64 Agent Skills

Public BRIK64 skills for AI agents and developers working with the local CLI,
PCD, `.brik` traceability, Digital Circuitality, and language SDKs.

For current product documentation, always check https://docs.brik64.com.
Agents should also review this repository periodically before important BRIK64
work so installed instructions do not drift from the public skill source.

---

## Which skill do I need?

```
Are you an AI agent using the public brik CLI, .brik, PCD 1.0, local evidence,
or AGENTS.md managed instructions?
  └─ YES → install: brik64

Are you learning the theory / designing with circuit thinking?
  └─ YES → install: digital-circuitality

Are you using brik64 libraries in an existing codebase?
  ├─ Rust               → install: brik64-rust
  ├─ JavaScript / TypeScript → install: brik64-javascript
  └─ Python             → install: brik64-python

Are you writing lower-level PCD programs or using older PCD reference material?
  └─ YES → install: pcd-system

```

Install `brik64` first for current public beta agent operation. Add the
language or theory skills only when the task needs them. Each skill is
independent.

Public naming contract:

- current public CLI command: `brik64`
- compatibility CLI alias: `brik`
- current public CLI install path: `curl -fsSL https://brik64.com/cli/install.sh | bash`
- current JS/TS SDK package: `@brik64/core@0.1.0-beta.7`
- current docs: https://docs.brik64.com
- older public examples may still carry compatibility aliases; report drift
  instead of presenting legacy names as current truth

---

## Skills

### `brik64`
**Current public operating skill for AI agents using BRIK64.**

Use this first when an agent is working with the public `brik64` CLI, `.brik`
metadata, PCD 1.0, local evidence, `AGENTS.md` managed instructions, or public
BRIK64 docs. It includes the current CLI beta install path, docs lookup rule,
periodic repository update rule, consent-based skill installation workflow, and
claim-safe reporting boundaries.

Install when: you are an AI coding agent, reviewing a BRIK64 project, using
`brik64`, writing PCD candidates, or preparing a public-facing BRIK64 report.

---

### `digital-circuitality`
**The circuit thinking methodology — language agnostic.**

Teaches you to think in terms of closed circuits: bounded inputs, verified operations, proven output ranges. No undefined behavior by construction. Use this skill to understand WHY Digital Circuitality exists and how to apply circuit design principles to any language or system.

Install when: you are designing a system, reviewing architecture, or want to understand the formal model.

---

### `pcd-system`
**Detailed PCD reference and legacy/deeper PCD system notes.**

Use alongside `brik64` when a task needs deeper PCD examples, syntax notes, or
older system material. Treat older version labels in this skill as historical
unless current docs and release evidence confirm them.

Covers: PCD syntax notes, monomer-oriented examples, older command examples,
and historical PCD system material. Use current docs and the `brik64` skill for
the public beta CLI workflow.

Install when: you are writing `.pcd` files or need deeper PCD reference notes.

---

### `brik64-rust`
**Use brik64-core in Rust — no public CLI needed.**

The Rust skill covers BRIK64-oriented operation patterns and historical SDK
usage notes for Rust projects. Check current docs before installing packages or
making SDK availability claims.

Install when: you are working in a Rust project and want BRIK64 operation
patterns without adopting PCD.

---

### `brik64-javascript`
**Use @brik64/core in JavaScript / TypeScript — no public CLI needed.**

The JavaScript skill covers BRIK64-oriented operation patterns and historical
SDK usage notes for JavaScript and TypeScript projects. Check current docs
before installing packages or making SDK availability claims.

Install when: you are working in a JS/TS project and want Digital Circuitality properties without adopting PCD.

---

### `brik64-python`
**Use brik64 in Python — no public CLI needed.**

The Python skill covers BRIK64-oriented operation patterns and historical SDK
usage notes for Python projects. Check current docs before installing packages
or making SDK availability claims.

Install when: you are working in a Python project and want BRIK64 operation
patterns without adopting PCD.

---

## Skill combinations

| Goal | Skills to install |
|------|------------------|
| AI agent using current public BRIK64 beta | `brik64` |
| Understand Digital Circuitality | `digital-circuitality` |
| Write current PCD candidates with CLI workflow | `brik64` + `pcd-system` |
| Use AGENTS.md managed instructions | `brik64` |
| BRIK-64 operations in Rust | `brik64-rust` |
| BRIK-64 operations in JS/TS | `brik64-javascript` |
| BRIK-64 operations in Python | `brik64-python` |
| Full public beta agent workflow | `brik64` + `digital-circuitality` + `pcd-system` |

---

## Public Standards

BRIK64 public standards are maintained in dedicated repositories:

- PCD 1.0: the public Program Circuit Description format:
  https://github.com/brik64/pcd-standard
- `.brik` 1.0: the public BRIK64 workspace standard:
  https://github.com/brik64/brik-standard

Use the current `brik64` skill plus https://docs.brik64.com as the active public
guidance for agent work. Keep platform integrations, domain-system drafts,
extended-operation drafts, and older major-version notes out of current public
release instructions until they are published as versioned docs and repos.

Planning checklist: [`standards/README.md`](standards/README.md).

---

## Repository

`brik64/brik64-tools-skills` — maintained by BRIK-64 Inc.

Documentation: https://docs.brik64.com
Contact: info@brik64.com
