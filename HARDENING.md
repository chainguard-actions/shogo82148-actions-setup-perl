<!-- markdownlint-disable -->

# Hardening Report: shogo82148--actions-setup-perl/v1.41.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shogo82148--actions-setup-perl/v1.41.0** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation of ${{ matrix.installer }} inside run: shell commands. The matrix.installer value is workflow-controllable and is interpolated directly into the shell command string before the shell sees it, enabling command injection. Offending lines: `- run: ${{ matrix.installer }} --help` and `- run: ${{ matrix.installer }} --version`.

Locations:

- `.github/workflows/test-cpan-installer.yml:113`
- `.github/workflows/test-cpan-installer.yml:114`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation of ${{ steps.setup-perl.outputs.perl-version }} and ${{ steps.setup-perl.outputs.perl-hash }} inside run: shell commands. steps.*.outputs.* is in scope as a workflow-controllable context and is interpolated directly into the shell command string. Offending lines appear in the 'show outputs' steps across multiple jobs: `echo "perl-version: ${{ steps.setup-perl.outputs.perl-version }}"` and `echo "perl-hash: ${{ steps.setup-perl.outputs.perl-hash }}"` (bash jobs), and `Write-Output "perl-version: ${{ steps.setup-perl.outputs.perl-version }}"` / `Write-Output "perl-hash: ${{ steps.setup-perl.outputs.perl-hash }}"` (Windows jobs).

Locations:

- `.github/workflows/test.yml:46`
- `.github/workflows/test.yml:47`
- `.github/workflows/test.yml:116`
- `.github/workflows/test.yml:117`
- `.github/workflows/test.yml:186`
- `.github/workflows/test.yml:187`
- `.github/workflows/test.yml:256`
- `.github/workflows/test.yml:257`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and none of its jobs define job-level `permissions:`. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g., write access to contents). Specific minimal permissions should be declared.

Locations:

- `.github/workflows/test.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and the `check-dist` job has no job-level `permissions:`. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. Specific minimal permissions should be declared.

Locations:

- `.github/workflows/check-dist.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and the `installer` job has no job-level `permissions:`. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. Specific minimal permissions should be declared.

Locations:

- `.github/workflows/test-cpan-installer.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed all 5 findings across 3 workflow files:

1. test-cpan-installer.yml: Added `permissions: {}` top-level block. Fixed script injection in the two installer run steps by moving `${{ matrix.installer }}` into an `env:` block as `INSTALLER` and referencing it as `$INSTALLER` in the shell.

2. test.yml: Added `permissions: {}` top-level block. Fixed script injection in all four 'show outputs' steps (test-linux, test-darwin, test-windows, test-strawberry) by moving `${{ steps.setup-perl.outputs.perl-version }}` and `${{ steps.setup-perl.outputs.perl-hash }}` into `env:` blocks as `PERL_VERSION` and `PERL_HASH`. Bash jobs reference them as `$PERL_VERSION`/`$PERL_HASH`; PowerShell jobs reference them as `$env:PERL_VERSION`/`$env:PERL_HASH`.

3. check-dist.yml: Added `permissions: {}` top-level block.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in .github/workflows/test-cpan-installer.yml at lines 110 and 114. The `run:` steps used YAML double-quoted strings `"$INSTALLER --help"` and `"$INSTALLER --version"`, where the YAML double quotes are just YAML string delimiters — the shell sees the bare unquoted expansion `$INSTALLER`. Changed to YAML single-quoted strings `'"$INSTALLER" --help'` and `'"$INSTALLER" --version'` so the shell receives `"$INSTALLER" --help` and `"$INSTALLER" --version`, properly double-quoting the variable in the shell to prevent metacharacter injection from attacker-controlled matrix values.

