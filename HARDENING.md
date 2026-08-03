<!-- markdownlint-disable -->

# Hardening Report: shogo82148--actions-setup-perl/v1.43.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shogo82148--actions-setup-perl/v1.43.0** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: blocks. In the 'test-linux' job's 'show outputs' step, `${{ steps.setup-perl.outputs.perl-version }}` and `${{ steps.setup-perl.outputs.perl-hash }}` are interpolated directly into echo commands inside a run: block. The same pattern repeats in the 'test-darwin' job. These step outputs flow through YAML template substitution before the shell sees them, enabling script injection if the output contains shell metacharacters.

Locations:

- `.github/workflows/test.yml:48`
- `.github/workflows/test.yml:49`
- `.github/workflows/test.yml:96`
- `.github/workflows/test.yml:97`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: blocks. In the 'test-windows' and 'test-strawberry' jobs' 'show outputs' steps, `${{ steps.setup-perl.outputs.perl-version }}` and `${{ steps.setup-perl.outputs.perl-hash }}` are interpolated directly into Write-Output commands inside run: blocks. Any ${{ ... }} inside a run: script is a script-injection risk regardless of context.

Locations:

- `.github/workflows/test.yml:155`
- `.github/workflows/test.yml:156`
- `.github/workflows/test.yml:215`
- `.github/workflows/test.yml:216`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: blocks. The lines `- run: ${{ matrix.installer }} --help` and `- run: ${{ matrix.installer }} --version` interpolate the matrix.installer expression directly as the shell command to execute. This allows the matrix value to be interpreted as a shell command, which is a script injection risk.

Locations:

- `.github/workflows/test-cpan-installer.yml:112`
- `.github/workflows/test-cpan-installer.yml:113`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs (test-linux, test-darwin, test-windows, test-strawberry, test-version-file, format). Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/test.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` key on the 'installer' job. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/test-cpan-installer.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` key on the 'check-dist' job. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/check-dist.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed all 6 findings across 3 workflow files:

1. test.yml - Added `permissions: {}` at top level.
2. test.yml (test-linux, test-darwin) - Moved `${{ steps.setup-perl.outputs.perl-version }}` and `${{ steps.setup-perl.outputs.perl-hash }}` out of echo commands into env: blocks (PERL_VERSION, PERL_HASH), referenced as $PERL_VERSION/$PERL_HASH in bash.
3. test.yml (test-windows, test-strawberry) - Same fix for PowerShell Write-Output commands; referenced as $env:PERL_VERSION/$env:PERL_HASH.
4. test-cpan-installer.yml - Added `permissions: {}` at top level. Fixed `run: ${{ matrix.installer }} --help/--version` by moving matrix.installer into an env: block as INSTALLER and running `"$INSTALLER --help"` / `"$INSTALLER --version"`.
5. check-dist.yml - Added `permissions: {}` at top level.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two steps in .github/workflows/test-cpan-installer.yml (lines 107 and 110) where `$INSTALLER` was unquoted in shell commands. Changed `run: "$INSTALLER --help"` and `run: "$INSTALLER --version"` (YAML double-quoted strings where the shell received unquoted `$INSTALLER`) to `run: '"$INSTALLER" --help'` and `run: '"$INSTALLER" --version'` (YAML single-quoted strings with shell double-quotes around the variable), preventing word splitting on the matrix.installer value.

