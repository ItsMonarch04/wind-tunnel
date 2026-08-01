# Product Studio

**A wind tunnel for SaaS pricing & packaging decisions.** Model your buyers as explicit assumptions, design tiers and fences as a screening mechanism, and watch segments self-select — revealing revenue, conversion, and the surplus you're leaving on the table, before you ship a price.

> **Status: planning (v0.0.1).** The build plan is complete ([CONTEXT.md](CONTEXT.md)); application code lands phase by phase. This README stays honest about what runs today: nothing yet.

## The idea

Most SaaS packaging is decided by gut and competitor-copying. But Good/Better/Best isn't decoration — it's a *screening mechanism* (second-degree price discrimination): a menu designed so buyers with different willingness-to-pay sort themselves into the right tier. Once you write your assumptions down — who the segments are, what they'd pay, which features they value — the menu's consequences stop being opinions. They can be computed.

This studio runs entirely on **your assumptions, made explicit**. It is not a billing tool and not an optimizer that needs live revenue data. It's useful before your first customer exists: a think-before-you-ship reasoning layer for the highest-leverage decision most SaaS teams make on vibes.

## What it does (v1 scope)

- **Model** — buyer segments with willingness-to-pay *distributions* (not point guesses), a segment × feature value matrix, and provenance tags for every assumption (guess / interview / survey / conjoint / benchmark).
- **Design** — tiers, feature fences, free tier, add-ons, flat or per-seat pricing — with a built-in linter that flags dominated tiers, fence inversions, free-tier leakage (in dollars), cannibalization, and competitor loss.
- **Simulate** — closed-form self-selection: every edit instantly re-sorts buyers into tiers. Revenue, conversion, ARPA, capture rate, competitor-loss share; a value waterfall of surplus captured vs. left on the table; per-tier demand curves; A/B two designs against the same buyers.
- **Analyze — Uncertainty** — Monte Carlo over your assumption uncertainty with a tornado chart that tells you *which assumption to validate first*.
- **Analyze — Research** — Van Westendorp Price Sensitivity Meter, Choice-Based Conjoint with MNL part-worth estimation (and a price-attribute → WTP bridge back into the value matrix), MaxDiff-lite importance scoring, bundling economics (pure vs. mixed).
- **Analyze — Positioning** — a competitive price-value map (frontier hull, iso-utility rays) plus each competitor entered as a real alternative in the simulator — so "what % of enterprise would leave for this competitor?" is a number, not a picture.
- **Communicate** — a pricing-page mock and an exportable Pricing Decision Record: the assumptions, the design, the numbers, and the why — the artifact you defend in the room.

The economics is the point: the engine is one rigorously tested primitive (utility upper-envelope selection over an offer menu), with every formula unit-tested against closed-form results and a value-conservation identity enforced on every simulation. See [CONTEXT.md](CONTEXT.md) §4 for the math contract.

## What it is not

No accounts, no server, no telemetry, no data connectors. Fully client-side: your assumptions never leave your browser. Scenarios save locally and share as self-contained URLs.

## Stack

Next.js (App Router, static export) · React · TypeScript (strict) · Tailwind CSS · Zustand · Zod — fully client-side, no server, deployed on Vercel (the built bundle stays portable to any static host). Light + dark themes, both WCAG-AA audited. Hand-rolled SVG visualization (no chart libraries). Tested with Vitest + fast-check (property tests against closed-form economics) and Playwright.

## Roadmap

| Phase | Delivers | Status |
|---|---|---|
| P0 | Scaffold, CI, deployed shell | — |
| P1–P2 | Economics engine (selection, KPIs, surplus, sweeps) + math spec | — |
| P3 | Scenarios: persistence, import/export, share-by-URL, templates | — |
| P4–P5 | Model & Design surfaces + design linter | — |
| P6 | The wind tunnel: live reveal, mechanism view, A/B | — |
| P7 | Analyze: Monte Carlo + tornado, Van Westendorp, bundling, Conjoint (CBC) + MaxDiff, competitive positioning | — |
| P8 | Communicate: pricing page mock + decision record export | — |
| P9 | Hardening, accessibility & performance audit → **v1.0.0** | — |

## Project docs

- [CONTEXT.md](CONTEXT.md) — working/handoff doc: full build plan, economic spec, decisions ledger, verification protocol.
- `docs/MODEL-SPEC.md` — the math contract (extracted in Phase 1; every engine test cites it).

## License

[MIT](LICENSE) © 2026 Sidakpreet Singh
