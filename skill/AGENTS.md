# Elegant Objects Java — agent instructions

This folder is a portable Agent Skill. If your agent runtime supports skills, load `SKILL.md`.
If it only supports a plain `AGENTS.md`, treat this file as the entry point.

## How to use

1. Read `SKILL.md` — it has the canon, the 20 non-negotiable rules, the task workflow, the
   Definition of Done, and the reference map.
2. Open the ONE reference file that matches the task (see the table in `SKILL.md`).
3. Only if you need provenance or edge cases, search `knowledge/elegant-objects-java.md`
   (the full book + articles + codebase knowledge base) or `knowledge/distilled-rules.md`.

## Default behavior when writing Java here

- `final` classes, `private final` fields, code-free constructors, one primary ctor last.
- Interfaces for every public method; behavior added by decorators, never by `extends` of a concrete class.
- No `null`, no `public static`, no getters/setters, no `instanceof`/casting, no `-er` names.
- Tests: arrange by constructing objects, then a single `MatcherAssert.assertThat(...)`.
- Run `mvn --errors --batch-mode clean install -Pqulice` before considering work done.
- One ticket per change; commit message starts with `#<ticket>`.

## Install into other agents

Copy this folder (or symlink it) into the runtime's skill directory:

| Runtime | Target |
|---|---|
| opencode | `.opencode/skill/elegant-objects-java/` |
| Claude Code | `.claude/skills/elegant-objects-java/` |
| generic Agents | `.agents/skills/elegant-objects-java/` |
| other | point the loader at this folder; `SKILL.md` has portable YAML frontmatter |
