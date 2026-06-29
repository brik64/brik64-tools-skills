---
name: brik64
description: Public BRIK64 operating skill for AI agents using the brik64 CLI, .brik traceability, PCD 1.0, local evidence, and claim-safe workflows. Use when working with BRIK64 projects, PCD files, agent instructions, CLI commands, evidence reports, or public BRIK64 documentation.
version: 0.1.0-beta.17
triggers:
  - using brik64 CLI
  - BRIK64 project workflow
  - writing or reviewing PCD 1.0
  - creating .brik traceability
  - installing BRIK64 AGENTS.md instructions
  - reviewing local evidence
  - preparing claim-safe reports
  - checking docs.brik64.com
---

# BRIK64 Agent Operating Skill

Use this skill when acting as an AI agent inside a BRIK64 project or when
helping a user adopt the public BRIK64 CLI beta.

BRIK64 is a local-first workflow for making critical software logic easier to
inspect, describe, compose, and review with bounded evidence. The current public
beta surface is `0.1.0-beta.17`. The CLI uses a local/offline runtime for
free command-line workflows.
The CLI install channel is curl-only:
`curl -fsSL https://brik64.com/cli/install.sh | bash`. npm, PyPI, and
crates.io carry SDK packages. Treat the live installer output as authoritative
for the current public beta.

Primary documentation:

- Docs: https://docs.brik64.com
- CLI install: https://docs.brik64.com/cli/install
- GitHub Release: https://github.com/brik64/brik64-cli/releases
- JS/TS SDK package: https://www.npmjs.com/package/@brik64/core
- Python SDK package: https://pypi.org/project/brik64/0.1.0b17/
- Rust SDK package: https://crates.io/crates/brik64-core
- Public skills repo: https://github.com/brik64/brik64-tools-skills

## Agent Rule

Do not start by writing code. Start by shaping the requirement into a bounded
circuit:

```text
requirement -> inputs and outputs -> fail-closed behavior -> PCD Blueprint
-> EVA structure -> Polymer candidate -> .brik traceability -> local evidence
-> bounded report
```

## Keep The Skill Current

Before important work, check the live docs and the public skills repository:

1. Read the relevant page on https://docs.brik64.com.
2. Run `brik64 --version` when the CLI is installed.
3. Run `brik64 skill check-version` when available to detect local skill drift.
4. Check the current `brik64-tools-skills` repository or installed skill version.
5. Prefer the newer public docs/repo state over stale local notes.
6. If local instructions conflict with live docs, report the drift and follow the
   more authoritative current public source unless the user provides a stricter
   repo-local policy.

For recurring or long-running projects, re-check docs and the skills repository
periodically, especially before publishing, installing agent instructions,
reporting evidence, or describing release status.

## Public Changelog Rule

Public changelog entries must describe user-visible product changes only:
commands added or changed, local workflow behavior, SDK API changes, supported
formats, compatibility improvements, install behavior, and user-facing fixes.

Do not write public changelog bullets about internal release decisions,
coordination process, unpublished readiness states, approval flow, evidence
collection mechanics, rollback planning, unpublished engineering names, or
cross-repository synchronization. Put that information in internal release
reports, manifests, gates, or evidence files instead.

If a public changelog reads like an operations report, classify it as drift and
rewrite it into functional release notes before treating the surface as ready.

## Install And Verify The CLI

Use the curl installer for the current public CLI beta:

```bash
curl -fsSL https://brik64.com/cli/install.sh | bash
brik64 --version
brik64 skill check-version
brik64 help
```

Current version boundary:

- Public CLI version: `0.1.0-beta.17`
- Runtime banner should be checked with `brik64 --version`.
- JS/TS SDK package: `@brik64/core@0.1.0-beta.17`.
- Python SDK package: `brik64==0.1.0b17`.
- Rust SDK package: `brik64-core@0.1.0-beta.17`.

Treat runtime output as observed evidence. Treat the installer route and GitHub
Release as the public CLI surface. Treat npm, PyPI and crates.io as SDK-only
channels. If they disagree, report the drift instead of hiding it.

Runtime boundary:

- The free/local CLI path uses the local/offline runtime and must work without login after
  installation.
- Managed cloud runtime is reserved for future paid/cloud platform workflows and must not be
  presented as the local downloadable engine.
- Avoid unsupported internal production claims, compiler-origin claims, proof-grade claims, or toolchain-closure claims unless current public docs and fresh release evidence explicitly authorize that wording.

## Source Lift Preview

Use source lift only as a local preview path:

```bash
brik64 lift js path/to/file.js --preview
brik64 lift ts path/to/file.ts --preview
brik64 lift python path/to/file.py --preview
brik64 adoption report --json
```

Rules:

- `lift --preview` generates PCD candidates and warning reports under
  `.brik/lift-preview/`.
