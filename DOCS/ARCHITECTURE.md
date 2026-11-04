# Natally v2.0 — ARCHITECTURE

Written 2026-11-04 against the FROZEN UI (`design/FROZEN.md`). Greenfield: nothing is
imported from natally-ai-studio code. Operator terms: one-shot, the agent owns the whole
result. Rules: Admin-Manual global rules 0a, 0b, 0c. Anything not in the entity tables
below is not built.

## 1. Product in one paragraph

Natally is a cute robot astrologer in a candlelit atelier. She calculates the customer's
natal chart exactly, explains any planet, sign, house or aspect for free with authored
lessons, and gives personal spoken readings paid with credits (Cr). A US$30 souvenir turns
the customer's reading into a keepsake: a generated PDF and a hosted frozen copy of the
atelier. Android ships as a native Google Play app; Windows and Linux as desktop apps; the
web app runs at the new Natally domain. The old v1 app keeps running at natally.robin.mba
and natally.surge.sh in parallel.

## 2. System shape

```
             ┌──────────────── client (one codebase) ────────────────┐
 Web (domain) │ React + Vite UI  ─ chart engine (Swiss Ephemeris WASM)│
 Android (Play)│ Tauri 2 shell   ─ voice (on-device TTS)               │
 Win / Linux   │                  ─ RevenueCat SDK (web: card; Android: Play Billing)
             └──────────┬───────────────────────────┬────────────────┘
                        │ Firebase ID token          │ direct, short-lived key
                        ▼                            ▼
          billing function (Netlify, existing)    OpenRouter (inference)
          · reads RevenueCat (secret key)
          · grants ROCHE, mirrors to Firebase
          · mints 60 s OpenRouter keys
          · souvenir delivery: SFTP/FTP + SMTP
                        │
        RevenueCat (authority)   Firestore (working copy)   Stack shared hosting
```

Hosting of the billing function stays on the existing Netlify mechanism until the approved
Oracle Always Free target exists (CLAUDE.md infrastructure boundary).

## 3. Authorities (single source of truth per fact)

| Fact | Authority | Working copy |
|---|---|---|
| Identity | Firebase Auth | — |
| Purchases, transactions | RevenueCat | Firestore receipts (`credit_grants`) |
| Credit balance | RevenueCat virtual currency code `ROCHE` | Firestore `accounts/{uid}.creditUnits` |
| Chart | computed on device from birth details | Firestore `charts/{uid}` |
| Conversation | Firestore `conversations/{uid}` | device cache |
| Souvenir | Firestore `souvenirs/{uid}/{purchaseId}` (create-only) | hosted files on Stack |

Currency rules: only RevenueCat code `ROCHE`. 1 ROCHE = 1 Cr = US$0.01 of inference spend
limit. A purchase grants `floor((gross − tax) × 100 × 0.40)` ROCHE, once per RevenueCat
purchase id. NATALTURN and AICREDIT do not exist. Prices are RevenueCat/env configuration,
never code.

## 4. Screens (frozen) → components

| Screen (design/stitch-v2) | Component | Notes |
|---|---|---|
| Landing desktop / chart-in-view | `src/landing/Landing.tsx` | sticky hero (atelier + inset chart + controls), copy beside it on wide screens, under it on phones; chart section in view → inset chart zooms to the main view, out of view → back (IntersectionObserver) |
| Sign in (as v1.42) | `src/account/SignIn.tsx` | "Readings by Natally", Google; Email Log In / Sign Up / Phone tabs; gold Log In |
| Birth details | `src/account/BirthDetails.tsx` | month names (never numeric dates); unknown birth time |
| App atelier | `src/atelier/Atelier.tsx` | header: name, small "N Cr · Top Up", "Souvenir", Profile (SVG icon); scene with real Natally art; "Move chart" inset; Sound On; element chooser; Share chart · Readings · My Horoscope · Synastry · Room · Your chart · Pause motion; conversation |
| Profile | `src/account/Profile.tsx` | name, email; balance; Top Up; ALL transactions; small Restore; Sign out at the bottom |
| Top Up | `src/billing/TopUp.tsx` | "What credits do", then 5/10/25 packs, then payment |
| Souvenir offer + delivery | `src/souvenir/SouvenirOffer.tsx` | inclusions, US$30, buy; ready state with atelier + PDF links |
| Souvenir atelier | `src/souvenir/SouvenirViewer.tsx` (+ static build `souvenir.html`) | frozen export, no inference; new question → authored Top Up reply |
| Souvenir PDF | `src/souvenir/pdf/` | group cover + 2 pages per element; layout from Stitch, text from generator |

