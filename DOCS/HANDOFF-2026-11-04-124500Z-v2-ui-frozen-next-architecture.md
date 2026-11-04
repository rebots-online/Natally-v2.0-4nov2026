---
written_utc: 2026-11-04T12:45:00Z
session: https://claude.ai/code/session_01Q4wdhSBmBCZGmF2siH9dL2
repo: rebots-online/Natally-v2.0-4nov2026 (master)
operator terms: one-shot greenfield; no sign-off stops; the agent owns the whole result (Robin, 2026-11-04)
---

# Handoff — Natally v2: UI designed, next is ARCHITECTURE

## Done
- Repo initialized: v1 ARCHITECTURE + PRD as reference only (`DOCS/REFERENCE/`), look-and-feel assets (`design/assets/`), no v1 code.
- UI source of truth = v1.42 (Robin). Real v1.42 screens: `design/reference-screens/v1.42.95682/`.
- Claude Design canvas (10 artboards, clickable): https://claude.ai/artifact/KZY7iqNENoN5hcpy9WdHTb; files in `design/ui/project/`.
- Google Stitch connected (key: Admin-Manual `CREDENTIALS/global-infrastructure/google-stitch.env`).
  - New project `8666922522707666120` "Natally v2.0 — latest UI (4 Nov 2026)": 10 screens in `design/stitch-v2/` (PNG, HTML, notes, raw replies).
  - Stitch returns screens only in each generate/edit reply (list_screens stays empty); downloads are JSON `{dataString}` (PNG base64, HTML plain text).
  - Corrected by edit: atelier (real chibi Natally, placeholder chat), PDF pages (Robin's 4 changes). Souvenir offer + souvenir atelier mascot/placeholder edits were running at hand-off (task output: `_reply-Souvenir-*.json`).
  - Old 22 Oct Stitch export kept as `design/stitch-2026-10-22-OUTDATED-reference-only/`.
- UI decisions from Robin's canvas comments: `DOCS/UI-DECISIONS.md` (no disclaimers; Natally Gen Z aside per deep-dive page; element chips with glyphs, links and hover relation; chip links to hosted site with upsells, split test; "Hear Natally" badge in PDF opens hosted atelier; Stitch art and chart text are not used — real assets and computed values are).

## Next (in order)
1. Review the two souvenir screens after their edit; then FREEZE the UI (Stitch v2 + canvas + UI-DECISIONS).
2. Write `DOCS/ARCHITECTURE.md` for v2 against the frozen UI, with an exhaustive entity table. Must carry over from v1's R13/R13b: RevenueCat code `ROCHE` only (1 ROCHE = 1 Cr = US$0.01; purchase grants 40% of pre-tax paid), purchase history = every RevenueCat transaction for the user id, Top Up opens 5/10/25 packs, souvenir US$30 with content generator, Stack SFTP/FTP hosting + SMTP, 60 s keys with remaining limit, no NATALTURN/AICREDIT.
3. `CHECKLIST.md` from it; coder subagents implement rows; evaluate semantically with CodeGraph and real flows (no unit tests).
4. A build script that sets versions automatically (Admin-Manual Rule 0b).

## Global rules to follow (Admin-Manual DOCS/TOOLING_CONVENTIONS/global-agent-rules.md)
Rule 0a (UI first → CodeGraph → ARCHITECTURE → CHECKLIST → code), 0b (versions only from the build script), 0c (items 0–16, 1a–1h: pipeline with coder subagents and semantic evaluation; root cause first; architect fully, one iteration; check everything up to your work; breakage → back to architecture; no unit tests; no disclaimers; LAST LEFT-OFF POINT in the session title; task list every turn).
