# odysseus-watch digest
<!-- odysseus-dev/odysseus (was pewdiepie-archdaemon/odysseus — org transfer 2026-07-20) -->

**Last reviewed:** 2026-08-09
**Last reviewed SHA:** `42da399` (fix(email): route summaries through shared LLM adapter (#5841), branch `dev`)
**Prior review:** 2026-07-28 @ `01790c2`

---

## TL;DR for Adam

- **Quiet-ish cycle.** 32 commits on `dev` since 07-28 (~12 days) — much slower than the prior 1,018-commit stretch. Issue numbering now at #5952, 954 open issues, 121 open PRs.
- **Star count dipped then recovered — not a real signal.** Shows ~85.0k now (was 62.3k), but Discussion #5754 confirms a maintainer accidentally wiped ~20k stars and old PR history during a 07-25 repo cleanup; the number reflects recovery, not organic doubling.
- **#5600 (tts_cache unbounded growth) is fixed** — the one concrete win from our watch list this cycle.
- **#5815 (deterministic agent execution authority)** — the prompt-injection/agent-authority hardening issue flagged as brand-new last cycle now has four draft PRs against it (#5817–#5821). Worth a close look next review; this is the most architecturally significant thing in flight.
- **Nothing here is an emergency for our box.**

---

## Watching (carry-forward)

| Item | State now | Note |
|------|-----------|------|
| #3136 research `/api/research/status` 404 | **OPEN** | unchanged since 07-09 |
| #3148 gallery null-owner read bypass (auth on) | **OPEN, stale** | no movement; #5946 (new) reports a similar gallery-auth inconsistency — same neighborhood |
| #3158 PR: deny agent tools access to cred/session stores | **CLOSED unmerged** | no confirmed direct-commit replacement found; still worth a spot-check on our deployment |
| #236 PR: SOPS encrypted-at-rest secrets | **CLOSED unmerged** | not landed |
| #605 Architecture v3 proposal | **OPEN, in progress** | `routes/*` subpackage refactors continue landing (vault, webhook, document, search since 07-28) |
| #5815 deterministic agent execution authority | **OPEN, active** | 4 draft PRs now implementing it (#5817–#5821) — capability registry, sandboxing, thread authority modes, provenance |
| #5782 scheduler interrupted by status polling | **OPEN** | couldn't get a clean re-check this cycle (fetch issue) — re-verify next time |
| #5757 `manage_tasks` writes tool-call JSON but doesn't execute | **OPEN** | unchanged |
| #5946 gallery provider image actions — inconsistent auth (new) | **OPEN** | added to watch; echoes #3148 |
| #5950 provider detection accepts suffix-lookalike hostnames (new) | **OPEN** | added to watch; spoofing-flavored |

*Dropped from the table (resolved 2+ cycles ago, no longer active): #3144, #3139, #3143, #3068, #3141, #3100, #3150, #3140. #5600 resolved this cycle, see below.*

---

## 2026-08-09 — review (delta since 2026-07-28)

### Watching-list movement
- **#5600 tts_cache unbounded growth — CLOSED, fixed.** Merged PR **#5822** "feat(tts): implement TTS cache size limit and eviction policy" (07-29) adds `ODYSSEUS_TTS_CACHE_MAX_BYTES` + LRU eviction.
- **#5815 agent execution authority — now has real movement.** Four draft PRs (#5817–#5821) appeared implementing the proposal in slices. Not merged yet; flagging for next cycle since it touches prompt-injection/tool-authority boundaries directly relevant to how we run agents.
- **#3158 (deny agent access to cred/session stores) — still unresolved.** Confirmed still closed/unmerged, no new activity. A June 19 commit by contributor Kukkerem ("harden agent path confinement and prompt-injection surface") may partially cover this but scope overlap is unconfirmed — treat as still open on our end.

### Security — new since 07-28
- **#5727 → merged (`9d68618`, 08-04):** pin `api_call` to the SSRF-validated IP — closes a DNS-rebind/TOCTOU gap between validation and request.
- **#5856 → merged (`20e7fc0`, 08-04):** require explicit `manage_skills` action/permission — tightens a capability check.
- **New reports, not yet fixed (opened 08-08, batch #5941–#5952):** **#5950** provider-hostname spoofing (suffix lookalikes accepted), **#5946** gallery provider-image auth inconsistency (same family as #3148), **#5952** assistant/threat docs drifted from actual runtime behavior.

### Scheduler / deep-research / Cookbook
- No movement. #3136 (research status 404), #5782 (scheduler polling interruption), #5757 (manage_tasks not executing) all unchanged. Cookbook: no activity since the 07-23 purge closed #3100.

### Other notable
- Architecture v3 (#605) continues: `routes/*` split landed for search (07-28), vault (08-03), webhook (08-03), document (08-04).
- Routine UX/bug fixes 08-06–08-08: memory-store overwrite guard, arrow-key eating unsent prompt, session URL hash regression fix, stale skills panel on reopen, startup tips rotation, email summaries routed through shared LLM adapter.
- Still no GitHub Releases/tags published.

### External
- No significant new external coverage (HN/Reddit/press) found dated after 07-28 — only pre-baseline launch coverage and an unrelated June SSRF writeup turned up in search.
- Note: this cycle's GitHub data came via public web pages, not the API — the connector used for this tracker is currently scoped to this repo only and can't reach `odysseus-dev/odysseus` directly. Numbers should be treated as best-effort; worth re-checking the connector scope if that matters going forward.

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
