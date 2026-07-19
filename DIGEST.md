# odysseus-watch digest
<!-- pewdiepie-archdaemon/odysseus -->

**Last reviewed:** 2026-07-19  
**Last reviewed SHA:** `68ba51c` (fix(llm): enhance fallback logic to handle empty completions and improve metadata handling, PR #5491)

---

## Watching

| Item | Why |
|------|-----|
| #3136 | Deep-research — /api/research/status returns 404 |
| #3158 | Security — deny agent file tools access to credential/session stores (PR open, needs work) |
| #3150 | Security — Harden API token route capabilities (draft PR open) |
| #3148 | Security — gallery null-owner read bypass when auth enabled |
| #236  | Security — encrypted-at-rest secrets via SOPS (long-running PR) |
| #605  | Architecture v3 proposal — stale; live work moved to ADRs #987, #1013, #1023 |

*(Resolved and dropped from watch this round: #3144/#3139 merged; #3100 closed unmerged — see below.)*

---

## 2026-07-19

> Six weeks of activity: 961 commits to `dev`, stars up 58k→83k, forks ~11k, open issues/PRs now in the thousands. Way too much volume to walk commit-by-commit, so this pass leans on merged-PR search by topic. Given the pace, treat "nothing found" for a keyword as "not surfaced by this search," not a guarantee.

### Watching list — status changes

- **RESOLVED** #3144 / **MERGED #3139** (Jun 7): event bus no longer skips event-triggered tasks (dropped a bogus `next_run` check); bulk session deletion now respects starred-session protection and reuses the single-session delete path for accurate counts.
- **CLOSED, not merged** #3100 (Jul 12): the big Windows Cookbook download/serve PR was closed without merging — its ground seems to have been covered piecemeal instead by ~20 smaller `fix(cookbook)` PRs merged through June (adopt-host validation, SSH target validation, serve command-injection guards, port-collision/watchdog fixes). Worth a sanity check that nothing from the original PR's fixes fell through the cracks.
- **#3136 still open** — `/api/research/status` / `/api/chat/stream_status` 404s remain unresolved.
- **#3158, #3150, #3148, #236 still open**, no movement. Note: #3158 (credential-store file access) overlaps with several sensitive-file deny-list PRs that *did* merge this window (see Security) — worth checking if it's now redundant.
- **#605 still open** but maintainer flagged it stale in a Jun 27 comment; real decisions are landing in ADRs #987, #1013, #1023 instead.

### Security (act on these first)

- **MERGED #4433**: closed an agent-path **RCE** via `manage_mcp`'s `add` action — now allowlisted. Highest-severity fix this window; update if self-hosting.
- **SSRF hardening**: **#5147** pins webhook delivery to the SSRF-validated IP (closes a DNS-rebinding gap), **#5145** applies the outbound SSRF guard to integration `api_call` URLs, **#5142** extends it to the reminder ntfy sender.
- **ReDoS sweep**: five separate merges (**#4941, #4877, #4708, #4704, #4943**) fixed exponential-blowup regexes across the XML/args tool-call parser, `<think>`-stripping, email→calendar extraction, and LLM-output parsing.
- **Credential hygiene**: **#4750** redacts credential-bearing URLs and PII from logs; **#4197** locks the API-key encryption key file to `0o600`.
- **Sensitive-file deny-list**: **#5013/#5097/#4986/#4981** progressively harden the agent tool deny-list (grep tool coverage, case-insensitivity, research path confinement, gallery endpoint URL handling) — this is the area #3158 lives in.
- **Session/owner scoping**: **#5238** scopes owner-less email accounts to a mailbox match, **#5195** scopes `send_to_session` to the exact owner, **#5212** sanitizes the email rich-body render path.
- **#3506**: agent no longer gets tool access during the pre-setup window.
- No published GHSA security advisories on the repo despite the volume of fixes above — all landed as regular PRs.

### Task scheduler

- Nothing beyond the #3139 merge above — no other scheduler/event-bus/cron work surfaced this window.

### Deep-research

- **MERGED #4784** (Jun 27): improves uploaded-document retrieval and result reuse.
- **MERGED #2189** (Jun 7): preserves partial deep-research findings on non-timeout errors instead of discarding them.
- #3136 (status-endpoint 404s) is still the open gap here.

### Cookbook

- Heavy iteration (~20 merged fixes) rather than one big PR since #3100 was closed — see status-changes note above for the list. Notable ones: **#4429** validates agent SSH targets, **#4345** hardens remote serve host handling, **#4283** stops orphaned Windows process trees.

### Other notable

- Two high-reaction **reverts**: **#3438** (39 👍) pulled the Tools Auto/On/Off control back out of the local-endpoint UI, **#3437** (13 👍) reverted the model-picker remove-from-recent button — both framed as "stabilize first, add UI later." Community reaction reads as preferring less UI churn right now.
- Project scale: 83.3k stars / ~11k forks / open issues+PRs now in the thousands — maintainer backlog is large.

### External discussion

- No significant new external coverage found since the last run — searches turned up the same May/June launch-cycle articles and the existing HN thread. Nothing indicating a fresh controversy or notable technical writeup this window.

## 2026-06-06 — bootstrap entry

> First run. Project launched 2026-05-31; 58k+ stars in one week. Very active — 111 commits and ~50 PRs/issues in the last six days alone.

### Security (act on these first)

- **MERGED `e87a1ad`** `fix(deep-research)`: wrap fetched webpage content in untrusted-context sandbox — raw page content was going straight into LLM prompts via string substitution, bypassing `prompt_security.py`. Prompt-injection hole, now fixed. Fixes #3044. **Update now if self-hosting.**
- **PR #3158 open** `fix(tools)`: deny agent file tools access to Odysseus' own credential/session stores — agent could read auth tokens off disk; needs merge.
- **PR #3150 open** Harden API token route capabilities — adds default-deny enforcement for `ody_...` bearer tokens; companion to issues #3092 / #3090 / #3097.
- **Issue #3148 open** Order-dependent test failures + gallery null-owner read bypass when auth is enabled — unauthenticated users can read gallery items with no owner set.
- **Issue #3068 open** Agent executes shell commands despite explicit guide-only / no-tools instruction — correctness + safety concern.
- **PR #236 open** Support optional encrypted-at-rest secrets via SOPS — long-standing security enhancement, still awaiting merge.

### Task scheduler

- **Issue #3144 open** Event-triggered tasks may be skipped: `next_run` validation rejects immediate triggers; also, bulk session deletion bypasses `is_important` / starred protection. Two bugs in one report.
- **PR #3139 open** Fix logical bugs in event bus and bulk session deletion — likely the fix for #3144; worth reviewing.

### Deep-research

- **Issue #3136 open** Frontend calls `/api/research/status` and `/api/chat/stream_status` — both return 404. Research status polling is broken.
- **`e87a1ad`** (security fix above) is also the deep-research extractor; see Security section.

### Cookbook

- **PR #3100 open** `fix(cookbook)`: Windows local download + serve fixes — bash routing, progress display, CUDA llama-server support, busy-endpoint watchdog. Windows Cookbook was largely broken; this is the fix.
- **Issue #3142 open** / **PR #3143 open** `duckduckgo-search` package deprecated — switching to `ddgs`. Affects web-search tool used by deep-research and agents.

### Other notable

- **MERGED `b03d934`** fix: restore backup import after skills migration (#2980) — backup/restore was broken after a recent skills refactor; now fixed.
- **MERGED `e87a1ad`** and **MERGED `893cb82`** fix(sessions): dead spinner bug when agent run detaches and page reloads — resumeStream retry in poll loop.
- **Issue #3141 open** Commit `dc365a1` breaks tool calling / memory management for local Ollama models on Windows — regression worth watching.
- **PR #3140 open** Add Gemini-native REST support in LLM core — Gemini models were silently falling back to OpenAI-compatible endpoint (#3145).
- **Issue #605 open** Proposal: Architecture & Codebase Structure v3 — large refactor proposal; restructures `background/` layer including `task_scheduler`, adds `cookbook.py` module. Worth reading before making changes in those areas.

### External discussion

- Active HN thread: [Odysseus – self-hosted AI workspace](https://news.ycombinator.com/item?id=48346693) — mostly positive; feature requests around plugin ecosystem and multi-user support.
- Project went viral on launch (May 31); 58k stars, Trendshift tracking it.
