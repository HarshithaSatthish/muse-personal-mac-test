# Meta Muse Personal - macOS 15 Runner Diagnostic Suite

This repository is a disposable test project designed to evaluate and verify **Meta Muse Personal for Mac** on an **Apple Silicon macOS 15 (Sequoia)** GitHub Actions runner (`macos-15`).

## Repository Contents

* `.github/workflows/muse_mac_test.yml`: Manual GitHub Actions workflow running on `macos-15` (M1 Apple Silicon, 3 vCPU, 7 GB RAM, 14 GB SSD) with strict `read-only` contents permissions.
* `scripts/verify_and_install_muse.sh`: Non-interactive verification script to inspect system architecture, verify official Meta distribution endpoints, test Gatekeeper signatures, and evaluate headless launch behavior.
* `SECURITY_AUDIT_GITHUB_MAC_REMOTE.md`: Full security and policy audit regarding GitHub Actions Acceptable Use Policies, VNC tunneling risks, and macOS 15 TCC permission constraints.
* `MUSE_GITHUB_RUNNER_REPORT.md`: Comprehensive execution report detailing test results and findings.

## Triggering the Test Workflow

You can manually trigger this workflow using the GitHub CLI:

```bash
gh workflow run "macOS 15 Apple Silicon - Meta Muse Personal Diagnostic"
```

Or via the GitHub web interface:
1. Go to the **Actions** tab in this repository.
2. Select **macOS 15 Apple Silicon - Meta Muse Personal Diagnostic**.
3. Click **Run workflow**.

Once complete, download the `macos15-muse-diagnostics` artifact bundle to review the detailed execution logs and hardware reports.
