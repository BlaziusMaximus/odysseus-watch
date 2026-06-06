# odysseus-watch digest
<!-- pewdiepie-archdaemon/odysseus -->

**Last reviewed:** 2026-06-06  
**Last reviewed SHA:** `b03d934` (fix: restore backup import after skills migration)

---

## Watching

| Item | Why |
|------|-----|
| #3144 | Task scheduler — event-triggered tasks may be skipped (next_run validation bug) |
| #3139 | PR: Fix logical bugs in event bus and bulk session deletion |
| #3136 | Deep-research — /api/research/status returns 404 |
| #3100 | Cookbook — Windows local download + serve fixes (PR open) |
| #3158 | Security — deny agent file tools access to credential/session stores (PR open) |
| #3150 | Security — Harden API token route capabilities (PR open) |
| #3148 | Security — gallery null-owner read bypass when auth enabled |
| #236  | Security — encrypted-at-rest secrets via SOPS (long-running PR) |
| #605  | Architecture v3 proposal — restructures scheduler, cookbook, deep-research |

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
