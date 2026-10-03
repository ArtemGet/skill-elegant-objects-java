# 10 — Branching, Commits & Merge Policy

Read this when you start work on a ticket, make commits, or open/merge a pull request.
Every rule carries a priority tag: **[MUST]** blocks the merge, **[SHOULD]** is the strong default
(deviate only with a written reason in a ticket), **[NICE]** is review guidance.

Grounded in the real conventions of Yegor Bugayenko's projects, primarily EOLANG
(`objectionary/eo`), with `yegor256/takes` and `yegor256/cactoos` as corroboration. Evidence:
`code/eo/README.md` §"How to Contribute", `code/eo/.rultor.yml`, `code/eo/.github/workflows/`,
and live GitHub REST data (commits/PRs/branches sampled 2026-10).

---

## 1. Branching model

The trunk (`master`) is **read-only for humans**: nothing lands on it except through the merge
gate. This is the same rule as `05-cicd-release.md` §1 and `06-repository-docs.md` §6.

- **[MUST]** The trunk is `master` (not `main`) and is **never committed to directly**.
  `objectionary/eo` default branch is `master`; every workflow triggers on `master`
  (`README.md:311`; `.github/workflows/mvn.yml:8-10`).
- **[MUST]** Branch once per issue, cut from the latest trunk, and tie the branch to the ticket.
  No ticket → no branch → no PR (`06-repository-docs.md:216`).
- **[MUST]** Name the branch after the issue number. The primary pattern is the **bare number**:
  `README.md:317` says *"name your branch after the issue you are working on, e.g. `42`"*.
  Real eo branches confirm it: `9046`, `9100`→`9046`, `9106`, `9108`, `9114`, `9122`
  (PR `head.ref` for #9100, #9109, #9121, #9123, #9125). Older eo branches: `3257`, `3538`, `5432`.
- **[SHOULD]** When a bare number reads poorly, append a short slug: `<issue>-<topic>`
  (`5678-socket-io`). Keep the issue number first.
- **[NICE]** Tooling branches keep their generator's prefix and are exempt from the issue-number
  rule: `renovate/...`, `claude/...`, `copilot/fix-4140`. Do not rename a Renovate branch.
- **[MUST]** Never commit to `master`, never push a red branch, and never force-push a shared
  branch (`06-repository-docs.md:219`, `07-process.md:170`).

```text
master (read-only) ──┐
                     ├── 42          <- branch per issue, bare number
                     ├── 42-add-cache
                     └── renovate/org.apache.maven-maven-core-3.x   (bot)
```

---

## 2. Commit frequency & size

- **[MUST]** Commit **atomically and often**: one logical change per commit. Do not mix a fix, a
  refactor, and a formatting pass in one commit or one PR (`07-process.md:167`,
  `06-repository-docs.md:238`).
- **[MUST]** Every commit references its issue, either in the subject (`type(#42): ...`,
  `#42: ...`) or with a `Refs #42` footer. An unreferenced commit is untraceable
  (`SKILL.md:88`, `06-repository-docs.md:217`).
- **[SHOULD]** Keep the whole PR small. eo asks for **40–100 hits-of-code** and the CI **fails
  above 200 changed lines** (`README.md:319-321`; `.github/workflows/pr-size.yml:25` sets
  `MAX_LINES_CHANGED: 200`).
- **[SHOULD]** A PR is judged by the **whole branch**, not one commit, so a few clean commits are
  normal and welcome. Real evidence: PR #9107 shipped **3 commits** (`commits: 3`,
  `additions: 94 / deletions: 5`); PR #9125 shipped **1** (`commits: 1`, `50 / 28`).
- **[MUST]** No WIP in the final PR. Finish the work, or convert the remainder into a `@todo`
  puzzle (`@todo #N:30min ...`) — a WIP commit left in the merged branch is a defect
  (`06-repository-docs.md:250-253`, `07-process.md:54`).
- **[SHOULD]** If you must park work-in-progress, open the PR as a **draft** or keep the
  `@Disabled` test plus a puzzle; do not leave `TODO`/`FIXME` or "wip" commits in the branch.
- **[NICE]** Squashing the branch to one commit is **not** required in eo (the merge is a true
  merge commit; see §4), but a tidy 1–3 commit branch is preferred over a noisy one.

### HoC vs. lines changed

```text
Hits-of-Code  = meaningful lines added (the review-relevant size)   README.md:319
Lines changed = additions + deletions, CI cap 200                   pr-size.yml:25
```

---

## 3. Commit message format

eo's canonical template is a typed subject carrying the issue number, a blank line, a body that
explains **why**, and (optionally) a `Refs #N` footer:

```text
type(#42): imperative short description

Explain in one short paragraph WHY this change is needed, not how. Name the
observed problem (a bad behavior, a broken invariant), and stop at ~72 columns
per line. Wrap at 72. Do not narrate the mechanics of the diff.

Refs #42
```

- **[MUST]** Put the issue number in the **first line** and keep the subject **imperative** and
  **≤72 columns**. `README.md:318`: *"prefix your commit messages with `fix(#42):` followed by a
  short description."*
- **[MUST]** Leave one **blank line** between subject and body; the body explains **why** and
  references the problem, not the code.
- **[SHOULD]** End with `Refs #42` when the commit is part of a multi-commit branch (used by ~99
  eo commits); use `Closes #42` in the PR description so GitHub auto-closes
  (`README.md:322-323`).
- **[SHOULD]** Use a recognized type prefix. Observed in eo: `bug(`, `fix(`, `test(`, `build:`,
  `ci:`, `chore(`, `feat(`, `docs(`. Renovate uses `fix(deps):` / `chore(deps):`.

### Real examples (copy the shape)

