# PHP 7.4 Windows security build

This fork builds the upstream `PHP-7.4-CVE-2026-109618-109619` branch, anchored at commit `712e60f9edfd03f0960fc4fc3c2c375f42b90aa2`. The upstream commit describes this as a **minimal mitigation for an end-of-life version**. These are unofficial builds, not new supported PHP releases, and do not establish that other vulnerabilities are fixed.

## Build and download

[Workflow runs](https://github.com/qaz50237/php-src/actions/workflows/php74-windows.yml)

The workflow uses standard `windows-2022` GitHub-hosted runners in this public repository. It builds x64 TS and NTS using the VC15 14.16 toolset and official PHP SDK 2.8.4 pinned at `cfe7325dfdc9f20e54e85fb8929e32625d5f624e`. PHP dependencies come from the official PHP Windows dependency service; their series may change over time. At build time the PHP 7.4 stable series lists legacy dependencies including OpenSSL 1.1.1s, curl 7.70.0 and libxml2 2.9.10. This workflow does not update those libraries and its output must not be represented as a comprehensively security-updated runtime. See https://downloads.php.net/~windows/php-sdk/deps/series/packages-7.4-vc15-x64-stable.txt .

Only successful smoke tests and selected PHPT tests (`tests/basic`, `ext/session/tests`) allow upload of the binary artifact. These are not the full PHP suite or CVE-specific exploit validation. The upstream mitigation unconditionally disables Session upload progress in ext/session/session.c; applications relying on session.upload_progress will lose that feature. Exactly eight existing upload-progress tests are marked XFAIL with that explicit reason; the workflow verifies the upstream disabling condition before doing so. Other unexpected failures remain fatal. Source/PHPT fixtures are checked out with LF endings. Each binary artifact includes ZIP files, SHA256SUMS.txt, BUILD-INFO.json, PHP configuration information, and test output. Artifacts are retained for 7 days.

Push changes to `.github/workflows/php74-windows.yml` on `build/php74-windows-security` to trigger a build. GitHub only exposes `workflow_dispatch` normally when the workflow also exists on the default branch; this fork does not modify upstream master to enable that UI.

## Before replacing an existing installation

1. Check the current PHP minor version, x86/x64 architecture, thread safety, compiler, and PHP/extension API build identifiers. This build targets PHP 7.4 x64 only, not all PHP 7 versions.
2. Match TS/NTS to the current hosting setup. Check every third-party extension DLL separately; do not assume compatibility.
3. Back up the existing PHP directory, php.ini, and web server/FastCGI configuration. Extract into a separate directory and test the application there before switching the server.
4. Compare extension availability and configuration against the old environment. Run application, database, session, upload and scheduled-task checks.
5. Verify the downloaded distribution ZIP against SHA256SUMS.txt using `Get-FileHash -Algorithm SHA256`.
6. Keep a rollback path. This build does not install, deploy, or modify any production server.

The upstream main/php_version.h is stale (7.4.31-dev), while configure.ac specifies 7.4.34-dev. The workflow regenerates only that generated header from configure.ac and adds -cve202610, yielding 7.4.34-dev-cve202610. This identifies an unofficial build; it must not be interpreted as an official release designation. Runtime implementation files are otherwise unchanged from the selected upstream branch. Record BUILD-INFO.json and the complete workflow URL with your deployment evidence.
