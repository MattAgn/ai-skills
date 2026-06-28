---
name: upgrade-deps
description: MANDATORY skill whenever the user mentions bumping, updating, or upgrading dependencies — triggers include "bump dep", "outdated", "update package", "upgrade dep", "package upgrade", "bump X to Y", "what's outdated". Enforces exact-version pinning, supply-chain sanity checks on lockfile diffs, and coordinated bumps for package families (a scope that appears more than once). Never recommends `@latest`.
argument-hint: [package-name?]
---

# Upgrading Dependencies

Guide for safely bumping outdated dependencies. The core principle is **supply-chain hardening**: pin exact versions, vet every new version before installing, and audit the lockfile diff. Use the project's package manager — the commands below are illustrative; adapt them.

> If the project uses a framework that manages a set of dependency versions for you (a meta-framework, an SDK, a monorepo toolchain), those framework-tied packages move only when the framework moves — bump them through the framework's own upgrade path, not here.

## Phase 1 — Pre-flight

1. Refuse to start on a dirty tree: `git status` must be clean. Mixing unrelated diffs into a bump commit hides the supply-chain audit trail.
2. Sync the base branch (e.g. `git pull origin main`).
3. Branch: `git checkout -b chore/bump-<scope>`.
4. List what's actually outdated (e.g. `npm outdated`). Capture the output. Note current vs. latest, and whether the gap is patch / minor / major.

## Phase 2 — Triage

> Skip this phase if the user specified a package name as argument — go straight to Phase 3.

Sort the outdated list into buckets before touching anything:

- **Framework-tied** — managed by a framework/SDK; bump through its upgrade path, not here.
- **Family bumps** (Phase 4): any **scope that appears more than once** (e.g. `@scope/a`, `@scope/b`). Bump siblings together.
- **Standalone bumps**: everything else.

Present the buckets to the user and ask which to bump in this session. Don't try to do all of them at once.

## Phase 3 — Read release notes and apply cooldown

For each candidate:

1. Read CHANGELOG / release notes for every version between current and target. Fetch version-matched docs if needed.
2. Flag breaking changes, deprecations, and required migration steps.
3. **Cooldown + advisory check** — both required, before install:
   - **Cooldown**: refuse any target version published <5 days ago. Fresh-publish is the prime attack window; 5d gives the community time to spot undetected compromises.
   - **Advisory check**: cross-check the target against known security advisories. Registries typically only allow unpublish within a short window after release — past that, a compromised version stays available indefinitely, so the cooldown alone won't filter it out.

   Check publish dates per version, whether the target is deprecated, and a vulnerability audit. If either check fails, don't bump — surface it to the user. Don't mechanically pick "the version before the latest" — if it's the version already installed, there's just no bump to do; wait.
4. Majors: surface the migration steps and ask the user to confirm the target version explicitly.

## Phase 4 — Coordinated package families

When bumping a scoped package, find every sibling in the same scope and bump them together in one command. Mismatched versions inside a family cause runtime errors that don't show up at install time.

Align the **major** version across siblings (that's the compat boundary). Minor and patch can — and often do — differ per sibling; resolve each one individually. Install with **exact, explicit pinned versions** (never `@latest`):

```bash
# all siblings in one install, each pinned to its own resolved version
<pm> add --exact <scope>@<vA> @<scope>/<sib1>@<vB> @<scope>/<sib2>@<vC>
```

If the package manager defaults to a caret/tilde range, force exact pinning (a config flag or `--exact`), then verify the manifest didn't get a `^`/`~` range written into it:

```bash
git diff package.json | grep -E '"\^|"~' && echo "FOUND CARET/TILDE — strip it" || echo "ok"
```

Do NOT use anything that resolves `@latest` for one or many packages at once — `@latest` is the supply-chain attack vector that exact pinning exists to close.

## Phase 5 — Supply-chain sanity check

Before running anything else, eyeball the lockfile diff and look for anomalies that don't match the size of the bump:

```bash
git diff <lockfile> | less
git diff --stat <lockfile> package.json
```

Red flags — surface to the user, do not proceed silently:

- Patch / minor bump pulling in many new transitive packages — especially unfamiliar names, typo-squat candidates, or single-letter / homoglyph names.
- Integrity hash changing on a package whose version did not move.
- New `postinstall`, `preinstall`, or `prepare` lifecycle scripts on any added dep (`git diff <lockfile> | grep -iE 'postinstall|preinstall|prepare'`). Do not auto-trust them — flag the package name and what its install script does.
- Resolved URLs pointing somewhere other than the official registry (random tarballs, custom registries).
- Package rename or maintainer / ownership transfer mentioned in release notes.
- Lockfile diff size disproportionate to the `package.json` diff.

If anything looks off, stop, paste the suspicious diff hunks to the user, and wait for explicit go-ahead. Do not auto-resolve.

## Phase 6 — Verification

Run the project's checks (type-check, lint, format, tests) and stop at the first failure. If the bump required code changes (API migration, type fix), make those edits and re-run.

If the bump touched a native/compiled module, JS/source-level checks are not enough — flag to the user that a full rebuild is needed before merging (leave the rebuild to them).

## Phase 7 — Commit

One commit per family or per standalone bump — never lump unrelated bumps together. Stage only the manifest and lockfile (plus any migration code edits). Use the `commit` skill to finalize.

```
chore(deps): bump @scope/* to 1.127.0
chore(deps): bump dayjs to 1.11.14
```

## Phase 8 — PR

Use the `open-pr` skill. The PR description should list, per family / package: old → new version, link to release notes, breaking changes touched, and a one-line confirmation that the supply-chain check passed.
