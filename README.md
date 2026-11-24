# Claude skills

Personal [Claude Code skills](https://docs.claude.com/en/docs/claude-code/skills). This repo is the
`~/.claude/skills` folder itself, so every Claude Code session on the machine loads them.

## Install on another machine

```bash
git clone git@github.com:elijahsimkin/claude-skills.git ~/.claude/skills
```

If `~/.claude/skills` already exists, clone elsewhere and move its contents in. `synced/` is managed
by the Claude app and is ignored.

## Skills

| Skill | Use it to |
|---|---|
| [contract-coverage-map](contract-coverage-map/SKILL.md) | Map an endpoint's input contracts to the tests that cover them, as a Mermaid diagram |
| [testing-checklist](testing-checklist/SKILL.md) | Check what a new feature needs tested for thorough coverage |
