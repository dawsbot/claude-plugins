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

Updates ship when this repo does. Run `/plugin marketplace update dawsbot-plugins` to pull them.

## Plugins

### codex-opinion

Claude cannot QA itself. After a long task it tells a confident, articulate story about what it did, and that story is the least reliable part of the output. This skill sends the actual artifacts to Codex (OpenAI's CLI agent) as an unattributed proposal, stripped of every hint that Claude wrote them or believes they work. A different model with different blind spots then judges the work on its merits, and Claude verifies each finding against the code before relaying it.

Trigger it by name: `/codex-opinion`, "get a codex opinion", "what does codex think". It never fires on its own.

**Prerequisite:** the [Codex CLI](https://github.com/openai/codex) must be installed and authenticated (`codex login status`). Codex usage is billed to your own OpenAI account. If Codex is missing, the skill reports that and stops rather than faking a second opinion.

## Team setup

To enable a plugin by default for everyone in a project, add this to that repo's `.claude/settings.json`:

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
    "codex-opinion@dawsbot-plugins": true
  }
}
```

Teammates get an install prompt the next time they trust the project.

## License

MIT
