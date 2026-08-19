<!-- markdownlint-disable -->

# Hardening Report: shogo82148--actions-setup-perl/v1.41.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shogo82148--actions-setup-perl/v1.41.1** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: blocks. In test-cpan-installer.yml, the steps `run: ${{ matrix.installer }} --help` and `run: ${{ matrix.installer }} --version` interpolate a matrix-controlled value directly into the shell command string before the shell ever sees it, enabling command injection. In test.yml, multiple jobs contain `echo "perl-version: ${{ steps.setup-perl.outputs.perl-version }}"`, `echo "perl-hash: ${{ steps.setup-perl.outputs.perl-hash }}"`, and `Write-Output "perl-version: ${{ steps.setup-perl.outputs.perl-version }}"` / `Write-Output "perl-hash: ${{ steps.setup-perl.outputs.perl-hash }}"` — all of which interpolate step output expressions directly inside run: shell command strings. These should be passed via env: variables and then referenced as shell variables (e.g. $PERL_VERSION) instead.

Locations:

- `.github/workflows/test-cpan-installer.yml:107`
- `.github/workflows/test-cpan-installer.yml:108`
- `.github/workflows/test.yml:46`
- `.github/workflows/test.yml:47`

### missing-permissions (severity: medium)

The workflow files check-dist.yml, test.yml, and test-cpan-installer.yml have no top-level `permissions:` key and no job-level `permissions:` keys on any of their jobs. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (which may be broad). Each workflow should declare minimal required permissions (e.g. `permissions: contents: read`) at the top level or per job.

Locations:

- `.github/workflows/check-dist.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/test-cpan-installer.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed script injection in test-cpan-installer.yml by moving matrix.installer into env: block (INSTALLER) and referencing as $INSTALLER in run steps. Fixed script injection in test.yml for all four jobs (test-linux, test-darwin, test-windows, test-strawberry) by moving steps.setup-perl.outputs.perl-version and steps.setup-perl.outputs.perl-hash into env: blocks (PERL_VERSION, PERL_HASH) and referencing as shell variables ($PERL_VERSION/$PERL_HASH for bash jobs, $env:PERL_VERSION/$env:PERL_HASH for PowerShell jobs). Added top-level `permissions: contents: read` to all three workflow files: check-dist.yml, test.yml, and test-cpan-installer.yml.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted shell expansions of $INSTALLER (sourced from matrix.installer) in .github/workflows/test-cpan-installer.yml. Changed `run: $INSTALLER --help` to `run: "$INSTALLER" --help` and `run: $INSTALLER --version` to `run: "$INSTALLER" --version`. The INSTALLER env var is already correctly set via the step's env block; only the quoting in the run command was missing.

