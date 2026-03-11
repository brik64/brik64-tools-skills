---
name: pcd-policy-circuits
description: "Design and implement AI safety policy circuits in PCD. Use when building ALLOW/BLOCK guardrails for AI agents, validating action proposals, or implementing access control circuits. Ensures Φ_c = 1 (no ambiguous paths)."
---

# PCD Policy Circuits — AI Safety Guardrails

Design and implement formally verified policy circuits that intercept AI agent actions and return ALLOW or BLOCK. Policy circuits are the core AI safety primitive of BRIK-64.

**Documentation:** https://docs.brik64.dev/pcd/examples

---

## What is a Policy Circuit

A Policy Circuit is a PCD program that:
1. Receives a proposed AI action (JSON via stdin or argument)
2. Evaluates it against a set of rules
3. Returns `ALLOW` or `BLOCK: <reason>`
4. Has **Φ_c = 1** — the compiler proves that every possible input produces a definitive result

The key guarantee: **there is no input that causes undefined behavior, a crash, or an ambiguous result.**

## Template: Minimal Policy Circuit

```pcd
import "stdlib/string.pcd";
import "stdlib/json.pcd";

PC policy {
    let action_json = MC_56.READ("/dev/stdin");
    let action = parse(action_json);

    let action_type = get(action, "type");
    let resource = get(action, "resource");

    // Rule 1: block dangerous paths
    if (starts_with(resource, "/etc/")) {
        OUTPUT "BLOCK: /etc/ access forbidden";
    }

    // Rule 2: block shell execution
    if (action_type == "shell") {
        OUTPUT "BLOCK: shell execution forbidden";
    }

    // All rules passed
    OUTPUT "ALLOW";
}
```

## Design Rules for Policy Circuits

### Rule 1: Every path MUST terminate with OUTPUT

The compiler enforces this via Φ_c = 1. If any input can fall through without an OUTPUT, compilation fails.

```pcd
// GOOD: fallback OUTPUT at the end
if (action_type == "write") { OUTPUT "BLOCK: no writes"; }
if (action_type == "read")  { OUTPUT "ALLOW"; }
OUTPUT "BLOCK: unknown action type";  // catches everything else

// BAD: what if action_type is "execute"?
if (action_type == "write") { OUTPUT "BLOCK"; }
if (action_type == "read")  { OUTPUT "ALLOW"; }
// Falls through! Φ_c < 1 — WILL NOT COMPILE
```

### Rule 2: BLOCK before ALLOW

Evaluate blocking rules first, allow at the end. This is the "deny by default" pattern:

```pcd
// Check dangerous conditions first
if (is_dangerous(action)) { OUTPUT "BLOCK: dangerous"; }
if (is_unauthorized(agent)) { OUTPUT "BLOCK: unauthorized"; }
if (is_rate_limited(agent)) { OUTPUT "BLOCK: rate limited"; }

// Only reached if ALL checks pass
OUTPUT "ALLOW";
```

### Rule 3: Include reason in BLOCK

Always format as `BLOCK: <human-readable reason>`. This enables auditing:

```pcd
OUTPUT "BLOCK: writes to /etc are forbidden for agent " + agent_id;
```

### Rule 4: Validate agent identity

```pcd
let known_agents = ["claude-code", "codex-cli", "gemini-cli"];
let is_known = contains(known_agents, agent_id);
if (!is_known) {
    OUTPUT "BLOCK: unknown agent_id: " + agent_id;
}
```

## Complete Policy Circuit: File Access Control

