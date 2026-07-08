# Developing nw-watchdog

`nw-watchdog` is a single POSIX-sh program (`nw-watchdog`) plus its documentation.
This note records the branch model, pull-request flow, and versioning scheme so the
process isn't only tribal knowledge.

> **This is a development/contributor document — not part of a release.** When a release
> is cut, exclude `documentation/DEVELOPMENT.md` from the released files.

## Branches

- **`development`** — integration branch. New work happens here, optionally on
  short-lived feature branches off it.
- **`main`** — the TESTING branch: stable enough to download and run if you want
  functionality that isn't in a release yet. Note: unlike many projects, `main` is
  *not* the latest release — releases are tags (below).
- **Releases** — git tags `vX.Y.Z`.

After a PR merges into `main`, `development` is fast-forwarded back to it, so the two
stay in step (they differ only by the `VERSION` line — see Versioning).

## Making changes

Changes reach `main` through pull requests:

1. Work on `development` (or a feature branch off it).
2. Open a PR into `main`.
3. The maintainer reviews and merges it, as a **merge commit** (not squash —
   squashing would make the long-lived `development` branch diverge from `main`).
4. `development` is fast-forwarded back to `main`.

Force-pushes to `main`/`development` are blocked; fix mistakes with a follow-up commit
rather than rewriting history.

## Versioning

The `VERSION` line at the top of the `nw-watchdog` script identifies the build.
`--version` and `--help` treat any version string containing a `-` as unreleased.

| Where            | Format                          | Example                        |
|------------------|---------------------------------|--------------------------------|
| Release (tag)    | `X.Y.Z`                         | `1.1.6`                        |
| `main` (testing) | `X.Y.Z-testing-YYYYMMDD-PR#`    | `1.1.6-testing-20260708-73`    |
| `development`    | `X.Y.Z-development-YYYYMMDD-PR#` | `1.1.6-development-20260708-73` |

- `X.Y.Z` is the release it is based on; `CHANNEL` is `testing` (stable enough to run, normally the
  next release) or `development` (unstable, in progress); `YYYYMMDD` is the change date; and `PR#` is
  the pull request that introduced it (for a `development` build, the `testing` build it is based on).
- The channel word denotes *maturity*, not the branch name, so it stays `testing` even if that branch
  is later renamed.
- The maintainer sets the `testing` version **inside the PR** — the PR number is known once the PR is
  opened, so the stamp is visible at review and correct at merge.
- After the fast-forward sync, `development`'s version is the `testing` version with the channel word
  swapped (`testing` → `development`).

## Releases

Releases are tags `vX.Y.Z` cut off `main`; there is no permanent release branch.

The step-by-step release procedure will be documented here when the next release is cut.
Among other things it must exclude development-only docs (this `DEVELOPMENT.md`) from the
released files.

## Code style & checks

- **POSIX sh** is a hard goal. Linux-only *dependencies* (`ip`, `ping` iputils flags,
  `/proc`) are intended and fine; non-POSIX *shell-language* constructs are not.
- Keep the existing terse style; comment the *why*, in English.
- Before committing, run:
  - `sh -n nw-watchdog` (and `dash -n nw-watchdog` if available) — syntax check.
  - `shellcheck nw-watchdog` — should be clean. A `.shellcheckrc` pins `shell=sh` and
    disables the reviewed-intentional info/style notes.
- `documentation/help.md` mirrors the built-in `--help`; keep the two in step.
- Record unreleased changes in `documentation/changelog.md` under the **TESTING** section.
