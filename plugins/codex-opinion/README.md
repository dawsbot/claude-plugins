# codex-opinion

A second opinion that means something. This skill sends Claude's work to Codex (OpenAI's CLI agent) as an unattributed proposal, with every trace of Claude's own verdict stripped out, so Codex judges the code instead of agreeing with a confident summary. Claude then verifies each finding before relaying it.

## Why the blind framing

The first version of this skill led the witness. It presented Claude's work as the answer and asked Codex to check it, and Codex mostly agreed. Agreement obtained that way is worth nothing. The current version passes only the task, the raw artifacts, and the honest unknowns, worded so Codex cannot tell whether it is reviewing something good or something broken.

## Usage

Ask for it by name inside Claude Code:

- `/codex-opinion:codex-opinion` (plugin skills are namespaced by plugin name)
- "get a codex opinion"
- "what does codex think"
- "have codex grade this"

The skill's first instruction forbids running unrequested. An unrequested Codex call spends your money without asking, so Claude is told to suggest it in one sentence instead and let you decide.

## Prerequisites

- [Codex CLI](https://github.com/openai/codex) installed
- Authenticated to your OpenAI account (`codex login status`)

Codex runs in a read-only sandbox. It reviews; it never edits.
