# odysseus-watch digest
<!-- pewdiepie-archdaemon/odysseus -->

**Last reviewed:** 2026-06-14  
**Last reviewed SHA:** `9d7a3d6` (test(memory): cover owner isolation for memory search)

---

## Watching

| Item | Why |
|------|-----|
| #3136 | Deep-research — /api/research/status returns 404 (still open; no fix landed) |
| #3100 | Cookbook — Windows local download + serve fixes (PR still open) |
| #3158 | Security — deny agent file tools access to credential/session stores (PR still open) |
| #3150 | Security — Harden API token route capabilities (PR still open; #3899 is a partial fix) |
| #3148 | Security — gallery null-owner read bypass; now spawned 5-issue auth-disabled owner series (#4200–4204) |
| #236  | Security — encrypted-at-rest secrets via SOPS (long-running PR, still open) |
| #605  | Architecture v3 proposal — had activity today 2026-06-14 |
| #4192 | Security NEW — Fernet key created world-readable (0o644); any local user can read all provider credentials |
| #4190 | Security NEW — shell injection in Codex cookbook SSH bridge (unquoted remoteHost/sshPort → subprocess) |
| #4189 | Security NEW — SSRF in gallery image proxy (outbound URL check skipped on response-derived URL) |
| #4163 | Task scheduler NEW — tasks runner blocked from bash/python tools even as admin |

---

## 2026-06-14

> 8 days since bootstrap. ~203 PRs merged; project at 69.6k stars (+11.6k). Security researchers are actively filing issues — four new high-severity reports on 2026-06-13. **Update recommended** for anyone self-hosting.

### Watching list changes

- **#3139 MERGED** 2026-06-07 — fixed both halves of #3144 (event bus `next_run` skip + starred session bypass in bulk delete). **Both closed.** ✓
- **#3141 CLOSED** — Ollama Windows tool-calling/memory regression resolved. ✓
- **#3150 partial** — #3899 merged (owner check on token update/delete routes), but full hardening PR still open.
- **#3148 active** — spawned a 5-issue "Default/Local owner" series (#4200–4204) filed 2026-06-14; structural fix in progress but nothing merged yet.
- **#605 active** — Architecture v3 proposal updated today; watch before touching scheduler/cookbook/deep-research.
- #3136, #3100, #3158, #236 — all still open, no change in merge status.

### Security — new issues (act on these)

- **#4192 open** `fix(security)`: Fernet key `data/.key` created 0o644 — decrypts every provider credential and webhook secret; any local account can exfiltrate it. **High.**
- **#4190 open** `fix(security)`: Codex cookbook SSH bridge interpolates `remoteHost`/`sshPort` into `create_subprocess_shell` without quoting — shell injection via stored cookbook config. **High.**
- **#4189 open** `fix(security)`: gallery inpaint/harmonize proxies fetch the upstream-response URL server-side without running `check_outbound_url` — SSRF. **Medium-high.**
- **#4193 open** `fix(security)`: RAG delete route lacks path-confinement that the add route applies — path traversal parity gap. **Medium.**
- **#4194 open** `fix(security)`: `owner_is_admin_or_single_user()` doesn't honor `AUTH_ENABLED=false` after setup writes an auth store — agent tools silently locked out in no-auth installs. **Medium.**

### Security — merged since 2026-06-06

- **#3489 MERGED** 2026-06-08 — `fix(security)`: fail closed in `/api/models` auth gate on unexpected errors.
- **#3899 MERGED** 2026-06-11 — `fix(tokens)`: owner check on token update and delete routes (partial #3150 fix).
- **#1314 MERGED** 2026-06-11 — `ci(security)`: full scanning suite — gitleaks, CodeQL (Python+JS), actionlint/zizmor, Trivy image scan, dependabot. Big infra addition.
- **#3707 MERGED** — `fix(settings)`: scrub camelCase secret keys from settings output.
- **#3807 MERGED** — `fix(memory)`: validate session owner on manual memory add.
- **#3724 MERGED** — `fix(email)`: scope learned sender signatures by owner.
- **#3733 MERGED** — `fix(auth)`: fail closed when deleting user tokens fails.
- **#3727 MERGED** — `fix(auth)`: drop reserved usernames from auth config on load.
- **#3622 MERGED** — `fix(sessions)`: apply `owner_filter` in `list_sessions` even when auth is disabled (closes a hole that fed #3148-class issues).

