# v2 UI — FROZEN 2026-11-04 13:00 UTC

The UI below is the input to v2 ARCHITECTURE. Changes after this point go through
architecture first (Admin-Manual Rule 0a); they are not ad hoc code edits.

Source of truth, in precedence order:
1. `DOCS/UI-DECISIONS.md` (Robin's rulings during design).
2. `design/stitch-v2/` — Stitch project 8666922522707666120 "Natally v2.0 — latest UI (4 Nov 2026)":
   Landing (desktop, incl. Google Play install), Landing chart-in-view, Sign in (as v1.42),
   Birth details, App atelier (v1.42 layout, real Natally), Profile (all transactions, sign out
   at bottom), Top Up (what credits do, then 5/10/25), Souvenir offer + delivery,
   Souvenir atelier (frozen export), Souvenir PDF pages (layout only).
3. `design/ui/project/` — Claude Design canvas https://claude.ai/artifact/KZY7iqNENoN5hcpy9WdHTb
   (interaction behaviour: sticky hero, chart zoom on scroll, pack selection, souvenir flow,
   element chips with hover relation).
4. `design/reference-screens/v1.42.95682/` — the real v1.42 screens the UI is based on.

Not part of the frozen UI: Stitch's mascot art and any chart values or lore text Stitch wrote.
The build uses `design/assets/` art and computed chart values only.
