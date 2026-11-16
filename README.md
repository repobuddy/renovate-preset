# renovate-preset

Preset config for my repositories.

<https://github.com/renovatebot/renovate>

## Usage

```json
{
  "extends": ["github>repobuddy/renovate-preset"]
}
```

`github>repobuddy/renovate-preset` resolves to `default.json` on the default
branch, so a change here reaches every consuming repo on its next run. There is
no tag to move.

The body of `default.json` is the canonical baseline kept in
[unional/skills](https://github.com/unional/skills)
(`skills/apply-repo-baseline/assets/renovate-preset.json`). Change it there
first, then copy it here.

## What the preset provides

- **Renovate's recommended base** (`config:recommended`): the dependency
  dashboard, monorepo grouping, and Renovate's standard replacements and
  workarounds.
- **Semver ranges are preserved** (`:preserveSemverRanges`). A `^1.2.3` range
  stays a range; Renovate does not pin it.
- **Semantic commit messages** (`:semanticCommits`), such as
  `fix(deps): update dependency foo to v1.2.4`.
- **A 3 day release soak** (`minimumReleaseAge: "3 days"`). Renovate proposes a
  version only once it has been published for 3 days. Security fixes soak for
  1 day instead (see below).
- **Automerge through GitHub** (`platformAutomerge: true`). Renovate turns on
  GitHub's auto-merge for the PR, and GitHub merges it once the required checks
  pass. Branch protection decides what "pass" means.
- **What merges by itself:**
  - minor, patch, pin and digest updates;
  - any update to a `devDependencies` entry, including a major one;
  - vulnerability fixes, after a 1 day soak (`vulnerabilityAlerts`);
  - weekly lock file maintenance (`lockFileMaintenance`).
- **`@types/node` never merges by itself.** It tracks the Node range a package
  supports, so every update, of any size, waits for a person. Its majors also
  wait for dependency dashboard approval, like Node majors.
- **Majors of runtime dependencies stay manual.** They do not automerge, and
  their branches carry a `major-` prefix so they are easy to find.
- **Changeset labels for the shared `dep-changeset.yml` workflow**
  ([repobuddy/.github](https://github.com/repobuddy/.github)). Updates to
  `dependencies`, `optionalDependencies` and pnpm catalog entries
  (`pnpm.catalog.*`) get `dep-changeset:auto` for minor, patch, pin and digest
  updates, and `dep-changeset:human` for majors. Every `peerDependencies` update
  gets `dep-changeset:human`. The workflow fails closed on a runtime change with
  neither label.
- **`rebaseWhen: "behind-base-branch"`**: branches are refreshed whenever the
  base branch moves, not only when they conflict.

Two behaviours come from Renovate's defaults rather than from keys in this file:

- Renovate does not move a dependency from a stable version onto a prerelease
  ([`ignoreUnstable`](https://docs.renovatebot.com/configuration-options/#ignoreunstable),
  default `true`). A dependency already on a prerelease line keeps getting that
  line's later prereleases.
- A version still inside the soak is skipped, not proposed
  ([`internalChecksFilter`](https://docs.renovatebot.com/configuration-options/#internalchecksfilter),
  default `"strict"`).

### Overriding

A consuming repo's own settings and `packageRules` are applied after the
preset's. A repo that wants a different soak, no automerge, or a bound on one
dependency sets it locally:

```json
{
  "extends": ["github>repobuddy/renovate-preset"],
  "packageRules": [
    {
      "matchPackageNames": ["typescript"],
      "allowedVersions": "<6"
    }
  ]
}
```

### Runtime majors in private apps

The preset holds every `dependencies` major for dependency dashboard approval.
That protects published packages, but a private app (such as `apps/website`)
publishes nothing, so the wait is noise. Only the repo knows which packages are
private, so the preset cannot lift it. A repo lifts it with a local rule that
matches the app's package file:

```json
{
  "extends": ["github>repobuddy/renovate-preset"],
  "packageRules": [
    {
      "description": "Private apps publish nothing: their runtime majors need no approval",
      "matchFileNames": ["apps/**"],
      "matchUpdateTypes": ["major"],
      "matchDepTypes": ["dependencies"],
      "dependencyDashboardApproval": false
    }
  ]
}
```
