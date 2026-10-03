# Knowledge base (deep dive / RAG)

These files are the "max" version of the skill: the full evidence behind the lean `SKILL.md` and
`references/` guides. Read them only when you need provenance, edge cases, or more examples.

- `elegant-objects-java.md` (~6000 lines) — a superset knowledge base: every postulate from
  Yegor Bugayenko's book *Elegant Objects* (verbatim, `B*` ids), additions distilled from 333 blog
  articles (`A*` ids), and generalizable findings from ten real repositories (`code-*.md` sources:
  cactoos, takes, qulice, eo, rultor, jare, s3auth, xembly, requs, rehttp). Sections 1–12 mirror the
  skill topics; 13–16 add reference architecture, CI/CD, a tool catalogue and pragmatic deviations.
- `distilled-rules.md` — a compact, prioritized checklist: **315 rules** tagged `[MUST]` /
  `[SHOULD]` / `[NICE]`, deduplicated across sources, with an anti-pattern catalogue and a
  source-disagreement resolution table.

Search strategy: Grep for a rule id (`B2.1`, `Atq.21`, `Ada.3`), a topic word, or a class name.
The `references/` guides cite these ids, so you can jump from a rule to its evidence.

The base rulebook these were built from defines priorities:
[MUST] = the build should fail on it · [SHOULD] = strong default, deviate with a written reason ·
[NICE] = review guidance.
