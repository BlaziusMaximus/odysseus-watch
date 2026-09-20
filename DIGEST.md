# odysseus-watch digest
<!-- odysseus-dev/odysseus (was pewdiepie-archdaemon/odysseus — org transfer 2026-07-20) -->

**Last reviewed:** 2026-09-20
**Last reviewed SHA:** `3b6c169` (unchanged since 09-16 — dev had zero new commits this window)
**Prior review:** 2026-09-16 @ `3b6c169`

---

## TL;DR for Adam

- **Dev branch is frozen, but a security disclosure wave just landed as open PRs.** Zero new commits, zero merges, zero releases since 09-16 (`3b6c169` confirmed identical to current `dev` HEAD). But a private security advisory (**GHSA-q6px-cw6j-7c5v**, not yet public) triggered ~9 coordinated fix PRs from afonso and isharak7m, all opened 09-17→09-20, **none merged yet**. Worth watching land — this is the most security activity in one window since #6280.
- **What the advisory wave covers:** SSRF guards (agent-tool endpoints #6327, model endpoint URLs #6316, IMAP/SMTP hosts #6355, a `::1` loopback misclassification #6368), a **prompt-injection fix in deep-research**: the "Discuss" spin-off was seeding a crawled report as a `system`-role message instead of untrusted context (**#6354**, closes #6351 — this is a real one, straight violation of their own threat model), a Cookbook tool-policy bypass for non-admin owners (**#6353**, closes #6350), agent tools re-enabling admin-disabled tools (**#6357**, closes #5523 — adjacent to but *not* the #5815 slice-2 we're waiting on), and credential redaction in logs (#6356).
- **Matching issue wave:** ~12 new issues filed 09-18 by the same two reporters (desmvu, afonso) — issue numbers line up 1:1 with the PRs above, plus 4 fresh Cookbook-only bugs (#6360–#6365) with no PRs yet: SAM checkpoint exposed as a servable model, SAM masks missing, HF token field doesn't persist, FastEmbed leaks CUDA VRAM in the idle process.
- **#5815 agent-authority umbrella still stalled** — still OPEN, still no slice-2 PR filed against it directly (6 weeks now). Don't confuse #6357 above for slice 2; it's a related but separately-tracked bug.
- Stats basically flat: stars 87.3k→87.4k, forks 887→923, open issues ~1,205→~1.1k.
- No notable new external discussion this window beyond the project's usual profile (HN launch thread, review sites) — nothing new picked up.

---

## Watching (carry-forward)

| Item | State now | Note |
|------|-----------|------|
| #5815 agent execution authority deterministic | **OPEN (umbrella)** | Still 1 of 4 slices. No slice 2 in 6 weeks. #6357 (closes #5523) is adjacent, not slice 2 — don't conflate. |
| #5600 tts_cache grows without bound | **CLOSED / FIXED** | (08-24: #5822) |
| #5782 scheduled tasks killed by status polling | **CLOSED / FIXED** | (08-24: #5789) |
| #5814 API model providers show empty in picker | **CLOSED / FIXED** | #6087/#6089 "show API models by default" landed 08-17. |
| #5757 manage_tasks writes tool-call JSON as text | **OPEN** | no visible movement this window |
| #5792 non-English IMAP folder names not found | **OPEN** | no visible movement this window |
| #5788 improve HUD and voice controls | **OPEN** | enhancement, no movement |
| #3136 `/api/research/status` 404 | **OPEN** | stale, no movement |
| #3148 gallery null-owner read bypass (auth on) | **OPEN** | stale, no PR filed yet |
| #605 Architecture v3 proposal | **OPEN** | no movement this window |
| #3158 PR deny agent tools access to cred/session stores | **CLOSED unmerged** | still no replacement |
| #236 PR SOPS encrypted-at-rest secrets | **CLOSED unmerged** | still not landed |

---

## 2026-09-20 — review (delta since 2026-09-16)

### Activity by the numbers (window `3b6c169`, ~4 days)
- **0 commits** on `dev` — HEAD still `3b6c169`; `compare/3b6c169...dev` confirms "identical."
- **0 PRs merged, 0 releases** (repo still has never cut a Release).
- **21 new PRs opened**, all still open — dominated by the security wave below.
- **~12+ new issues** opened 09-18, mostly by desmvu and afonso, matching the PR wave 1:1.
- Open issues ~1,205 → **~1.1k**; stars 87.3k → **87.4k**; forks 887 → **923**.

### Security — filed, NOT yet merged (the thing to watch)
A private advisory, **GHSA-q6px-cw6j-7c5v** (404s publicly — still under embargo), appears to be behind a coordinated fix wave from **afonso** and **isharak7m**, all opened 09-17→09-20 and labeled "ready for review" or "needs runtime validation" — none merged as of this review:
- **#6354** `fix(research): seed spin-off report as untrusted context` (closes #6351) — the Deep Research "Discuss" spin-off was injecting a crawled report as a `system`-role message instead of `untrusted_context_message`, a direct violation of the project's own threat model. Real prompt-injection bug, real fix.
- **#6355** guard user-supplied IMAP/SMTP hosts against SSRF (closes #6352)
- **#6327** SSRF validation on the `manage_endpoints` agent tool
- **#6316** block SSRF via user-supplied model endpoint URLs
- **#6368** fix `::1` misclassified as reserved instead of loopback in the SSRF guard
- **#6353** block read-only Cookbook tools for non-admin owners (closes #6350)
- **#6357** stop the agent from re-enabling tools an admin disabled (closes #5523 — adjacent to, not part of, the #5815 umbrella)
- **#6358** enforce admin-only action policy in `manage_tasks`
- **#6356** redact credentials in scheme-less endpoint URLs (logging)

### Cookbook / deep-research (priority area)
- #6354 and #6353 above both land directly in this area.
- **#6331** `fix(research): expose manage_research as a native tool` — open, unmerged.
- Four fresh Cookbook-only bugs from 09-18, no PRs yet: **#6365** internal SAM checkpoint (facebook/sam-vit-base) listed as servable, **#6362** SAM object masks reported missing even when deps installed, **#6361** HF token field doesn't persist, **#6360** FastEmbed reserves CUDA VRAM in the idle web process.

### Not landed / still open
- **#5815 agent-authority umbrella** — still OPEN, no slice 2 filed against it (6 weeks since #5817).
- **#3148** gallery null-owner read bypass — still open, stale, no PR.
- **#236** SOPS encrypted-at-rest secrets — still not landed.

### Other notable
- **#6318** `feat(dashboard): add homescreen widget dashboard with a weather widget` — open, first widget-dashboard feature seen in this repo.
- Web search: no significant new external discussion this window — same general profile (HN launch thread, review-site writeups) as prior reviews, nothing new since 09-16.

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
