# 05 — CI/CD & Release

Reference for an AI agent creating or auditing the CI/CD and release flow of an Elegant
Objects (EO) Java project. Tags: **[MUST]** required · **[SHOULD]** strong default ·
**[NICE]** improves maturity.

Distilled from `knowledge/distilled-rules.md` §11 and the real pipelines in
`knowledge/elegant-objects-java.md` §D and `knowledge/elegant-objects-java.md` §D.

---

## 1. Split pipelines: GHA gates, Rultor merges/releases

**[MUST]** Run two pipelines with non-overlapping authority:

- **GitHub Actions = PR gate only.** Lint, build, tests, coverage, static analysis.
  It never tags, never merges, never deploys.
- **Rultor (chat-ops bot) = the sole merger and releaser.** `master` is write-only through
  Rultor; it re-runs the gate on top of master, GPG-signs the merge, and performs tagged
  releases.

**[MUST]** `master` is read-only for humans. Merges are PR-only and bot-validated.
A green branch build is *not* protection — only a merge gate on the write path is
(`Apa.33`). Never let a GHA workflow push to `master`, tag, or deploy.

---

## 2. One small workflow per concern

**[MUST]** Split CI into many single-purpose workflows, each with its own trigger, `paths:`
filter and `timeout-minutes` — build, coverage, static analysis, duplication, license,
typos, YAML/Markdown/XML lint, dependency audit, docs-version. Never one mega-report.

**[MUST]** Every workflow declares:

```yaml
permissions:
  contents: read                # least privilege by default
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true      # kill superseded runs
timeout-minutes: 15             # bounded, never the default 360
```

```yaml
# Maven cache key keyed on the POM hash; restore-keys is the fallback
- uses: actions/cache@v4
  with:
    path: ~/.m2/repository
    key: ${{ runner.os }}-jdk-${{ matrix.java }}-maven-${{ hashFiles('**/pom.xml') }}
    restore-keys: ${{ runner.os }}-jdk-${{ matrix.java }}-maven-
```

- **[SHOULD]** Use **restore-only** caches on PR workflows (`actions/cache/restore`) so PRs
  never evict master's cache.
- **[SHOULD]** Before install in multi-module builds, delete sibling snapshots so they
  re-resolve: `rm -rf ~/.m2/repository/org/<group>`.
- **[SHOULD]** `fetch-depth: 2` when only the merge-base is needed; documented with a
  comment (avoids cloning a multi-GB `gh-pages`).
- **[NICE]** Document *why* in YAML comments (cache eviction, fetch depth, security triggers).

---

## 3. OS × JDK matrix

**[SHOULD]** Test across OS × one modern JDK to catch portability bugs; do not let one OS
dictate the code.

- **[MUST]** If an OS genuinely cannot build (e.g. Windows for a Docker-dependent build),
  fail fast in the POM with a clear message and a Qulice-only escape — do not silently skip.
- **[SHOULD]** Use per-OS conditional steps for tooling; declare capability skips rather than
  disabling a whole matrix leg.
- **[NICE]** A second CI (AppVeyor-style) as an independent cross-check.

---

## 4. `.github/workflows/build.yml` example

```yaml
# SPDX-FileCopyrightText: Copyright (c) 2009-2026 Yegor Bugayenko
# SPDX-License-Identifier: MIT
name: mvn
'on':
  push:
    branches: [master]
  pull_request:
    branches: [master]

# Least privilege: the workflow only reads the repo.
permissions:
  contents: read

# One running instance per ref; cancel superseded runs.
concurrency:
  group: mvn-${{ github.ref }}
  cancel-in-progress: true

jobs:
  build:
    runs-on: ${{ matrix.os }}
    timeout-minutes: 20          # bounded job
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-24.04, windows-2022, macos-15]   # portability gate
        java: [21]                                   # one modern JDK == release
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 2         # merge-base only; avoid huge history

      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: ${{ matrix.java }}

      # Cache keyed on the POM hash; restore-keys falls back gracefully.
      - uses: actions/cache@v4
        with:
          path: ~/.m2/repository
          key: ${{ runner.os }}-jdk-${{ matrix.java }}-maven-${{ hashFiles('**/pom.xml') }}
          restore-keys: ${{ runner.os }}-jdk-${{ matrix.java }}-maven-

      # Force sibling snapshots to re-resolve (multi-module gotcha).
      - run: rm -rf ~/.m2/repository/org/example

      - name: Build
        run: mvn --errors --batch-mode clean install -Pqulice   # the one command
```

