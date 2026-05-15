---
title: "GIT Version Control"
description: "## Monorepo Structure"
audience: tech
type: reference
status: planned
phase: 2
order: 99
lang: en
publish: true
---
# GIT Version Control

## Monorepo Structure

```
dex/
├── sdk/           # Core SDK (library, no dev server)
│   └── src/
│       ├── eth/   # Tokens, chains, contracts metadata
│       └── pool/  # Pool SDK
├── front/         # Preact frontend (Vite)
│   └── src/
│       ├── components/
│       ├── hooks/
│       ├── lib/
│       └── pages/
├── back/          # Backend services
│   └── collector/ # Bun WebSocket server
├── contracts/     # Solidity contracts (Foundry)
│   ├── src/
│   │   ├── interfaces/
│   │   ├── modules/
│   │   ├── libraries/
│   │   └── tokens/
│   ├── test/
│   └── script/
├── scripts/       # Shared build/dev scripts
├── docs/          # Documentation
│   └── 5. Contributing/
│       ├── GIT.md
│       ├── FRONTEND.md
│       ├── BACKEND.md
│       ├── SMART_CONTRACTS.md
│       ├── SECURITY.md
│       └── QUANT.md
├── specs/         # Technical specifications
├── salts/         # CREATE3 deployment salts
└── sim/           # Python quant research (UV)
```

### Module Dependencies

```
front  ──import──> sdk
back   ──import──> sdk
contracts          (independent)
sim                (independent)
```

**Rules**:
- SDK must build before front/back: `bun run build` builds sdk → back → front
- Contracts are independent (no TS dependencies)
- Use workspace protocol for internal deps: `"@sdk/eth": "workspace:*"`

### Module Commands

| Command | SDK | Front | Back | Contracts |
|---------|-----|-------|------|-----------|
| `bun run dev` | - | ✓ | ✓ | - |
| `bun run build` | ✓ | ✓ | ✓ | - |
| `bun run typecheck` | ✓ | ✓ | ✓ | - |
| `bun run lint` | ✓ | ✓ | ✓ | - |

### Root Commands

```bash
bun run dev          # Start front + back in parallel
bun run build        # Build: sdk → back → front
bun run typecheck    # Type check all modules
bun run lint         # Lint entire codebase
bun run fmt          # Format entire codebase
```

---

## Atomic Commits

**All commits must be atomic** - one focused change per commit.

✅ Good: Add single function, fix one bug, update one doc section
❌ Bad: Multiple features + bugs + docs in one commit

**Rule**: If your commit message needs "and", split it into multiple commits.

---

## Commit Categories

| Category | Prefix | Purpose |
|----------|--------|---------|
| **Feature** | `feat` | New functionality, enhancements |
| **Fix** | `fix` | Bug fixes, error corrections |
| **Documentation** | `docs` | Documentation updates only (no code logic) |
| **Refactor** | `refac` | Code restructuring, same behavior |
| **Operations** | `ops` | CI/CD, build, tooling |

---

## Branch Naming

Format: `<category>/<brief-description>`

**Rules**:
- Lowercase only
- Separate words with hyphens
- Keep brief (2-4 words)

**Examples**:
- `feat/unified-volatility`
- `fix/oracle-precision`
- `docs/update-readme`
- `refac/pricing-lib`
- `ops/ci-pipeline`

---

## Commit Messages

Format:
```
[category] Brief description in imperative mood

Optional detailed explanation.
```

**Rules**:
1. Prefix: `[category]` in lowercase
2. Imperative mood: "Add" not "Added"
3. First line ≤ 72 characters
4. Optional body after blank line
5. **NEVER mention AI tools** (Claude, GPT, Copilot, etc.) in commit messages

**Examples**:

✅ Good:
```
[feat] Add baseline volatility calculation
[fix] Correct coverage ratio haircut
[docs] Update fee system documentation
[refac] Extract helper function
```

