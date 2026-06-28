# odysseus-watch digest
<!-- pewdiepie-archdaemon/odysseus -->

**Last reviewed:** 2026-06-28  
**Last reviewed SHA:** `927b1f7` (fix(llm): normalize OpenAI-compatible chat URLs)

---

## Watching

| Item | Why |
|------|-----|
| #3158 | Security — deny agent file tools access to credential/session stores (stalled: merge conflict) |
| #3150 | Security — Harden API token route capabilities (stalled: draft, no activity) |
| #3148 | Security — gallery null-owner read bypass when auth enabled |
| #3068 | Security — agent executes shell commands despite guide-only / no-tools instruction |
| #236  | Security — encrypted-at-rest secrets via SOPS (active, 22 comments) |
| #3136 | Deep-research — /api/research/status returns 404 (partially addressed, monitor) |
| #3100 | Cookbook — Windows local download + serve fixes (companion #4894 open) |
| #4986 | Security — research result file paths not confined (new, path traversal risk) |
| #3566 | Security — Codex audit remediation, 16 issues (stalled: merge conflict) |
| #605  | Architecture v3 proposal — ADRs in progress; routes refactor underway |

---

## 2026-06-28

### Watching — status changes

- **RESOLVED #3144 + #3139** — event bus next_run skip bug + bulk-delete starred-session bypass: both closed/merged 2026-06-07. Good news; both bugs fixed within a day of the last review.
- **#3158 STALLED** — "needs work" + merge conflict labels; no maintainer activity since June 12. The credential/session store read-gap (allows prompt injection → token exfiltration → account takeover) is still unpatched. New #4844 (vault contract) and #4986 (research path confinement) are moving in the same direction but don't close this.
- **#3150 STALLED** — still a draft; zero activity since it opened June 6.
- **#236 (SOPS) ACTIVE** — 22 comments, last updated June 26; still open. Companion PR #4254 (align `.gitignore`/`.dockerignore` for SOPS artifacts) was **merged June 15** — incremental progress.
- **#3136 (research 404) PARTIAL** — research domain moved to `routes/research/` subpackage (#4975 merged) and deep-research reuse improved (#4784 merged); original issue state unconfirmed, monitor.
- **#3148 (gallery null-owner bypass) OPEN** — new PR #4981 (harden gallery endpoint URL handling) is open and related; issue itself unchanged.
- **#605 (Architecture v3) ACTIVE** — 102 comments, updated June 27; being formalized into ADRs via PRs #987 + #1013. Gallery routes (#4903) and research routes (#4975) already moved — v3 is shipping in pieces.

### Security (act on these first)

- **MERGED — ReDoS sweep** (June 26–28): Four regex denial-of-service bugs found and patched in rapid succession. All in LLM output-parsing paths — an attacker who can steer model output could hang the process. **Update now if self-hosting.**
  - #4704: LLM-output tool/think parsers
  - #4877: agent_loop `<think>` tag stripper
  - #4941: XML and args tool-call parsers
  - #4943: verdict-prose and continuation matchers
- **MERGED #4554** — codex cookbook routes now gated behind admin check for cookie sessions; unauthenticated users could previously access codex endpoints.
- **MERGED #3584** — untrusted LLM context isolated from visible user prompts (companion to the deep-research sandbox fix last digest).
- **MERGED #4447** — config lock added around auth migration methods (race condition).
- **NEW OPEN #4986** — research result file paths not confined; path traversal risk in research routes. PR open, not yet merged.
- **NEW OPEN #4981** — harden gallery endpoint URL handling; related to watched #3148.
- **NEW OPEN #4965** — wrap email style, integration, and MCP descriptions as untrusted context.
- **STALLED #3158** — credential/session store read gap still unpatched (see Watching above).

### Task scheduler

- **MERGED #4878** — scheduled-task run time moved from system prompt into a user-role message, preserving the Anthropic prompt cache; previously the minute-stamped time was busting cache on every single scheduled run (issue #4850 closed). Big efficiency win for anyone running frequent scheduled tasks.
- **MERGED #4812** — MCP builtin startup tasks now retained, npx probe reaped on cancel.
- **MERGED #4545** — cookbook serve tasks now preserve `sshPort`/`platform` metadata across server restarts (issue #4544 closed).

### Deep-research

- **MERGED #4784** — improve uploaded document retrieval and deep-research result reuse across sessions.
- **MERGED #4975** — research domain refactored into `routes/research/` subpackage (Architecture v3 progress).
- **OPEN #4986** — new: research result file paths not confined (path traversal, see Security above).

### Cookbook

- **MERGED #4803** — cookbook serve now accepts `$(find)` subshells in command validation.
- **MERGED #4811** — runtime `--user` installs now replay user-site `.pth` hooks; fixes packages like realesrgan/CUDA tools that install but aren't recognized (issue #4810 closed).
- **MERGED #4834** — self-host troubleshooting cookbook added to docs.
- **OPEN #4894** — new: activate local Windows venv in bash runner (companion to watched #3100 Windows fixes).
- **OPEN #4535** — new: proxy ollama serve to Docker host when running in Docker.
- **OPEN #4921** — replace deprecated `hf_transfer` with `hf_xet` in cookbook routes.

### Other notable

- **OPEN #4984** — full-packaged Electron desktop app with bundled Python server; first serious desktop-native path. Significant for non-technical users.
- **OPEN #4959** — Design Maker: dedicated Claude-Design-style surface. New major feature.
- **OPEN #4076** — unified Markdown model + preview-panel editor for notes. Substantial notes rework.
- **OPEN #3566** — Codex integration audit remediation: 16 issues resolved (3×P1 including removing fragile `_as_owner` state-patching, plugin zip hardened, structured audit log). Currently stalled on merge conflict — worth watching.
- **OPEN #3400** — companion mobile contract (backend-only; 19 comments, active discussion); Ed25519 signed commands, scoped bearer tokens, owner-scoped pairing.
- **OPEN #4844** — internal secrets vault contract and System configuration; moving in the direction of #236 and #3158.

### External discussion

- Project hit ~77k stars + 10k forks by mid-June. Growth steady.
- XDA Developers review: "I tried PewDiePie's open-source AI workspace, and it's weirdly great."
- Multiple tech blog write-ups (explainx.ai, ChatForest, gavinj.net, DEV community, Medium × 2). Tone: positive on local-first approach, caveats around setup complexity and agent permissions.
- Same HN thread from launch still cited; no major new HN discussion found since June 6.

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