Pair it with `codecov.yml` (master only, `fail_ci_if_error: true`) and the hygiene linters
(`actionlint`, `yamllint`, `markdown-lint`, `shellcheck`, `typos`, `reuse`, `pdd`, `xcop`),
each as its own tiny workflow.

---

## 5. `.rultor.yml` example

Rultor is the merge/release contract. Same skeleton in every repo:
`docker.image` (isolation), `readers` (authorization), `assets` (secrets from a separate
repo), `install` (pre-fetch), `merge.script` (the gate), `release.script` (the deploy).

```yaml
# .rultor.yml
docker:
  image: yegor256/java                 # pinned; build runs in this container
readers:
  - "urn:github:526301"                # who may issue merge/release commands
assets:
  # Secrets are checked out from a SEPARATE repo, never stored here.
  settings.xml: yegor256/home#assets/example/settings.xml
  id_rsa: yegor256/home#assets/heroku-key
  id_rsa.pub: yegor256/home#assets/heroku-key.pub
install: |-
  pdd --file=/dev/null                 # puzzle validation before anything runs
merge:
  script: |-
    # Preflight: run the full gate on master+PR, inside the container.
    mvn help:system clean install -Pqulice --errors --settings ../settings.xml
release:
  pre: false                           # no pre-release workflow
  sensitive:
    - settings.xml                     # refuse release if this leaks into the repo
  script: |-
    mv ../netrc ~/.netrc
    # Tag must be strict semver or the release aborts.
    [[ "${tag}" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]] || exit -1
    echo "Author of the request: ${author}"
    # Scripted, reproducible version bump from the tag.
    mvn -ntp versions:set "-DnewVersion=${tag}" --quiet
    git commit -am "${tag}"
    cp ../settings.xml settings.xml    # secret copied in, never committed for long
    mvn clean package -Pqulice --errors --batch-mode --quiet
    # Stamp the build identity into the manifest.
    build=$(git rev-parse --short HEAD)
    sed -i "s/BUILD/${build}/g" src/main/resources/META-INF/MANIFEST.MF
    git add src/main/resources/META-INF/MANIFEST.MF
    git commit -m 'build number set'
    git add settings.xml
    git commit -m 'settings.xml'
    # Deploy = force-push to the PaaS remote, then health-check.
    git remote add dokku dokku@example.com:example
    rm -rf ~/.ssh && mkdir ~/.ssh
    mv ../id_rsa ../id_rsa.pub ~/.ssh
    chmod -R 600 ~/.ssh/*
    echo -e "Host *\n  StrictHostKeyChecking no\n  UserKnownHostsFile=/dev/null" > ~/.ssh/config
    git push -f dokku "$(git symbolic-ref --short HEAD):master"
    git reset HEAD~1                   # undo the secret commit
    rm -rf settings.xml
    curl --insecure -f --connect-timeout 30 --retry 8 --retry-delay 60 https://example.com
```

- **[MUST]** Release is tag-driven and scripted; validate the tag, `versions:set`, commit,
  signed `deploy -Psonatype`. Never ad-hoc.
- **[MUST]** Mark secret files `sensitive:` so a leaked file aborts the release.

---

## 6. `release.sh` example (manual fallback / DR)

**[SHOULD]** Keep a manual script that mirrors the bot so a release is recoverable. Wrap the
secret handling in `trap ... EXIT` so cleanup happens on success *and* failure.

```bash
#!/usr/bin/env bash
# SPDX-FileCopyrightText: Copyright (c) 2009-2026 Yegor Bugayenko
# SPDX-License-Identifier: MIT
set -ex -o pipefail
cd "$(dirname "$0")"

# Validate the tag before touching anything.
[[ "${1}" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]] || { echo "usage: release.sh X.Y.Z" >&2; exit 1; }
tag="${1}"

# Copy the secret in, commit, and guarantee removal on ANY exit.
cp /code/home/assets/example/settings.xml .
git add settings.xml
git commit -m "${tag} settings.xml"
trap 'git reset HEAD~1 && rm -f settings.xml' EXIT

mvn -ntp versions:set -DnewVersion="${tag}" --quiet
git commit -am "${tag}"
mvn --errors --batch-mode clean deploy -Pqulice -Psonatype --settings settings.xml
git tag "${tag}"
git push origin "${tag}"
```

