# OSS Security Triage Log — Wazuh & Velociraptor

Tracking real, public GitHub Discussions/Issues answered as part of the resume-building
project (Security lane, OSS support triage). Goal per reviewer feedback: 1-2 weeks,
real hands-on/practitioner-reviewed work, tracked stats.

## Summary stats (update as we go)
- Total questions answered: 10
- Projects covered: Velociraptor (3), Wazuh (7)
- Self-diagnosed from scratch (original research): 8
- Found existing fix/doc and connected it: 2
- Acknowledged by maintainer/OP: pending
- Fact-check pass run 2026-07-20 (independent agent, re-verified all 7 entries against primary source): #4058, #3044, #36860, #4107 fully confirmed. #35231 and #33262 each contain one real error, correction comments not yet drafted. #33590 has one minor imprecise detail (cited path doesn't exist, underlying claim still true).
- Fact-checking is now a standing step before every post going forward (not a one-off), per Zach's instruction — see feedback memory.
- No maintainer/OP replies yet on any of the first 8 posted entries as of 2026-07-24 recheck.

## Entries

### 2026-07-24 — Wazuh
- **Question:** "Issues running Wazuh Dashboard Repo dev build locally (plugins not loading / dependency conflicts)" — OP cloned wazuh-dashboard alone, followed its DEVELOPER_GUIDE.md, got 404s on all Wazuh plugin bundles, then tried hand-gluing wazuh-dashboard-plugins into plugins/ and hit dependency conflicts and path errors across three release tags (open since 2026-03-14, only reply was a low-effort "is this fixed?")
- **Link:** https://github.com/wazuh/wazuh/discussions/34981#discussioncomment-17773378
- **Type:** Self-diagnosed / original research (built directly on prior research from the #36860 entry below, same root cause approached from the other direction)
- **Summary:** Confirmed wazuh-dashboard's DEVELOPER_GUIDE.md is unmodified upstream OpenSearch Dashboards boilerplate (literally tells you to fork opensearch-project/OpenSearch-Dashboards), not Wazuh-specific guidance, and that wazuh-dashboard's own `plugins/` folder is deliberately empty in the repo (just a `.gitignore`), so a bare clone can never produce the Wazuh plugin bundles the OP was expecting. Pointed to the actual supported path, `docker/osd-dev/dev.sh` in the wazuh-dashboard-plugins repo, and confirmed it exists identically on all three tags the OP tried (v4.14.5, v4.14.4, v4.14.2).
- **Status:** Posted, awaiting response
- **Fact-check:** Independent agent pass confirmed all 6 claims exactly against primary source (file contents, empty plugins/ dir, tag-level repo structure). No changes needed before posting.

### 2026-07-24 — Wazuh
- **Question:** "Reason for the removal of Threat Intelligence and Correlation Rules" — why Threat Intelligence and Correlation Rules are being removed from the Security Analytics dashboard plugin in Wazuh 5.0, OP linked two task-checklist issues as their only source (open since 2025-12-11, zero replies)
- **Link:** https://github.com/wazuh/wazuh/discussions/33746#discussioncomment-17773332
- **Type:** Self-diagnosed / original research (synthesized across 5 linked issues in two repos, no existing consolidated answer anywhere)
- **Summary:** Traced the OP's two linked issues back through their parent objective (wazuh-dashboard-plugins#7862) and closeout/evidence issue (#7921), confirming this is an explicitly presentation-layer-only reorg ("purely aesthetic," backend/API/data models untouched, still runs on `log_type` internally). Distinguished Threat Intelligence (fully removed from the UI) from Correlations/Alerts/Correlation rules (described as "hidden" from nav, a meaningfully different claim than deleted, though some smaller sub-elements are explicitly "removed"). Clarified this is the OpenSearch Security Analytics plugin's own detector/correlation layer, not Wazuh's core analysisd detection engine. Also surfaced a related cleanup PR (#7898) removing legacy Rules/Decoders/CDB lists apps. Explicitly flagged that no deeper strategic rationale exists anywhere in the public issue trail, rather than guessing at one.
- **Status:** Posted, awaiting response
- **Fact-check:** Independent agent pass confirmed all 7 claims against primary source, no errors found. Tightened one nuance (the hidden-vs-removed distinction only applies to the three top-level nav apps, not every element in #25) before posting.

### 2026-07-20 — Wazuh
- **Question:** "systemd-native wazuh-agent redesign" — reported that a crashed sub-daemon (e.g. wazuh-logcollector) doesn't trigger systemd restart because wazuh-control's internal multi-daemon supervision hides the crash from systemd (open since 2026-03-12, zero replies)
- **Link:** https://github.com/wazuh/wazuh/discussions/34911#discussioncomment-17706820
- **Type:** Self-diagnosed / original research (no existing answer anywhere; traced actual source across two repos)
- **Summary:** Confirmed the reported gap is real and architectural: `wazuh-control` forks/supervises sub-daemons itself, so systemd's unit only tracks the top-level process, never the sub-daemons. Traced Wazuh's separate agent-rewrite repo (`wazuh/wazuh-agent`, targeting 6.0.0 alpha) and confirmed via closed issues #127/#130 that the new single-process C++ agent structurally eliminates this specific failure mode. But also found (and correctly caveated) that the new agent's shipped systemd unit still has no `Restart=` directive, and its restart_handler module is command-triggered (config reload/upgrade, or `systemctl restart` under systemd) rather than a crash watchdog, so general auto-recovery still isn't solved. Suggested a practical `ExecStartPost` health-check workaround for the current 4.x setup.
- **Status:** Posted, awaiting response
- **Fact-check:** Independent agent pass caught one real inaccuracy pre-post (draft described the wrong restart code path for the systemd case, fork/SIGTERM/execve vs. the actual `systemctl restart wazuh-agent` call) — corrected before posting. Everything else confirmed exactly (repo, issue states/closure, exact unit file contents).

### 2026-07-20 — Velociraptor
- **Question:** "Velociraptor Standby Mode" — how to deploy clients that don't beacon/collect until manually activated (open since 2025-03-06, zero replies)
- **Link:** https://github.com/Velocidex/velociraptor/discussions/4107#discussioncomment-17695241
- **Type:** Self-diagnosed / original research (traced actual Go source, no existing answer anywhere on the repo)
- **Summary:** No documented feature for this, but the Windows client's service wrapper implements native SCM pause/continue (`AcceptPauseAndContinue` in `bin/installer_windows.go`), which calls `SetPause(true)` on the HTTP communicator (`http_comms/comms.go`), halting the send/receive loop while the process stays resident. Drivable via `sc pause`/`sc continue`, PowerShell, or WMI/RMM. Confirmed this is Windows-only, no equivalent in the Linux/macOS service wrappers. Independently fact-checked line-for-line before posting.
- **Status:** Posted, awaiting response

### 2026-07-20 — Wazuh
- **Question:** "How to run wazuh-dashboard from source with wazuh-dashboard-plugins in development mode?" — dev environment setup, relative import path resolution / duplicate plugin registration error (open since 2026-06-12, zero replies)
- **Link:** https://github.com/wazuh/wazuh/discussions/36860#discussioncomment-17694846
- **Type:** Self-diagnosed / original research (no existing answer to link to)
- **Summary:** OP was manually cloning wazuh-dashboard and wazuh-dashboard-plugins side by side and gluing them together with `--plugin-path`, which is the source of both the import path errors and the duplicate plugin registration. Read the actual repo docs (`docs/dev/setup.md`, `docs/dev/run-sources.md`, `docker/osd-dev/README.md`) and confirmed the current supported workflow is a Docker-based dev environment (`docker/osd-dev/dev.sh`) that auto-detects versions and auto-mounts the internal plugins (main/wazuh-core/wazuh-check-updates, which already live inside the plugins repo) at the correct path depth. Verified this workflow exists on the 4.14.x branch the OP is actually using, not just `main`, before answering.
- **Status:** Posted, awaiting response

### 2026-07-19 — Wazuh
- **Question:** "Bug or feature? analysisd will request latest scan results only if a hash of a related policy file was changed" — SCA source-code design question (open since 2025-12-18, zero replies)
- **Link:** https://github.com/wazuh/wazuh/discussions/33590#discussioncomment-17685444
- **Type:** Self-diagnosed / original research (read actual C source references + traced design intent)
- **Summary:** Confirmed the hash-gating behavior in analysisd's SCA module is intentional, not a bug, traced it to issue #3006 which specifically introduced this integrity check to stop agents resending full scans on every restart. Explained the two-hash system (policy file hash vs scan results hash) to directly address the OP's worry about missed config improvements/regressions, and pointed them to the exact ossec.log message to disambiguate which hash they're hitting. Also confirmed the same logic persists in the new Wazuh 5.0 C++ engine rewrite (builder/optransform/sca), so it's active design.
- **Status:** Posted, awaiting response

### 2026-07-18 — Velociraptor
- **Question:** "How to Handle Failed Hunts" — how to re-run a hunt against clients where it failed/timed out (open since 2023-10-23, zero replies)
- **Link:** https://github.com/Velocidex/velociraptor/discussions/3044#discussioncomment-17677638
- **Type:** Found existing official doc and connected it
- **Summary:** Velociraptor has an official (undersurfaced) knowledge base tip covering exactly this: a manual GUI method (copy failed collection, re-add to hunt) and an automated VQL method using `hunt_flows()` to find `ERROR`-state flows, `collect_client()` to re-collect with higher limits, and `hunt_add()` to fold the new flow back into the original hunt. Posted both methods plus the VQL snippet and noted the OP's own label-based idea would work but isn't the native mechanism.
- **Status:** Posted, awaiting response

### 2026-07-18 — Wazuh
- **Question:** "Ism and datastream" — ISM policy not applying to new backing indices after rollover (open since 2025-11-22, zero replies)
- **Link:** https://github.com/wazuh/wazuh/discussions/33262#discussioncomment-17677550
- **Type:** Self-diagnosed / original research
- **Summary:** Root cause is an OpenSearch ISM behavior (not Wazuh-specific): manually attaching a policy to an index only manages that one index, doesn't carry to future rollover-created backing indices. Fix: use an `ism_template` block with a pattern matching the data stream's backing indices, which auto-applies the policy to new matching indices. Cited that Wazuh's own `wazuh-alerts`/`wazuh-archives` indices use this same mechanism via index template settings. Sourced from OpenSearch docs, an upstream OpenSearch GitHub issue with the same symptom, and Wazuh's index lifecycle management docs.
- **Status:** Posted, awaiting response
- **Fact-check note (2026-07-20):** The claim about `wazuh-alerts`/`wazuh-archives` using `ism_template` by default is wrong, those are legacy Filebeat date-suffixed indices, not data streams (no `data_stream` block in `wazuh-template.json`), and Wazuh's docs present `ism_template` as something the admin sets up manually, not a built-in default. The core `ism_template` fix advice for the OP's own case is still correct. Correction comment not yet drafted/posted.

### 2026-07-17 — Wazuh
- **Question:** "Detection Engineering with Sigma + Wazuh — Seeking Industry Guidance on SOP, Automation, Testing and Normalisation" (open since 2026-04-01, zero replies)
- **Link:** https://github.com/wazuh/wazuh/discussions/35231#discussioncomment-17677434
- **Type:** Self-diagnosed / original research (no existing answer anywhere to link to)
- **Summary:** Multi-part question on Wazuh + pySigma detection engineering. Answered 4 of 9 sub-parts with sourced research: (1) confirmed Wazuh's analysisd processing order (decode -> rule match happens before any downstream ECS normalization, so custom/default rules don't break), (2) confirmed the pySigma-Wazuh backend's known `if_sid` scoping gap and named community tools that patch it (SigWaz, theflakes/sigma_to_wazuh), (3) noted there's no official Sigma-severity-to-Wazuh-level mapping table, just a practitioner convention, (4) confirmed `wazuh-logtest` is the official testing tool. Explicitly flagged the remaining sub-questions (ECS pipeline, bulk field migration, self-discovering field mapping) as genuinely unanswered elsewhere rather than guessing.
- **Status:** Posted, awaiting response
- **Fact-check note (2026-07-20):** Sub-part (2) overstates it, there's no official pySigma-Wazuh backend at all (per SigmaHQ maintainer, Jan 2026), only independent non-pySigma converter tools that don't fully solve `if_sid` scoping either. Correction comment not yet drafted/posted.

### 2026-07-17 — Velociraptor
- **Question:** "How to properly run Windows.Memory.Acquisition artifact" (open since 2025-02-11, unanswered follow-up)
- **Link:** https://github.com/Velocidex/velociraptor/discussions/4058#discussioncomment-17677272
- **Type:** Found existing fix (issue #4209) and connected it to the unanswered thread
- **Summary:** WinPmem kernel driver fails to load from a path containing a space (e.g. `C:\Program Files\...`). Root-caused and fixed upstream via a `DriverPath` parameter (default `C:\Windows\Temp\winpmem.sys`), shipped after v0.74.2. Posted explanation + fix + link to source issue.
- **Status:** Posted, awaiting response

---

## How to add a new entry
1. Date, project (Wazuh / Velociraptor / other)
2. Question title + link
3. Type: self-diagnosed / found-and-connected
4. One or two sentence summary of the actual technical answer
5. Status: posted / awaiting response / acknowledged
