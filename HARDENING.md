<!-- markdownlint-disable -->

# Hardening Report: shogo82148--actions-setup-perl/v1.40.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shogo82148--actions-setup-perl/v1.40.1** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are directly interpolated inside run: shell command strings. In test.yml, the 'show outputs' steps in test-linux, test-darwin, test-windows, and test-strawberry jobs use `echo "perl-version: ${{ steps.setup-perl.outputs.perl-version }}"` and `echo "perl-hash: ${{ steps.setup-perl.outputs.perl-hash }}"` directly in run: blocks. These step output values flow through YAML template substitution before the shell sees them, enabling script injection if the values contain shell metacharacters.

Locations:

- `.github/workflows/test.yml:43`
- `.github/workflows/test.yml:44`
- `.github/workflows/test.yml:113`
- `.github/workflows/test.yml:114`
- `.github/workflows/test.yml:183`
- `.github/workflows/test.yml:184`
- `.github/workflows/test.yml:248`
- `.github/workflows/test.yml:249`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are directly interpolated as the entire run: command. In test-cpan-installer.yml, two steps use `run: ${{ matrix.installer }} --help` and `run: ${{ matrix.installer }} --version`. The matrix.installer value is substituted directly into the shell command before execution, allowing an attacker who controls the matrix value to inject arbitrary shell commands.

Locations:

- `.github/workflows/test-cpan-installer.yml:74`
- `.github/workflows/test-cpan-installer.yml:75`

### missing-permissions (severity: medium)

The workflow files check-dist.yml, test.yml, and test-cpan-installer.yml have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/check-dist.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/test-cpan-installer.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed script injection in test.yml by moving steps.setup-perl.outputs expressions to env: blocks in all 4 'show outputs' steps (using $PERL_VERSION/$PERL_HASH for bash jobs and $env:PERL_VERSION/$env:PERL_HASH for PowerShell jobs). Fixed script injection in test-cpan-installer.yml by moving matrix.installer to an env: block (INSTALLER) and referencing it as $INSTALLER in the run commands. Added 'permissions: contents: read' at the top level of all three workflow files (check-dist.yml, test.yml, test-cpan-installer.yml).

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted shell variable expansions in `.github/workflows/test-cpan-installer.yml` at lines 76 and 79. Changed `run: "$INSTALLER --help"` and `run: "$INSTALLER --version"` to `run: '"$INSTALLER" --help'` and `run: '"$INSTALLER" --version'` respectively. The YAML single-quoted strings now contain shell double-quotes around `$INSTALLER`, ensuring the variable is properly quoted in the shell and cannot be exploited via shell metacharacters in the matrix value.