- It does not create certificates.
- It does not upload source by default.
- Unsupported constructs are warnings, not proof failures.
- Review generated PCD candidates before using `certify`, `verify`, `emit`, or
  `polymerize`.

## `.brik` Traceability

Use `.brik` as local project metadata for BRIK64 work:

- project manifest;
- local ledger events under `.brik/ledger/`;
- decisions and evidence references;
- local candidate artifacts;
- update and release notes where supported by the CLI.

Rules:

- `brik64 init` prepares local BRIK64 metadata.
- `brik64 init` must not create or modify `AGENTS.md`.
- `brik64 ledger verify --json` checks local event-chain consistency.
- `brik64 ledger export --redacted` exports local ledger metadata without raw
  source, raw PCD content, absolute paths, or secrets.
- Do not store secrets, raw tokens, private keys, or private source snapshots in
  `.brik`.
- Treat `.brik` metadata and `.brik/ledger` as operational traceability, not as
  certification or as a distributed blockchain.

## PCD 1.0 Working Model

PCD means Program Circuit Description. In public beta work, treat PCD as a
reviewable structural description for logic, not as a free-form script.

Agent checklist for PCD work:

- Declare inputs and outputs.
- Name failure behavior.
- Keep monomer selection explicit.
- Keep EVA composition explicit.
- Preserve generated files and evidence references.
- Review generated PCD before using it in a larger composition.
- Do not claim that a PCD candidate certifies an entire application.

Use docs before writing PCD:

- https://docs.brik64.com/language/pcd-file-standard
- https://docs.brik64.com/language/pcd-syntax
- https://docs.brik64.com/language/pcd-tutorial
- https://docs.brik64.com/agents/pcd-authoring-rules

## CLI Workflow

Current public beta commands:

```bash
brik64 help
brik64 --version
brik64 doctor
brik64 init
brik64 certify <file.pcd>
brik64 emit <file.pcd>
brik64 emit <file.pcd> --target ts --out out-ts --tests
brik64 emit <file.pcd> --target rust --out out-rust --tests
brik64 emit <file.pcd> --target python --out out-python --tests
```

Use commands only within their documented scope. If docs mention a command that
is missing from the installed beta, report the observed output and installed
package version before continuing. Advanced PCD generation, polymerization,
compiler, update, engine, and platform commands require current docs and release
evidence before an agent treats them as available.

## Agent Instructions In `AGENTS.md`

BRIK64 agent instructions are written only with explicit user consent. The
current CLI beta does not expose `brik skill` subcommands, so agents must install
or update skills through their host agent environment or by reviewing the public
skills repository directly:

```text
https://github.com/brik64/brik64-tools-skills
```

Rules:

- `AGENTS.md` writes must be explicit, reviewable, and user-approved.
- Preserve human-authored content outside the managed BRIK64 section.
- Report exactly what changed.
- The managed section is guidance for agents. It is not a certificate for the
  repository.

## Evidence And Certification Boundaries

Use precise language:

- local candidate;
- local evidence;
- smoke report;
- release evidence;
- package metadata;
- GitHub Release;
- public claim authorization.

Do not claim any of the following unless the current public evidence proves the
exact scope:

- assurance levels, runtime capabilities, reproducibility claims, independence
  claims, or proof statements that are not listed in public release notes;
- broad correctness;
- complete platform support;
- bundled runtime motor, unless the release artifact shows it.

If evidence is missing, say what is missing and keep the result scoped.

## Reporting Template

When closing work, report:

```text
BRIK64 status:
- Package:
- Runtime output:
- Docs checked:
- Commands run:
- Files changed:
- Evidence produced:
- Claim boundary:
- Open blockers:
```

## Public Surface Rule

BRIK64 public surfaces should be confident but bounded. Prefer:

```text
This produces local evidence for the inspected scope.
```

Avoid:

```text
This proves the whole application is correct.
```

## User-Facing UI Language Rule

Do not expose PCD, Polymer, monomer, DTL, ledger, certification, or internal
engine vocabulary in end-user frontend copy unless the UI is explicitly an
expert developer console.

For product/application frontends, translate internal BRIK64 concepts into
domain language:

- `PCD` -> bounded logic, domain rule, validation contract, or review artifact.
- `ledger` / `DTL` -> local trace, trace history, audit trail, or review state.
- `certified` / `secured` -> checked, validated for this scope, or traceable.
- `polymer` -> composed rule set or system rule set.

Keep raw PCD names, hashes, `.brik` paths and command output in expandable
developer diagnostics, logs, reports, or agent-readable evidence, not in primary
user-facing labels.

## Source Authority Order

Use this order when facts conflict:

1. Current repo files and command output.
2. https://docs.brik64.com.
3. Current GitHub Release and package metadata.
4. Public `brik64-tools-skills` repository.
5. Older local notes or chat context.

When a fact is drift-prone, verify it again before publishing or reporting it.
