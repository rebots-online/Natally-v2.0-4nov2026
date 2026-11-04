# UI decisions captured during UI design (2026-11-04, Robin via canvas comments)

- No unrequested disclaimers anywhere in product content (Admin-Manual Rule 0c 1h).
- PDF deep dive: Natally art with an eager Gen Z aside on every deep-dive page; text flows around her.
- Chart elements in text are chips (subtle rounded border, glyph where one exists), clickable to that element's page; hover/focus shows the customer's own relation to it (web/atelier; PDF keeps links only).
- Chip links: eventually to the hosted interactive souvenir site (with upsells); split-test against internal PDF links.
- Natally voice/animation: PDF media does not play in most viewers; each PDF page carries a "Hear Natally" badge linking to that page in the hosted atelier, where she animates and speaks it.
- Stitch draws its own mascot and invents chart facts; the build uses the real Natally assets in design/assets/ and only computed chart values.
- Stitch PDF pages are the LAYOUT reference only: their chart values and lore text were invented by Stitch; the build uses computed chart values and the content generator's text.
- Stitch atelier header leaks an icon-font name ("person") where the Profile icon belongs; the build uses an inline SVG icon.
- Android: the landing steers Android visitors to the native Google Play app (quality, voice, and room for on-device local inference later), not the PWA; Windows and Linux get the desktop app (Robin 2026-11-04).
- Deployment: the old app (natally.robin.mba, natally.surge.sh, v1.x from natally-ai-studio) keeps working in parallel; v2 is deployed alongside on its own address and never replaces the old URLs without Robin's say (Robin 2026-11-04).
- Domain: Robin is buying a dedicated Natally domain for branding; v2 goes live there (domain set in env/build config, never hardcoded), alongside the old natally.robin.mba and natally.surge.sh (Robin 2026-11-04).
