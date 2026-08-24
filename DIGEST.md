# odysseus-watch digest
<!-- odysseus-dev/odysseus (was pewdiepie-archdaemon/odysseus — org transfer 2026-07-20) -->

**Last reviewed:** 2026-08-24
**Last reviewed SHA:** `b4d1293` (fix(agent): drop the empty assistant turn from an approved-action replay (#6124), branch `dev`, 2026-08-20)
**Prior review:** 2026-07-28 @ `01790c2`

---

## TL;DR for Adam

- **Calm, healthy cycle — no purge this time.** 117 commits on `dev` since the last review (`01790c2`…`b4d1293`). Of the 100 PRs closed in the window, **91 merged / only 9 closed-unmerged** — a normal maintain-and-merge rhythm, the opposite of the 93-PR bulk-purge we saw on 2026-07-23. Nothing swept, nothing dramatic.
- **Still exploding.** ~**86.1k stars** (was ~62.3k), 621 forks. Issue/PR numbering `5820 → 6158` (~340 new items in ~4 weeks). **1,109 open** (984 issues + 125 PRs). Ships from `dev`, no GitHub Releases/tags, no published security advisories.
- **The big story: agent-authority hardening landed (#5817).** The prompt-injection / "make agent execution authority deterministic" request we flagged as *new & open* last time (#5815) got its **first real implementation merged 2026-08-15** — a 16-commit, 52-file, +5,303/−448 stack (`RaresKeY/fix/agent-external-context-gate`). It adds a **server-owned tool-capability registry** and carries **run-local "external-context" taint** through the agent loop: once an untrusted/external result enters a run, high-impact and unknown tools are **blocked before execution** (same-batch and on later rounds). In their words: *"model output is now an action request rather than an authorization decision."* This is directly relevant to how we run Odysseus as our agent platform. **It's explicitly "the first of four" — more agent-security slices are coming.**
- **Three carry-forward bugs we were watching are now FIXED on `dev`:** the unbounded `tts_cache` leak (#5600 → capped/evicted, #5822), the scheduler-killed-by-status-polling bug (#5782 → #5789), and continued SSRF hardening (skill import now resists **DNS-rebinding + TOCTOU**, #5986; `api_call` pinned to the SSRF-validated IP, #5727).
- **⚠️ Our box is far behind.** `~/odysseus` on this server is pinned at **`463713c` (2026-06-05)** — that predates *every* fix in this digest **and** the last one: no agent-authority gate, no SSRF/DNS-rebind hardening, no DB-0600, no email token-hiding, no tts-cache cap. It is **~900+ commits behind** `dev`. Not an emergency (the box isn't public-facing), but if we ever expose Odysseus or run untrusted agent content through it, we're running the pre-hardening code. See **"Our deployment" below.**
- **Infra bugs that hit OUR stack specifically:** several fresh issues target the exact Docker services we run — **ChromaDB volume mounted at the wrong path / RAG goes silently empty after a container recreate** (#6132, #6146, fix PR #6147), and **migrated SearXNG settings silently disabling JSON** (#6066, fix PR #6136). Worth a spot-check on our Compose. See **"Watch for our stack."**

---

## Watching (carry-forward)

| Item | State now | Note |
|------|-----------|------|
| #5815 agent execution authority deterministic | **OPEN (umbrella)** | First slice **MERGED** as #5817 (2026-08-15); umbrella stays open — "1 of 4" |
| #5600 tts_cache grows without bound | **CLOSED / FIXED** | cap + eviction shipped (#5822, `5104a9a`) |
| #5782 scheduled tasks killed by status polling | **CLOSED / FIXED** | fixed by #5789 (+ #6009 treat unread-state as passive poll) |
| #5814 API model providers show empty in picker | **OPEN** (upd 08-08) | related UI fixes landed (#6087/#6089 "show API models by default") — re-verify |
| #5757 manage_tasks writes tool-call JSON as text | **OPEN** (upd 07-28) | agent tool-call reliability; not yet resolved |
| #5792 non-English IMAP folder names not found | **OPEN** (upd 07-28) | localized Sent/Trash/Archive/Junk |
| #5788 improve HUD and voice controls | **OPEN** (upd 07-28) | enhancement |
| #3136 `/api/research/status` 404 | **OPEN** (upd 07-09) | stale; research status polling |
| #3148 gallery null-owner read bypass (auth on) | **OPEN** (upd 06-06) | **still stale**; unauth read of null-owner gallery items — verify vs `dev` |
| #605 Architecture v3 proposal | **OPEN** (upd 06-27) | being executed incrementally — `routes/*` subpackage carve-out continues (search, vault, webhook, document, mcp, task all moved this cycle) |
| #3158 PR deny agent tools access to cred/session stores | **CLOSED unmerged** | swept 07-23; **no direct-commit replacement found.** The #5817 gate reduces blast radius (blocks high-impact tools after untrusted context) but does **not** specifically block file-read of cred/session files. Still worth a deployment spot-check. |
| #236 PR SOPS encrypted-at-rest secrets | **CLOSED unmerged** | **still not landed**; secrets at rest in plaintext config unless self-rolled |

---

## 2026-08-24 — review (delta since 2026-07-28)

### Activity by the numbers (window `01790c2`…`b4d1293`)
- **117 commits** on `dev`; HEAD `b4d1293` (2026-08-20).
- **PRs:** 100 closed → **91 merged**, 9 closed-unmerged. **No mass purge** this cycle.
- **Issues:** 139 opened, 78 closed. Net open climbing (~984 issues + 125 PRs = 1,109).
- Maintainers still commit directly to `dev`; outside PRs now mostly get **merged** rather than swept (healthier than the 07-23 cycle).

### Security — landed on `dev` (good)
- **#5817 "gate tools after external context" — MERGED 08-15 (the headline).** Server-owned tool-capability registry + run-local external-context taint. After an external/untrusted result enters a run, high-impact & unknown tools are blocked pre-execution (same-batch and later rounds). Follow-on commits in the same series: taint prefetched web context, taint stored document tool results, taint model-visible tool responses, keep ambient context fail-closed, harden/retire the approval lifecycle, seal document-approval content. Later touch-ups: `#6113` allow remaining actions for an approved task, `#6124` drop empty assistant turn from approved-action replay. **Explicitly slice 1 of 4 — watch for the next three.**
- **SSRF / DNS-rebinding:** `#5986` harden skill import against **DNS rebinding + SSRF TOCTOU**; `#5727` pin `api_call` to the SSRF-validated IP; `#5961` bind provider detection to DNS labels.
- **Auth/session:** `#6048` derive the session-cookie **Secure** flag from the request scheme; `#5807` normalize mounted request paths; `#5795` define Default/Local **owner contract**; `#5856` require `manage_skills` action for skill management; `#6069` scope the `action_tidy_research` broken-file sweep to **admins only**.
- **Email/OAuth:** `#5802` preserve OAuth SMTP security; `#5995` derive Google OAuth redirect URI scheme from request; `#6032` stop MCP assuming `http://localhost:7000` for the OAuth callback.
- **Data safety:** `#5831` don't let an unreadable memory store get overwritten with an empty one; `#6068` clean up orphaned temp files on atomic-write failure; `#5721` stop atomic writes colliding on a constant PID suffix.

### Security — still open / not landed
- **#3158** deny agent file-tools access to Odysseus' own credential/session stores — **still closed unmerged, no replacement.** The new gate helps but doesn't target on-disk cred reads. Spot-check on our deployment.
- **#236** SOPS encrypted-at-rest secrets — **not landed.**
- **#3148** gallery null-owner read bypass — **open & stale.**
- **#4991** (new) *Benchmark how often the prompt-injection guard actually holds on small models* — open; a good sanity issue given the new gate is only as strong as the model behind it (relevant to our local-small/-large tiers).

### What shipped / themes (last ~4 weeks)
- **Agent-security stack (#5817 + follow-ups)** — see above; the defining change of this cycle.
- **Architecture v3 (#605) grinding forward** — `routes/` domains carved into subpackages this cycle: `search` (#5779), `vault` (#5780), `webhook` (#5781), `document` (#5885), `mcp` (#5899), `task` (#6081); `session` in flight (#6133).
- **Memory overhaul in flight (#6142/#6143/#6144)** — a 3-PR "hybrid recall + sleep-time + lifecycle (growth/consolidation/worthiness) + brain view/API" series; not merged yet (needs runtime validation).
- **Model routing / LLM plumbing:** omit temperature for major-only Opus ids incl. **`claude-opus-5`** (#5761); add **`deepseek-v4`** to thinking-model patterns (#6000); alias tool names colliding with gpt-oss built-ins (#5878); parse Hermes/Qwen JSON tool bodies inside wrappers (#5887); keep selected models strict (#5801); show API models by default (#6089).
- **Perf pass:** email prewarm made idle/bounded, session loading no longer blocks the shell, deferred transcript hydration, batched live-thinking rendering, vendored KaTeX/Mermaid + first-use loading, preloaded Fira Code faces.
- **Desktop:** native **Tauri** desktop installer + lifecycle harness in flight (#6137); frozen-Windows-build MCP connection fixes.
- **Podman support** proposed/in-flight (#3194 / #6149 rootless Compose overlays).

### Watch for our stack (Docker infra bugs that map to services we run)
- **ChromaDB:** #6132 *"ChromaDB volume mounted at the wrong path — the entire vector store"* and #6146 *"RAG retrieval goes silently empty after a ChromaDB container recreate"* (fix PR **#6147**, ready for review). We run ChromaDB behind Odysseus — worth checking our Compose mount path and that RAG still returns hits after any container recreate.
- **SearXNG:** #6066 *"Migrated SearXNG settings can disable JSON while root health passes"* (fix PR **#6136**). We run SearXNG; a health-green-but-JSON-disabled state would silently break agent web search.
- **Docker runtime:** #6157 vLLM can't create runtime caches after container recreation; #6156/#6155 Real-ESRGAN wheel build timeout + fastembed conflict; #6158 repair app-cache parent ownership. General Docker-recreate fragility this cycle.

### Our deployment (this server) — ⚠️ action item
- `~/odysseus` HEAD = **`463713c` "feat(search): unify session transcript search (#2877)" (2026-06-05)**.
- That is **older than the last review** (2026-07-28) and older than the prior one (2026-06-06). Relative to `dev` it is **~906 commits behind** (graph shows `diverged` because upstream `dev` history was rewritten around the org transfer / 07-23 curated merge, so an exact ancestor count is fuzzy — but the "behind ~900" magnitude is real).
- **Missing everything security-relevant from BOTH digests:** the agent-authority gate (#5817), all SSRF/DNS-rebind hardening, DB `0600` restriction, email OAuth token-hiding, tts-cache cap, scheduler-polling fix.
- **Not an emergency** — the box isn't public-facing and we don't (currently) pipe untrusted external content through Odysseus agents. **But:** if we ever expose it or run untrusted agent input, we're on pre-hardening code. Recommend a planned pull to a post-`#5817` `dev` commit (i.e. ≥ `b4d1293`) as a deliberate, tested upgrade — not a blind `git pull`, since `dev` moves fast and has had history surgery.

### External / method notes
- GitHub **search API works again** for this repo this cycle (the 07-28 422 "too large / rename" errors have cleared) — counts above are from `search/issues`, deltas from the `compare` API, commit list from `compare 01790c2...dev`.
- No published GitHub **security advisories** (`/security-advisories` empty) and no Releases/tags — as before, "version" is internal only.

---

*Tracker: `BlaziusMaximus/odysseus-watch` · routine = migrated-routines task #2 (Odysseus GitHub tracker). Next review: on next scheduled run (~4-weekly cadence observed). Local snapshot archived at `~/agents/projects/migrated-routines/sandbox/odysseus-github-tracker/`.*
