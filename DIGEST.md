# odysseus-watch digest
<!-- pewdiepie-archdaemon/odysseus -->

**Last reviewed:** 2026-06-07  
**Last reviewed SHA:** `b22c2b2` (fix(hwfit): detect NVIDIA GPU on WSL and other minimal-PATH environments)

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

## 2026-06-07 — 44 commits, ~30 PRs merged, security sweep

### Watching list status changes

- ✅ **PR #3139 MERGED** (`2a6921a`, 15:08 UTC) — fixes logical bugs in event bus and bulk session deletion; this is the fix for **#3144** (next_run validation + starred-session bypass). Both should now be closed.
- ⚠️ **#3148** (gallery null-owner bypass) — Vykos landed `67aeea4` "Scope gallery image endpoints by owner" (#3001); a duplicate PR #3286 was explicitly closed as dup. Likely addressed but watch for formal close.
- ⚠️ **#3136** (deep-research 404) — two related fixes merged (PR #2189: preserve partial findings; PR #2229: fix IndexError in category parser) but the `/api/research/status` 404 itself has no direct fix yet. Still open.
- 🔴 **PR #3158, #3150, #3100, #236** — no activity today; still open.
- 📝 **#605** (Arch v3 proposal) — saw fresh activity today; still open.

### Security — NEW open issues (act on these)

- **#3200 OPEN** DNS rebinding SSRF in `services/search/content.py` `_get_public_url` — attacker can route fetch requests to private network hosts. No fix yet. **Critical for network-accessible deployments.**
- **#3201 OPEN** Unauthenticated bash/python tool access before first admin is created — `owner_is_admin_or_single_user()` grants full tool access during the setup window. Exploitable on first-run.
- **#3227 OPEN** `app_api` token can reach shell-adjacent Cookbook control routes — partial: `a3784da` blocked direct shell routes, but Cookbook mutation endpoints remain reachable.
- **#3229 OPEN** `allow_bash` flag only read from form data, ignored for JSON-body API requests — bash/web tools silently disabled for all API callers; also a correctness regression.

### Security — fixes merged today

- `a3cb15d` **fix(agent): enforce guide-only tool policy** (#3088) — closes **#3068** (agent executing shell despite guide-only instruction).
- `a3784da` **fix: block app_api access to shell routes** (#3225) — closes **#3224**.
- `cbbb41d` **fix: avoid double bcrypt on login** (#3236) — closes **#3230** (login bug, not a direct attack vector but worth noting).
- `f78539b` **fix(caldav): disable redirects on sync/write-back DAVClient** (#2663) — SSRF via CalDAV redirect following.
- `95c2dca` **fix(security): add HSTS and Permissions-Policy headers** (#3081) — hardens TLS enforcement and feature access.
- Vykos sweep (~8 commits): owner-scoped gallery (#3001), gallery albums (#3004), doc session links (#3005), task chains (#3006), model helpers (#3007), vision model resolution (#3009), RAG upload privilege (#2990), auxiliary LLM endpoints (#2996). Multi-user data-isolation tightening across the board.
- `000932a` **fix(auth): gate api tokens from user routes** (#2992) — part of the #3150 capability-hardening work; PR #3150 itself still open.

### Task scheduler

- **PR #3139 MERGED** — see watching list above; resolves #3144.
- `5d3e3c7` feat(tasks): assign folder='Tasks' at creation + backfill migration — new UX polish, not a bug fix.
- `7b4e6c4` Enforce task chain owner scope — multi-user isolation fix.

### Deep research

- **PR #2189 MERGED** — preserve partial findings when a non-timeout error aborts a research run (data no longer lost on intermittent failures).
- **PR #2229 MERGED** — fix double `split()` / IndexError in category parser.
- **#3136** (`/api/research/status` 404) — still open; no direct fix merged.

### Cookbook

- **PR #3306 MERGED** (`b22c2b2`) — NVIDIA GPU detection on WSL / minimal-PATH envs; closes #3307.
- **PR #2657 MERGED** — APFEL platform support (Apple Silicon dependency set + models in catalog).
- `573d431` **MERGED** — don't infer server OS from browser user-agent (#3221 closed) — llama.cpp was emitting Windows-only commands on Mac/Linux servers when accessed from a Windows browser. Significant bug.
- **#3100** (Windows local download/CUDA fixes) — no activity, still open.

### Other notable

- **PR #3183 MERGED** (`adbcb37`) — fix(agent): don't abort legitimate tool batches as runaway loops — agent was misidentifying valid multi-tool responses as infinite loops. Reliability improvement.
- `04d6a5c` **MERGED** — Fix CORS preflight 401'd by AuthMiddleware before CORS headers applied — broke cross-origin requests with `AUTH_ENABLED=true`.
- **PR #3301 open** — feat(auth): generic OIDC SSO — big feature, enables enterprise SSO with any OIDC provider.
- **PR #3310 open** / **#3309 open** — Electron desktop wrapper; community demand is high.
- **#3254 open** — AI hallucinates dates in tool calls (June 10 requested → Jan 15 executed) — prompt/date-context bug.
- **#3229 open** — bash/web tools silently disabled for API JSON callers (see security section too; also a UX bug).
- **#3266/#3265** open — refactor streamed agent turn out of `agent_loop.py`; relates to the Architecture v3 direction (#605).

### External

- Project available one-click via [Pinokio](https://beta.pinokio.co/apps/github-com-pewdiepie-archdaemon-odysseus); broadening reach.
- Coverage in Cybernews, DEV.to, and tech blogs (launch hype still running). 44k+ stars as of June 4; expect issue volume to stay high.

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
