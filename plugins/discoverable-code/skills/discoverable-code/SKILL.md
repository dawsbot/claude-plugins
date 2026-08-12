---
name: discoverable-code
description: Code style rules for making a codebase legible to coding agents, which navigate by grep rather than by semantic analysis. Use when naming exports, writing type signatures, placing comments, organizing files, or reviewing a diff for agent-readability. Covers the three-word naming threshold, branded ID types, comment placement at definitions, and conventions files.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
  - Edit
  - Write
---

# Writing Code Coding Agents Can Find

Source: [How Coding Agents Read Your Code](https://modem.dev/blog/how-coding-agents-read-your-code) (modem.dev).

## Core Insight

Coding agents retrieve code with **text search — grep/ripgrep — not language servers, not dependency graphs, not semantic indexes.** Every consequence below follows from that one fact:

- **Symbol names and file paths are both search terms.** A name is an address, not just a label.
- **Agents cannot do reverse lookups through imports.** To find callers, an agent searches for the name. If the name is generic, the search is useless.
- **Every irrelevant file the agent opens costs hundreds of tokens** and dilutes attention across the context window.
- **Models trust what the code says it does.** A misleading name is worse than no name — the article reports misleading names cut accuracy on code-reasoning tasks by roughly 23%, and chain-of-thought recovers only part of that loss.

Discoverability is therefore a performance characteristic of the codebase, not a matter of taste.

---

## Rule 1: Names Need Two to Three Words

> "Three words is roughly where a name stops being ambiguous and starts working as an address."

Across 7,922 exported TypeScript names, grep-result uniqueness by word count:

| Words | Example | Unique in grep |
|---|---|---|
| 1 | `get` | 61% |
| 2 | `getUser` | 88% |
| 3 | `getUserProfile` | **96%** |

The same effect on real search cost:

| Name | grep hits |
|---|---|
| `create()` | 1,585 |
| `createClient()` | 466 |
| `createStripeClient()` | **43** |

**Pattern: domain word + specific descriptor.** Write `diffUserObjects`, not `diff`.

**Apply to exports.** Local variables inside a short function are not search targets and do not need this treatment — the cost lands on names that cross file boundaries.

**Banned as standalone export names:** `create`, `update`, `get`, `handle`, `process`, `run`, `data`, `config`, `utils`, `helpers`, `manager`, `item`, `value`.

---

## Rule 2: One Spelling Per Concept

> "Don't call it `organizations` in one place and `customers` somewhere else — even using a local import alias."

An agent that greps `organization` and misses the `customer` half of the system will confidently report on a fragment. Import aliases are a trap: they break the text-search link between definition and use even when the types line up.

Pick one word per concept and use it in symbol names, file names, table names, and prose.

---

## Rule 3: Make Types Answer the Question

Precise signatures let the agent stop reading. A vague signature forces it into the implementation.

**Use branded/newtype IDs for distinct concepts:**

```typescript
// Ambiguous — three interchangeable strings, and a swap compiles fine
function transferOwnership(userId: string, orgId: string, projectId: string)

// Discoverable — a swap becomes a build error the agent can fix in one turn
function transferOwnership(userId: UserId, orgId: OrgId, projectId: ProjectId)
```

The point is not type-safety theater: it converts a silent argument swap into compiler feedback, which is the fastest signal an agent can act on.

**Avoid `any`.** Every `any` increases the chance the agent must read the implementation to learn what it is dealing with.

**Prefer specific input/output types over broad ones** so the agent sees the answer in the grep result and keeps moving.

---

## Rule 4: Put the Comment on the Definition

> "The definition is the one spot you can count on it reading. That makes the one-line comment directly above a definition the most cost-effective documentation you can write."

Grep results land the agent at definitions. Documentation anywhere else — a wiki, a README section, a comment at the call site — is documentation the agent will usually not see.

- One line directly above each exported definition, stating what it does and any non-obvious constraint.
- Prefer this over block comments elsewhere in the file.
- Mark legacy code with `@deprecated` (TypeScript's tag, or the language equivalent) so the agent knows not to pattern-match on it.

---

## Rule 5: Make File Paths Searchable

Paths are search terms too.

- **Name test files after their source:** `stripe.test.ts` tests `stripe.ts`. The agent finds the tests for a file by transforming the path, with no extra search.
- **Name modules after the concept they contain**, not after their layer. `email-delivery.ts` beats `helpers.ts`.
- **Split very large files along concept lines.** The article's real-world case: splitting a 4,943-line email subsystem into concept-named modules cut token spend 31–34% for models that had been getting lost in it, and moved Haiku and Sonnet from 7/16 to 16/16 on bug detection under a fixed budget.

---

## Rule 6: Write the Conventions File

Maintain an `AGENTS.md` (or `CLAUDE.md`) at the repo root recording anything non-obvious:

- Naming conventions this repo follows
- Where each kind of source code lives
- The canonical word for each domain concept (see Rule 2)
- Build/test commands
- Known-deprecated areas to avoid

---

## Review Checklist

When reviewing a diff for discoverability, check each new or renamed export:

1. **Two to three words**, including a domain word?
2. **Distinctive** — would grepping it return a workable number of hits, or hundreds?
3. **Honest** — does the name describe what the code actually does? (Misleading beats missing for damage.)
4. **Consistent** — does it use the repo's existing word for this concept, with no alias?
5. **Typed** — precise parameter and return types, branded IDs for distinct identifiers, no `any`?
6. **Commented at the definition** — one line, non-obvious constraints stated?
7. **Test file named after the source file?**
8. **Legacy marked `@deprecated`?**

To sanity-check a name's distinctiveness before committing to it:

```bash
rg -w 'proposedName' --stats
```

Hundreds of hits means the name is an address collision. Add a domain word.

---

## Evidence

From the article's experiments:

- **Generated-library test** (TypeScript, 1,680 runs): code written under a discoverable-code skill cut token spend **6–66%** across fourteen model/harness pairs. Generic-named versions produced confidently wrong answers; distinctive-named versions eliminated those errors.
- **Real-world refactor** (Odysseus email subsystem): concept-named module split cut token spend **31–34%**, and raised bug-detection from **7/16 to 16/16** for Haiku and Sonnet under fixed budgets.
- **Misleading names** reduce model accuracy on code reasoning by roughly **23%**.

The consistent finding: the gains are largest for smaller/faster models, which have the least budget to spend recovering from a bad search.
