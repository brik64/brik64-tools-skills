---
name: brik64
description: Public BRIK64 operating skill for AI agents using the brik CLI, .brik traceability, PCD 1.0, local evidence, and claim-safe workflows. Use when working with BRIK64 projects, PCD files, agent instructions, CLI commands, evidence reports, or public BRIK64 documentation.
version: 0.1.0-beta.3
triggers:
  - using brik CLI
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
CLI package is `@brik64/cli@0.1.0-beta.3`.

Primary documentation:

- Docs: https://docs.brik64.com
- CLI install: https://docs.brik64.com/cli/install
- GitHub Release: https://github.com/brik64/brik64-cli/releases/tag/v0.1.0-beta.3
- npm package: https://www.npmjs.com/package/@brik64/cli
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
2. Check the current `brik64-tools-skills` repository or installed skill version.
3. Prefer the newer public docs/repo state over stale local notes.
4. If local instructions conflict with live docs, report the drift and follow the
   more authoritative current public source unless the user provides a stricter
   repo-local policy.

For recurring or long-running projects, re-check docs and the skills repository
periodically, especially before publishing, installing agent instructions,
reporting evidence, or describing release status.

## Install And Verify The CLI

Use npm for the current public beta:

```bash
npm install -g @brik64/cli@beta
brik --version
brik help
```

Current version boundary:

- Public package/release version: `0.1.0-beta.3`
- Runtime banner should be checked with `brik --version` or `brik version`.

Treat runtime output as observed evidence. Treat package metadata and GitHub
Release as the public package surface. If they disagree, report the drift instead
of hiding it.

## `.brik` Traceability

Use `.brik` as local project metadata for BRIK64 work:

- project manifest;
- decisions and evidence references;
- local candidate artifacts;
- update and release notes where supported by the CLI.

Rules:

- `brik init` prepares local BRIK64 metadata.
- `brik init` must not create or modify `AGENTS.md`.
- Do not store secrets, raw tokens, private keys, or private source snapshots in
  `.brik`.
- Treat `.brik` metadata as operational traceability, not as certification.

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
brik help
brik version
brik --version
brik init
brik certify <file.pcd>
brik emit <file.pcd>
brik emit <file.pcd> --target ts --out out-ts --tests
brik emit <file.pcd> --target rust --out out-rust --tests
brik emit <file.pcd> --target python --out out-python --tests
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

- formal certification of an application;
- N4/N5, L5+/L6+, self-hosting, Rust independence, fixpoint, or proof;
- universal correctness;
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

## Source Authority Order

Use this order when facts conflict:

1. Current repo files and command output.
2. https://docs.brik64.com.
3. Current GitHub Release and package metadata.
4. Public `brik64-tools-skills` repository.
5. Older local notes or chat context.

When a fact is drift-prone, verify it again before publishing or reporting it.
