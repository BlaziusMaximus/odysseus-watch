# odysseus-watch digest
<!-- odysseus-dev/odysseus (was pewdiepie-archdaemon/odysseus — org transfer 2026-07-20) -->

**Last reviewed:** 2026-11-08
**Last reviewed SHA:** `1c15090` (fix(calendar): render multi-day events within their day cells (#6948), branch `dev`, 2026-11-08)
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

## 2026-11-08 — review (delta since 2026-09-16 @ `3b6c169`)

*Caveat: this run could read git history + GitHub search, but not the issue/release API for odysseus-dev/odysseus (access not enabled). Watch-list states below come from public issue pages.*

**Numbers:** ~754 commits on `dev`, ~166 PRs merged, ~28 closed-unmerged, ~290 new issues; **1,118 open issues** (down from 1,205 — someone triaged). Still **no GitHub Releases or tags**.

### Watching
- **#5815 agent authority umbrella — still OPEN, now broken into slices:** #6091 (slice 2, process sandbox; PR #6120 + seccomp #6119 / egress #6118 / Docker-AppArmor #6121), #6083 (slice 3), #6092 (slice 4). Relabeled p2 on 10-01. Slice PR merge status not confirmed — the "no slice 2" worry is now "slice 2 designed, not confirmed landed."
- **#5757** (manage_tasks JSON-as-text) — OPEN; fix PR #5790 referenced, not confirmed merged.
- **#5792** (non-English IMAP folders) — OPEN; fix PR #5791 referenced, labeled "ready for review."
- **#3148** (gallery null-owner bypass) — OPEN, now **p1** (10-02). Related: #6268 (10-14) denies generated-image access when ownership check fails.
- **#4991** (prompt-injection guard benchmark) — OPEN, no PR in this repo; #7030 offline benchmark evidence auditor merged adjacent.
- **#5788, #3136, #605** — not re-checked this run.

### Security (biggest cluster this cycle)
- **Pre-Ajax release freeze (#6503, 10-06):** a big security sweep — regex/parser DoS removal, host-bridge hardening, workspace confinement, delegated authority in local web fetch, subprocess env credential scrubbing (`2a3d0c6`). Why it matters: the post-external-context approval gate is now **on by default** (`ee13ed1`).
- **CA trust incident handling (#6785, #6791, `cd9f1f7`):** rejects "published compromised intermediates" as trust anchors; refreshes outbound HTTPS trust + SearXNG. Why: if our Docker image predates this, rebuild.
- **SSRF closures:** #6316 (model endpoint URLs), #6327 (manage_endpoints tool), #6355 (user-supplied IMAP/SMTP hosts).
- **Admin/auth:** #6832 revoke API tokens on admin demotion, #6830 privilege type validation, #6821 username validation, #6357 agent can't re-enable admin-disabled tools, #6353 non-admin cookbook tools blocked.
- **#6705 / #6722:** approved bash/python/file writes sealed to agent workspace when none set.
- **#6956:** Docker image requires patched Debian Brotli decoder (dependency CVE).

### Task scheduler / deep research / Cookbook
- **Scheduler:** #6358 admin-only action policy in `manage_tasks`; #6825 refuses cron with seconds field; #6473 non-numeric research settings fall back to defaults; #6926 daily-brief calendars in local time; singleflight cache cleanup (`6e4b3aa`); unscoped remote scheduled SSH deterministically refused (`29c31a4`).
- **Deep research:** #7043 keep reports after failed deletion; #6929 math typeset in visual reports.
- **Cookbook:** #6969 endpoints preserved across model launches; #7031 endpoint registration refactor; #6728 vLLM too-old NVIDIA driver diagnosis; #6080 stale llama-server shim recovery; #6369 NameError in hf-gguf-files; Cookbook wrappers enforce local control (`3834cd7`).

### Other notable
- Photo/image **editor** overhaul (#6741, #6739, #6992); **email** EML/PDF export (#6742); **Windows** test-compat sweep; maintainer governance handbook (#6736); latest: #7054 web-search query fix (11-06), #6768 timeout message in chat (11-08).

### External
- Coverage is still launch-era (May 31, 2026 release) explainers and Railway templates; mixed notes on hardware/install difficulty. Nothing new of significance; search surfaced no discussion of the security work or the org move.

### Action for Adam
- Our box was ~920 commits behind in September; it is now ~1,700. The CA-trust and SSRF fixes alone justify updating.

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
