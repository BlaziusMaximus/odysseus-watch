# odysseus-watch digest
<!-- odysseus-dev/odysseus (renamed from pewdiepie-archdaemon/odysseus, ~Jul 2026) -->

**Last reviewed:** 2026-07-26  
**Last reviewed SHA:** `cf4e240` (Merge verified Odysseus fixes, main)

---

## Watching

| Item | Status | Why |
|------|--------|-----|
| #3144 | ✅ Fixed (merged Jun 7) | Task scheduler — event-triggered tasks may be skipped (next_run validation bug) |
| #3139 | ✅ Merged Jun 7 | PR: Fix logical bugs in event bus and bulk session deletion |
| #3136 | 🔴 Still open | Deep-research — /api/research/status returns 404 |
| #3100 | ⚪ Closed, not merged | Cookbook — Windows local download + serve fixes (maintainer asked for smaller PRs) |
| #3158 | ⚪ Closed, superseded | Security — deny agent file tools access to credential/session stores (broader hardening landed instead) |
| #3150 | ✅ Landed Jul 23 | Security — Harden API token route capabilities (default-deny capability manifest) |
| #3148 | 🔴 Still open | Security — gallery null-owner read bypass when auth enabled |
| #236  | 🟡 Stalled | Security — encrypted-at-rest secrets via SOPS (conflicting/stale since Jun 26, needs rebase) |
| #605  | 🟡 Moved to ADRs | Architecture v3 proposal — active work now in #987, #1013, #1023 |

---

## 2026-07-26

> Repo renamed `pewdiepie-archdaemon/odysseus` → `odysseus-dev/odysseus` (GitHub redirects old links). ~7 weeks of activity: 1,344+ merged PRs total, 5,757+ issues filed. This entry covers only what's notable, not a full changelog.

### Watching list — status changes

- **#3144 / #3139 fixed** — merged Jun 7 into `dev`. Event-triggered tasks no longer skipped; bulk session deletion now respects starred/`is_important` sessions.
- **#3150 landed** (Jul 23) — default-deny capability manifest for bearer-token API routes; token/admin/pairing routes now blocked unless explicitly registered.
- **#3158 closed, not merged as-is** (Jul 23) — the deny-list approach (credential/session-store files off-limits to agent file tools) was superseded by broader hardening; reviewer had flagged basename-matching and symlink-bypass gaps. Worth confirming the replacement actually covers the original threat (agent-mode prompt injection → token exfiltration).
- **#3136 still open** — `/api/research/status` and `/api/chat/stream_status` still 404. No PR linked; reporter says they won't fix it themselves.
- **#3148 still open** — gallery null-owner auth bypass unresolved; no PR yet.
- **#3100 closed, not merged** — maintainer rejected the Windows Cookbook fix as too broad/stale, asked for it to be split into focused PRs. Nobody has picked that up yet.
- **#236 (SOPS secrets) stalled** — conflicting against `dev` since Jun 26, needs a rebase from the author.
- **#605 (Architecture v3) superseded by ADRs** — if touching scheduler/cookbook/research code, check #987 (ADR-001), #1013 (ADRs 002–024, draft), #1023 (Python tooling) instead of the old proposal.

### Security (beyond the watch list)

- **#5250** (Jul 21) — CodeQL advanced setup now scans PRs before merge.
- **#5261** (Jul 14) — blocks private-network SSRF targets, revalidates redirects in the skills importer.
- Jul 4 cluster: SSRF guards extended to webhook delivery, integration `api_call`, and reminder ntfy sender; `send_to_session` scoped to exact owner; email rich-body render path sanitized.
- **#3926** (Jul 2) — Cookbook's Ollama runner no longer executes the install one-liner (was a local RCE-shaped footgun).
- **#4902** (Jun 30) — host Docker socket is now opt-in, not default-mounted.
- **#4420** (Jul 11) — `data/app.db` permissions restricted to 0600.
- **#5264** (Jul 11) — `manage_tasks` mutations scoped to the exact task owner.

### Task scheduler

- **#4398** (Jun 16) — scheduled-task agents now get shell/file tools **by default**. Worth a look if you care about the agent-file-tools attack surface from #3158/#3150 above — default tool grants and hardening are moving in opposite directions.
- **#5757 / #5755 (new, open, Jul 25)** — `manage_tasks` can write a tool-call as plain text without executing it, and there's no durable per-run record of tool executions (`task_runs.steps` always NULL). Correctness/observability gap, not yet fixed.

### Deep-research / Cookbook

- Research file-path handling hardened (#4986, #5129 — path confinement + CodeQL-friendly lookup), but the status-endpoint 404 (#3136) is unrelated and still broken.
- Cookbook got a Docker-socket opt-in fix and Ollama RCE fix (see Security above) but the Windows-specific PR (#3100) is dead; no replacement in flight.

### Other notable

- **Org rename broke the production Docker image.** New package `ghcr.io/odysseus-dev/odysseus` is private/unauthorized; old `ghcr.io/pewdiepie-archdaemon/odysseus` is frozen (no updates since Jul 12); no git tags or GitHub Releases exist as a fallback. Tracked in **#5728** (open, unresolved). If you self-host from the published image, you're currently stuck on a stale build — an unofficial community rebuild exists (`ghcr.io/worph/odysseus:1.0.2-dev.7c8070f`) but isn't official.
- **Critical startup crash** — `NameError: name 'Any' is not defined` in `src/agent_loop.py` breaks app startup entirely (issues **#5732**/**#5723**, filed Jul 24). One-line fix identified and up as **PR #5733**, not yet merged as of Jul 25.
- **New, open, unresolved** (Jul 25): **#5743** — app pegs 100% CPU from an AnyIO cancellation-loop storm (reported on TrueNAS); marked "needs more info," no fix in progress.

### External discussion

- No significant new external coverage turned up since the last review's HN thread (project press has mostly covered the May 31 launch and the ~58k→77k star growth in the following weeks). The org rename hasn't generated visible outside discussion yet, despite the real breakage it caused.

---

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
