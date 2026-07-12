# odysseus-watch digest
<!-- pewdiepie-archdaemon/odysseus -->

**Last reviewed:** 2026-07-12  
**Last reviewed SHA:** `9844a2f` (chore(release): bump version to 1.0.2)

---

## Watching

| Item | Why |
|------|-----|
| #3136 | Deep-research — /api/research/status returns 404 (still unresolved) |
| #3158 | Security — deny agent file tools access to credential/session stores (PR open, changes requested) |
| #3150 | Security — Harden API token route capabilities (PR open, draft; several PRs now depend on it) |
| #3148 | Security — gallery null-owner read bypass when auth enabled (still open) |
| #236  | Security — encrypted-at-rest secrets via SOPS (long-running PR, awaiting rebase) |
| #605  | Architecture v3 proposal — maintainer calls it possibly stale; see ADR #1013 instead |
| #5263 | Security — `manage_tasks` owner-check bypass, any user can run/edit/delete others' scheduled tasks (fix PR #5264 open) |
| #5460 | Security — email "attach from Odysseus" leaks owner-less gallery images cross-tenant + bulk zip download (fix PR #5461 open) |
| #5431 | Task scheduler — one stuck task wedges the entire scheduler for all users (unresolved regression, see #5294/#4826) |

---

## 2026-07-12

> 5+ weeks since last review. ~530+ PRs merged — repo stayed very hot. Dominant theme: a sustained security-hardening sprint (SSRF/IDOR/RCE/ReDoS) through June, then a rocky 1.0.2 release on Jul 11-12 that broke chat for everyone and needed same-day hotfixes.

### Watching list — status changes

- **RESOLVED** #3144 (scheduler skip bug) — fixed by merged PR #3139 (Jun 7). Both dropped from Watching.
- **CLOSED unmerged** #3100 (Windows Cookbook fixes) — closed Jul 12 as stale/conflicted; maintainer asked for smaller per-issue resubmissions. Underlying Windows Cookbook bugs likely still present. Dropped from Watching.
- #3136, #3148, #605 — no progress, still open.
- #3158 — reviewer requested exact-path matching + symlink hardening before merge.
- #3150 — still draft, but now blocking #3400/#3944/#3899/#3946 — worth landing soon.
- #236 — awaiting rebase against `dev`, no material change.

### Security (act on these first)

- **RCE closed**: #4433 — MCP `add` path allowed remote code execution; now allowlisted.
- **SSRF/DNS-rebinding cluster** (Jul 4): #5147, #5145, #5142, #704, #347 — pinned webhook/integration/reminder/search/diffusion-server outbound requests to SSRF-validated, resolved IPs. Recurring bug class across nearly every outbound-request feature — good sign it's now been swept systematically.
- **IDOR/ownership-bypass cluster** (recurring "owner-less record ⇒ visible to all" pattern): #5238, #5195, #5264, #3899, #3807, #1511, #3006 fixed. **Two instances of the same bug pattern still open** — see #5263 and #5460 in Watching above.
- **Secrets**: CardDAV password now encrypted at rest (#1741); API-key file locked to 0600 (#4197); logs redact credentials/PII (#4750).
- **ReDoS**: fixed across LLM tool/XML/calendar/email parsers (#4877, #4704, #4708, #4712, #4941, #4943).

### Task scheduler

- **New, unresolved and concerning**: #5431 / #5294 / #4826 — a single wedged agent-loop stream holds the scheduler's only execution slot, so one stuck task freezes the entire system ("Queued — waiting for idle" forever, cascades to all users). Added to Watching.
- Fixed: #5109 (PR #5110) — two racing pollers could double-send the same scheduled email.
- #5264 — owner-scope fix for `manage_tasks` (see Security; companion issue #5263 still open).
- #4027 — background tasks (auto-title, memory audit) moved to a cheaper "utility model" (cost fix).

### Deep-research

- #3136 still unresolved (see Watching).
- Research routes refactored into `routes/research/` subpackage (#4975); file-path handling hardened (#4986/#5129).
- Requested: restart/edit sessions (#5284), `/report` regeneration command (#5152).

### Cookbook

- #3100 closed unmerged (see Watching).
- Admin gate added to cookbook routes (#4554, #5235 — security).
- Windows: orphaned processes after cancelled download (#3995, open); serve-button UX confusion (#3594/#4510, open).

### Other notable

- **1.0.2 release regression (Jul 11-12)**: a `NameError: _explicit_web_intent is not defined` shipped in chat routing and broke chat for all users (duplicate reports #5291/#5297/#5305/#5310, #5424) — patched same day (#5290, #5313, #5469).
- Route-layer reorganization continues: contacts (#5227), history (#5090), memory (#5007), research (#4975) split into subpackages.
- Multi-arch GHCR image publishing added (#3423); CI security gate added (#3498).

### External discussion

- No new HN front-page moment; the May 31 launch thread (243pts/105 comments) remains the high-water mark.
- ~77k stars / 10k forks by Jun 24, ~1,400 open issues — growth continues but issue backlog is building.
- Press coverage (XDA, Cybernews, others) settled into "useful, but less battle-tested than Open WebUI"; recurring criticism that AI-assisted ("vibecoded") code shipped with security gaps in Agent Mode — consistent with the IDOR/SSRF cleanup above.
- No CVE/advisory, funding, or acquisition news.

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
