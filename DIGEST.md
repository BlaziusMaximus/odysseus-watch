# odysseus-watch digest
<!-- odysseus-dev/odysseus (was pewdiepie-archdaemon/odysseus — org transfer 2026-07-20) -->

**Last reviewed:** 2026-11-01
**Last reviewed SHA:** `8450793` (feat(admin): rename model endpoints inline (#6970), branch `dev`, 2026-11-01)
**Prior review:** 2026-09-16 @ `3b6c169`

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

## 2026-11-01 — review (delta since 2026-09-16 @ `3b6c169`)

> ⚠️ **Partial run.** Commit data only (read-only clone). The GitHub API for issues/PRs/releases isn't enabled for `pewdiepie-archdaemon/odysseus` in this session, so **Watching-list status changes, new issues, merged-PR metadata and Releases were NOT checked.** Re-run with API access to cover those. No git tags exist on `dev`/`main` as of this run.

- **Volume:** ~726 commits on `dev` in ~7 weeks (many are merge/test-remediation commits). Heavy test-hardening push in late Oct.
- **Security (biggest theme):**
  - **SSRF** closed on several fronts: user-supplied model endpoint URLs (#6316), `manage_endpoints` agent tool (#6327), email IMAP/SMTP hosts (#6355). *Why: the SSRF gap our pinned box lacked.*
  - **Upload/thumbnail path confinement** + symlinked-root canonicalization (10-29, `571ac09`, `59cca6b`). *Path-traversal class fix.*
  - **Agent can no longer re-enable tools an admin disabled** (#6357); **delegated authority enforced in local web fetches** (`82cfd7d`); external-context approval gate kept on by default (`ee13ed1`). *Continues the agent-authority series (#5815 umbrella) — runtime authority binding landed 10-01…10-02 as many commits, not tracked as "slice 2" by name.*
  - Generated-image access denied when ownership check fails (#6268); `AUTH_ENABLED=false` honored in admin config routes (#6029); regex-DoS paths removed (`ee48c9c`); SearXNG/CA-trust refresh after an incident-CA check (#6791, `cd9f1f7`).
- **Task scheduler / research:** admin-only action policy in `manage_tasks` (#6358); cron with seconds field rejected (#6824); non-numeric/non-finite research settings no longer crash scheduler; daily-brief calendars in local time (#6926); singleflight cleanup on cancel. Big deep-research reliability pass 09-17 (breadth recovery, streaming + draft reconciliation).
- **Cookbook:** non-admin owners blocked from read-only cookbook tools (#6353); llama-server launch/PATH fix; vLLM old-NVIDIA-driver diagnosis (#6728); endpoints preserved across model launches (#6969); NameError fix in hf-gguf (#6369).
- **Docker/Compose (our stack):** TTS cache default stops compose warning (#6645) — related to old #5600; `ODYSSEUS_FIRST_TOKEN_TIMEOUT` forwarded (#6778); `SEARXNG_HOST_PORT` param (#6598); realesrgan wheel retry (#6407); patched Debian Brotli required (#6956).
- **Release line:** commit `dab6605` "close pre-integration release blockers" and a release smoke suite (09-30). Still no tags — confirm via Releases API next run.
- **Notable features:** deployment-declared local MCP tools (#6330), inline endpoint rename (#6970), photo editor hardening, regenerate/edit warning + fork (#6225).
- **External discussion:** nothing new surfaced; web search only returned launch-era (May–June) coverage (Open Source For U, NetInfluencer, MindStudio). Sources disagree on the repo name — it's now `odysseus-dev/odysseus` per the header note.
- **Our box:** `~/odysseus` still pinned at `463713c` (06-05), now well over 1,600 commits behind. Several fixes above are security-relevant — worth updating.

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