```text
#9105: read the shared entries document of `Uses` under its own lock
```
(commit `9a6aaee2dde06bc19a778defbd7d1d50fe09e986`, PR #9107 — classic eo `#N: subject` form)

```text
bug(#9046): let a goal that only reads a stage find it without numbering it
```
(PR #9100 title / merge message, branch `9046` — typed `type(#N):` form)

```text
build: #9108 bump lints to 0.5.0 and run every lint in eo-runtime

Lints 0.5.0 ships new rules, several of them experimental, and eo-runtime used
to skip experimental lints altogether. It now runs every lint and skips the ones
it cannot satisfy yet one by one, each with a puzzle that says why: ...
```
(commit `03e8bc73cd83b2cff9c1400b6eb2c1ab1e8521ec` — long WHY body, wrapped at 72)

- **[MUST]** Do **not** rewrite published history to fix a message: amend is allowed only on a
  **local, unpushed** commit; once pushed, add a new commit
  (`06-repository-docs.md:219`, `07-process.md:170`).
- **[NICE]** GPG-sign your commits (eo's merge commits are verified; sign if you have a key).
  Signing is a trust signal, not a merge requirement (`git log --show-signature`).

---

## 4. Merge policy

- **[MUST]** Merges are **bot-validated and PR-only**. A human does not press "merge" and does
  not push to `master`; the merge runs through the **Rultor** chat-ops bot, and only architects
  listed in `.rultor.yml` may command it (`.rultor.yml:5-6` `architect: [yegor256]`; live PRs show
  `merged_by: yegor256`, e.g. PR #9107 / #9125).
- **[MUST]** The merge is a **true merge commit** (`Merge pull request #N from <owner>/<branch>`),
  **not** a squash and **not** a rebase. Every sampled eo merge has two parents, e.g.
  `25ee984969cc9b4284f2bd8ed6b289552c82bc73` = `Merge pull request #9100 from objectionary/9046`.
  This is `--no-ff` semantics: the branch topology and its individual commits are preserved.
- **[MUST]** The **pre-merge gate is `merge.script` in `.rultor.yml`**, run inside the pinned
  Docker image. It must be green before the bot merges; a green PR workflow is *not* sufficient
  (`.rultor.yml:17-19`, `05-cicd-release.md:22-24`).

```yaml
# .rultor.yml:17-19 — the merge gate (eo)
merge:
  script: |
    mvn clean install -ntp -DskipTests -Pqulice --errors -Dstyle.color=never
```

- **[SHOULD]** Multiple commits are merged as-is (no squash), which is why §2 asks for clean,
  atomic commits rather than one big blob. Do not rely on the merge to tidy your history.
- **[SHOULD]** After merge the source branch may be left in place. eo does **not** aggressively
  delete branches — the remote still lists `3257`, `3538`, `5432`, `5678-socket-io`, `9048`, etc.
  Delete only if its owner wants to (GitHub `delete_branch_on_merge` is not relied upon).
- **[MUST]** Never merge into a broken `master`. A build fix is its own PR
  (`07-process.md:91`, `SKILL.md:111`).

### Who does what

```text
author   -> push branch, open PR, answer review comments (no force-push)
reviewer -> prove the code is bad; request changes or approve
architect-> issue the merge command to Rultor (@rultor merge) on a green gate
Rultor   -> re-runs merge.script, creates the signed merge commit, reports in the PR
reporter -> closes the ticket after the merge lands
```

---

## 5. Interaction with Rultor (the merge/release contract)

- **[MUST]** The merge command is issued as a PR comment and executed by the bot — the standard
  is `@rultor merge`. Rultor checks the requester against `architect:`/`readers:`, runs
  `merge.script`, and merges (`05-cicd-release.md` §1, §5).
- **[MUST]** The **release is tag-driven and scripted**, tied to the same contract. Rultor's
  `release.script` validates the tag (strict semver), runs `mvn versions:set -DnewVersion=${tag}`,
  commits, and does the signed `deploy -Psonatype` (`.rultor.yml:20-30`;
  `05-cicd-release.md:193-195`).

```yaml
# .rultor.yml:20-28 (eo) — release is tag-driven
release:
  pre: false
  script: |-
    [[ "${tag}" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]] || exit -1
    mvn flatten:flatten
    mvn -ntp versions:set "-DnewVersion=${tag}" -Dstyle.color=never
    git commit -am "${tag}"
    mvn clean deploy -ntp ... -Pobjectionary -Psonatype --errors --settings ../settings.xml
```

- **[SHOULD]** Never issue a release tag from a branch other than the merged trunk; release the
  trunk, not a feature branch (`05-cicd-release.md:310`).
- **[NICE]** Secrets for the release come from a separate repo and are listed under
  `release.sensitive:`, so a leak aborts the release (`.rultor.yml:11-14`;
  `05-cicd-release.md:241-242`).

---

## 6. Pre-PR checklist

```text
[ ] Branch cut from master, named `<issue>` (or `<issue>-slug`); never commit to master
[ ] One concern only; fix != refactor != formatting
[ ] Commits atomic and referenced: `type(#N): ...` or `#N: ...`, `Refs #N` where needed
[ ] Subject imperative, <=72 cols; body explains WHY after a blank line
[ ] PR 40-100 HoC and under the 200-line CI cap
[ ] No WIP, no bare TODO/FIXME (use `@todo #N:30min ...`)
[ ] PR body says `Closes #N`; ping the architect if the repo asks for it
[ ] Full gate green: mvn --errors --batch-mode clean install -Pqulice
[ ] No force-push, no history rewrite
[ ] Merge by Rultor (`@rultor merge`), not by hand; tag the trunk for release
```
