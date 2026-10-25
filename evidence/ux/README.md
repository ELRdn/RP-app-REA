# UX measurement evidence

Same-harness walkthrough of the primary "first chat" flow on 2026-10-25.

- Viewport: 390×844 / DPR2 / ja-JP / Asia-Tokyo
- Browser: system Chrome, headful, Playwright, anti-automation init script
- Neutral prompt: `こんにちは。今日は何をしていますか？` (Zeta / MiraiMind)
- "Taps" count pointer clicks / Enter presses, not typing
- Times are wall-clock single samples and include SPA settling waits. Use deltas (tap→next screen, send→reply), not absolute page-load times.

## Flow captures

| App | Step 1 | Step 2 | Step 3 | Step 4 | Step 5 |
|---|---|---|---|---|---|
| Zeta | home | plot detail | chat | AI reply | — |
| SynClub | feed | QR gate | — | — | — |
| MiraiMind | discover | chat | AI reply | — | — |
| CrushOn | gender onboarding | home | detail sheet | chat | login gate |

## Key numbers

| Metric | Zeta | SynClub | MiraiMind | CrushOn |
|---|---|---|---|---|
| Taps to chat | 2 | — | 1 | 3 |
| Taps to send | 3 | — | 2 | 4 |
| Card→chat | 2.13s | — | 1.05s | 2.09s |
| Send→reply | 3.19s | — | 2.96s | blocked |
| Gate | soft app/login | QR | 5-request / feature | login banner |

Raw JSON:

- `raw-measurement-1.json` — first pass (SynClub, CrushOn; Zeta chat selector failed)
- `raw-measurement-2.json` — Zeta and MiraiMind with reply-selector fix

Curated data: `data/ux-metrics.json`, `data/ux-flows.json`.

## Unknowns

- Single sample; network/A-B variance is not modelled.
- CrushOn cannot send as a guest, so reply latency is unknown.
- MiraiMind's 5-request limit is from the runtime log `requestLimit: 5`; the 6th request was not forced.
- Native app UX (voice calls, notifications, purchase sheets) is not measured.
