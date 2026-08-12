# discoverable-code

Code style for a codebase that coding agents can navigate. Agents retrieve code with grep — not language servers, not dependency graphs — so every symbol name and file path is an address. A generic name is a bad address, and the agent pays for it in wasted file reads and confidently wrong answers.

## What it enforces

- **Two-to-three-word names.** Across 7,922 exported TypeScript names, one word is 61% unique in grep results, two is 88%, three is 96%. `createStripeClient()` returns 43 hits where `create()` returns 1,585.
- **One spelling per concept**, including no import aliases — an alias breaks the text-search link between definition and use.
- **Types that answer the question.** Branded `UserId`/`OrgId` instead of three interchangeable `string`s, so an argument swap becomes a build error the agent can fix in one turn. No `any`.
- **Comments on definitions**, because that is where grep results land.
- **Searchable paths.** Test files named after their source, modules named after concepts rather than layers.
- **A conventions file** recording the non-obvious.

It also ships a review checklist for auditing a diff, and an `rg -w 'name' --stats` check for testing whether a name is distinctive before you commit to it.

## Usage

Ask by name — `/discoverable-code:discoverable-code` (plugin skills are namespaced) — or let it load when you are naming exports, writing signatures, or reviewing a diff for readability.

## Source

Rules and figures are drawn from [How Coding Agents Read Your Code](https://modem.dev/blog/how-coding-agents-read-your-code) (modem.dev), which measured 6–66% token reductions across fourteen model/harness pairs on a generated library, and 31–34% on a real 4,943-line refactor. The gains are largest for smaller, faster models — they have the least budget to spend recovering from a bad search.
