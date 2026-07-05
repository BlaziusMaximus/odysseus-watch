# odysseus-watch digest
<!-- pewdiepie-archdaemon/odysseus -->

**Last reviewed:** 2026-07-05  
**Last reviewed SHA:** `2826dcf` (fix(tasks): gate cookbook serve task execution, #5235) — tip of `dev`, the active integration branch

---

## Watching

| Item | Why |
|------|-----|
| #3136 | Deep-research — /api/research/status returns 404 (fix now in PR #3541) |
| #3100 | Cookbook — Windows local download + serve fixes (PR open, near merge-ready) |
| #3158 | Security — deny agent file tools access to credential/session stores (PR open, blocked on review) |
| #3150 | Security — Harden API token route capabilities (draft PR, not yet merged) |
| #3148 | Security — gallery null-owner read bypass when auth enabled (still no PR) |
| #236  | Security — encrypted-at-rest secrets via SOPS (long-running PR, needs rebase) |
| #605  | Architecture v3 proposal — restructures scheduler, cookbook, deep-research (routes/ split now underway) |
| #3541 | New: PR adding the missing /api/research/status and /api/chat/stream_status endpoints — fixes #3136 |

*(#3144 and #3139 dropped — resolved, see below.)*

---

## 2026-07-05

> ~1 month since last review. This is a genuinely huge volume — **907 commits, 509 merged PRs** on `dev` since `b03d934`. This entry is curated to your priorities, not exhaustive; a full changelog isn't useful here.

### Watching list — status changes

- **RESOLVED** #3144 / **MERGED** #3139 (Jun 7) — event-triggered tasks no longer skipped; bulk session deletion now respects starred protection and reports accurate counts. Closed the loop from last digest.
- #3136 still open, but **PR #3541 opened** to add the missing `/api/research/status` and `/api/chat/stream_status` endpoints — worth reviewing/merging.
- #3100 (Windows Cookbook) still open but close: synced with `dev`, multiple contributors confirm it works locally with GPU acceleration.
- #3158 (deny agent tools access to credential stores) still open — reviewer wants exact-path matching (not basename) plus symlink-expansion guards before merge; PR currently has conflicts.
- #3150 (API token route hardening) still a draft, not merged.
- #3148 (gallery null-owner read bypass) **still open, no PR** — note: PR #4981 "harden gallery endpoint URL handling" (merged Jun 30) is a *different* gallery fix (SSRF/host-validation, fixes #4980), not this one. The owner-bypass bug is still live.
- #236 (SOPS secrets) still open, reported `DIRTY/CONFLICTING`, needs rebase onto `dev`.
- #605 (Architecture v3) still open as a proposal, but concretely progressing: routes are actively being split into subpackages (`routes/research/`, `routes/gallery/`, `routes/contacts/`) — the restructuring it called for is happening piecemeal via merged PRs even though the tracking issue itself sits open.

### Security (top of the pile — a security-fix wave, not one fix)

- **SSRF / DNS-rebinding cluster (Jul 2–4):** webhook ntfy sender (#5142), webhook delivery pinned to SSRF-validated IP (#5147), integration `api_call` URL validation (#5145), httpx connections pinned to resolved IP (#704) — a coordinated hardening pass across every outbound-request path.
- **ReDoS fixes (Jun 27–28):** verdict-prose/continuation matchers (#4943), XML and tool-call-args parsers (#4941).
- **New feature, new leaks:** Odysseus shipped **Email + Contacts** as a feature this window, and it immediately produced a string of cross-tenant owner-scoping bugs, all since patched: #5238, #5234, #5195, #2996, #3006, #3614, #3397, #2057. Pattern worth watching if you touch this code — new surface area keeps reintroducing the same owner-scope mistake.
- **Sensitive-file deny-list hardening:** case-insensitive matching fixed for grep/rg tool exclusions (#5013, #5097, #5189) — same threat model as your watched #3158, landed via a separate mechanism.
- Docker host socket is now opt-in, not default-exposed (#4902).
- Cookbook's Ollama runner no longer executes the install one-liner (#3926) — closed a code-execution vector.
- Codex cookbook routes now gated behind an admin check for cookie sessions (#4554).
- Credential-bearing URLs and PII now redacted from logs (#4750).

### Task scheduler

- #4878 (Jun 27): scheduled-task time moved into a user-role message to preserve prompt cache.
- #4398: scheduled task agents now get shell/file tools by default — a capability expansion, not just a fix; relevant given #3158/#3148 above.
- #5235 (Jul 5, today): gate cookbook serve task execution — security fix in the scheduler path.

### Deep-research

- #4975: research routes moved into `routes/research/` subpackage (part of the v3 restructure).
- #4784: improved uploaded-document retrieval and deep-research reuse.
- #4006: Discuss chats now stay grounded on their source report.
- #4986 / #5129: research file-path confinement hardened, made CodeQL-clean.
- Still nothing merged yet for #3136 itself — see PR #3541 above.

### Cookbook

- #3100 (your watched item) still the big one — near merge.
- #4803: serve-command validation now accepts `$(find)` subshells without opening injection.
- #4811: correctly recognizes runtime `--user` installs.
- #4760: model launch only blocks on real port collisions.
- #3975: local Windows now correctly treated as Windows for serve commands.

### Other notable

- No releases published yet — still tag-less, all activity is on `dev`.
- External: project has crossed **76k+ GitHub stars** (up from 58k at launch). Coverage has turned more critical — pieces calling it "vibecoded" (AI-written, no formal security audit) are circulating alongside a detailed vulnerability write-up from security researcher Jamieson O'Reilly. The steady drip of owner-scoping/SSRF fixes above is consistent with that narrative rather than contradicting it.

### External discussion

- Continued mainstream coverage (DEV Community, XDA, SourceForge mirror) — reception still broadly positive but now paired with "no formal audit yet" caveats.
- No major new controversy beyond the general "vibecoded, fast security churn" narrative; original HN thread from launch week is the reference point, nothing bigger has superseded it.

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
