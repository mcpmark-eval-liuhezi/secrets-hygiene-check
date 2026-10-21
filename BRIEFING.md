# Secrets Hygiene Briefing

Prepared by HeziLiu for the squad, as part of our personal GitHub hygiene pass. Goal: understand what GitHub actually catches today, look at one real incident, and agree on the settings we should all run.

## What secret scanning covers

GitHub's secret scanning (now part of **GitHub Secret Protection**) works in two layers:

- **Alerts (detective).** Scans repositories for credential patterns — provider tokens and keys such as AWS, Azure, Google, OpenAI, Stripe, Slack and hundreds of other partner patterns — across the repository's commit history and branches. As of April 2024 it also scans **issues, discussions and pull request bodies**, not just code (GitHub changelog, "Secret scanning for historically existing secrets found in..."). Validity checks tell you whether a detected secret is **active or inactive**, which helps triage.
- **Push protection (preventive).** Checks for high-confidence secrets **as you push** and blocks the push before the secret ever lands (GA April 2022; github.blog "Proactively prevent secret leaks with GitHub Advanced Security"). It is now **enabled by default for all public repositories** (GitHub changelog, March 11, 2024), and individual users can turn it on for all pushes from their personal account (GitHub community discussion #63294, August 2023).

Important nuances for the squad:

- **Free for public repos.** Secret scanning alerts and push protection are available at no cost for all public repositories; for private repos they require GitHub Advanced Security / Secret Protection.
- **Coverage is pattern-based.** It only catches secrets it has patterns for. As of mid-2025, security teams can additionally **choose which secret scanning patterns are included in push protection** (github.blog changelog, June 25, 2025), rather than accepting a fixed high-confidence set.
- **It is not a substitute for rotation.** Detection does not equal remediation: a secret found by scanning was still readable by anyone until it was revoked.

Sources: docs.github.com (Push protection; About secret scanning; REST API endpoints for secret scanning push protection), github.blog changelogs (April 2022, March 2024, June 2025), GitHub community discussions (#63294, extended metadata checks, October 2025).

## Real incidents

**ArtiPACKED — leaked access tokens in GitHub Actions artifacts (reported August 2024).** Yaron Avital, a researcher with **Palo Alto Networks Unit 42**, published research in mid-August 2024 (August 13) showing that build artifacts produced by GitHub Actions in **public repositories routinely contained live access tokens** — including GitHub's own tokens and credentials for third-party cloud services. He found exposed secrets in artifacts across **dozens of public repositories**, and the attack chain he dubbed "ArtiPACKED" combined a race condition in the artifact system with poisoned artifacts to enable **repository takeover and cloud account access**. The work was widely covered: **CSO Online** ("Major GitHub repos leak access tokens putting code and cloud environments at risk", August 15, 2024), **SecurityWeek** ("GitHub Actions Artifacts Leak Tokens and Expose Cloud Accounts", August 15, 2024), and **The Hacker News** ("GitHub Vulnerability 'ArtiPACKED' Exposes Repositories to Ransomware-Like Attacks", August 15, 2024).

Why it matters for us: tokens were not in `main` source files — they were buried in **CI/CD artifacts and build logs**. That is exactly the blind spot you have if you only eyeball your diffs.

Follow-on context: Unit 42's later analysis traced the March 2025 mass GitHub Actions supply-chain compromise back to **stolen tokens** (reported by The Register, April 7, 2025, "Stolen SpotBugs tokens sparked the massive GitHub attack") — a reminder that one leaked token can cascade across hundreds of downstream projects.

## Recommended settings

- [ ] Turn on **push protection for your personal account** (Settings → Code security → Push protection for yourself) so every push from your machine is checked, not just repo-level protection.
- [ ] Confirm **secret scanning alerts + push protection are enabled** on every repo you own, including public ones (on by default, but verify after forks/imports).
- [ ] Enable **validity checks** so alerts tell you which leaked secrets are still live.
- [ ] **Rotate and revoke** any credential that ever touched a public repo or a CI artifact — treat any exposure as a compromise; assume the git history still contains it even after you delete the file.
- [ ] Keep secrets out of **GitHub Actions artifacts, logs and step summaries**; these are the paths that burned projects in the ArtiPACKED research.
- [ ] Prefer **environment-scoped secrets** with minimal `GITHUB_TOKEN` permissions over broad repo/org secrets.
- [ ] Add a local pre-commit hook (e.g. `gitleaks`) and keep `.env` files in `.gitignore` — defense in depth beyond platform scanning.
- [ ] If you maintain public Actions, review what your workflows write into artifacts, since anyone can download them.

---

*Prepared as part of the personal GitHub hygiene pass. Sources cited inline; primary links: docs.github.com/concepts/secret-security, github.blog (April 2022 / March 2024 / June 2025 changelogs), CSO Online (Aug 15, 2024), SecurityWeek (Aug 15, 2024), The Hacker News (Aug 15, 2024), The Register (Apr 7, 2025).*
