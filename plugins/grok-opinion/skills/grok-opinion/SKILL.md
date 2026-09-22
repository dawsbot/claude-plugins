---
name: grok-opinion
description: Get a second opinion from Grok (xAI's CLI agent) on work Claude just produced. Sends the artifacts — not Claude's summary of them — to Grok under neutral framing and reports back what a different model with different biases catches. ONLY run when the user explicitly asks for it, by name ("/grok-opinion", "get a grok opinion", "what does grok think", "have grok grade this", "second-model check"). NEVER invoke proactively, never as a self-review step, never because a task felt risky or finished.
---

# Grok Opinion

## Invocation rule, read first

**Only run this when the user asks for it by name.** Do not fire it because work
just landed, because a change looks risky, because tests are green, or as a
courtesy QA pass. An unrequested Grok call spends the user's money and their
attention. If you think a second opinion would help and they haven't asked, say
so in one sentence and let them decide.

## Purpose

Claude cannot QA itself. After a long task it is confident, articulate, and
tells clean stories about what it did — and those stories are the least reliable
part of the output. A different model, with different training and different
blind spots, catches what Claude missed. This skill is the mechanism.

The value comes entirely from Grok reaching an **independent** judgment. Every
rule below exists to protect that independence.

## The failure mode this skill is built around

A review prompt that frames Claude's work as *the answer* and asks the reviewer
to check it gets agreement, because the reviewer pattern-matches on the
confident frame instead of evaluating. Agreement obtained that way is worth
nothing.

The fix: **present the work as an unattributed proposal, and strip every trace of
Claude's own verdict on it.** Grok should not be able to tell whether it is
looking at something good or something broken.

### Never pass to Grok

- Any statement that the work is correct, complete, verified, or passing
- Claude's summary, narrative, or explanation of what it did or why
- The words "Claude", "I", or "the assistant" anywhere in the framing
- Claude's own commit messages or PR descriptions (these are argument, not evidence)
- Words like "confirm", "validate", "double-check", "sanity-check", "make sure"
- Passing test output presented as proof
- Findings from any other reviewer (a prior Codex or human review). Grok must
  not know what has already been found, or it anchors on that list.

### Always pass to Grok

- The original task, in the user's words where possible
- The actual artifacts: the diff, the files, the commands and their real output
- The test *code*, not just the fact that tests passed — a test can be vacuous
- Honest unknowns: what was not run, not covered, or assumed

## Procedure

### 1. Identify the target

Ask the user only if genuinely ambiguous. Otherwise default to the most recent
substantive unit of work: the current branch's diff against its merge-base, or
the specific files just written.

```bash
DEFAULT_BRANCH=$(git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null || echo origin/main)
git diff $(git merge-base HEAD "$DEFAULT_BRANCH")...HEAD
git diff HEAD                              # uncommitted work is part of the proposal
git status --porcelain                     # untracked files may be too
```

The merge-base diff alone misses staged, unstaged, and untracked changes. If
the work just produced is not yet committed, include the working-tree diff and
the relevant untracked files, or the packet reviews an empty or obsolete
proposal. Prefer real artifacts over prose. If the work is not in git at all,
pass the files.

### 2. Assemble the packet

Write a prompt file to a temporary directory. Structure:

```
# Task
<the original request, in the user's words>

# Proposal under review
<the diff, or the file contents>

# What was run
<commands and their verbatim output, including failures>

# Not established
<what was not run, not covered, or assumed>
```

Then the instruction block. Use this wording; it is load-bearing:

```
You are reviewing a proposed change from an unnamed contributor. It has not
been accepted, and it may be wrong in ways that are not obvious.

Judge it on its merits against the task. Read the code that matters, including
the tests — a passing test suite is evidence about the tests, not about the
code. You have read-only access to the working directory; use it.

Report:
1. Anything actually broken: wrong behavior, wrong values, a real failure mode.
   Give file:line and the concrete input or state that breaks it.
2. Anything the tests appear to cover but do not. Say what would still pass if
   the implementation were wrong.
3. Anything about the approach you would have done differently, and why it
   matters — skip pure style.
4. What you verified by reading versus what you are assuming.

If you find nothing real, say so plainly. Do not manufacture findings, and do
not soften real ones. Disagreeing with the proposal is a normal outcome.
```

### 3. Run Grok

Headless, read-only tools only. It is reviewing, not editing. Grok's headless
mode is `-p` / `--prompt-file`; `--tools` is an allowlist that removes every
other built-in tool, so the agent physically cannot edit, run shell commands,
or spawn subagents.

```bash
grok --prompt-file "$PROMPT_FILE" \
  --cwd "$(git rev-parse --show-toplevel 2>/dev/null || pwd)" \
  --tools "read_file,grep,list_dir,web_search,web_fetch" \
  --disallowed-tools "Agent" \
  --permission-mode dontAsk \
  --output-format plain \
  --max-turns 40 \
  2>&1 | tee "$OUT_FILE"
```

Notes:
- `--prompt-file` avoids shell-quoting damage on large diffs. Do not pass the
  packet with `-p "$(cat ...)"`.
- `-m <model>` if the user names one; otherwise take the configured default
  (`grok models` shows it). `--effort high` is reasonable for a review.
- Drop `web_search,web_fetch` from `--tools` if the packet contains anything
  the user would not want leaving the machine beyond xAI itself.
- Grok is slow on large packets. Run it in the background and keep working if
  there is anything useful to do meanwhile.
- If `grok` is missing or unauthenticated (`grok models` fails with an auth
  error), report that and stop. Do not fall back to reviewing the work yourself
  and presenting it as a second opinion — that defeats the entire purpose.

### 4. Adjudicate, don't just relay

Grok is a second opinion, not an oracle. It will be wrong sometimes, and it
lacks the conversation's context.

For each finding, verify it against the code before accepting it. Then report:

- **Confirmed** — real, with the evidence you checked. Fix these or list them.
- **Wrong** — say why, specifically. Grok missing context is common.
- **Unclear** — needs the user's judgment or domain knowledge.

Lead with anything confirmed. If Grok found a real problem, that is the headline
and it should not be buried under process narration. If it found nothing real,
say that plainly too — but only after checking, since "Grok agreed" is exactly
the outcome a leading prompt manufactures.

Never suppress a finding because it criticizes work done earlier in the session.
That is the specific bias this skill exists to counter. If another model has
already reviewed the same work, say which findings are new, which overlap, and
which contradict, after both have been checked against the code.

## Reference invocation

Working end-to-end example, for reuse:

```bash
S=$(mktemp -d)             # working dir for the packet and output
B=$(git merge-base HEAD origin/main)

# build packet (see step 2 for the full structure)
{ ...; } > "$S/grok-prompt.md"

# audit the packet for leading language before sending. Hits inside the diff
# itself are fine — that is the artifact under review, and Grok should judge
# its claims. Hits in YOUR framing are the bug.
rg -in "claude\|codex\|correct\|verified\|confirm\|validate" "$S/grok-prompt.md"

grok --prompt-file "$S/grok-prompt.md" --cwd "$(git rev-parse --show-toplevel)" \
  --tools "read_file,grep,list_dir,web_search,web_fetch" --disallowed-tools "Agent" \
  --permission-mode dontAsk --output-format plain --max-turns 40 \
  > "$S/grok-out.txt" 2>&1
```

Elide large generated data files from the diff and pass a representative sample
plus a shape description instead. A 2000-line JSON snapshot crowds out the code
that actually needs judging.
