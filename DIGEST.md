# odysseus-watch digest
<!-- odysseus-dev/odysseus (was pewdiepie-archdaemon/odysseus — org transfer 2026-07-20) -->

**Last reviewed:** 2026-10-11
**Last reviewed SHA:** `a8c147b` (fix(reminders): retry failed note delivery through the configured channel (#6566), branch `dev`, 2026-10-07)
**Prior review:** 2026-09-16 @ `3b6c169` (⚠️ no longer exists upstream — history was rewritten, see below)

---

## TL;DR for Adam

- **Quiet, healthy cycle — no purge.** 17 commits on `dev` since the last review (`b4d1293`…`3b6c169`), 14 PRs merged, 8 closed-unmerged (mostly Dependabot + two junk PRs "Main"/"Work"). ~62 new issues opened in the window. **1,205 open** (was ~1,109 on 08-24). Stars ~**87.3k** (was ~86.1k), 887 forks (was 621 on 07-28).
- **The big story: a 1.0 release line started.** `chore(release): align dev version with 1.0.3` (#6168, 08-25) + `fix(image): prepare v1.0.3 hotfix` (#6167) + `chore(release): sync main with dev` (**#6260, merged 09-07**). Odysseus has always shipped from `dev` with no Releases/tags — a `main` sync means they're preparing to actually cut a release. Watch for a GitHub Release.
- **No slice 2 of the agent-authority series.** #5817 (external-context gate, "slice 1 of 4") landed 08-15; umbrella **#5815 is still OPEN** (last touched 08-19, no new PRs since). The follow-on security slices we were promised haven't appeared in ~5 weeks.
- **New security-relevant fix: token cache race (#6280, 09-13).** `_refresh_token_cache` atomic swap — race in token refresh, fixed with a regression test. Plus **MCP input hardening (#6215)**: `Add MCP Server` now rejects malformed `Args` instead of silently defaulting to `[]`.
- **Infra fixes that hit OUR stack:** **#6158 `fix(docker): repair app cache parent ownership`** (09-05) — a Docker app-cache parent-ownership repair, exactly the kind of Compose-cache bug that breaks our Dockerized Odysseus. Also **#6228** cache a successful-but-empty Tailscale lookup (discovery), and **#5937** psycopg2 → psycopg2-binary (Docker/dependency hygiene).
- **⚠️ Our box is still far behind.** `~/odysseus` pinned at `463713c` (2026-06-05) — now ~920+ commits behind `dev`, still missing the agent-authority gate, the SSRF/DNS-rebind hardening, the 1.0.3 release prep, and the token-cache race fix. (Local `~/odysseus` checkout not found on this box this run — the 08-24 note referenced it; worth re-checking where our Odysseus actually runs.)

---

## Watching (carry-forward)

| Item | State now | Note |
|------|-----------|------|
| #5815 agent execution authority deterministic | **OPEN (umbrella)** | Still 1 of 4 slices. No slice 2 in ~5 weeks. |
| #5600 tts_cache grows without bound | **CLOSED / FIXED** | (08-24: #5822) |
| #5782 scheduled tasks killed by status polling | **CLOSED / FIXED** | (08-24: #5789) |
| #5814 API model providers show empty in picker | **CLOSED / FIXED** | #6087/#6089 "show API models by default" landed 08-17 — re-verify passed, closing the watch item. |
| #5757 manage_tasks writes tool-call JSON as text | **OPEN** | not yet resolved |
| #5792 non-English IMAP folder names not found | **OPEN** | not yet resolved |
| #5788 improve HUD and voice controls | **OPEN** | enhancement |
| #3136 `/api/research/status` 404 | **OPEN** | stale |
| #3148 gallery null-owner read bypass (auth on) | **OPEN** | stale |
| #605 Architecture v3 proposal | **OPEN** | `routes/*` subpackage carve-out continues (task domain moved 08-17) |
| #3158 PR deny agent tools access to cred/session stores | **CLOSED unmerged** | still no replacement; #5817 gate reduces blast radius only |
| #236 PR SOPS encrypted-at-rest secrets | **CLOSED unmerged** | still not landed |

---

## 2026-10-11 — review (delta since 2026-09-16)

_Caveat: the GitHub connector is scoped to this repo only, so upstream issues/PRs/releases/stars were NOT queryable this run. Everything below is from the `dev` git history (≈442 commits since 09-13). Watching-list issue states are unverified._

- **⚠️ Upstream history was rewritten.** `main` backed up as `backup/main-before-history-cleanup-20260910`, and our old anchor `3b6c169` no longer exists on `dev`. *Why it matters:* anything of ours pinned to an old SHA won't fetch/rebase cleanly; re-anchor on the new history. No tags/Releases yet (1.0.3 line still unreleased; `main` last synced 09-07 via #6260).
- **Agent-authority series finally moved (watch #5815).** Not as numbered slices, but as a big wave of runtime work 10-01→10-06: server request authority, containment of every Python/Bash/PTY/tmux execution, resource-identity binding (fs/process/browser/remote), a durable effects journal requiring evidence for "done" claims. *Why it matters:* this is the substance of the promised slices 2–4; confirm #5815's status by hand.
- **Security default flipped:** `ee13ed1` post-external-context approval gate now **on by default** (opt out with `ODYSSEUS_TOOL_APPROVAL_GATE=0`). *Why it matters:* an email/fetched page can no longer silently reach send/delete/bulk-email tools. Expect more approval prompts in our setup.
- **Security sweep 10-05/06 (`pre-ajax` release freeze, #6503):** subprocess env allowlist + credential scrubbing (`2a3d0c6`), delegated authority enforced in local web fetches, workspace-existence-check confinement, regex- and parser-DoS fixes, host-bridge request hardening, registered-endpoint authority, session cost-ledger key isolation, SVG DOM reparse fix, CodeQL fixes. *Why it matters:* child processes no longer inherit provider tokens/DB URLs — check our agent tooling doesn't rely on inherited env.
- **Task scheduler:** scheduled remote SSH without scope is now deterministically refused (`29c31a4`); tasks recover registered-endpoint runtime credentials (`9aa58b2`); singleflight cache cleanup on cancel (`6e4b3aa`); reminders retry failed note delivery via configured channel (#6566). *Why it matters:* unattended jobs that SSH out may start failing by design.
- **Deep research:** big 09-17 burst — required-research-before-synthesis, bounded breadth/depth recovery, distinct refinements, reject fabricated follow-up queries, stream answer tokens while reconciling drafts; accessible research map restored (09-22). *Why it matters:* deep-research behavior is materially different from our pinned build.
- **Cookbook:** local control enforced across Cookbook wrappers (`3834cd7`); pid sweep skipped on hosts without procfs (`30ef765`).
- **Other:** #6342 blank `memory_id` rejected on edit/delete; ipv6-localhost loopback policy honored; Windows bash/PTY fixes; CI now 4 pytest shards; release smoke suite added; new top-level SECURITY.md, THREAT_MODEL.md, ROADMAP.md. *Why it matters:* THREAT_MODEL/ROADMAP are worth a skim for #3158/#236-type gaps.
- **Contributors:** ~197 commits Alexandre Teixeira, ~173 pewdiepie-archdaemon, ~65 Léo — dev is now dominated by an integration/"wave" branch workflow.
- **External chatter:** web search surfaced only launch-era coverage (MindStudio, HN thread, DEV) — nothing new since last run.

---

## 2026-09-16 — review (delta since 2026-08-24)

### Activity by the numbers (window `b4d1293`…`3b6c169`, ~23 days)
- **17 commits** on `dev`; HEAD `3b6c169` (2026-09-13).
- **PRs:** 14 merged, 8 closed-unmerged (Dependabot #6199/#6140/#6195, junk #6313 "Main" / #6312 "Work", #6297 "Agent/odysseus/astra context budget", #6163 "Codex/upstream dev modernized", #6011 Poe tool-name alias). **No mass purge.**
- **Issues:** ~62 opened in the window. Open count climbing: ~1,109 → **1,205**.
- Stars ~86.1k → **~87.3k**; forks 621 → 887 (the fork growth is the real signal this cycle).

### Security — landed on `dev` (good)
- **#6280 atomic token cache swap (09-13).** Race in `_refresh_token_cache` eliminated via atomic swap, with a regression test that exercises the production path.
- **#6215 MCP `Add Server` rejects malformed `Args` (09-11)** instead of silently defaulting to `[]` — input-validation hardening on a config surface.
- **#5937 psycopg2 → psycopg2-binary (09-01)** — dependency hygiene for Docker deployments.

### Infra — relevant to our stack
- **#6158 `fix(docker): repair app cache parent ownership` (09-05).** Docker app-cache parent-ownership repair — spot-check our Compose for the same symptom (cache dir owned by wrong uid after a recreate).
- **#6228 cache a successful-but-empty Tailscale lookup (09-02)** — discovery no longer hammers Tailscale when the tailnet legitimately returns empty.
- **#6174 clean up singleflight cache on cancellation (09-01)** — task-cache leak on cancelled tasks.

### Release line (new)
- **#6167 prepare v1.0.3 hotfix (08-25)**, **#6168 align dev version with 1.0.3 (08-25)**, **#6260 sync main with dev (09-07)**. First evidence of a real release cadence — `main` now tracks `dev`. Watch for a GitHub Release / tag.

### Not landed / still open
- **Agent-authority slices 2–4** — no PRs since #5817 (08-15).
- **#3158** deny agent file-tools access to cred/session stores — still closed-unmerged, no replacement.
- **#236** SOPS encrypted-at-rest secrets — still not landed.
- **#3148** gallery null-owner read bypass — open & stale.
- **#4991** benchmark how often the prompt-injection guard holds on small models — open (relevant to our local-small/-large tiers).
