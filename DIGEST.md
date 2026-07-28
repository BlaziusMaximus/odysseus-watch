# odysseus-watch digest
<!-- odysseus-dev/odysseus (was pewdiepie-archdaemon/odysseus — org transfer 2026-07-20) -->

**Last reviewed:** 2026-07-28
**Last reviewed SHA:** `01790c2` (fix(mcp): keep built-in servers on SDK v1 (#5820), branch `dev`)
**Prior review:** 2026-06-06 @ `b03d934`

---

## TL;DR for Adam

- **The repo moved.** `pewdiepie-archdaemon/odysseus` → **`odysseus-dev/odysseus`** (org transfer, ~2026-07-20). The old URL/API 301-redirects, so nothing breaks yet, but the tracker now points at the new canonical name.
- **Still exploding.** ~62.3k stars (was 58k). ~1,018 commits on `dev` since the last review; issue/PR numbering jumped from ~3,150 to ~5,820 (~2,700 new items in ~7.5 weeks). 991 open issues+PRs.
- **Ships from `dev`, not releases.** Still no GitHub Releases/tags. Internal version bumped to **1.0.2** (chore commits, 2026-07-11 / 2026-07-18). Maintainers commit directly to `dev`; most outside PRs get cherry-picked or closed rather than merged.
- **Mass PR purge on 2026-07-23.** The owner (`pewdiepie-archdaemon`) bulk-closed **93 PRs in one pass**, alongside a curated `d8a2059 "Merge verified Odysseus fixes"`. Several security PRs we were watching (#3158, #3150, #236) were swept in this — **closed unmerged**. Good news: the underlying concerns were largely fixed via *direct commits* instead (see Security). Bad news: any self-hoster who was waiting on those specific PRs should re-check against `dev`, not the PR.
- **Nothing here is an emergency for our box.** We run Odysseus as the AI-agent platform; the security fixes below have already landed on `dev`. If we ever pin a specific commit, pin at/after the fixes noted below.

---

## Watching (carry-forward)

| Item | State now | Note |
|------|-----------|------|
| #3144 scheduler skip / bulk-delete bypass | **CLOSED** | Fixed by #3139 (merged 2026-06-07) |
| #3139 PR: event-bus + bulk-session-deletion fix | **MERGED** 06-07 | resolved |
| #3143 PR: ddgs swap (duckduckgo-search deprecated) | **MERGED** 06-10 | resolved (#3142 closed) |
| #3068 agent runs shell despite guide-only | **CLOSED** | resolved 06-07 |
| #3141 Ollama tool-calling regression (Win) | **CLOSED** | resolved 06-07 |
| #3136 research `/api/research/status` 404 | **OPEN** | still open (upd 07-09); research status polling |
| #3148 gallery null-owner read bypass (auth on) | **OPEN** | **stale** (upd 06-06); unauth read of null-owner gallery items — verify against `dev` |
| #3100 PR: Cookbook Windows fixes | **CLOSED unmerged** | swept in 07-23 purge; Windows Cookbook may still be rough |
| #3158 PR: deny agent tools access to cred/session stores | **CLOSED unmerged** | swept 07-23; see Security below |
| #3150 PR: harden API token route caps | **CLOSED unmerged** | concern addressed by merged #4054 (see Security) |
| #236 PR: SOPS encrypted-at-rest secrets | **CLOSED unmerged** | swept 07-23; not landed |
| #3140 PR: Gemini-native REST | **CLOSED unmerged** | swept 07-23 |
| #605 Architecture v3 proposal | **OPEN** | being implemented incrementally — `routes/*` subpackage refactors are landing on `dev` |

---

## 2026-07-28 — review (delta since 2026-06-06)

### Security — landed on `dev` (good)
Even though the security *PRs* got closed in the purge, the *fixes* mostly shipped as direct commits:

- **`#4054` MERGED (06-15)** `fix(api): attribute bearer-token actions to the token owner on owner-scoped routes` — this is effectively the fix behind watched PR **#3150** (API token capability hardening). Bearer (`ody_...`) actions are now attributed/scoped to the token owner.
- **`c7b3b84` (#4420)** `fix(db): restrict data/app.db to 0600` — DB file no longer world/group-readable.
- **`93f97e7` (#5651)** `fix(email): hide OAuth tokens from config response` + **`16b04a9` (#5648)** verify mailbox identity on OAuth reconnect — the new email/OAuth stack got token-leak + identity hardening.
- **`45b6771` (#5261)** `fix(skills): block private SSRF targets and revalidate redirects in importer` and **`b3f8b77` (#5474)** `fix(url-safety): reject RFC 6598 shared address space in strict mode` — SSRF hardening in the skills importer / URL safety layer.
- **`0f259ea`** `fix(docker): bump Docker CLI to a patched release` — CVE bump.
- **`b07c1e3` (#5250)** `ci: add CodeQL advanced setup to scan PRs before merge` — they added static security scanning to CI.

### Security — still open / not landed
- **#3158** (deny agent file-tools access to Odysseus' own credential/session stores) — **closed unmerged**; no obvious direct-commit replacement found. Agents may still be able to read auth/session files off disk. Worth a spot-check on our deployment.
- **#236** SOPS encrypted-at-rest secrets — **not landed**; secrets still at rest in plaintext config unless you roll your own.
- **#3148** gallery null-owner read bypass — **open & stale**.
- **#5815** (new, 07-28) *Make agent execution authority deterministic across untrusted context and prompt content* — new agent-authority / prompt-injection hardening request; open, no comments yet.

### What shipped / themes (last ~7.5 weeks)
- **Email + OAuth push (07-22):** SMTP OAuth accounts, Google email OAuth through Compose, mailbox identity verification, token hiding. New first-class email/reminders capability.
- **Architecture v3 (#605) in motion:** `routes/` being carved into per-domain subpackages (`compare`, `admin_wipe`, `cleanup`, `note`, …). The big refactor proposal is being executed incrementally on `dev`.
- **Models capability schema (#2739):** `feat(models): define capability schema and readers` — groundwork for per-model capability gating.
- **MCP hardening:** concurrent server connections w/ timeout handling, then pin built-in MCP servers to SDK v1 (`#5820`, 07-28) — MCP has been churny.
- **RAG:** skip hidden/junk dirs when indexing (`#5633`).

### Current open-issue flavor (most recently active)
- **#5782** (4 comments) Scheduled tasks interrupted by `/api/tasks/runs/recent` polling and left in a bad state — **task-scheduler reliability again** (same family as the old #3144).
- **#5757** `manage_tasks`: model writes tool-call JSON as plain text but never executes it — agent tool-call reliability.
- **#5814** API model providers show empty/offline in picker despite healthy backend.
- **#5600** `tts_cache` grows without bound (no cap/TTL/eviction) — resource leak.
- **#5792** Sent/Trash/Archive/Junk folders not found for localized (non-English) IMAP.
- **#5788** Improve HUD and voice controls.

### External
- GitHub search API refuses to index-search this repo (422 — too large / rename), so keyword issue-search is unavailable; digest built from the issues/pulls/commits list APIs instead.

---

*Tracker: `BlaziusMaximus/odysseus-watch` · routine = migrated-routines task #2 (Odysseus GitHub tracker). Next review: on next scheduled run (~weekly).*
