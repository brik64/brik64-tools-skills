# BRIK64 Public Standards Roadmap

This repository hosts public agent skills. It does not replace the dedicated
standard repositories that BRIK64 should publish for PCD 1.0 and `.skill` 1.0.

Current maturity:

- PCD 1.0: public standard in preparation.
- `.skill` 1.0: public agent-instruction standard in preparation.
- docs.brik64.com: primary public documentation surface.
- `brik64/brik64-tools-skills`: current public skill source for agents.

## Planned Repositories

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

### `brik64/skill-standard`

Purpose: define the public `.skill` packaging and instruction format for AI
agents consuming BRIK64 workflows.

Required artifacts:

- `README.md`: standard overview, maturity label, and docs links.
- `SPEC.md`: file layout, metadata, triggers, versioning, and validation rules.
- `VERSIONING.md`: compatibility, deprecation, and migration policy.
- `EXAMPLES.md`: minimal skill, product skill, and claim-sensitive skill examples.
- `NOTICE`: BRIK64 Inc. ownership and attribution.
- `LICENSE` or terms file appropriate for a public standard.
- `AGENTS.md`: agent-readable usage and claim boundary.

## Publication Rule

Do not present PCD 1.0 or `.skill` 1.0 as finalized public standards until:

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
