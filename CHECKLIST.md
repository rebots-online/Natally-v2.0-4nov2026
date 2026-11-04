# Natally v2.0 — CHECKLIST

Source: `DOCS/ARCHITECTURE.md` (43b0a8e). Every row names the ARCHITECTURE entity it builds;
a row with no entity is not built. Colours are subproject divisions (red onboarding/mobile,
blue readings/inference, purple voice, green accounts/sync, yellow billing, brown
launch/infrastructure). Status: ⬜ not started · 🔨 coded · ✅ verified by real flow +
CodeGraph against ARCHITECTURE. No unit tests.

## 🟤 0. Foundation
| # | Row | Entity (§) | Status |
|---|---|---|---|
| 0.1 | Vite + React + TS scaffold, Tauri 2 shell (Android, Windows, Linux targets) | §2 client | ⬜ |
| 0.2 | `src/ui/` tokens from the Stitch theme (surface #161021, primary #ebc165/#c9a24a, Fraunces, Cinzel, Nunito Sans) | §4 Shared | ⬜ |
| 0.3 | `.env.example` with every name in §6; loader resolves Admin-Manual pointers at build time only; nothing hardcoded | §5.5 env, §6 | ⬜ |
| 0.4 | `build_all.sh`: stamps version every run (Rule 0b); web deploy first to `WEB_DOMAINS`; then AAB+symbols+APK, Windows EXE/NSIS, Linux AppImage/DEB/RPM | §5.5 `build_all.sh` | ⬜ |
| 0.5 | CodeGraph index of this repo (`codegraph init -i .`) | §7 | ⬜ |

## 🔴 1. Onboarding and landing
| # | Row | Entity (§) | Status |
|---|---|---|---|
| 1.1 | Landing: sticky hero (atelier + inset chart + controls), copy beside it wide / under it on phones; IntersectionObserver zooms inset chart in/out; Install section → Google Play + desktop apps | §4 `Landing.tsx` | ⬜ |
| 1.2 | Sign in as v1.42: "Readings by Natally", Google; Email Log In / Sign Up / Phone tabs; gold Log In (Firebase Auth) | §4 `SignIn.tsx`, §3 Identity | ⬜ |
| 1.3 | Birth details: month names, unknown birth time | §4 `BirthDetails.tsx` | ⬜ |

## 🔵 2. Chart and readings
| # | Row | Entity (§) | Status |
|---|---|---|---|
| 2.1 | `calculateChart(BirthDetails) → Chart` on Swiss Ephemeris WASM, Placidus default | §5.1 `calculateChart`, `Chart` | ⬜ |
| 2.2 | `elementsFor(Chart)`: planets, 12 signs, 12 tightest aspects, 12 houses, each with id/glyph/label/`yourRelation` | §5.1 `elementsFor` | ⬜ |
| 2.3 | `ElementChip`: rounded subtle border, glyph, link to element page, hover/focus callout | §5.1 `ElementChip` | ⬜ |
| 2.4 | Atelier: header (name, "N Cr · Top Up", Souvenir, Profile SVG), real Natally art, Move chart inset, Sound On, element chooser, action row, conversation | §4 `Atelier.tsx` | ⬜ |
| 2.5 | `authoredLesson(elementId, Chart)`: free, no inference | §5.2 `authoredLesson` | ⬜ |
| 2.6 | `readingKey()` + `streamReading(question, Chart, history, key)` via OpenRouter, models from `OPENROUTER_MODELS` | §5.2 | ⬜ |
| 2.7 | Out-of-credit authored reply → opens Top Up packs | §5.2 out-of-credit reply | ⬜ |
| 2.8 | Conversation stored at Firestore `conversations/{uid}`, chart at `charts/{uid}` | §3 | ⬜ |

## 🟣 3. Voice
| # | Row | Entity (§) | Status |
|---|---|---|---|
| 3.1 | `src/voice/`: on-device TTS speaks readings; Sound On toggle | §2, §4 Shared | ⬜ |

## 🟡 4. Billing (server `billing/`, Netlify)
| # | Row | Entity (§) | Status |
|---|---|---|---|
| 4.1 | `listPurchases(uid)`: RevenueCat v2 REST, secret key, all pages | §5.3 | ⬜ |
| 4.2 | `grantPaidPurchases(uid)`: each owned production purchase → `floor((gross−tax)×100×0.40)` ROCHE once, idempotency key = purchase id; no checkout-id/webhook precondition | §5.3, §3 | ⬜ |
| 4.3 | `readBalance(uid)`: RevenueCat `ROCHE`, mirrored to `accounts/{uid}.creditUnits`; 1 ROCHE = 1 Cr | §5.3, §3 | ⬜ |
| 4.4 | `/billing/access`: grant + read; transactions = every RevenueCat purchase for the uid + redemptions; provider failure returns the error, never "none" | §5.3 | ⬜ |
| 4.5 | `/billing/key`: OpenRouter key, limit = remaining spend limit, expires now + 60 s | §5.2 | ⬜ |
| 4.6 | `/billing/redeem`: code record defines behaviour; `ADMIN_REDEMPTION_CODE` adds `ADMIN_REDEMPTION_CR` | §5.3 | ⬜ |
| 4.7 | `/billing/webhook`: triggers grant early; never required | §5.3 | ⬜ |
| 4.8 | Client `useAccount`: RevenueCat SDK customer first, then `/billing/access`; Restore runs the same | §5.3 | ⬜ |
| 4.9 | `TopUp.tsx` + `TopUpFlow`: "What credits do", 5/10/25 packs from `CREDIT_PACKS_JSON`, payment (web card / Android Play Billing) | §4, §5.3 | ⬜ |

## 🟢 5. Account
| # | Row | Entity (§) | Status |
|---|---|---|---|
| 5.1 | Profile: name, email, balance, Top Up, ALL transactions, small Restore, souvenir rows, Sign out at the bottom | §4 `Profile.tsx`, §5.4 Profile souvenir row | ⬜ |

## 🟡 6. Souvenir
| # | Row | Entity (§) | Status |
|---|---|---|---|
| 6.1 | `SouvenirOffer.tsx`: inclusions, US$30 (`SOUVENIR_PRODUCT_ID`), buy; ready state with links | §4 | ⬜ |
| 6.2 | `createSouvenir(Chart, conversation, purchaseId)`: create-only `souvenirs/{uid}/{purchaseId}` | §5.4 | ⬜ |
| 6.3 | `generateSouvenirContent(Snapshot, key)`: per element cover + deep dive + Natally Gen Z aside; per group cover; resumable; inputs only computed chart + conversation | §5.4 | ⬜ |
| 6.4 | `buildSouvenirPdf`: letter; group cover + 2 pages per element; chips as links; "Hear Natally" badge; no disclaimers | §5.4, §4 `pdf/` | ⬜ |
| 6.5 | `SouvenirViewer.tsx` + static `souvenir.html`: frozen export, no inference; new question → authored Top Up reply | §4 | ⬜ |
| 6.6 | `/billing/souvenir-deliver`: verify purchase in RevenueCat; SFTP/FTP to `SOUVENIR_HOST_ROOT/<uid>/<purchaseId>/`; SMTP email of both links; write `atelierUrl`, `pdfUrl` to receipt | §5.4 | ⬜ |

## 🟤 7. Release and verification
| # | Row | Entity (§) | Status |
|---|---|---|---|
| 7.1 | Web deployed to whatever `WEB_DOMAINS` holds in `.env` (nothing waits on a domain purchase; DNS set via Namecheap connector when a domain is named); old v1 URLs untouched | §1, §5.5 | ⬜ |
| 7.2 | Real-flow pass: sign in → chart → free lesson → Top Up 5/10/25 → Cr shown → reading streams → profile lists every transaction → souvenir → PDF + atelier links in history and email | §7 | ⬜ |
| 7.3 | Robin's account shows 79 Cr and both US$0.99 transactions | §7 | ⬜ |
