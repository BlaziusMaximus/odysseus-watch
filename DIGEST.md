# odysseus-watch digest
<!-- odysseus-dev/odysseus (was pewdiepie-archdaemon/odysseus — org transfer 2026-07-20) -->

**Last reviewed:** 2026-10-18
**Last reviewed SHA:** `9712a54` (Merge PR #6434 oauth-expired-state, branch `dev`, 2026-10-16)
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

## 2026-10-18 — review (delta since 2026-09-16, `3b6c169`…`9712a54`)

_Caveat: GitHub API tools can't reach `odysseus-dev/odysseus` from this session (git read only), so this is from commit history + public pages. Issue/PR open-closed counts and new-issue list not collected._

**Watching**
- **#5815 agent authority** — still OPEN, now "1 of 6" slices done (was "1 of 4"); six sub-issues filed 08-17 (#6083/#6090–#6093/#6103), labelled p2 on 10-01. Why it matters: the *work* is landing as a big in-repo "Wave 3/4/5" effort (see below) even though the sub-issues haven't closed.
- **#5757** (manage_tasks JSON-as-text) and **#5792** (IMAP folders) — both still OPEN, triaged `p2` 10-01; fix PRs #5790 / #5791 exist but unmerged.
- No movement seen on #3136, #3148, #605, #4991 (no matching commits).

**Security (dominant theme, ~25 commits)**
- **Pre-ajax security sweep (10-05/06):** endpoint authority, browser boundaries, host-bridge requests, regex/parser DoS, workspace-existence confinement, session-cost-ledger key isolation, CodeQL fixes. Why: broad hardening, mostly invisible to users.
- **`ee13ed1` post-external-context approval gate ON by default** — this is the follow-through on #5817. Behavior change: agents will prompt more.
- **`82cfd7d` delegated authority enforced in local web fetches**; **`2a3d0c6` subprocess env sanitized / credentials scrubbed.**
- **#6268 deny generated-image access when ownership check fails** (10-14); **#6029 `AUTH_ENABLED=false` honored in admin-gated config routes** (10-14) — check this one if running no-login.
- **Runtime containment/authority, Waves 3–5 (10-01→10-03):** process/PTY/job/browser/remote resources bound to authority, functional bwrap containment in CI, durable append-only effect log. Why: this is the deterministic-execution-authority design #5815 asked for.

**Scheduler / Cookbook / research**
- **#6566 reminders retry failed note delivery via configured channel**; `9aa58b2` scheduled tasks recover registered-endpoint credentials; `29c31a4` remote scheduled SSH refused deterministically when unscoped.
- **Cookbook:** `3834cd7` local control enforced across Cookbook wrappers; `30ef765` pid sweep skipped without procfs. Why: scheduler/Cookbook are now under the new authority model — expect stricter behavior on scheduled shell jobs.
- Nothing notable on deep-research.

**Release / infra**
- **Still no GitHub Release or tags.** But history was restructured: root commit is now "Odysseus v1.0", `dev` carries a `release/pre-ajax-freeze2` merge (10-06) and a "release smoke suite" (09-30). A cut looks close.
- **#6166 Docker: ChromaDB volume mounted where image persists** (10-14) — likely affects our Compose; check we weren't losing vector data.
- Commit volume jumped: **469 commits** in 32 days (mostly the wave merges above).

**Other**
- #6693 skills tagged remote-only; #6495 attach Notes/Todos as Agent context; #6572 opt-in reasoning effort for utility calls; #6643 published skills injected with Auto-approve off.

**External**
- Coverage is still launch-era (OpenSourceForU, TBS News, DEV, MindStudio review); one source cites ~87k stars as of 09-12. Nothing new/critical, no security disclosures found.

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