Shared: `src/chart/` (engine, wheel, element model), `src/voice/`, `src/ui/` (tokens from
Stitch design theme: surface #161021, primary #ebc165 / #c9a24a, Fraunces, Cinzel, Nunito Sans).

## 5. Entities

### 5.1 Chart
| Entity | Signature | Role |
|---|---|---|
| `calculateChart` | `(BirthDetails) → Chart` | Swiss Ephemeris WASM; Placidus default |
| `Chart` | `{ person, angles{asc,mc,dsc,ic}, bodies[], houses[12], signs[12], aspects[] }` | only computed values ever reach the UI or generator |
| `elementsFor` | `(Chart) → Element[]` | planets, 12 signs, 12 tightest aspects, 12 houses; each with id, glyph, label, `yourRelation` text |
| `ElementChip` | `<ElementChip element>` | rounded subtle border, glyph, link to the element's page, hover/focus callout with `yourRelation` |

### 5.2 Readings and inference
| Entity | Signature | Role |
|---|---|---|
| `authoredLesson` | `(elementId, Chart) → text` | free explanations, no inference |
| `readingKey` | `() → {key, expiresAt}` | before every inference: use held key if valid and funded, else `/billing/key` |
| `/billing/key` | `POST → {key, limitUsd, expiresAt}` | OpenRouter key, limit = remaining spend limit, expires now + 60 s |
| `streamReading` | `(question, Chart, history, key) → stream` | OpenRouter, model chain from env |
| out-of-credit reply | authored | "Add credits for your personal reading" → opens Top Up packs |

### 5.3 Billing (server: `billing/`)
| Entity | Signature | Role |
|---|---|---|
| `listPurchases` | `(uid) → RCPurchase[]` | RevenueCat v2 REST with secret key; all pages |
| `grantPaidPurchases` | `(uid) → {granted}` | every owned production purchase → ROCHE once (idempotency key = purchase id); no checkout id / webhook precondition |
| `readBalance` | `(uid) → creditUnits` | RevenueCat `ROCHE` balance; mirrored to Firestore |
| `/billing/access` | `GET → {creditCents, transactions[]}` | runs grant + read; transactions = every RevenueCat purchase for the uid (product, gross, date, status, refunds) + redemptions; on provider failure the actual error, never a false "none" |
| `/billing/redeem` | `POST {code}` | code's own record defines behaviour; env admin code adds 400 Cr per redemption |
| `/billing/webhook` | `POST` (RevenueCat) | triggers `grantPaidPurchases` early; never required |
| client `useAccount` | hook | reads the RevenueCat SDK customer first (transactions shown immediately), then `/billing/access`; Restore runs the same |
| `TopUpFlow` | Top Up and chat "Add credits" | open 5/10/25 packs and start that purchase; souvenir offered only from Souvenir buttons |

### 5.4 Souvenir
| Entity | Signature | Role |
|---|---|---|
| `createSouvenir` | `(Chart, conversation, purchaseId) → Snapshot` | create-only, keyed by purchase id |
| `generateSouvenirContent` | `(Snapshot, key) → Generated` | per element: cover (title, subtitle, yourRelation), deep dive (origins, traditional interpretation, lore, examples), Natally aside (eager, Gen Z); per group: cover title + intro; resumable; inputs only the computed chart and conversation |
| `buildSouvenirPdf` | `(Snapshot+Generated) → Blob` | letter; group cover + 2 pages per element; chips as links; "Hear Natally" badge per page → hosted atelier page; no disclaimers |
| `/billing/souvenir-deliver` | `POST {purchaseId, snapshot, pdf}` | verifies the purchase in RevenueCat; uploads `souvenir.html` build + snapshot.json + souvenir.pdf to `SOUVENIR_HOST_ROOT/<uid>/<purchaseId>/` (SFTP/FTP); emails both links (SMTP); writes `atelierUrl`, `pdfUrl` to the receipt |
| Profile souvenir row | — | "Open your atelier" and "Souvenir PDF" links |

### 5.5 Build and release
| Entity | Role |
|---|---|
| `build_all.sh` | the only build entry; stamps the version automatically on every run (Rule 0b); deploys web first to the configured domain (`WEB_DOMAINS`), then Android AAB + symbols + APK, Windows EXE/NSIS (cross), Linux AppImage/DEB/RPM |
| env | every domain, price, product id, key and host comes from `.env` resolved from Admin-Manual pointers; nothing hardcoded |

## 6. Configuration (env names)

`APP_NAME`, `APP_IDENTIFIER`, `WEB_DOMAINS`, `VITE_FIREBASE_*`, `VITE_REVENUECAT_PUBLIC_KEY`,
`VITE_REVENUECAT_ANDROID_PUBLIC_KEY`, `REVENUECAT_SECRET_API_KEY`, `REVENUECAT_PROJECT_ID`,
`CREDIT_PACKS_JSON` (5/10/25 → RevenueCat package ids), `SOUVENIR_PRODUCT_ID`,
`BILLING_ALLOCATION_BPS=4000`, `ADMIN_REDEMPTION_CODE`, `ADMIN_REDEMPTION_CR=400`,
`OPENROUTER_ADMIN_KEY`, `OPENROUTER_MODELS`, `SOUVENIR_HOST_*`, `SOUVENIR_SMTP_*`,
`SURGE_TOKEN` (or the new host's deploy credential).

## 7. Verification (no unit tests)

Each checklist row is verified by the real flow and checked semantically with CodeGraph
against this document: sign in → chart → free lesson → Top Up 5/10/25 → balance shows
the granted Cr → personal reading streams → profile lists every transaction → souvenir
purchase → PDF + hosted atelier links in history and email. Robin's own account
(79 ROCHE, two US$0.99 purchases) must show 79 Cr and both transactions.

## 8. Planned, not in this build

On-device inference on Android (why Android steers to the native app); Alby per-turn and
x402 rails; split test of PDF chip links (hosted site vs in-PDF).
