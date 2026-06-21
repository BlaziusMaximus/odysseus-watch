# odysseus-watch digest
<!-- pewdiepie-archdaemon/odysseus -->

**Last reviewed:** 2026-06-21  
**Last reviewed SHA:** `076e8c9` (fix(ui): escape model name in model-info popup — DOM-XSS)

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

## 2026-06-21

### Watched list — status changes

- **#605 Architecture v3**: Most active watched item — 3 open implementation PRs: **#2567** (Nix devShell), **#2568** (NixOS/nix-darwin modules, updated Jun 20), **#4540** (chatRenderer refactor, **has merge conflict**). Read before touching `background/`, scheduler, or cookbook.
- **#3144, #3136, #3148** (scheduler next_run bug, research 404, gallery null-owner bypass): No detectable new activity since Jun 6. All presumed still open. #4620 (below) is a related-but-separate new deep-research timeout report.
- **#3139, #3100, #3158, #3150, #236** (watched PRs): Not in recent merge activity; presumed still open, no merges detected.

### Security (act first)

- **MERGED `93569b1`** `fix(security)`: allowlist `manage_mcp 'add'` to close **agent-path RCE** (#4433, Jun 16) — agent could register arbitrary MCP servers via the manage_mcp tool. If self-hosting, update now.
- **MERGED `076e8c9`** `fix(ui)`: escape model name in model-info popup — **DOM-XSS** + 2 latent sinks (#4605, Jun 19) — LLM response could inject HTML via `innerHTML`; fixes issue #4604 (closed same day).
- **MERGED `1602674`** `fix(personal)`: scope RAG file delete to caller's own upload dir (#4602, Jun 19) — was using shared root; path-traversal-adjacent. Fixes issue #4603 (closed same day).
- **MERGED `7b09491`** `fix`: calendar digest **leaked every user's events** (missing owner scope) (#1925, Jun 16) — multi-user data exposure, now fixed.
- **MERGED `4e47774`** `harden(agent-loop)`: wrap non-native tool results as untrusted data (#1629, Jun 16) — tightens prompt-injection surface on external tool responses.
- **MERGED** owner/scope fixes: email MCP boundaries (`260ce8b`), memory server per-owner (`422f23f`), gallery image path confinement (`745c10e`) — all Jun 16; closes a cluster of cross-user data-access holes.
- **MERGED `396e26b`** `fix(auth)`: tie remember-me cookie lifetime to `TOKEN_TTL` (#4472, Jun 18) — sessions were outliving token expiry.
- **PR #4659 open** `feat(api)`: self-serve API token management in Settings → Account — user-facing companion to watched #3150 (admin-only token hardening).
- **Issue #3222 open** (web search): Agent executes Markdown code fences as tool calls → accidental commands + loops — safety concern, not yet fixed.

### Task scheduler

- **MERGED `dd20c2b`** `fix(tasks)`: offer shell/file tools to scheduled task agents by default (#4398, Jun 16) — agents were running without their expected tools.
- **Issue #4653 open** / **PR #4652 open** Scheduled task agents describe tool calls in prose instead of calling them — small models role-play tools instead of invoking them; PR #4652 is the proposed fix.
- **Issue #4618 open** Scheduled tasks with explicit base `endpoint_url` silently return empty response — bare URL not normalized to `/chat/completions`.

### Deep-research

- **Issue #4620 open** Deep Research times out frequently — new report, related to watched #3136 but distinct; no fix landed yet.
- **PR #4628 open** `fix(research)`: increase LLM call timeouts in research handler — prevents cold-start failures with large local models.

### Cookbook

- **MERGED** four fixes Jun 18–19: stop Windows process trees (`057ec05`), validate agent SSH targets (`1cc8a37`), pull llama.cpp from ggml-org GHCR namespace (`a52ac68`), scope "Kill vLLM" diagnosis to actual vLLM tracebacks (`e7ffc69`).
- **PR #4374 open** Auto-detect CUDA on Windows + install GPU `llama-cpp-python` wheels — extends watched #3100 scope.
- **Issue #4615 open** Cookbook detects GPU but llama-cpp still falls back to CPU.
- **PR #3515 open** Clean up stale `.incomplete` markers on download completion.

### Other notable

- **PR #4033 open** `feat`: native Google Gemini API support with payload builder + streaming — first-class Gemini alongside OpenAI-compat hosts; companion to #3140 from last digest.
- **MERGED `d795d9a`** `feat(launcher)`: portable Windows launcher (#976, Jun 16) — no-Docker Windows entry point finally lands.
- **PR #4687 open** `feat`: centralized Axios API client — replaces scattered `fetch()` calls across the frontend.
- **PR #4423 open** `refactor(tools)`: split 4,101-line `tool_implementations.py` into `src/tools/` package (Phase 0) — prerequisite for #605 restructure.
- **Issue #4541 open** Every message invalidates llama.cpp KV cache — system prompt changes each turn; significant perf impact for local model users.
- **Issue #4644 open** / **PR #4661 open** Browser OOM/freeze in long agent sessions (multi-GB RAM from tool call accumulation).
- **MERGED `a226c94`** Remove unused `@anthropic-ai/sdk` npm dependency (Jun 19).

### External discussion

- Stars: **75.3k** (was 58k at last review; +17k in ~15 days).
- Pinokio one-click install now available — lowers barrier to entry significantly.
- ChatForest published a review; DEV.to article ("the point is bigger than the hype") getting traction; AyyazTech cheap-VPS guide circulating.

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
