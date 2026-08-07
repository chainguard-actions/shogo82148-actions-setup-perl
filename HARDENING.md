<!-- markdownlint-disable -->

# Hardening Report: shogo82148--actions-setup-perl/v1.43.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shogo82148--actions-setup-perl/v1.43.1** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): GitHub Actions expressions are interpolated directly inside run: shell command strings. In the 'show outputs' step of the test-linux and test-darwin jobs, `echo` commands embed `${{ steps.setup-perl.outputs.perl-version }}` and `${{ steps.setup-perl.outputs.perl-hash }}` directly in the shell string. In the test-windows and test-strawberry jobs, `Write-Output` commands do the same. The `steps.*.outputs.*` context is workflow-controllable and flows through YAML template substitution before the shell sees it, making this a script injection risk. These values should be passed via env: variables and referenced as `"$ENV_VAR"` in the shell.

Locations:

- `.github/workflows/test.yml:45`
- `.github/workflows/test.yml:46`
- `.github/workflows/test.yml:115`
- `.github/workflows/test.yml:116`
- `.github/workflows/test.yml:186`
- `.github/workflows/test.yml:187`
- `.github/workflows/test.yml:256`
- `.github/workflows/test.yml:257`

### script-injection (severity: high)

Rule (a): Two run: steps in test-cpan-installer.yml use `${{ matrix.installer }}` as the entire shell command to execute (e.g. `run: ${{ matrix.installer }} --help` and `run: ${{ matrix.installer }} --version`). The `matrix.*` context is workflow-controllable and is interpolated directly into the shell command string before the shell parses it, allowing an attacker who controls the matrix value to inject arbitrary shell commands. These should use an env: variable and reference it as `"$INSTALLER"` in the run: block.

Locations:

- `.github/workflows/test-cpan-installer.yml:107`
- `.github/workflows/test-cpan-installer.yml:108`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs (check-dist). Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. A minimal permissions block (e.g. `permissions: contents: read`) should be added.

Locations:

- `.github/workflows/check-dist.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and none of its six jobs (test-linux, test-darwin, test-windows, test-strawberry, test-version-file, format) have a job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. A minimal permissions block should be added at the top level or on each job.

Locations:

- `.github/workflows/test.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and the single job `installer` has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. A minimal permissions block (e.g. `permissions: contents: read`) should be added.

Locations:

- `.github/workflows/test-cpan-installer.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed script injection in test.yml by moving steps.setup-perl.outputs.* expressions into env: blocks for all 4 'show outputs' steps (test-linux, test-darwin use bash env vars $PERL_VERSION/$PERL_HASH; test-windows, test-strawberry use PowerShell env vars $env:PERL_VERSION/$env:PERL_HASH). Fixed script injection in test-cpan-installer.yml by moving matrix.installer into an env: block as INSTALLER and referencing it as "$INSTALLER" in run steps. Added 'permissions: contents: read' at the top level of all three workflow files (check-dist.yml, test.yml, test-cpan-installer.yml).

