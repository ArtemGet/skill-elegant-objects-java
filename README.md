[![EO principles respected here](https://www.elegantobjects.org/badge.svg)](https://www.elegantobjects.org)

# Elegant Objects Java — Agent Skill

A portable **Agent Skill** that lets an AI agent write, review, test, build, release and maintain
Java projects in [Yegor Bugayenko](https://www.yegor256.com)'s
[Elegant Objects](https://www.elegantobjects.org) style — plus the build, CI/CD, testing and
repository conventions his real projects (Cactoos, Takes, Qulice, Rultor, eo, …) actually use.

## What's here

```
skill/                                  the skill (portable core + deep knowledge)
  SKILL.md                              ~200-line core: canon, 20 rules, workflow, DoD, reference map
  AGENTS.md                             entry point for agents that only read AGENTS.md
  references/                           11 task-scoped guides (progressive disclosure)
    01-design.md 02-testing.md 03-code-style.md 04-build-maven.md 05-cicd-release.md
    06-repository-docs.md 07-process.md 08-tooling.md 09-recipes.md
    10-branching-and-commits.md 11-reviewer-playbook.md
  knowledge/
    elegant-objects-java.md             full evidence base (book + articles + real codebases), for RAG
    distilled-rules.md                  prioritized rulebook (MUST / SHOULD / NICE)

.opencode/skills/elegant-objects-java/  ready-to-use copy for opencode
.claude/skills/elegant-objects-java/    ready-to-use copy for Claude Code
.agents/skills/elegant-objects-java/    ready-to-use copy for generic agents
```

> The study sources (downloaded articles, cloned repositories, the book PDF, the analysis output and
> the fetch/OCR scripts) are intentionally **not** published here — only the resulting skill is.

## Install / use

The skill uses portable YAML frontmatter (`name` + `description`), so it works in Claude, opencode,
Cursor, Gemini and other Agent-Skill runtimes. Pick the matching location:

| Runtime | Put the `skill/` folder at |
|---|---|
| opencode (project) | `.opencode/skills/elegant-objects-java/` |
| opencode (global) | `~/.config/opencode/skills/elegant-objects-java/` |
| Claude Code | `.claude/skills/elegant-objects-java/` |
| generic agents | `.agents/skills/elegant-objects-java/` |

The copies in this repo are already in place for opencode, Claude Code and generic agents. Re-copy
after editing `skill/`.

## Skill best practices this follows

- **Progressive disclosure:** ~200-line `SKILL.md` (always cheap to load) → `references/*.md` (loaded
  per task) → `knowledge/*.md` (loaded only for deep dives / RAG).
- **One job:** "own a Java project the Elegant Objects way," not "all of software engineering."
- **A description that says *what* and *when*:** the only text always in context, so it names both
  the capabilities and the triggers ("Use when writing, refactoring, reviewing Java…").
- **Imperative, checklisted, example-first:** instructions an agent can execute, not essays.
- **Enforceable:** every `[MUST]` maps to a build gate (Qulice, Jacoco/PIT, Revapi, bot-merged
  read-only `master`). A rule that cannot fail the build gets ignored.
- **Portable layout:** identical core deployed to each runtime's skill directory.

## Sizes (why they are what they are)

| Layer | Size | Role |
|---|---|---|
| `SKILL.md` | ~200 lines | always-on core |
| `references/*.md` | ~150–420 lines each | one per task |
| `knowledge/distilled-rules.md` | ~720 lines | checklist / RAG |
| `knowledge/elegant-objects-java.md` | ~6000 lines | full evidence base / RAG |

The working ceiling for a skill body is roughly 5k tokens (~300–500 lines of Markdown): everything
beyond that is paid on every activation, so depth lives in files the agent opens on demand.

## How this was built (pipeline)

1. Cloned the 10 most popular Java (or Java-focused) `yegor256` repositories, incl. `takes`, `eo`,
   `cactoos`, `qulice`.
2. Downloaded every article from `yegor256.com/tag/java.html` and the key tags of
   `yegor256.com/contents.html` (programming, oop, tdd/testing/tests, quality, maintainability,
   style, devops, docker, maven, rultor, github, specs, pdd, xdsd, zerocracy, oss, management,
   agile, architect, aop, restful, http, eolang) — 333 unique articles, plus `elegantobjects.org`.
3. OCR'd the *Elegant Objects* book PDF (234 scanned pages) with Tesseract.
4. One agent extracted the book's postulates with examples; eight parallel agents added findings
   from the articles (isolated); a merge agent combined them; seven agents analysed the codebases
   for real CI/CD, testing, style, repository and tooling practice; a final agent produced the
   superset knowledge base and the distilled rulebook.
5. Wrote the lean core (`SKILL.md`) by hand and the task-scoped reference guides, then deployed the
   portable layout to each runtime.

## Sources

- https://www.elegantobjects.org
- https://www.yegor256.com (blog, tag pages)
- *Elegant Objects* by Yegor Bugayenko (vol. 1)
- https://github.com/yegor256 (and `objectionary/eo`)
