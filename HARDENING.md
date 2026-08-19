<!-- markdownlint-disable -->

# Hardening Report: shogo82148--actions-setup-perl/v1.39.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shogo82148--actions-setup-perl/v1.39.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation inside run: blocks. In test.yml, the 'show outputs' steps in multiple jobs directly interpolate ${{ steps.setup-perl.outputs.perl-version }} and ${{ steps.setup-perl.outputs.perl-hash }} inside echo/Write-Output shell commands. These steps context expressions are substituted by the YAML template engine before the shell sees them, enabling script injection. In test-cpan-installer.yml, `run: ${{ matrix.installer }} --help` and `run: ${{ matrix.installer }} --version` directly interpolate a matrix context value as the shell command itself.

Locations:

- `.github/workflows/test.yml:41`
- `.github/workflows/test.yml:42`
- `.github/workflows/test.yml:112`
- `.github/workflows/test.yml:113`
- `.github/workflows/test.yml:183`
- `.github/workflows/test.yml:184`
- `.github/workflows/test.yml:254`
- `.github/workflows/test.yml:255`
- `.github/workflows/test-cpan-installer.yml:68`
- `.github/workflows/test-cpan-installer.yml:69`

### missing-permissions (severity: medium)

Workflow files have no top-level permissions: key and no job-level permissions: keys on any job. Without explicit permissions, workflows inherit the default repository permissions (which may include write access to contents), violating the principle of least privilege. Affected files: test.yml (jobs: test-linux, test-darwin, test-windows, test-strawberry, test-version-file, format), check-dist.yml (job: check-dist), and test-cpan-installer.yml (job: installer).

Locations:

- `.github/workflows/test.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/test-cpan-installer.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed script injection in test.yml by moving ${{ steps.setup-perl.outputs.perl-version }} and ${{ steps.setup-perl.outputs.perl-hash }} expressions from run: blocks into env: blocks (PERL_VERSION/PERL_HASH) for all 4 affected jobs (test-linux, test-darwin, test-windows, test-strawberry). For PowerShell steps, used $env:PERL_VERSION/$env:PERL_HASH syntax. Fixed script injection in test-cpan-installer.yml by moving ${{ matrix.installer }} into an env: block (INSTALLER) and referencing it as "$INSTALLER" in the run: commands. Added 'permissions: contents: read' at the top level of all three workflow files (test.yml, test-cpan-installer.yml, check-dist.yml) to enforce least privilege.

