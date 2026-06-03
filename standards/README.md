# BRIK64 Public Standards Roadmap

This repository hosts public agent skills. It does not replace the dedicated
standard repositories for PCD 1.0 and `.brik` 1.0.

Current maturity:

- PCD 1.0: public standard repository at https://github.com/brik64/pcd-standard.
- `.brik` 1.0: public workspace standard repository at https://github.com/brik64/brik-standard.
- docs.brik64.com: primary public documentation surface.
- `brik64/brik64-tools-skills`: current public skill source for agents.

## Public Standard Repositories

### `brik64/pcd-standard`

Purpose: define the public Program Circuit Description format as a versioned
standard.

Required artifacts:

- `README.md`: standard overview, maturity label, and docs links.
- `SPEC.md`: syntax, envelopes, canonicalization, hashes, examples, and invalid cases.
- `VERSIONING.md`: compatibility, deprecation, and migration policy.
- `EXAMPLES.md`: tested minimal and realistic examples.
- `NOTICE`: BRIK64 Inc. ownership and attribution.
- `LICENSE` or terms file appropriate for a public standard.
- `AGENTS.md`: agent-readable usage and claim boundary.

### `brik64/brik-standard`

Purpose: define the public `.brik` workspace format for local BRIK64 project
metadata, traceability, evidence references, and agent-readable project state.

Required artifacts:

- `README.md`: standard overview, maturity label, and docs links.
- `SPEC.md`: workspace layout, metadata, ledger, references, versioning, and
  validation rules.
- `VERSIONING.md`: compatibility, deprecation, and migration policy.
- `EXAMPLES.md`: minimal workspace, evidence reference, and agent workflow examples.
- `NOTICE`: BRIK64 Inc. ownership and attribution.
- `LICENSE` or terms file appropriate for a public standard.
- `AGENTS.md`: agent-readable usage and claim boundary.

## Publication Rule

Do not present PCD 1.0 or `.brik` 1.0 as stronger than their current public
repositories and docs support. Before using either as a public authority, verify:

1. the dedicated repository exists;
2. the standard has a versioned `SPEC.md`;
3. docs.brik64.com links to the standard;
4. examples and invalid cases are reviewed;
5. public copy uses the same maturity label across docs, README files, and skills.

## Public Boundary

This roadmap is operational planning for public standards work. The active
surface is the current public CLI beta, docs.brik64.com, and this skills repo.
Platform integrations, distribution catalogs, and stronger certification
surfaces need their own versioned release evidence before they appear in public
instructions.
