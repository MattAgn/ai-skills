---
name: upgrade-deps
description: MANDATORY skill whenever the user mentions bumping, updating, or upgrading dependencies — triggers include "bump dep", "outdated", "update package", "upgrade dep", "package upgrade", "bump X to Y", "what's outdated". Enforces exact-version pinning, supply-chain sanity checks on lockfile diffs, and coordinated bumps for package families (a scope that appears more than once). Never recommends `@latest`.
argument-hint: [package-name?]
disable-model-invocation: true
---

# Upgrading Dependencies

Safely bump outdated deps. Core principle — **supply-chain hardening**: pin exact versions, vet each new version before installing, audit the lockfile diff. Use the project's package manager; commands below are illustrative — adapt them.

> Framework-tied packages (managed by a meta-framework, SDK, or monorepo toolchain) move only when the framework moves — bump them through the framework's own upgrade path, not here.

## Phase 1 — Pre-flight

1. Refuse to start on a dirty tree: `git status` must be clean (unrelated diffs hide the audit trail).
2. Sync the base branch (`git pull origin main`).
3. Branch: `git checkout -b chore/bump-<scope>`.
4. List what's outdated (`npm outdated`); capture output, noting current vs. latest and whether the gap is patch / minor / major.

## Phase 2 — Triage

> Skip if the user named a package as argument — go to Phase 3.

Sort the outdated list into buckets before touching anything, then present them and ask which to bump this session (don't do all at once):

- **Framework-tied** — bump through the framework's upgrade path, not here.
- **Family** (Phase 4): any **scope appearing more than once** (e.g. `@scope/a`, `@scope/b`). Bump siblings together.
- **Standalone**: everything else.

## Phase 3 — Read release notes and apply cooldown

For each candidate:

1. Read CHANGELOG / release notes for every version between current and target (fetch version-matched docs if needed). Flag breaking changes, deprecations, and required migration steps.
2. **Cooldown + advisory check** — both required before install:
   - **Cooldown**: refuse any target published <5 days ago. Fresh-publish is the prime attack window; 5d lets the community spot compromises.
   - **Advisory check**: cross-check the target against known security advisories. Registries only allow unpublish briefly after release — past that, a compromised version stays available indefinitely, so cooldown alone won't filter it.

   Check per-version publish dates, whether the target is deprecated, and run a vulnerability audit. If either check fails, don't bump — surface it. Don't mechanically pick "the version before latest"; if that's what's installed, there's no bump to do — wait.

3. Majors: surface migration steps and have the user confirm the target version explicitly.

## Phase 4 — Coordinated package families

Bump every sibling in the same scope together in one command — mismatched family versions cause runtime errors invisible at install time. Align the **major** across siblings (the compat boundary); minor/patch may differ per sibling, so resolve each individually. Install with **exact pinned versions, never `@latest`**:

```bash
# all siblings in one install, each pinned to its own resolved version
<pm> add --exact <scope>@<vA> @<scope>/<sib1>@<vB> @<scope>/<sib2>@<vC>
```

If the package manager defaults to caret/tilde ranges, force exact pinning (config flag or `--exact`), then verify no range leaked into the manifest:

```bash
git diff package.json | grep -E '"\^|"~' && echo "FOUND CARET/TILDE — strip it" || echo "ok"
```

Never use anything resolving `@latest` — it's the supply-chain attack vector exact pinning exists to close.

## Phase 5 — Supply-chain sanity check

Eyeball the lockfile diff for anomalies disproportionate to the bump:

```bash
git diff <lockfile> | less
git diff --stat <lockfile> package.json
```

Red flags — surface to the user, don't proceed silently:

- Patch/minor bump pulling in many new transitive packages — especially unfamiliar, typo-squat, or single-letter / homoglyph names.
- Integrity hash changing on a package whose version didn't move.
- New `postinstall` / `preinstall` / `prepare` scripts on any added dep (`git diff <lockfile> | grep -iE 'postinstall|preinstall|prepare'`). Don't auto-trust — flag the package and what its script does.
- Resolved URLs pointing somewhere other than the official registry (random tarballs, custom registries).
- Package rename or maintainer/ownership transfer noted in release notes.
- Lockfile diff size disproportionate to the `package.json` diff.

If anything looks off, stop, paste the suspicious hunks to the user, and wait for explicit go-ahead. Don't auto-resolve.

## Phase 6 — Verification

Run the project's checks (type-check, lint, format, tests), stopping at the first failure. If the bump needs code changes (API migration, type fix), make them and re-run. If it touched a native/compiled module, source-level checks aren't enough — flag that a full rebuild is needed before merging (leave the rebuild to them).

## Phase 7 — Commit

One commit per family or standalone bump — never lump unrelated bumps. Stage only the manifest, lockfile, and any migration edits. Use the `commit` skill.

```
chore(deps): bump @scope/* to 1.127.0
chore(deps): bump dayjs to 1.11.14
```

## Phase 8 — PR

Use the `open-pr` skill. Per family/package, list: old → new version, release-notes link, breaking changes touched, and a one-line confirmation the supply-chain check passed.
