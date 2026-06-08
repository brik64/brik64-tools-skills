---
name: pcd-system
description: Current public-beta PCD reference notes for BRIK64 agents. Use alongside the current brik64 skill and docs.brik64.com. Keep older platform-integration, domain-system, extended-operation, multi-target, certification, and distribution-catalog material out of current public instructions unless a current public release explicitly supports it.
triggers:
  - writing PCD notes
  - using brik CLI public beta
  - reviewing PCD examples
  - checking current docs before public claims
version: 0.1.0-beta.11-public-reference
---

# PCD System — Public Beta Reference Notes

This skill contains current public-beta PCD orientation for agents. For current
public agent operation, install and use the `brik64` skill first, then check
https://docs.brik64.com before making release, command, or capability claims.

Current public CLI command: `brik64`.
Compatibility alias: `brik`.
Current public CLI version: `0.1.0-beta.11`; verify public release status
against docs and GitHub before presenting it as published.
Current public CLI install path: `curl -fsSL https://brik64.com/cli/install.sh | bash`.
Do not use npm to install the CLI. npm is reserved for SDK packages.

Current public surface:

- `brik64` CLI public beta
- `.brik` local traceability metadata
- PCD 1.0 public standard: https://github.com/brik64/pcd-standard
- agent workflows that point to docs.brik64.com

Roadmap or non-current surface unless separately published:

- platform integration servers
- distribution catalogs
- domain-system drafts
- extended-operation drafts
- multi-target compilation claims
- formal certification claims
- compiler-sovereignty or reproducibility claims

---

## Quick Start

```bash
curl -fsSL https://brik64.com/cli/install.sh | bash
brik64 --version
brik64 help
brik64 init
brik64 certify path/to/program.pcd
brik64 verify path/to/program.pcd
brik64 emit path/to/program.pcd
brik64 polymerize path/to/program.pcd --out polymer.pcd
```

## PCD Orientation

Treat PCD as Program Circuit Description: a circuit description format for
bounded logic. In public beta writing, point to `brik64/pcd-standard` and
docs.brik64.com for the current versioned standard.

Current agent posture:

- write small, inspectable PCD candidates;
- keep input and output boundaries explicit;
- avoid claiming certification from syntax, examples, or CLI output alone;
- report exact CLI/runtime evidence when available;
- link to docs.brik64.com for the current command surface.

## Example Pattern

```pcd
// ai-safety-policy.pcd — ALLOW/BLOCK guardrail
PC file_write_policy {
    let action  = MC_63.ENV("ACTION");      // "write_file"
    let path    = MC_63.ENV("TARGET_PATH"); // "/etc/passwd"
    let agent   = MC_63.ENV("AGENT_ID");   // "gpt-4"

    fn is_protected_path(p) {
        let is_etc   = MC_42.SUBSTR(p, 0, 4) == "/etc";
        let is_sys   = MC_42.SUBSTR(p, 0, 4) == "/sys";
        let is_root  = p == "/";
        return is_etc + is_sys + is_root;
    }

    fn is_allowed_action(act) {
        // Allowlist approach: DENY by default
        let allowed = 0;
        if (act == "read_file")    { let allowed = 1; }
        if (act == "write_temp")   { let allowed = 1; }
        return allowed;
    }

    let protected = is_protected_path(path);
    let allowed   = is_allowed_action(action);

    let verdict = "BLOCK";
    if ((protected == 0) && (allowed == 1)) {
        let verdict = "ALLOW";
    }

    let _ = MC_58.WRITE(1, verdict, MC_43.LEN(verdict));
    let _ = MC_58.WRITE(1, "\n", 1);

    OUTPUT verdict;
    return verdict;
}
```

### EVA Composition

```pcd
// Sequential: A ⊗ B — output of A feeds input of B
let h = hash_sha256_int(data);
let keyed = xor_bytes_int(h, key);

// Parallel: A ∥ B — independent execution
let (a, b) = (compute_x(input), compute_y(input));

// Conditional: A ⊕ B — select based on predicate
let result = if (check) { path_a(x) } else { path_b(x) };
```

---

## CLI Boundary

Use the current public command name in examples:

```bash
brik64 --version
brik help
brik init
```

When you need command details, check docs.brik64.com and the installed CLI
itself. Do not copy older `brik64` command examples into public instructions
unless the current release proves that exact command.

## Standards Boundary

PCD 1.0 is maintained in the public standard repository:

```text
https://github.com/brik64/pcd-standard
```

Use that repository and docs.brik64.com for current PCD 1.0 file format,
authoring, examples, compatibility, and claim boundary. This skill orients
agents; it should not replace the standard repository or present historical PCD
notes as current normative grammar.
