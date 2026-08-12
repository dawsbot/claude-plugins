# claude-plugins

A [Claude Code plugin marketplace](https://docs.anthropic.com/en/docs/claude-code/plugins) by [@dawsbot](https://github.com/dawsbot).

## Install

Add the marketplace once:

```
/plugin marketplace add dawsbot/claude-plugins
```

Then install whichever plugin you want:

```
/plugin install codex-opinion@dawsbot-plugins
```

Updates ship when this repo pushes a version bump. Claude Code compares the installed plugin version against the manifest, so a fix pushed without bumping `version` in `plugin.json` is invisible to existing installs. Run `/plugin marketplace update dawsbot-plugins` to pull the latest.

## Plugins

### codex-opinion

Claude cannot QA itself. After a long task it tells a confident, articulate story about what it did, and that story is the least reliable part of the output. This skill sends the actual artifacts to Codex (OpenAI's CLI agent) as an unattributed proposal, stripped of every hint that Claude wrote them or believes they work. A different model with different blind spots then judges the work on its merits, and Claude verifies each finding against the code before relaying it.

Trigger it by name: `/codex-opinion:codex-opinion` (plugin skills are namespaced), or in plain words: "get a codex opinion", "what does codex think". The skill's instructions forbid proactive invocation; Claude only loads it when you ask.

**Prerequisite:** the [Codex CLI](https://github.com/openai/codex) must be installed and authenticated (`codex login status`). Codex usage is billed to your own OpenAI account. If Codex is missing, the skill reports that and stops rather than faking a second opinion.

### discoverable-code

Coding agents retrieve code with grep, not language servers or dependency graphs, which makes every symbol name and file path an address. `create()` returns 1,585 hits; `createStripeClient()` returns 43. This skill turns that into working rules: two-to-three-word export names, one spelling per concept, branded ID types so an argument swap becomes a build error, comments on definitions where grep lands, and test files named after their source. It ships a review checklist for auditing a diff.

Trigger it by name — `/discoverable-code:discoverable-code` — or let it load while naming exports, writing type signatures, or reviewing a diff for readability. Rules and figures come from [How Coding Agents Read Your Code](https://modem.dev/blog/how-coding-agents-read-your-code).

## Team setup

To enable plugins by default for everyone in a project, add this to that repo's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "dawsbot-plugins": {
      "source": {
        "source": "github",
        "repo": "dawsbot/claude-plugins"
      }
    }
  },
  "enabledPlugins": {
    "codex-opinion@dawsbot-plugins": true,
    "discoverable-code@dawsbot-plugins": true
  }
}
```

Teammates get an install prompt the next time they trust the project.

## License

MIT
