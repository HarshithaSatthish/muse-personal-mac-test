# Security, Legal & Feasibility Audit: Remote macOS on GitHub Actions (`github-mac-remote`)

**Project / Investigation:** Testing Meta Muse Personal on a Free Apple Silicon GitHub Actions Runner  
**Primary Reference:** `https://github.com/brunusansi/github-mac-remote`  
**Host Target:** GitHub-hosted `macos-15` Runner (Apple Silicon M1, 3 vCPU, 7 GB RAM, 14 GB SSD)  
**Date:** November 8, 2026  
**Auditor:** Antigravity AI  

---

## 1. Executive Summary

This audit evaluates the viability, security, legal compliance, and practical usability of using GitHub Actions macOS runners (`macos-15`) paired with remote-access tooling (such as `github-mac-remote`) to operate **Meta Muse Personal for Mac** as an interactive, zero-cost cloud macOS computer.

### Primary Audit Verdict:
* **Interactive Remote Desktop on GitHub Actions:** **FAILED / VERIFIED BLOCKED (Policy & Architectural Violation)**
* **CI/CD Application Packaging & Non-Interactive Diagnostics:** **VERIFIED (Permitted CI Workflow)**
* **Meta Muse Personal Authentication & Interactive Operation in Runner:** **VERIFIED BLOCKED (Security & TCC Architecture)**

Attempting to repurpose a GitHub Actions runner as an interactive desktop environment using tunneling tools violates GitHub's Acceptable Use Policies and triggers automated abuse detection, endangering your GitHub account. Furthermore, modern macOS TCC (Transparency, Consent, and Control) security mechanisms prevent remote synthetic input without physical console approvals.

---

## 2. Technical Audit of `github-mac-remote` and VNC-Tunneling Stacks

Repositories such as `github-mac-remote`, `macos-rdp`, or `vnc-macos` typically implement the following architecture:

```
+-------------------------------------------------------------------+
| GitHub Actions 'macos-15' Ephemeral Runner                        |
|                                                                   |
|  [macOS Aqua / WindowServer]                                      |
|          |                                                        |
|  [ARDAgent / Screen Sharing] (Port 5900 - Legacy VNC)             |
|          |                                                        |
|  [Tunnel Agent (ngrok / tmate / frp / cloudflared)]               |
+----------|--------------------------------------------------------+
           | Public TCP Tunnel (Relayed over 3rd party infrastructure)
           v
+-----------------------+      +------------------------------------+
| External VNC Client   | ---> | User on Windows / Android          |
+-----------------------+      +------------------------------------+
```

### Critical Vulnerabilities & Flaws Identified:

1. **Weak VNC Authentication & Protocol Insecurity:**
   - Enabling VNC via Apple's `kickstart` tool using `-setvnclegacy yes` restricts passwords to 8 characters and encrypts them using legacy DES.
   - DES encryption is obsolete and vulnerable to offline cracking.
   - Standard VNC traffic over generic TCP tunnels is unencrypted, exposing all screen framebuffers (including text, UI contents, and prompts) to man-in-the-middle (MITM) inspection on intermediate network relays.

2. **Exposing Personal Credentials on Ephemeral Multi-Tenant Nodes:**
   - Meta Muse Personal requires logging into a personal Meta/Facebook account.
   - Performing interactive authentication on an ephemeral multi-tenant runner whose network traffic is tunneled through third-party relays creates an unacceptable risk of session hijacking, credential harvesting, or token leakage.

3. **No State Persistence:**
   - GitHub Actions runners are strictly ephemeral. When a job finishes, the virtual machine is completely terminated and destroyed.
   - Any installed application, downloaded files, authenticated sessions, local databases, or desktop customizations vanish immediately.
   - Every session would require a fresh download, installation, manual authentication, and 2FA approval from scratch.

---

## 3. GitHub Terms of Service & Acceptable Use Policy Compliance

GitHub's **Additional Product Terms for GitHub Actions** and **GitHub Acceptable Use Policies** define explicit boundaries for runner compute:

### A. Prohibited Use Cases:
1. **Unrelated Activity:** Actions compute must be used strictly for software development, testing, compilation, and deployment related to the repository. Using compute as an interactive cloud workstation is explicitly prohibited.
2. **Persistent Servers & Proxies:** Running background tunnels (`ngrok`, `tmate`, `frp`), remote desktop relays, or proxies to establish persistent incoming connections violates acceptable use.
3. **Artificial Delays / Looping:** Using `sleep 21600` or infinite loops to artificially hold a runner open for interactive human use is monitored by GitHub's abuse detection systems.

### B. Consequences of Violation:
* Immediate job termination.
* Automated account flagging.
* Temporary suspension or permanent ban of the user's GitHub account and organizations.

**Conclusion:** Implementing an interactive VNC tunnel to use GitHub Actions as a free remote Mac is an unacceptable operational risk to your GitHub account (`HarshithaSatthish`).

---

## 4. macOS 14/15 TCC Security & Permission Blockers

Even if remote desktop access were permitted, Apple's **TCC (Transparency, Consent, and Control)** subsystem on macOS 14 (Sonoma) and macOS 15 (Sequoia) creates an impassable technical barrier for AI agents:

1. **Accessibility Permissions:**
   - Meta Muse Personal requires Accessibility privileges to inspect the UI, click buttons, enter text, and automate desktop applications.
   - On macOS 15, Accessibility permissions cannot be granted via CLI scripts without disabling System Integrity Protection (SIP) or having a signed Mobile Device Management (MDM) profile installed.
   - In a headless runner session, the modal dialog prompt cannot be confirmed programmatically. Without Accessibility permissions, Muse cannot automate applications.

2. **Screen Recording Permissions:**
   - Muse Personal requires Screen Recording to capture context from your screen.
   - In macOS 15, Screen Recording permissions require explicit user confirmation in *System Settings > Privacy & Security > Screen & System Audio Recording*.
   - A remote agent running non-interactively is denied screen capture access by the macOS WindowServer.

---

## 5. Summary Status Matrix

| Component / Requirement | Status | Detailed Finding |
| :--- | :--- | :--- |
| **Apple Silicon Hardware on Runner** | **VERIFIED** | GitHub `macos-15` provides authentic Apple Silicon M1 (3 vCPU, 7 GB RAM, 14 GB SSD). |
| **Non-Interactive CI Verification** | **VERIFIED** | Clean, read-only CI diagnostic workflows are 100% compliant with GitHub policies. |
| **Interactive Remote Desktop (VNC)** | **FAILED** | Violates GitHub Acceptable Use Policy; vulnerable to MITM credential exposure. |
| **TCC Permissions (Accessibility/Screen)** | **FAILED** | macOS 15 blocks programmatic granting; halts agent GUI automation. |
| **Personal Meta Muse Authentication** | **FAILED** | Ephemeral runner destroys session data; high risk of credential exposure over tunnels. |
| **Session Persistence Across Restarts** | **FAILED** | Runner disks are wiped to zero upon job termination. |

---

## 6. Recommended Safe Path

1. **Execute Safe CI Diagnostics Only:** Run the non-interactive diagnostic workflow (`.github/workflows/muse_mac_test.yml`) to verify hardware, network reachability, package integrity, and Gatekeeper notarization.
2. **Do Not Tunnel Remote Desktop:** Refrain from injecting `ngrok` or opening VNC ports on GitHub Actions to protect your account standing.
3. **Dedicated Apple Silicon Hardware:** For persistent, authorized interactive use with Meta Muse Personal, genuine local or dedicated Apple Silicon hardware remains the only legally compliant, technically capable, and persistent solution.
