# Standalone skill

`prose-engineered-copy/SKILL.md` packs this core's method into one Claude
skill that runs without the Hedgehog CLI. It keeps the brief, the
draft / `check-copy` / revise loop (6-run cap), the full rule set and the
per-format contracts, and drops the build graph, BMAD intake, scratch
directory and star prompt.

The skill clones this repository and installs `workspace/scripts/check-copy`
on first use. When Node or the network is unavailable it falls back to a
manual pass against the same rules and labels the result "not gate-checked".

To install it, copy the `prose-engineered-copy` folder into your skills
directory (for Claude Code, `~/.claude/skills/`), or upload `SKILL.md` as a
skill on claude.ai.
