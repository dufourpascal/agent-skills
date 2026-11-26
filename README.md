# agent-skills

Shared agent skills for [Claude Code](https://claude.com/claude-code) and [Codex](https://developers.openai.com/codex), packaged as plugins for both. The workflow skills assume T3 Code with both Claude Code and Codex available.

| Plugin | Skills |
| --- | --- |
| `ship` | `full-send`, `full-stop`, `babysit-pr`, `run-overnight`, `council` |
| `frontend` | `frontend-design`, `frontend-slides` |

## Install

Claude Code:

```sh
claude plugin marketplace add dufourpascal/agent-skills
claude plugin install ship@dufourpascal
claude plugin install frontend@dufourpascal
```

Codex:

```sh
codex plugin marketplace add dufourpascal/agent-skills
codex plugin add ship@dufourpascal
codex plugin add frontend@dufourpascal
```

Start a new session afterwards so the skills load.

## Update

```sh
claude plugin marketplace update dufourpascal && claude plugin update ship@dufourpascal && claude plugin update frontend@dufourpascal
codex plugin marketplace upgrade dufourpascal
```

## Global instructions

[`instructions/AGENTS.md`](instructions/AGENTS.md) is a reference for global agent instructions (model routing, git safety, independent review). It isn't installed by the plugins. Copy what fits into your own `~/.codex/AGENTS.md` (and import that from `~/.claude/CLAUDE.md` with `@~/.codex/AGENTS.md`). Several skills follow the rules defined there.

## Layout

```
.claude-plugin/marketplace.json     Claude Code marketplace
.agents/plugins/marketplace.json    Codex marketplace
plugins/<plugin>/
  .claude-plugin/plugin.json        Claude Code plugin manifest
  .codex-plugin/plugin.json         Codex plugin manifest
  skills/<skill>/SKILL.md           shared by both
```

## Contributing

Edit the skill under `plugins/<plugin>/skills/`, then bump `version` in **both** manifests of that plugin. Both tools cache plugins by version, so an unbumped change doesn't reach installed copies. Check with `claude plugin validate .`.

For live editing, link a skill directory into `~/.codex/skills/` and `~/.claude/skills/` instead of installing the plugin, so the skill doesn't load twice.

## Licenses

`frontend-design` is Apache-2.0 (based on Anthropic's frontend-design skill and Paul Bakaus's Impeccable; see its `LICENSE` and `NOTICE.md`). `frontend-slides` is MIT (by Zara Zhang; see its `LICENSE`). Both have been modified.