- **[MUST]** Secrets never reach history: copy → commit → push → `git reset HEAD~1` + `rm`.
- **[MUST]** `trap '...' EXIT` on any script that stages a secret.
- **[SHOULD]** Deploy to PaaS via `git push -f` to the remote, then curl the live URL with
  `--connect-timeout`/`--retry`/`--retry-delay`. Release is **not done** until the endpoint
  answers.
- **[SHOULD]** Keep `Procfile`/Dokku config/runtime declaration in-repo alongside the script.

---

## 7. Secrets hygiene

- **[MUST]** Assets (settings.xml, keys, netrc) live in a **separate repo**
  (`yegor256/home#assets/...`); the app repo never contains them.
- **[MUST]** `.rultor.yml` lists secret filenames under `release.sensitive:`; a leaked file
  aborts the release.
- **[MUST]** GHA secrets never printed; use OIDC where possible instead of long-lived tokens.
- **[SHOULD]** Scan for accidental secrets (`trufflehog-oss` / secret scanning workflow).
- **[NICE]** Trigger jobs that consume untrusted text only on `schedule`, never
  `issues:opened`; checkout PR refs read-only with `persist-credentials: false`.

---

## 8. Deploy to PaaS + health check

- **[SHOULD]** Declare the runtime in-repo (`Procfile`, `nginx.conf.sigil`, `app.json`) so
  the platform config is versioned.
- **[SHOULD]** Health-check the live URL with retries; best-effort ancillary deploys (the
  docs site) must not block the product release (`... || echo 'Failed to deploy site'`).
- **[NICE]** Provision integration infrastructure via profiles (DynamoDBLocal, H2, reserved
  ports), not mocks; name ITs `*ITCase` for failsafe.

---

## 9. Renovate

**[SHOULD]** Use Renovate (richer rules than Dependabot) to keep dependencies moving, with
explicit guard-rails for pins that must not float.

```json
{
  "extends": ["config:base"],                 // baseline update policy
  "ignoreDeps": ["xml-apis:xml-apis"],        // known-broken transitive dep
  "packageRules": [
    {
      "matchPackageNames": ["maven-compiler-plugin"],
      "allowedVersions": "3.8.1"              // frozen: newer breaks the build
    }
  ]
}
```

- **[MUST]** Every ignore/pin carries a reason (comment or PR note).
- **[SHOULD]** Renovate opens the PR; Rultor still merges it — never auto-merge outside the
  gate.

---

## 10. Doc / version sync

**[SHOULD]** On each new tag, run an `up.yml` workflow that bumps the version pinned in
`README.md` / usage docs and opens a signed PR.

```yaml
name: up
'on':
  push:
    tags: ['[0-9]+.[0-9]+.[0-9]+']
permissions:
  contents: write        # the only workflow that needs write
jobs:
  readme:
    runs-on: ubuntu-24.04
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - run: sed -i "s/<version>.*<\/version>/<version>${GITHUB_REF_NAME}<\/version>/" README.md
      - uses: peter-evans/create-pull-request@v6
        with:
          sign-commits: true
          commit-message: "sync README to ${GITHUB_REF_NAME}"
```

**[MUST]** The version in the POM, README/docs, badges and license are one truth.

---

## 11. Checklist

- [ ] GHA is PR-gate only; Rultor is the sole merger/releaser; `master` read-only.
- [ ] Every workflow: `permissions: contents: read`, `concurrency` cancel-in-progress,
      `timeout-minutes`, Maven cache keyed on `hashFiles('**/pom.xml')`.
- [ ] PR workflows use restore-only caches; sibling snapshots wiped before install.
- [ ] OS×JDK matrix where portability matters; capability skips, not silent gaps.
- [ ] Release: semver regex → `versions:set` → commit → signed `deploy -Psonatype`.
- [ ] Secrets from a separate repo; `sensitive:` markers; copy→commit→push→reset;
      `trap EXIT` in scripts.
- [ ] PaaS deploy force-push + curl health-check with retries.
- [ ] Renovate with reasoned guard-rails; docs/version synced on each tag.
