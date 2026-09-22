# grok-opinion

A second opinion that means something. This skill sends Claude's work to Grok (xAI's CLI agent) as an unattributed proposal, with every trace of Claude's own verdict stripped out, so Grok judges the code instead of agreeing with a confident summary. Claude then verifies each finding before relaying it.

It is the sibling of `codex-opinion`. Run both when a change matters: two models with different training disagree in useful ways, and the skill tells Claude to keep each reviewer blind to the other's findings so neither anchors.

## Why the blind framing

A review prompt that presents the work as the answer and asks the reviewer to check it gets agreement, because the reviewer pattern-matches on the confident frame. Agreement obtained that way is worth nothing. This skill passes only the task, the raw artifacts, and the honest unknowns, worded so Grok cannot tell whether it is reviewing something good or something broken.

## Usage

Ask for it by name inside Claude Code:

- `/grok-opinion:grok-opinion` (plugin skills are namespaced by plugin name)
- "get a grok opinion"
- "what does grok think"
- "have grok grade this"

The skill's first instruction forbids running unrequested. An unrequested Grok call spends your money without asking, so Claude is told to suggest it in one sentence instead and let you decide.

## Prerequisites

- [Grok CLI](https://x.ai/cli) installed (`curl -fsSL https://x.ai/cli/install.sh | bash`)
- Signed in (`grok login`; `grok models` succeeds when auth is good)

Grok runs headless with a read-only tool allowlist (`read_file`, `grep`, `list_dir`, plus web lookup). It cannot edit files, run shell commands, or spawn subagents. It reviews; it never edits.
