# odysseus-watch digest
<!-- odysseus-dev/odysseus (was pewdiepie-archdaemon/odysseus — org transfer 2026-07-20) -->

**Last reviewed:** 2026-10-25
**Last reviewed SHA:** `9d0d372` (fix(deps): require Pillow 13 with compatible FastEmbed (#6845), branch `dev`, 2026-10-25)
**Prior review:** 2026-09-16 @ `3b6c169`

---

## TL;DR for Adam (as of 2026-09-16 — newest entry is below the Watching table)

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

## 2026-10-25 — review (delta since 2026-09-16)

*Caveat: the GitHub connector was scoped to this repo only, so commits come from a `dev` clone and issue/star data from the public API. Merged-PR/new-issue lists and the 3b6c169 boundary were not enumerated; window is by date (09-13 → 10-25).*

- **Stars ~94.4k (was ~87.3k), forks 1,753 (was 887), open issues+PRs 1,516 (was 1,205 issues).** Forks doubled in 6 weeks.
- **No GitHub Release or tag yet** despite the 1.0.3 prep in September. Still shipping from `dev`.

### Watching
- **#5815 agent-authority umbrella: still OPEN**, last touched 10-01. But the slices are landing under other names: `ee13ed1` keeps the post-external-context approval gate on by default, `82cfd7d` enforces delegated authority in local web fetches, `9ee614e` enforces registered-endpoint authority, and the "runtime resource authority" waves (process/job/browser/remote resources bound to authority, `9d02571` server request authority) merged 10-01 to 10-03. This is the real follow-through, and #5815 just hasn't been updated.
- **#5757, #5792, #5788, #3136, #3148, #605:** all still open, no movement (last updated ~10-01/02).
- **#4991 (prompt-injection guard benchmark on small models): open, updated today (10-25).** Someone is active on it.
- **#3158 / #236:** nothing new. The 10-02 `2a3d0c6` subprocess env sanitization and credential scrubbing partly overlaps the #3158 goal.

### Security (priority 2)
- **#6316, #6327 (10-24): SSRF blocked for user-supplied model endpoint URLs and the `manage_endpoints` agent tool.** Matters because endpoints are the main user-configurable URL surface.
- **#6791 / #6788 (10-24) plus a 10-21 cluster: outbound-HTTPS trust refresh.** Incident CAs are distrusted in Chromium and gRPC, and custom trust stores are checked. SearXNG is bumped to a corrected upstream, and supporting services are hardened.
- **#6268 (10-14): generated-image access denied when ownership verification fails.** Looks like the fix for the gallery null-owner class (cf. #3148, which is still open).
- **#6029 (10-14): `AUTH_ENABLED=false` is honored in admin-gated config routes.**
- **10-05/06 hardening sweep:** regex and parser DoS paths, workspace existence confinement, host-bridge request boundaries, browser boundaries, session cost-ledger key isolation.
- **#6705 (10-23): approved bash/python and file writes are sealed to the agent workspace when no workspace is set.**

### Scheduler / deep-research / Cookbook (priority 3)
- **Scheduler:** #6358 applies the admin-only action policy in `manage_tasks`. #6473 / `35e3b29` stop non-numeric or non-finite research settings from crashing the scheduler (#5049). `9aa58b2` recovers registered-endpoint credentials for tasks. `6e4b3aa` fixes the task singleflight cache leak on cancellation.
- **Deep research:** a long 09-17 to 09-19 series enforces breadth and distinct refinement, bounds search attempts by depth, and streams answer tokens while reconciling the final draft. This is a rework of research behavior, so expect output changes.
- **Cookbook:** `30ef765` skips the pid sweep on hosts without procfs, and `3834cd7` enforces local control across the Cookbook wrappers.

### Our-stack / other
- **#6166 (10-14): the Docker ChromaDB volume is now mounted where the image persists.** Check our Compose. The old mount may mean memory never persisted.
- **#6845 (10-25): Pillow 13 required with a compatible FastEmbed.** Dependency bump, so rebuild images.
- **New features:** email export to EML/PDF (#6742), Notes/Todos attachable as Agent context (#6495), opt-in reasoning effort for utility calls (#6572), remote-only skill tag (#6693), chat history exposed to the agent (#6443).
- **Housekeeping:** the maintainer handbook and governance docs were published, and the repo is on the `odysseus-dev` org (a few commits still reference `pewdiepie-archdaemon`).
- **Our box:** still pinned at `463713c` (June). I couldn't verify this run, but it is now far behind on all the security items above.

### External discussion
- Nothing new since launch coverage (TBS News, 80.lv, Dexerto, MindStudio). No security write-ups or notable critiques surfaced.

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