❌ Bad:
```
Update stuff
Fixed things
feat: Add feature (wrong format)
[FEAT] Add (uppercase)
Co-Authored / Generated with Claude (never mention AI)
```

---

## Pull Requests

**Title**: Same format as commit messages: `[category] Brief description`

**Size**:
- Small: < 200 lines (ideal)
- Medium: 200-500 lines (acceptable)
- Large: > 500 lines (split into smaller PRs)

---

## Dead Code Policy

**⚠️ ZERO TOLERANCE for dead code**

- ❌ NO deprecated code, backward compatibility layers, or compatibility shims
- ❌ NO commented-out code blocks
- ❌ NO unused imports, functions, types, or variables
- ❌ NO "TODO: remove this later" comments
- ✅ Delete unused code immediately
- ✅ Update all usages when refactoring
- ✅ Clean slate - if it's not used, it's gone

**When refactoring:**
1. Update all usages first
2. Delete old code completely
3. No transition period with both versions

---

## Pre-Commit Checklist

- [ ] Change is atomic (one logical change)
- [ ] Correct category prefix used
- [ ] Commit message in imperative mood
- [ ] First line ≤ 80 characters
- [ ] No AI tool mentions in comment or commit message
- [ ] Code is type checked, linted and compiles without errors
- [ ] Only related files included
- [ ] No dead code, deprecated exports, or unused imports

---

## Comment Policy

**Concise but informative - explain WHY, not WHAT**

| Keep | Remove |
|------|--------|
| Non-obvious logic, invariants, safety constraints | Decorative ASCII art, verbose headers |
| Inline comments for complex calculations or edge cases | Obvious comments like `// Check if x is zero` for `if (x == 0)` |
| Contract-critical behavior (revert conditions, access control) | Trivial getters/setters |
| Formula implementations | Redundant descriptions |

**Function comments**:
- ✅ Explain contract-critical behavior (revert conditions, access control, state changes)
- ✅ Document non-obvious side effects
- ✅ For complex math, note the formula being implemented
- ❌ Skip trivial getters/setters

**When in doubt**: A comment that saves a future reader 5 minutes of head-scratching is worth keeping.

---

## Code Philosophy

**Performance. Elegant. Signal-Driven. DRY. Concise.**

| Principle | Description |
|-----------|-------------|
| **Performance first** | Optimize hot paths, profile before optimizing |
| **Lean** | Minimal dependencies, small bundle size |
| **DRY** | Don't repeat yourself - extract common patterns |
| **No over-abstraction** | Three similar lines of code is better than a premature abstraction |
| **Consistent formatting** | Use project's linter/formatter |
| **Silent** | No emojis, minimal verbose output |

---

## Problem Solving (For AI Agents)

**When you don't know how to do something:**

1. **WebSearch first** - Use WebSearch or web-search-prime MCP to find current information
2. **Ask second** - If web search doesn't help, ask the user

**Examples**:
- Don't know a CLI flag? → WebSearch it
- Unclear about a tool feature? → WebSearch official docs
- Need installation instructions? → WebSearch the official guide

---

## Agent Git Identity

**⚠️ Always use the user's git identity**

- ❌ NEVER use agent or "Gemini" or "Claude" identity for git commit authors or co-authors
- ✅ Use the user's configured git identity

---

## Agent Communication Style

**Keep responses SHORT**

- ❌ NO long summaries/verbose explanations
- ✅ Brief status (1-2 lines), ask when needed

---

## Related Documentation

- [`BACKEND.md`](./BACKEND.md) - Back-end best practices
- [`FRONTEND.md`](./FRONTEND.md) - Front-end best practices
- [`SMART_CONTRACTS.md`](./SMART_CONTRACTS.md) - Smart contract development
- [`SECURITY.md`](./SECURITY.md) - Security best practices
- [`QUANT.md`](./QUANT.md) - Quant/researcher guide
