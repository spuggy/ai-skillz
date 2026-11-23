# ai-skillz

Agent skills for [Claude Code](https://claude.com/claude-code).

## Skills

- **[gherkin-spec](skills/gherkin-spec/SKILL.md)**: turns a loose feature request into a Gherkin `.feature` spec that says
  *what* the system does, never *how*, and ends with a list of the decisions you need to confirm. Writes the spec only,
  never the step code or implementation.

## Install

Copy a skill folder into your project's `.claude/skills/` (or `~/.claude/skills/` to use it everywhere):

```sh
git clone https://github.com/spuggy/ai-skillz.git
cp -r ai-skillz/skills/gherkin-spec ~/.claude/skills/
```

Then ask Claude to "spec this" or run `/gherkin-spec`.