```pcd
import "stdlib/string.pcd";
import "stdlib/json.pcd";
import "stdlib/array.pcd";

PC file_access_policy {
    let action_json = MC_56.READ("/dev/stdin");
    let action = parse(action_json);

    let op       = get(action, "operation");    // "read" | "write" | "delete"
    let path     = get(action, "path");
    let agent_id = get(action, "agent_id");

    // === RULE 1: Agent allowlist ===
    let known = ["claude-code", "codex-cli"];
    if (!contains(known, agent_id)) {
        OUTPUT "BLOCK: unknown agent " + agent_id;
    }

    // === RULE 2: No access to secrets ===
    if (contains(path, ".env")) {
        OUTPUT "BLOCK: .env access forbidden";
    }
    if (contains(path, "credentials")) {
        OUTPUT "BLOCK: credentials access forbidden";
    }
    if (starts_with(path, "/etc/shadow")) {
        OUTPUT "BLOCK: shadow file access forbidden";
    }

    // === RULE 3: Write restrictions ===
    if (op == "write") {
        if (!starts_with(path, "/tmp/") && !starts_with(path, "/home/")) {
            OUTPUT "BLOCK: writes only to /tmp/ or /home/";
        }
    }

    // === RULE 4: Delete restrictions ===
    if (op == "delete") {
        if (!starts_with(path, "/tmp/")) {
            OUTPUT "BLOCK: deletes only in /tmp/";
        }
    }

    // === RULE 5: No dotfiles ===
    if (contains(path, "/.")) {
        OUTPUT "BLOCK: dotfile access forbidden";
    }

    // All checks passed
    OUTPUT "ALLOW";
}
```

## Network Policy Circuit

```pcd
import "stdlib/string.pcd";
import "stdlib/json.pcd";
import "stdlib/array.pcd";

PC network_policy {
    let action_json = MC_56.READ("/dev/stdin");
    let action = parse(action_json);

    let method = get(action, "method");    // GET, POST, PUT, DELETE
    let url    = get(action, "url");
    let host   = get(action, "host");

    // Allowlisted domains
    let allowed = [
        "api.anthropic.com",
        "api.openai.com",
        "registry.brik64.dev",
        "github.com"
    ];

    if (!contains(allowed, host)) {
        OUTPUT "BLOCK: host " + host + " not in allowlist";
    }

    // No mutation on external APIs without explicit permission
    if (method == "DELETE") {
        OUTPUT "BLOCK: DELETE requests forbidden";
    }

    if (method == "PUT" && !starts_with(url, "/api/v1/drafts")) {
        OUTPUT "BLOCK: PUT only allowed for /api/v1/drafts";
    }

    OUTPUT "ALLOW";
}
```

## Compile and Deploy

```bash
# Compile to native binary
brikc compile policy.pcd -o policy

# Test with JSON input
echo '{"type":"write","resource":"/etc/passwd","agent_id":"claude-code"}' | ./policy
# BLOCK: /etc/ access forbidden

echo '{"type":"read","resource":"/tmp/data.txt","agent_id":"claude-code"}' | ./policy
# ALLOW

# Cross-compile for integration
brikc compile policy.pcd --target js -o policy.mjs    # Node.js middleware
brikc compile policy.pcd --target py -o policy.py     # Python wrapper
brikc compile policy.pcd --target wasm32 -o policy.wasm  # Browser sandbox
```

## Integration Patterns

### As CLI middleware

```bash
# Before executing an AI action:
echo "$ACTION_JSON" | ./policy
if [ $? -eq 0 ] && [ "$(echo "$ACTION_JSON" | ./policy)" = "ALLOW" ]; then
    execute_action "$ACTION_JSON"
fi
```

### As Node.js middleware

```javascript
import { policy } from './policy.mjs';

async function guardedAction(action) {
    const result = policy(JSON.stringify(action));
    if (!result.startsWith('ALLOW')) {
        throw new Error(result);
    }
    return executeAction(action);
}
```

## Checklist for Policy Circuits

- [ ] Every rule ends with `OUTPUT "BLOCK: reason"`
- [ ] Final line is `OUTPUT "ALLOW"` (deny-by-default)
- [ ] Agent identity is validated against an allowlist
- [ ] Sensitive paths (.env, /etc/, credentials) are blocked
- [ ] BLOCK messages include human-readable reasons
- [ ] `brikc check policy.pcd` returns Φ_c = 1.000
- [ ] Tested with both ALLOW and BLOCK scenarios