### Task scheduler

- **#3640 MERGED** — `fix(tasks)`: read `Memory.text` correctly in `classify_events` personal context.
- **#3859 MERGED** — `fix(webhooks)`: keep references to in-flight delivery tasks (prevents GC dropping live coroutines).
- **#4163 open** — tasks runner blocked from bash/python tools regardless of admin status — new regression.
- **#4165 open** — agent loop guards can silently terminate scheduled runs with no user-facing signal.

### Deep-research

- **#1931 MERGED** — `fix`: `odysseus-research --status complete` never matched because the writer stores `"done"` not `"complete"` — the CLI status flag was always silently broken.
- **#3637 MERGED** — `fix(research)`: stop rescanning the research directory on every status poll (N×disk-scan eliminated).
- **#4160 open** — non-English agent queries never trigger `web_search`/`web_fetch` — tool-selection intent classifier is English-only; silent no-op for all other languages.
- **#3136 still open** — `/api/research/status` and `/api/chat/stream_status` return 404; last updated 2026-06-08, no fix merged.

### Cookbook

- **#3143 MERGED** — `chore`: switch `duckduckgo-search` → `ddgs` (deprecated package; also fixes web-search in deep-research and agents).
- **#1538 MERGED** — expand cookbook error output tail 12→50 lines (makes failures actually debuggable).
- **#3100 still open** — Windows bash routing / CUDA llama-server / busy-endpoint watchdog PR; no merge yet.
- **#4131 open** — `cookbook_serve_lifecycle` silently drops UI state on concurrent writes and swallows malformed JSON.

### Agent sandboxing (notable new feature)

- **#3665 MERGED** — `feat(agent)`: confine agent file/shell tools to a user-selectable workspace directory. First real sandboxing boundary in the tool layer — directly mitigates the class of risk #3158 addresses even while that PR sits open.

### Other notable merges

- **#3117 MERGED** — `fix`: omit `temperature` for Opus 4.7+ on the native Anthropic path (API would reject it).
- **#3945 MERGED** — `fix(llm)`: stop sending llama.cpp slot-affinity fields to cloud providers (was causing 422 on Mistral/Gemini OpenAI-compat endpoints).
- **#3785 MERGED** — `fix(search)`: `web_fetch` now reads plain-text, Markdown, and JSON URLs (was silently skipping them).
- **#4229 open** — Proposal: Plugin system to de-escalate the PR backlog and keep core opinionated. Worth reading alongside #605.
- **#4121 open** — Any settings save materializes the 6000-token `agent_input_token_budget` default into `settings.json`, permanently capping long-context models at 6k tokens. Quiet footgun.
- **#4141 open** — `manage_calendar` creates events only in local SQLite, never writes back to CalDAV/Google Calendar when synced.

### External

- Stars: ~69.6k (from ~58k at last review — +11.6k in 8 days). 8.8k forks, 226 contributors.
- Security researchers filed four high/medium issues on a single day (2026-06-13): Fernet key permissions, shell injection, SSRF, path traversal. Indicates outside audit attention — expect more.
- Issue #187 ("not safe to expose publicly in default Docker config") resurfacing in community discussion; relevant if you or users expose Odysseus on a public network.
- [DeepWiki page for Cookbook + model management](https://deepwiki.com/pewdiepie-archdaemon/odysseus/6-cookbook-and-model-management) going live — contributor onboarding surface improving.

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
