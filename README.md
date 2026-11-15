# github-workflows

Reusable GitHub Actions workflows for building my apps and libraries and publishing the libraries to GitHub Packages. Each repository's own workflow file only calls one of these, so a fix here reaches every repository.

| Workflow | What it does |
|---|---|
| `gradle-library.yml` | Builds and tests on every push and pull request. Publishes the `-SNAPSHOT` version on every push to `main`, and from the "Run workflow" button (one module or all). |
| `npm-library.yml` | Builds, lints and tests on every push and pull request. On `main`, Changesets keeps a "Version Packages" pull request open while changesets are pending; merging it publishes, tags and creates GitHub Releases. The "Run workflow" button publishes a canary (`0.0.0-canary-<timestamp>`, dist-tag `canary`) from any branch with a pending changeset. |
| `pr-title.yml` | Checks that pull request titles follow Conventional Commits (`feat:`, `fix:`, `chore:` ...). |
| `gradle-app.yml` | Apps: `./gradlew build` with Java 21. |
| `maven-app.yml` | Apps: `./mvnw -B verify` with Java 21. |
| `node-app.yml` | Apps: `npm ci`, then `lint` and `test` when the scripts exist, then `build`. |

The app workflows only build and test, and take a `working-directory` input for repositories that keep the app in a subfolder. Publish jobs are skipped while the calling repository is a template. Actions is disabled on the `template-*` repositories themselves; repositories created from them start with Actions on.

## Calling them

```yaml
# .github/workflows/ci.yml
jobs:
  ci:
    uses: Pruthvik007/github-workflows/.github/workflows/gradle-library.yml@main
    with:
      module: ${{ inputs.module }}
    secrets: inherit
```

The caller's top-level `permissions` must allow what the jobs here ask for: `contents: read` and `packages: write` for Gradle, and `contents: write`, `pull-requests: write`, `packages: write` for npm. `PACKAGES_TOKEN` is used when the caller has it (needed to read packages from other repositories); otherwise the built-in token is used.

npm libraries also need two `package.json` scripts, `release` (`npm run build && changeset publish`) and `version-packages` (`changeset version && npm install --package-lock-only`), and "Allow GitHub Actions to create and approve pull requests" turned on under Settings → Actions → General, so Changesets can open its pull request.

Action versions are pinned to commit SHAs; Dependabot opens a pull request when a new version is out.
