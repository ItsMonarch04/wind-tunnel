# CONTEXT.md — Working Handoff

**Project:** product-studio — a SaaS pricing & packaging studio (product name kept as **Product Studio** for now; see §11)
**Doc role:** the agent working file — plan, decisions ledger, math spec, backlog, verification protocol, session log. The public-facing story lives in README.md; the two are kept distinct on purpose. This doc is the source of truth for the build.
**Version:** v0.0.2 · **Status:** review-hardened per owner S2/S2b — oracle reconciliation, corrected economic/positioning/Conjoint contracts, bounded sharing, and an unambiguous v1.0/v1.1 release contract; v0.0.1 committed post-dated 2026-07-13 15:00 · **Last updated:** 2026-07-13

---

## 0. How to use this doc (agent protocol)

1. Read §1–§3 for intent, §4 for the math you must implement, §6 for your phase, §7 for how your work is judged, §8 for conventions.
2. Precedence on conflict: Decisions ledger (§9) > this doc's body > README. Math truth is §4 until Phase 1 extracts it to `docs/MODEL-SPEC.md`; after that, MODEL-SPEC is the math contract and §4 defers to it.
3. Every session: append a Session log entry (§13). Every consequential choice you make: add a ledger entry (§9). Never silently deviate from the plan — deviate loudly, in the ledger.
4. **Commit protocol.** Never commit or push without an explicit ask from the owner. When you believe a commit is due: (a) stop, (b) propose the exact **version number** (per §8.3 mapping) and the exact **commit message** (per §8.3 format), (c) list the staged files, (d) wait for the owner's explicit go-ahead on both the version and the message before running `git commit`. No exceptions. Stage, propose, wait.
5. New runtime dependencies require a ledger entry and owner ping. Dependency budget: ≤ 7 production deps (§3.4).
6. Phase gate: complete the phase's acceptance checklist (§6), run the verification protocol (§7.5), update this doc, stop for review.

---

## 1. Vision & core loop

**User & job.** A founder, PM, or pricing owner at an early-to-mid-stage SaaS is designing or revisiting packaging and wants a decision they can *defend* — before committing, without a consultant, and before any billing or CRM data exists.

**Thesis.** Packaging is usually decided by gut and competitor-copying, but Good/Better/Best is really a *screening mechanism* (second-degree price discrimination): a menu designed so that buyer segments with different willingness-to-pay sort themselves. If you make the buyer assumptions explicit — WTP distributions, how each segment values each feature — the menu's consequences stop being a matter of opinion. They can be computed.

**Core loop.** **Model** the buyers (segments, WTP distributions, segment × feature value matrix) → **Design** the packaging (tiers, fences, add-ons, free tier, price metric) → **Simulate** self-selection (each buyer picks the utility-maximizing offer; closed-form, instant) → **Reveal** the economics (revenue, conversion, capture rate, surplus kept by buyers, value lost to fencing and non-conversion, cannibalization) → tighten the assumptions that matter most (sensitivity tells you which) → repeat.

**Positioning.** Runs entirely on the user's assumptions, made explicit. Not a billing tool (Stripe/Paddle), not a data-driven optimizer that needs live revenue data. Useful before the first customer exists. Fully client-side: no accounts, no server, no analytics; data stays in the browser, scenarios share as URLs.

**The honest answer to "it's just your assumptions."** Yes — deliberately. The studio's stance: (1) you are already pricing on assumptions, just implicit ones; (2) the simulation shows what your assumptions *imply*, which is where the surprises live; (3) the sensitivity layer + assumption-provenance tags tell you which assumption drives the answer, so validation effort goes where it pays; (4) when you *do* run a survey, PSM and Conjoint/MaxDiff (§4.7, §4.10) fold their findings into the same model — assumption → evidence, in one loop. The output isn't "the right price" — it's a defensible decision plus a ranked list of what to validate first.

**What keeps this out of slop territory** (checkable claims, not vibes):
- One rigorous economic primitive powers everything (§4.2): the utility upper envelope over an offer set. Tiers, free tier, add-ons, bundling analysis, and competitor alternatives are all the same math — no per-feature hand-waving.
- Every formula in §4 has a named unit test against a closed-form or independently computed result (§7). A value-conservation identity holds to 1e-6 on every simulation, enforced by property tests.
- A "mechanism view" shows the actual screening picture — utility lines vs. buyer type, indifference breakpoints — for people who know the textbook figure. The economics is inspectable, not a black box.
- Behavioral effects (decoy, anchoring, choice overload) are handled as deterministic *linter checks with citations*, never as invented numeric uplift multipliers (§4.9, ledger D-08).
- Research methods are shipped *complete when released* — v1.0 includes PSM and sensitivity; v1.1 adds Conjoint/MNL, MaxDiff, and bundling under the contracts in §4. No shallow "conjoint-flavored" placeholders (ledger D-07).

---

## 2. Scope

### 2.1 v1.0 core + v1.1 extensions

| Surface | Contents |
|---|---|
| **Model** | Segments (2–6): size, seat count, median account-level WTP with P10/P90 band, within-segment spread, segment × feature value matrix (entered as total WTP × allocation shares, editable directly in advanced view), assumption provenance tags (guess / interview / survey / conjoint / benchmark + confidence) |
| **Design** | 1–5 tiers as feature fences over a shared catalog (≤ 12 features); flat or per-seat price metric; free tier (price 0); up to 3 add-ons; multiple designs per scenario for A/B |
| **Simulate** | Always-live wind tunnel: closed-form self-selection per segment; KPIs (MRR, paid conversion, ARPA, capture rate); buyer-dot sorting viz; value waterfall (revenue / buyer surplus / fencing loss / non-conversion); per-tier price-sweep revenue curves; A/B compare with deltas and win-rate under uncertainty; mechanism view (envelope diagram); competitor-choice-share when the Positioning layer is active |
| **Analyze — Uncertainty** | Monte Carlo over assumption uncertainty (seeded, deterministic) with P10/P50/P90 and tornado chart, tornado × provenance → validation to-do list |
| **Analyze — Research** | Van Westendorp PSM (manual/CSV survey input, validation, interpolated crossing points, optional illustrative mode clearly labeled); Choice-Based Conjoint (attributes/levels, design generator, respondent input, pooled MNL part-worth estimation, and an explicitly mapped WTP proposal); MaxDiff-lite importance scoring; bundling economics (pure vs. mixed). Estimator engines are v1.0 quality gates; their full views are v1.1 extensions after the core ships (§6). |
| **Analyze — Positioning** | Competitor-as-alternative in the envelope engine (per-segment choice share choosing a competitor = quantified switching/churn risk) is in the core engine. The segment-scoped competitive map is a v1.1 extension: Pareto points and break-even rays, with a discrete dominance verdict rather than a fictitious interpolated market offer (§4.11). |
| **Critic** | Design linter: economic checks (dominated offers, fence inversion, free-tier leakage with $ counterfactual, downgrade/cannibalization mass, dead fences, competitor loss) + behavioral checks (choice overload, weak anchor) — deterministic triggers, cited explainers |
| **Communicate** | Pricing Decision Record (assumptions + provenance, design, simulated economics, sensitivity, research findings when available, linter findings, alternatives considered) exported as Markdown and print-CSS PDF — this is required for v1.0. The pricing-page mock is a v1.1 extension after the record and core loop are green (D-06). |
| **Templates** | 3 archetype scenarios as data presets: PLG collaboration tool (per-seat + free tier), API/infra (flat tiers + add-on), sales-led B2B (2 paid tiers + enterprise) — solve the cold start, double as e2e fixtures |

### 2.2 Deferred (v2+ backlog, §12) — with the cut rationale

- **Usage-based pricing engine.** Needs usage distributions per segment and price-as-function-of-consumption — a second engine. v1 models flat and per-seat metrics, where the screening insight lives. The value-metric question is still surfaced (Design field + Decision Record + linter guidance). (Ledger D-09.)
- **Trials & time dynamics** (retention, expansion, annual-vs-monthly terms). Temporal modeling is a different machine; v1 is a single-period screening model, stated plainly in the UI.
- **Joint price optimizer** ("suggest optimal prices"). The per-tier sweep + tornado gives the insight without over-claiming; a numerical joint optimizer invites "the tool said so" misuse. v1.1 candidate behind an explicit "local search, not truth" frame.
- **Hierarchical Bayes for Conjoint** (per-respondent part-worths). v1 does pooled MNL with pooled SEs — enough for the pedagogical arc and enough to bridge back to the value matrix with honest uncertainty; HB is a step-change in complexity that dilutes the correctness focus. (Ledger D-17.)
- **D-efficient design generation** for Conjoint. v1 uses random balanced designs (each level appears equally often per attribute); D-efficient search is a v1.1 refinement (ledger D-17).
- Currency/i18n, PNG/PPTX export, elasticity & substitution matrix readout, scenario version history. All listed in §12.

### 2.3 Explicitly out (any version)

Billing integration, live data connectors, accounts/auth, server-side anything, telemetry.

---

## 3. Architecture & stack

### 3.1 Decision

**Next.js (App Router) + React + TypeScript (strict) + Tailwind CSS.** Configured with `output: 'export'` for a fully static build — no Node runtime at request time, no serverless functions, no backend, ever, for v1. State in Zustand (+ zundo for undo); validation and schema versioning with Zod; scenario sharing via lz-string compressed URL hash; charts hand-rolled as React+SVG components; fonts via `next/font/local` (self-hosted Inter subset, no external network); tests with Vitest + fast-check + Testing Library + Playwright smoke; **deploy to Vercel** (owner-confirmed, S1) via the Next.js integration — the studio remains a plain static bundle behind Vercel's CDN.

### 3.2 Alternatives weighed

| Option | Verdict |
|---|---|
| **Next.js (App Router, `output: 'export'`, Vercel)** ✅ | Owner-fluent; App Router gives clean file-based structure without pulling in SSR; Next `output: 'export'` produces the same static bundle a plain Vite build would, but keeps `next/font`, `next/image`, and first-class TS/ESLint config on the shelf if we ever need them; Vercel deploys via repo-connect with zero config. Chosen (S1). |
| Vite + React + TS | Lighter toolchain, but reintroducing routing/asset-pipeline/font solutions we get free from Next; not owner-native. Rejected: owner fluency and free ecosystem win once we've decided against a backend anyway. |
| Vanilla ES modules | Maximally minimal, but this app is dense interdependent state (edit a WTP band → re-simulate → 8 views update). React's declarative model + TS types is materially safer for agent-built UI. Rejected for velocity and safety, not capability. |
| Chart library (Recharts/visx/d3) | Rejected as deps: ~18 chart instances, all bespoke (dot swarm, waterfall, envelope diagram, price-value map, tornado, PSM crossings — no library ships them). Hand-rolled SVG = full control, tiny bundle, and the craft is part of the showcase. `d3-*` micro-packages allowed later only via ledger entry. |
| Web Workers for Monte Carlo / MNL Newton-Raphson | Not needed by default: 1,000 MC draws over ≤ 6 segments × ≤ ~20 offers of closed-form arithmetic is sub-10 ms; MNL on ≤ 500 respondents converges in <10 Newton iterations in <100 ms. Revisit only if profiling disagrees (§7.6 perf gates). |
| Next SSR / server actions / RSC data fetching | Excluded by `output: 'export'` and by product stance (no server, no data). App Router is used purely as a static file-based structure. |

### 3.3 Module boundaries

```
app/            Next.js App Router routes:
  layout.tsx     root shell (header, footer, theme provider, aria-live region)
  page.tsx       studio (workbench + wind-tunnel; tabs are state, no sub-routes)
  globals.css    Tailwind entry
components/    UI components, views, charts (SVG), design tokens.
               May import lib/engine + lib/state.
lib/
  engine/      Pure TypeScript. No React, no DOM, no Date.now/Math.random
               (PRNG injected). Deterministic: same input → same output.
               ESLint-enforced: engine may not import from components/ or lib/state/.
    stats.ts       Φ, Φ⁻¹, erf, lognormal (pdf/cdf/quantile/partial expectation), band fitting
    offers.ts      offer expansion: tiers × add-on subsets + outside option (+ free tier + competitors)
    envelope.ts    upper-envelope selection: breakpoints, per-offer shares
    economics.ts   revenue, conversion, ARPA, surplus decomposition, capture rate,
                   counterfactual diffs (remove-offer), price sweeps
    montecarlo.ts  seeded PRNG (mulberry32), scenario draws, percentiles, tornado
    vanwest.ts     PSM validation, curve construction, intersection solving
    bundling.ts    regime comparison (components / pure bundle / mixed) + optimizer
    conjoint.ts    CBC: MNL log-likelihood + gradient + Hessian, Newton-Raphson estimator,
                   random-balanced design generator, price-attribute → WTP bridge
    maxdiff.ts     best-worst counting scores, normalization, design generator
    competitive.ts frontier (Pareto staircase, §4.11), iso-utility rays, competitor-offer synthesis into segment offer sets
    linter.ts      rule engine over scenario + simulation results
    index.ts       public API surface
  state/       Zod schemas (versioned), Zustand stores, persistence
               (localStorage autosave), JSON import/export, URL-hash codec.
content/      Glossary/explainer copy, linter messages with citations,
              template scenario data.
e2e/          Playwright smoke flows.
docs/         MODEL-SPEC.md (from Phase 1), screenshots.
public/       Static assets (favicon, font subset if not inlined via next/font).
```

**The load-bearing abstraction:** one primitive, `selectOffers(offers: {value, price}[], sigma, tieMode = 'conservative') → {activeOffers, breakpoints, shares}` (§4.2; `tieMode` is `'conservative'` everywhere in the app and `'seller-favorable'` only in the closed-form-oracle tests, D-20), consumed by tier simulation, free-tier counterfactuals, add-on choice, bundling regimes, *and* competitor alternatives alike. Every economics claim in the app reduces to this one tested function plus integration formulas (§4.3). This is deliberate: correctness concentrates where the tests are.

**Scenario data model:** one scenario = one buyer model + N designs (packaging variants) + optional competitor set + optional survey artifacts (PSM responses, Conjoint tasks/responses, MaxDiff tasks/responses) + settings `{seed, currency, theme}`. A/B compares designs *within* a scenario against the same buyers — apples to apples by construction. Autosaved to localStorage and exportable as versioned JSON (maximum 2 MiB before parsing). The URL payload is a separate, compact schema: it carries Model/Design/competitor/settings state only, never respondent-level survey records or derived research results; the Share UI says so and offers full JSON export for a complete handoff. The encoded hash is capped at 8 KiB and its decompressed form at 64 KiB before parsing; oversized or corrupt payloads fail closed with an actionable export/import message.

### 3.4 Dependency budget (hold the line)

Production: `next`, `react`, `react-dom`, `zustand`, `zundo`, `zod`, `lz-string` — **7, budget full** (bumped from 6 in S1 to accommodate `next`; ledger D-19). Dev: typescript, tailwindcss, vitest, fast-check, @testing-library/react, playwright, eslint (with `eslint-config-next`), prettier, axe-core (via playwright). Exact versions pinned at Phase 0 scaffold time and recorded in the ledger (latest stable at scaffold time; don't inherit this doc's assumptions about versions).

### 3.5 Non-functional requirements

- **Performance:** edit → re-simulate → repaint < 16 ms typical (closed form makes this cheap); Monte Carlo 1,000 draws < 150 ms end-to-end (the closed-form compute alone is ~sub-10 ms per §3.2; the 150 ms figure is a *ceiling* that also covers percentile + tornado aggregation and leaves margin — not a target); Conjoint MNL fit on ≤ 500 respondents < 100 ms; total client JS bundle ≤ 300 KB gzip (CI-gated; the 300 KB ceiling — raised from the pre-Next 250 KB working figure to absorb the Next static-export runtime overhead, D-01/S1 — with actual target "as small as possible under Next's static-export path", hand-audited in P0).
- **Privacy:** zero network calls at runtime, no external fonts/CDNs; Inter variable font subset via `next/font/local` (self-hosted). P0 enforces this two ways: static checks reject known network APIs/loaders, and a Playwright request allowlist fails any runtime request other than the document and same-origin static assets. The Vercel deployment sets `connect-src 'none'`; `docs/STATIC-HOST-HEADERS.md` records the equivalent header for portable hosts.
- **Accessibility:** WCAG 2.1 AA intent — full keyboard operation, every chart has a "view as table" toggle, `prefers-reduced-motion` respected everywhere, Okabe–Ito colorblind-safe categorical palette that meets contrast in both light and dark themes, axe CI gate with 0 serious/critical violations on both themes.
- **Determinism:** seeds visible and settable; identical scenario + seed → identical outputs, byte-stable exports (tested).

---

## 4. Economic engine & method catalog (the correctness contract)

Everything here is normative. Phase 1 extracts §4 into `docs/MODEL-SPEC.md` with section numbers preserved; every engine test cites the spec section it verifies (§7.3). Named tests (T-IDs) are the acceptance instruments — Opus reviews phases against them.

**Notation.** Segments `s` have prospect count `n_s`, seat count `c_s ≥ 1`, account-level per-feature median values `v[s,f] ≥ 0` ($/account/month for a representative account of `c_s` seats), and within-segment spread `σ_s`. Feature catalog `F`. An **offer** `o` has feature set `F_o` and list price `p_o`; flat `p_o` is $/account/month, while per-seat `p_o` is $/seat/month and has effective account price `P_s(o) = p_o · c_s`. Offer value `V_s(o) = Σ_{f ∈ F_o} v[s,f]` is always the account-level value for that segment (additive valuation — complementarity is v2, §12). The Model UI explicitly labels `W_s` as total account WTP at the stated seat count; it is not a per-seat WTP field.

### 4.1 Buyer model

Within segment `s`, an individual buyer draws a scale factor `ε ~ LogNormal(0, σ_s²)` (median 1) and values offer `o` at `ε · V_s(o)`. Buyer utility: `u(o) = ε·V_s(o) − P_s(o)`; outside option `u = 0`.

- Structure: **horizontal** differentiation across segments (different value *vectors*), **vertical** within a segment (one-dimensional scale ε). Within a segment, single-crossing holds by construction: offers rank identically for all buyers by `V_s`, and higher-ε buyers gain more from higher-V offers. This is a mixture of textbook vertical models (Mussa–Rosen / Tirole §3.5; Shapiro–Varian versioning), which is exactly the screening structure the product teaches.
- Input mapping: user enters median total WTP `W_s` for the full catalog and allocation shares `w[s,f]` (Σ_f w = 1) → `v[s,f] = W_s · w[s,f]`. Advanced view edits `v` directly.
- Two distinct uncertainty knobs, never conflated (UI must teach this): `σ_s` = *how much buyers within the segment differ* (analytic, §4.2); the P10/P90 **confidence band** on `W_s` and on `n_s` = *how unsure the user is* (Monte Carlo, §4.8).
- Lognormal band fitting: given quantiles `(q10, q90)` of a lognormal quantity, `μ = (ln q10 + ln q90)/2`, `σ = ln(q90/q10) / (2·z90)`, `z90 = 1.2815515655446004`.

Tests — **T-STAT-01** Φ matches reference values to 1e-7: Φ(0)=0.5, Φ(1)=0.8413447461, Φ(1.96)=0.9750021049, Φ(−3)=0.0013498980. **T-STAT-02** Φ⁻¹(Φ(x))=x to 1e-9 for x ∈ [−6, 6] grid. **T-STAT-03** band fit round-trips: quantile(fit(q10,q90), .1)=q10 and (.9)=q90 to 1e-9 rel. **T-STAT-04** lognormal partial expectation (§4.3) matches numeric integration (Simpson, 1e-6 rel) on 20 random (σ, a, b).

### 4.2 Offer selection — the upper envelope (the core primitive)

Buyer with scale ε chooses `argmax_o { ε·V_s(o) − P_s(o) }` including the outside option `(V=0, P=0)`. Exact ties (equal utility within the tie tolerance, §4.12) are resolved by an explicit **tie mode** — a parameter of `selectOffers`, default `'conservative'`:

- **`'conservative'`** — the product simulator everywhere. Break to **lower effective price, then lower value**, and prefer the outside option when a paid offer ties it. Conservative for revenue, deterministic, documented in-UI. This is the rule D-16 governs.
- **`'seller-favorable'`** — the closed-form-oracle validation harness only (§4.5 screening, §4.6 bundling), always at the σ→0 point-mass path. Break to **higher value, then higher price**, and prefer *participation* over the outside option on a tie. This is the δ→0 limit of pricing strictly inside each binding constraint: the textbook menus price served buyers exactly at a binding value, so the indifferent buyer is assumed to participate and take the intended (highest-value) offer. The two modes are exact mirror images.

Running the screening/bundling oracle fixtures under `'conservative'` sends every indifferent buyer to the outside option and reports revenue far below the textbook result (see the guard in §4.5) — that is a *test-configuration* error, not an engine bug. Fixtures not specifically exercising tie behavior should price strictly inside participation (no exact ties) so they read identically under either mode (D-20).

In ε-space each offer is a line `u_o(ε) = V_o·ε − P_o`. The chosen offer as a function of ε is the **upper envelope** of these lines: a sequence of active offers with increasing V and increasing breakpoints
```
ε*(o_k → o_{k+1}) = (P_{k+1} − P_k) / (V_{k+1} − V_k)
```
Algorithm (convex-hull trick): dedupe equal-V offers keeping min P; sort by V ascending; stack-scan popping any line whose intersection with its predecessor occurs at or before the predecessor's own entry breakpoint. Offers never on the envelope have **zero share** (dominated — the linter reuses this, §4.9).

Share of segment s choosing active offer k, with `Φ_ln(x) = Φ(ln(x)/σ_s)` and breakpoint interval `(a_k, b_k]` clipped to ε > 0:
```
share_k = Φ_ln(b_k) − Φ_ln(a_k)
```
Degenerate case σ_s = 0: all mass at ε = 1; choose by direct argmax under the active tie mode (§4.2). The screening/bundling oracles (§4.5, §4.6) exercise exactly this path with `tieMode = 'seller-favorable'`.

**Offer expansion.** The simulated offer set per segment = paid/free tiers × subsets of ≤ 3 add-ons not already contained in the tier (composite value = union of features; composite price = sum), plus outside option, plus (when Positioning is active, §4.11) any competitor alternatives valued for that segment. Add-on purchase is therefore *jointly* optimal with tier choice — no greedy approximation. Cap: ≤ 5 tiers × 2³ add-on subsets + 1 outside + up to 6 competitors = ≤ 47 offers/segment.

Tests — **T-ENV-01** dominated offer (V ≤, P ≥ another) gets zero share for every σ. **T-ENV-02** breakpoints strictly increasing; shares sum to 1 (incl. outside) to 1e-12. **T-ENV-03** *brute-force cross-validation*: for 200 fast-check-random offer sets and σ values, sample 1,000 ε quantile points; direct argmax agrees with envelope interval assignment at every point (away from breakpoint ties by > 1e-9). **T-ENV-04** σ=0 reproduces point-WTP argmax with documented tie-breaking. **T-ENV-05** free tier `(V>0, P=0)`: outside option gets zero share for ε > 0; free absorbs the low-ε mass. **T-ADD-01** add-on expansion agrees with brute-force enumeration over tier × subset combos on random fixtures.

### 4.3 Economics readouts & the conservation identity

With lognormal partial expectation `L(a,b) = E[ε·1(a<ε≤b)] = e^{σ²/2}·[Φ((σ²−ln a)/σ) − Φ((σ²−ln b)/σ)]` (limits: `ln 0 → −∞`), per segment s over active offers k on intervals `(a_k, b_k]`:

```
Revenue_s      = n_s · Σ_{k ∈ own tiers/add-ons} share_k · P_k
OwnBuyerSurplus_s = n_s · Σ_{k ∈ own offers (paid or free)} [ V_k · L(a_k, b_k) − P_k · share_k ]
FencingGap_s   = n_s · Σ_{k ∈ own offers (paid or free)} (V_full − V_k) · L(a_k, b_k)      where V_full = V_s(F)
Unserved_s     = n_s · V_full · L(outside interval)
CompetitorLoss_s = n_s · Σ_{k ∈ competitors} V_full · L(a_k, b_k)       (only when Positioning active)
Potential_s    = n_s · E[ε] · V_full ,   E[ε] = e^{σ²/2}
```

**Conservation identity (must hold exactly):** `Potential = Revenue + OwnBuyerSurplus + FencingGap + Unserved + CompetitorLoss`. This is deliberately an *our-catalog potential* identity: each buyer's `ε·V_full` is assigned to an own offer (price paid + buyer utility + features withheld), to non-conversion, or to a competitor. A free fenced offer participates in `FencingGap` even though its price is zero. Competitor utility/revenue is intentionally **not** folded into `OwnBuyerSurplus`; it may be reported separately as a non-conserved comparison metric, never in this waterfall. When no competitors are active, `CompetitorLoss = 0` and the identity collapses to the four-term form.

KPIs: `MRR = Σ_s Revenue_s`; `paid conversion = paid buyers to us / Σ n_s`; `ARPA = MRR / paid buyers to us`; **capture rate** `= MRR / Σ Potential_s` (headline metric); **competitor loss share** `= Σ CompetitorLoss / Σ Potential_s` (headline metric when Positioning is on). Counterfactual diffs: re-run with an offer removed (e.g., free tier, a competitor) → `Δ MRR` = that offer's leakage/contribution (used by linter E5, E7 and the add-on and competitor analyses).

Tests — **T-ECON-01** hand-computed 2-segment, 2-tier fixture matches all readouts to 1e-9. **T-ECON-02** (property, fast-check) conservation holds to 1e-6·Potential on random scenarios *with and without* competitor alternatives. **T-ECON-03** (property) raising one offer's price weakly decreases its own share. **T-ECON-04** analytic own-buyer surplus matches numeric integration on random fixtures (1e-6 rel). **T-ECON-05** counterfactual remove-offer: removing a zero-share offer changes nothing (to 1e-12). **T-ECON-06** has two hand-computed guards: a fenced free tier contributes its withheld value to `FencingGap`, and a competitor selection contributes only `CompetitorLoss` (not `OwnBuyerSurplus`); both preserve the identity exactly.

### 4.4 Price sweeps (demand & revenue curves)

For a selected tier, sweep its **list-price unit** over a 400-point grid, recomputing the envelope at each point. Flat price uses `0 → 1.5 × max_s V_full,s`; per-seat price uses `0 → 1.5 × max_s (V_full,s / c_s)`, so the grid and the editable price are both $/seat/month. Changing a tier price shifts every composite offer derived from that tier; `Q(p)` and `R(p)` aggregate the selected tier plus those composites. Render current price and grid-best-found markers. This is the residual demand curve given the rest of the menu (and any active competitors), labeled as such.

Tests — **T-SWP-01** sweep evaluated at the current price equals the scenario's simulated values (1e-9). **T-SWP-02** `Q(p)` weakly decreasing in own price (property). **T-SWP-03** golden fixture: argmax within one grid step of independently computed optimum.

### 4.5 Second-degree price discrimination oracle (screening correctness)

The classic two-type menu result the engine must reproduce (Mussa–Rosen; σ→0). Types L, H; tiers Basic ⊂ Pro; values `V_LB, V_LP, V_HB, V_HP` with single crossing (`V_HB ≥ V_LB`, `V_HP − V_HB ≥ V_LP − V_LB`). Optimal serve-both menu:
```
p_B = V_LB                       (L's IR binds — full extraction on Basic)
p_P = V_HP − V_HB + V_LB         (H's IC binds — H keeps information rent V_HB − V_LB)
Exclude L entirely  iff  n_H·(V_HB − V_LB) > n_L·V_LB   (then p_P = V_HP)
```
Canonical fixture: `V_LB=40, V_LP=55, V_HB=60, V_HP=100`, `n_L=100, n_H=50` → `p_B=40, p_P=80`, revenue `8000`; exclusion variant `n_L=10` → Pro-only at `p_P=100`, revenue `5000 > 4400`.

**Tie-convention guard — read before implementing these tests.** This menu prices each served type exactly at a binding value (L's IR: `p_B = V_LB`; H's IC: `p_P` leaves H indifferent between the two tiers), so every served buyer sits on an exact tie. The engine reproduces the textbook revenue **only** under `tieMode = 'seller-favorable'` at the σ→0 point-mass path (§4.2). Under the simulator's default `'conservative'` mode the same fixture yields ≈2000 (L → outside on its IR tie; H → Basic on its IC tie), and σ=1e-3 under *any* mode yields ≈5000 (pricing at the segment median drops the below-median half). **If T-SCRN-01 does not return 8000, the tie mode / σ is the suspect — do not modify the envelope or selection engine to force the number.** The engine is correct; the test must run σ=0 with `'seller-favorable'`. (Verified numerically during review S2, D-20.)

Tests (all at **σ=0** with **`tieMode = 'seller-favorable'`** per the guard) — **T-SCRN-01** engine at (40, 80): L→Basic, H→Pro, revenue exactly 8000, L surplus = 0, H surplus = 20/buyer. **T-SCRN-02** perturb `p_P` to 85 → H switches to Basic (cannibalization; strict preference, no tie), revenue 6000. **T-SCRN-03** exclusion variant: menu {Pro@100} beats serve-both 4400, L takes outside option. **T-SCRN-04** no-distortion-at-top sanity: at the optimal menu, H buys the highest-value tier.

### 4.6 Bundling analyzer (Adams–Yellen on the envelope engine)

Question it answers: *should capability X be a tier fence, a separate add-on, or both/mixed?* Model two goods A, B (feature bundles) with per-segment values from the same matrix. Regimes as offer sets on the standard engine (§4.2):
```
Pure components : {A@p_A, B@p_B, A+B@(p_A+p_B)}     (buying both is allowed)
Pure bundle     : {A+B@p_AB}
Mixed bundling  : {A@p_A, B@p_B, A+B@p_AB}, p_AB ≤ p_A + p_B
```
Within-segment ε is perfectly correlated across goods (one scale factor), so bundling gains come from *across-segment* value dispersion — the honest SaaS interpretation, stated in-UI. Optimizer: coordinate ascent over each regime's prices on the closed-form revenue, multi-start from candidate grid (segment valuations ± spread quantiles, plus a "not offered" sentinel = price above all WTP); report the **best price found under the documented search grid**, regime deltas, and the searched candidates. Comparisons at given prices are exact; the UI never calls a coordinate-ascent result a global optimum.

Tests (closed-form optima T-BND-01…03 at **σ=0** with **`tieMode = 'seller-favorable'`**: pricing a bundle at exactly its value is a participation tie, and the textbook optimum assumes the buyer participates — same guard as §4.5) — **T-BND-01** canonical negative-correlation case: segments (A=9,B=1) and (A=1,B=9), equal sizes → pure components optimum 9/9 (rev 18·n), pure bundle 10 (rev 20·n); bundle wins. (Under `'conservative'` the pure-bundle-at-10 fixture yields 0 — the guard applies here too.) **T-BND-02** mixed ≥ max(components, pure bundle) at optima (mixed nests both; property, tolerance = grid resolution). **T-BND-03** σ→0 optimizer matches brute-force grid over type valuations on random 3-segment fixtures. **T-BND-04** the simulator's default `'conservative'` mode: between two *offered* options of equal utility the buyer takes the cheaper one — an offer-vs-offer tie, distinct from the participation tie the other tests exercise — documenting why "not offered" sentinels matter.

### 4.7 Van Westendorp Price Sensitivity Meter

Survey-first stance: the PSM module analyzes *fielded* responses (manual entry or CSV paste: 4 prices per respondent). A model-generated illustrative mode exists but is labeled "simulated from your assumptions — for intuition, not evidence" (mapping documented in-UI; ledger D-11).

Construction: per respondent quadruple `(too cheap ≤ cheap ≤ expensive ≤ too expensive)`; violators flagged and excluded (count shown — data hygiene is part of the pedagogy). Cumulative curves on the union grid of response prices: "too cheap" and "cheap" descending; "expensive" and "too expensive" ascending; `not cheap = 100% − cheap`, `not expensive = 100% − expensive`. Crossings by piecewise-linear interpolation:
```
PMC = too cheap × not cheap      PME = too expensive × not expensive
IPP = cheap × expensive          OPP = too cheap × too expensive
Acceptable range = [PMC, PME]
```
(Conventions vary in the literature; ours pinned to the standard operationalization of van Westendorp 1976, stated in the explainer.) No crossing → point reported "undefined for this data" — never fabricated.

Tests — **T-VW-01** monotonicity validation drops exactly the violating respondents on a fixture. **T-VW-02** symmetric synthetic dataset (quadruples mirrored around 50) → OPP = IPP = 50 exactly; PMC, PME symmetric about 50. **T-VW-03** hand-computed 5-respondent fixture matches all four points to 1e-9. **T-VW-04** no-crossing fixture returns undefined markers, not numbers.

### 4.8 Monte Carlo & tornado (assumption uncertainty)

Uncertain parameters: each segment's `W_s` (value scale) and `n_s` (size), via user-set P10/P90 bands → lognormal draws (fitting per §4.1); `σ_s` and prices held fixed. PRNG: mulberry32, seed visible in UI and stored in the scenario. K = 1,000 draws (configurable 200–5,000); each draw is evaluated with the closed-form engine (no nested simulation). All A/B designs receive the **same indexed vector of parameter draws** (common random numbers), so a win rate is paired rather than a comparison of unrelated samples; an equal-MRR draw is a tie, not a win. Outputs: MRR distribution (P10/P50/P90 band), per-design; A/B win rate ("B beats A in 78% of draws"); tornado = one-at-a-time low/high (P10/P90) per parameter vs. base, bars sorted by |ΔMRR|. Tornado × provenance tags → the validation to-do list.

Tests — **T-MC-01** fixed seed → byte-identical percentiles and first 5 draws (snapshot). **T-MC-02** zero-width bands → P10 = P50 = P90 = analytic MRR exactly. **T-MC-03** MC mean within 3 standard errors of analytic value on a 1-uncertain-parameter fixture. **T-MC-04** tornado of a parameter with zero-width band has zero-length bar. **T-MC-05** two identical designs consume the same draw vector and produce 100% ties / 0% wins for either side.

### 4.9 Design linter (deterministic critic)

Every rule: id, exact trigger, severity, message, citation where applicable; all computed from the scenario + simulation results, no randomness. Economic rules:
- **E1 dead fence** — feature in no offer, or in all offers including free: fence carries no screening information. Info.
- **E2 dominated offer** — zero envelope share for every segment (§4.2): dead weight *or* deliberate decoy. Message explains asymmetric dominance honestly: the rational model scores it inert; behavioral evidence (Huber–Payne–Puto 1982) says a dominated decoy can lift the dominating tier — the model won't invent a number for that, but here's the trade-off to consider. Warning.
- **E3 fence inversion** — a higher-priced tier missing a feature a cheaper tier has (non-nested ladder): breaks the upgrade narrative; sometimes intentional ("good-better-different"), so warn with rationale. Warning.
- **E4 downgrade mass / cannibalization** — > 30% of a segment's buying mass chooses an offer with value below the segment's value-maximal tier; message names the price gap and fence causing it (read from breakpoints). Warning with $ quantification.
- **E5 free-tier leakage** — counterfactual remove-free re-run (§4.3): "free absorbs X% of would-be paid demand ≈ $Y MRR." Info/warning by threshold (default: warn > 15% of MRR).
- **E6 add-on cannibalization** — counterfactual remove-add-on: add-on's net MRR contribution after tier-mix shift; negative → warning.
- **E7 competitor loss (Positioning only)** — a segment loses > 25% of its Potential to a single competitor: message names the competitor and the tier(s) it beats on the envelope. Warning with $ quantification.
Behavioral rules (cited, deterministic):
- **B1 choice overload** — > 4 paid offers visible (Iyengar–Lepper 2000, framed as directional evidence with replication caveats). Info.
- **B2 weak anchor** — top tier takes < 2% share and < 1% of MRR: prune it, or reposition deliberately as an anchor/decoy (explainer covers anchoring; Tversky–Kahneman 1974). Info.

Tests — **T-LNT-01…09** one fixture per rule triggering it exactly, plus a clean fixture triggering none.

### 4.10 Conjoint (CBC) & MaxDiff-lite

**Purpose.** Estimate feature-level preference weights from choice data, then optionally bridge those weights into the value matrix. Positioned as "the survey-driven complement to the assumption-driven model" — the same pedagogical stance as PSM: your assumptions get pressure-tested by real choices.

**Choice-based conjoint (CBC).** User defines 2–5 non-price attributes with 2–4 levels. The bridge requires a **numeric price column** in $/account/month, the same unit as the value matrix; a categorical price attribute may be estimated and charted but cannot enable a WTP bridge. Design generator produces `T` choice tasks (default 12), each a set of `k` (default 3) concepts + an optional "None" alternative; concepts are level-balanced across each non-price attribute (random-balanced design v1; D-efficient generation deferred v1.1 — see §2.2 & ledger D-17). It rejects duplicate concepts within a task and validates full column rank before a study can be fielded. Respondent data entry is manual or CSV of `respondent_id, task_id, chosen_alternative`.

**Model.** Multinomial logit with effects-coded **non-price** part-worths. For attribute `a` and level `l`, part-worth `β_{a,l}` has identifying constraint `Σ_l β_{a,l} = 0`; implementation estimates `L_a−1` free coefficients and reconstructs the final level. For bridge-enabled studies, the numeric price coefficient `β_p` is utility per $/account/month. Concept `j` has utility `u_j = Σ_a β_{a, level(j,a)} + β_p·price_j`; None utility is `β_None`. Choice probability in set `S_t` for respondent `r`:
```
P(choose j | S_t; β) = exp(u_j) / Σ_{k ∈ S_t} exp(u_k)   (log-sum-exp for numerical stability)
```
Aggregate log-likelihood (pooled across respondents — HB deferred, §2.2):
```
ℓ(β) = Σ_r Σ_t log P(chosen_{r,t} | S_t; β)
```
`ℓ` is concave; it is strictly concave only when the design is full rank and the observed choices are not separated. Estimate via **damped Newton-Raphson** with analytic gradient and Hessian, backtracking only on likelihood-improving steps, to `‖∇ℓ‖∞ < 1e-8` or 50 iterations. Return one of `ok`, `nonIdentifiable`, `separated`, or `nonConverged`; only `ok` with invertible observed information may report `SE(β̂) = √diag(−H(β̂)⁻¹)`, 90% CIs, or a WTP bridge. The UI explains failure states instead of fabricating coefficients or CIs. Overall hit rate is training-set descriptive only, never validation accuracy.

**MaxDiff-lite** (importance scoring, no price required). User defines items (up to 12); design generator produces tasks each showing `k` (3–5) items; respondent picks best and worst per task. Analysis (v1): best–worst counting scores
```
s_i = (times_i_chosen_best − times_i_chosen_worst) / times_i_appeared
```
then linearly rescaled so `Σ_i normalized_s_i = 100` and `min ≥ 0` (shift-then-normalize: subtract `min_i s_i`, then scale to sum 100). **Degenerate branch (required):** when all raw `s_i` are equal — the symmetric case that T-MXD-01 feeds — the shift makes every value 0 and the sum-to-100 scaling divides by zero; return the uniform distribution `100/k` per item instead. Full MNL best-worst estimation deferred to v1.1.

**Bridge to the value matrix.** If the study has numeric account-month price, status `ok`, and `β̂_p` is significantly negative (`β̂_p + 1.645·SE(β̂_p) < 0`), a non-price part-worth contrast converts to account-month WTP: for a feature-attribute contrast between level `l₁` and `l₀`,
```
ΔWTP = −(β̂_{a,l₁} − β̂_{a,l₀}) / β̂_p
```
Before applying, the user must map each chosen contrast to a catalog feature, map respondents (or an explicit aggregate) to segments, choose a weighting rule, and confirm the account-month period. The UI shows a proposed diff against existing `v[s,f]` values; it never overwrites cells silently. Confirmed cells receive `conjoint (N=<respondents>)` provenance. Categorical/no price, a non-negative or insignificant price coefficient, or any non-`ok` status disables the bridge while keeping relative part-worths visible.

Tests — **T-CNJ-01** recovery: simulate 500 respondents from a known `β` on a full-rank 3-attribute × 3-level design with numeric price → estimated `β̂` within 3·SE of truth on every coefficient. **T-CNJ-02** log-likelihood strictly increases on every accepted Newton step until the gradient stop condition. **T-CNJ-03** the observed Hessian is negative-definite at the `ok` optimum of that full-rank, non-separated recovery fixture. **T-CNJ-04** effects-coding constraint `Σ_l β̂_{a,l} = 0` holds to 1e-10 per attribute in the returned estimate. **T-CNJ-05** an exactly balanced β=0 fixture returns near-zero part-worths and no bridge. **T-CNJ-06** bridge is disabled when numeric `β̂_p` is not significantly negative. **T-CNJ-07** rank-deficient and separated fixtures return the named status, no CIs, and no bridge. **T-MXD-01** best-worst counting: symmetric fixture (each item picked equally often best & worst) → all scores equal (uniform 100/k) after normalization via the degenerate branch. **T-MXD-02** normalized scores sum to 100 ± 1e-9 and are non-negative.

### 4.11 Competitive price-value map & competitor alternatives

**Two coupled surfaces powered by the same data.**

**(1) Positioning map.** Values vary by segment, so the map is explicitly **segment-scoped**: the user selects one segment and every competitor/tier point is plotted with that segment's value and effective price. Axes are value on X and price on Y. A zero-utility ray obeys `P = εV`, so its slope is `ε`, never `1/ε`. Because every segment has median `ε=1`, the UI shows selected-segment P10/P50/P90 rays (derived from that segment's `σ`) rather than pretending there are different median rays. Competitor points are the Pareto non-dominated set, not a convex hull: competitors are discrete alternatives and cannot be mixed (D-21). The map may draw a step-style visual connector, but no line is a market offer.

**(2) Competitor as an alternative in the envelope engine.** Each competitor becomes a per-segment offer `(V_s(competitor), P_competitor)` in that segment's offer set (§4.2), with a flag marking it as a competitor. Simulator computes:
- **Choice share going to a competitor** in a segment = **switching/loss risk** for that segment (headline number).
- Revenue accounting: competitor choice does **not** contribute to `Revenue_s`; it feeds `CompetitorLoss_s` in the conservation identity (§4.3). Our KPIs (MRR, capture rate) stay uncontaminated; a separate "competitor loss share" KPI surfaces when Positioning is active.
- Wind tunnel dot panel adds competitor swatches so churn/loss is *visible*, not just aggregated.

**Frontier construction and verdict.** Given competitor points `{(V_i, P_i)}`, drop dominated points (`V_i ≤ V_j ∧ P_i ≥ P_j` for some j, strict in one component), then sort survivors by V ascending. There is no linear interpolation and no vertical "distance to frontier": either would invent an unavailable competitor offer. A tier receives `directly dominated` only when a competitor has `V_c ≥ V_t ∧ P_c ≤ P_t` (strict in one); otherwise it receives `not directly dominated`. The envelope-engine competitor share remains the quantitative trade-off measure.

Tests — **T-CMP-01** frontier construction: 5-competitor canonical fixture matches an independently computed Pareto set — include a competitor that is Pareto-efficient but off the convex hull, to pin the discrete choice; single-competitor edge case returns that competitor. **T-CMP-02** a ray at slope `ε` passes through `(V, εV)` and a selected segment's P10/P50/P90 slopes match its lognormal quantiles. **T-CMP-03** duplicate competitor points collapse in the frontier (no double-count). **T-CMP-04** direct-dominance verdict triggers only for the stated discrete predicate; a point between two competitors is never judged by interpolation. **T-CMP-05** competitor as engine offer: on a fixture where a competitor priced below every tier at similar value exists, the segment's competitor-share exceeds 0 and paid-conversion drops by the corresponding amount (identity: `Δ (conversion + competitor share) = 0`). **T-CMP-06** identity preservation: enabling Positioning with an empty competitor set leaves all KPIs and shares unchanged (to 1e-12). **T-CMP-07** conservation with competitors: `Potential = Revenue + OwnBuyerSurplus + FencingGap + Unserved + CompetitorLoss` holds on random fixtures with 0–5 competitors (property).

### 4.12 Numerical conventions

Money as floating dollars (modeling tool, not accounting; display-rounded); prices ≥ 0; `σ_s` UI range [0.05, 2.0] with labeled presets (Low 0.25 / Medium 0.5 / High 0.9); tie tolerance 1e-9 (equal utility within this is a tie, resolved by the active tie mode, §4.2); percentages to 1 dp; currency USD-formatted via `Intl.NumberFormat` (multi-currency v2). Φ via a rational-approximation implementation tested to 1e-7 (T-STAT-01); Φ⁻¹ via Acklam/AS-241, tested by round-trip. Softmax via log-sum-exp for numerical stability (Conjoint MNL). All engine randomness flows through the injected seeded PRNG.

---

## 5. Interaction, visualization & UX

### 5.1 Layout — the loop on one screen

```
┌────────────────────────────────────────────────────────────────────────┐
│ ◆ name   [Scenario ▾]   Model · Design · Analyze · Share      seed 42  │
├──────────────────────────────────┬─────────────────────────────────────┤
│ WORKBENCH (Model or Design tab)  │ WIND TUNNEL (always on, always live)│
│                                  │  MRR · capture · conv · comp-loss   │
│ Model: segment cards (size,      │  ┌ buyer dots: segments → offers ┐  │
│  seats, WTP band, spread),       │  └ value waterfall ──────────────┘  │
│  value matrix (totals + shares;  │  per-tier price-sweep curves        │
│  advanced: direct $ edit),       │  [Mechanism view]  [Compare A/B]    │
│  provenance tags                 │                                     │
│ Design: tier cards, fence grid,  │  Linter findings dock (badge count) │
│  prices, metric, add-ons         │                                     │
└──────────────────────────────────┴─────────────────────────────────────┘
```
The wind tunnel never leaves the screen on desktop: **every edit re-simulates instantly** (closed form makes this free) — the product thesis as an interaction. Analyze — Uncertainty (Monte Carlo/tornado), Research (PSM, Conjoint/MaxDiff, Bundling), and Positioning (competitive map + competitor share) are full-screen views fed by the same scenario. Share (pricing page, decision record) also full-screen. Narrow viewports: tabbed, results one tap away. Tabs are state; the URL hash is reserved for the scenario payload.

### 5.2 Making the economics tangible

- **Buyer dots:** 100 dots per segment (1-dot = 1% granularity, disclosed), colored by segment (Okabe–Ito), arranged by chosen offer; competitor offers get distinct hatched swatches so switching is legible at a glance. On change dots re-sort with a short eased transition. Reduced motion: instant reposition + numeric deltas. This is self-selection made visible.
- **Value waterfall:** Potential → Revenue / Own-buyer surplus / Fencing loss / Unserved / (Competitor loss when active), per §4.3's identity — the money shot for "surplus captured vs left on the table vs walking to the competitor." Competitor buyer utility, when shown, is explicitly a separate comparison metric rather than a waterfall term.
- **Mechanism view (toggle):** the actual screening diagram — utility lines vs ε, upper envelope shaded, breakpoints labeled, segment density strip beneath. The textbook figure, live. For expert credibility; also the best debugging view.
- **Price sweeps:** small-multiple R(p) and Q(p) per tier with current-price and grid-argmax markers.
- **A/B:** side-by-side KPI columns with delta chips, dot-panels, and MC win rate; "promote B to active" action.
- **Conjoint part-worth chart:** horizontal bar per attribute-level with 90% CI whiskers; hit-rate badge; "apply bridge" primary CTA gated on the conditions in §4.10.
- **PSM curve panel:** four cumulative curves with crossing markers and shaded acceptable range; a violator-count chip; illustrative-mode toggle labeled "SIMULATED — not evidence."
- **Positioning map:** segment selector; 2D chart with selected-segment P10/P50/P90 break-even rays, Pareto competitor points, our tier dots, and a discrete direct-dominance verdict. The simulator's competitor share, not an interpolated chart distance, is the trade-off number.
- **Explainers:** every loaded term (WTP, fence, surplus, decoy, capture rate, PMC, part-worth, iso-utility ray…) gets a popover from `content/glossary.ts` — one consistent voice, citations where claims are empirical. The educational layer is content, not chrome.

### 5.3 First-run & empty states

First visit: template picker (3 archetypes, §2.1) or "start blank"; templates load pre-filled with provenance tags set to "benchmark guess" — teaching the tagging discipline by example. Every empty state says what to do next. A worked default scenario means the hero view is alive within seconds of first load.

### 5.4 Accessibility & motion (acceptance-tested, §7.6)

Keyboard-complete flows (matrix grid = arrow-key navigable with visible focus); all charts paired with a "view as table" toggle rendering the same numbers; KPI updates announced via a debounced `aria-live` region; color pairs pass contrast on both fills and text in **both light and dark themes**; `prefers-reduced-motion` disables all transitions (state changes remain fully legible); print stylesheet for the decision record.

### 5.5 Visual language

Restraint: neutral slate surfaces with **light + dark themes**, one accent, Okabe–Ito for categorical segment color (validated for AA contrast against both surfaces), self-hosted Inter subset via `next/font/local`, generous whitespace, numbers formatted like a good board deck (USD, 1-dp percentages, no seven-decimal floats). No emoji, no gradients-as-personality, no dashboard clutter. Theme is a scenario setting (`settings.theme: 'system' | 'light' | 'dark'`) with a manual override in the header; the Okabe–Ito palette gets subtly re-toned per theme so segment colors stay distinct on both backgrounds (ledger D-13).

---

## 6. Phased build plan

Sizing: S ≈ half an agent-session, M ≈ one, L ≈ two. Every phase ends with the §7.5 gate. Versions: minor bump per phase completion, patches within a phase (§8.3). "Review" = what Opus should scrutinize hardest.

**Global definition of done (every phase):** `npm run typecheck && npm run lint && npm run test && npm run build` green; no console errors/warnings in dev flows touched; CONTEXT.md session log + ledger updated; version bumped in `package.json` and footer; **stop and propose the exact version + commit message to the owner (§0 rule 4)** — do not run `git commit` until owner approves both.

---

**P0 — Scaffold, CI, deployed shell** · S/M · → v0.1.0
Next.js (App Router) + TS strict + Tailwind (with light + dark tokens defined in P0), `output: 'export'` for static build; ESLint (`eslint-config-next` + a custom **engine-purity import rule** blocking `lib/engine/*` from importing `components/*` or `lib/state/*`) + Prettier; Vitest + fast-check + Testing Library; Playwright (chromium); `.nvmrc` (current LTS); scripts (`dev/build/test/e2e/typecheck/lint/spec-coverage` stub). GitHub Actions: PR = typecheck+lint+test+build+bundle-size gate; main = Vercel deploys via the repo-connect integration (no CI-side deploy step; Vercel's own build runs `next build` and serves the static export). Add Vercel CSP headers (`connect-src 'none'`) and `docs/STATIC-HOST-HEADERS.md` for portable hosts. App shell: header (theme toggle + seed chip), tab skeleton, footer with version string from `package.json`. Manual scaffold (not `create-next-app`) — the dir is non-empty; don't clobber existing docs.
*Accept:* CI green on a test PR; shell deployed to public Vercel URL (needs owner's Vercel connection, §11); `README` run instructions work from clean clone; bundle gate active (≤ 300 KB gz, trivially passing now); theme toggle flips both light and dark tokens correctly on the empty shell; Playwright request allowlist proves the exported app makes no request beyond same-origin document/static assets.
*Review:* toolchain versions recorded in ledger D-19b; engine-purity lint rule actually fails on a violation (prove with a scratch test that then deletes); Next `output: 'export'` is set and no server-only features (`headers`, `cookies`, `revalidate`, image-optimization remote loaders) leak into `app/`.

**P1 — Engine: statistics + envelope core** · M · → v0.2.0
`stats.ts` (erf/Φ/Φ⁻¹, lognormal pdf/cdf/quantile/partial expectation, band fit), `offers.ts` (expansion incl. add-on subsets, per-seat effective pricing, and **functional** competitor-offer synthesis — a competitor is just another `(V_s, P)` offer carrying a competitor flag; building it for real here, not as a stub, is what lets the E7 linter rule be tested in P5 before the Positioning UI lands in P7e), `envelope.ts` (selection, shares, σ=0 path, **both tie modes** per §4.2). Extract §4 → `docs/MODEL-SPEC.md` (section numbers preserved). `spec-coverage` script: greps `@spec §x.y` tags in tests → coverage table; CI-reported.
*Accept:* T-STAT-01…04, T-ENV-01…05, T-ADD-01 pass; spec-coverage shows §4.1–4.2 covered; engine imports nothing from components/state (lint proves); `selectOffers` exposes both tie modes (default `'conservative'`) and a unit test asserts they diverge on a constructed exact-tie fixture (both a participation tie and an offer-vs-offer tie).
*Review:* brute-force cross-validation test (T-ENV-03) is genuinely independent (no shared helper with the envelope implementation); numerical edge cases (σ→0, equal-V offers, free tier).

**P2 — Engine: economics readouts** · M/L · → v0.3.0
`economics.ts`: KPIs, surplus decomposition + conservation (both 4-term and 5-term with competitors), counterfactual remove-offer diffs, unit-correct price sweeps. Screening oracle fixtures.
*Accept:* T-ECON-01…06, T-SWP-01…03, T-SCRN-01…04 pass; spec-coverage §4.3–4.5 complete.
*Review:* conservation property test uses adversarial fast-check generators (fenced free tiers, dominated offers, σ extremes, with and without competitors); waterfall term definitions match §4.3 exactly.

**P3 — Scenario schema, state, persistence, sharing** · M · → v0.4.0
Zod schemas (versioned, `schemaVersion: 1`, migration stub), Zustand stores + zundo undo, localStorage autosave (debounced), JSON export/import with validation errors surfaced, URL codec `#s=` (lz-string), template data files (3 archetypes, §2.1) as validated fixtures. Persisted full-scenario JSON may include survey artifacts; the URL `SharePayload` schema cannot. It has an 8 KiB encoded cap plus decompressed-byte guard, and the UI directs a complete scenario to JSON export/import. Schema includes competitor list slot + survey-artifact slots even though those layers ship later — versioning gets done once.
*Accept:* T-SCH-01 export→import round-trip byte-equal; T-SCH-02 corrupted/foreign or over-limit JSON rejected with human message; T-URL-01 compact hash round-trip incl. unicode segment names; T-URL-02 oversized/decompression-bomb hash is rejected without parsing and offers JSON export/import; templates validate against schema; undo/redo works across a scripted edit sequence (unit).
*Review:* schema is minimal-but-sufficient for §2.1 scope; no engine types leak into persisted schema (versioning safety); sharing never silently drops respondent data.

**P4 — Model surface** · L · → v0.5.0
Segment cards (size, seats, WTP median + P10/P90 band with live density strip, spread preset, provenance tag), value matrix (totals × allocation shares; advanced direct-$ grid), template picker + first-run flow, glossary popovers (initial set). Wind-tunnel rail may show placeholder KPIs wired to the engine (no charts yet). Theme toggle (light/dark/system) wired to state.
*Accept:* Playwright E2E-01: first visit → pick template → edit a WTP median → KPI text updates; axe 0 serious violations on both themes; matrix fully keyboard-navigable (e2e asserts focus traversal); band inputs reject q10 ≥ q90 inline.
*Review:* the σ-vs-confidence-band distinction is legible in the UI copy (the classic conflation, §4.1); input formatting (currency, %) consistent; theme switch survives reload.

**P5 — Design surface + linter** · L · → v0.6.0
Tier cards (name, price, metric flat/per-seat), fence grid (feature × tier checkboxes), free-tier toggle, add-ons (≤ 3), multiple designs per scenario (create/duplicate/rename), `linter.ts` + findings dock with cited explainers.
*Accept:* T-LNT-01…09 pass (E7 competitor loss asserts against a fixture with a competitor added directly to the scenario JSON, run through the **functional** competitor-offer synthesis built in P1 and the 5-term conservation from P2 — the full Positioning UI lands in P7e, but the engine path and the *rule* are real and testable now); E2E-02: build a 3-tier design from blank, see linter fire and clear; dominated-offer fixture shows E2 with per-segment detail; counterfactual $ figures match engine diffs (unit-checked against `economics.ts`).
*Review:* linter triggers exactly per §4.9 (no fuzzy thresholds beyond documented ones); messages are decision-useful, not scoldy.

**P6a — Wind tunnel: core reveal** · L · → v0.7.0
Live KPI header (MRR, conversion, ARPA, capture rate; competitor-loss placeholder that becomes live in P7e), buyer-dot panel (transition + reduced-motion path), value waterfall, per-tier price sweeps, "view as table" toggles, aria-live updates.
*Accept:* E2E-03: price edit → dots re-sort, waterfall + KPIs update; reduced-motion e2e (emulated) shows no transition but same end state; every chart has a table toggle with matching numbers (spot-assert two); perf: simulate+render interaction < 16 ms on templates (measured via Playwright trace, budget documented).
*Review:* charts hand-rolled cleanly (no dep creep); numbers in views reconcile with engine output exactly (one source of truth).

**P6b — Mechanism view + A/B compare** · M · → v1.1 extension (after v1.0)
Envelope diagram (lines, shaded envelope, breakpoints, density strip), A/B: design switcher, side-by-side KPIs + deltas, promote-to-active.
*Accept:* E2E-04: duplicate design, change a price, compare view shows correct deltas (engine-verified in test); mechanism view matches envelope.ts breakpoints (rendered labels asserted against engine values).
*Review:* mechanism view correctness — this is the expert-credibility surface; wrong here is fatal.

**P7a — Monte Carlo + tornado** · M · → v0.8.0
`montecarlo.ts` + Analyze — Uncertainty view: MRR band, per-design distributions, A/B win rate, tornado sorted by |ΔMRR|, validation to-do list (tornado × provenance).
*Accept:* T-MC-01…05 pass; E2E-05: set bands, run MC, tornado renders, seed change → different but deterministic results (re-run same seed → identical); 1,000 draws < 150 ms (measured).
*Review:* determinism end-to-end (no stray Math.random); statistical claims in copy are accurate (percentiles, not CIs).

**P7b — Van Westendorp** · M · → v0.8.x
`vanwest.ts` + PSM view (Analyze — Research): manual/CSV input, validation report, four-curve chart, crossing markers, acceptable-range shading, illustrative-mode toggle (labeled), demo dataset.
*Accept:* T-VW-01…04 pass; E2E-06: paste demo CSV → points match fixture values; violator count displayed; no-crossing data shows "undefined" state.
*Review:* interpolation and convention choices match §4.7 exactly; illustrative mode cannot be mistaken for real evidence (labeling).

**P7c — Bundling analyzer** · M · → v1.1 extension (after v1.0)
`bundling.ts` + view (Analyze — Research): pick goods A/B from catalog/add-ons, per-regime optima, comparison verdict with revenue deltas and "prices searched" disclosure.
*Accept:* T-BND-01…04 pass; E2E-07: canonical fixture reproduces bundle-beats-components verdict.
*Review:* optimizer honesty (multi-start coverage, "not offered" sentinels); regime nesting property holds in UI-reported numbers.

**P7d-1 — Conjoint estimator core + MaxDiff scoring** · M · → v1.1 extension (after v1.0)
Estimation only, no UI. `conjoint.ts`: MNL log-likelihood + **analytic gradient + analytic Hessian** (log-sum-exp softmax), damped Newton-Raphson to `‖∇ℓ‖∞ < 1e-8` or 50 iterations, explicit status outcomes, SEs only from invertible observed information, effects coding for non-price attributes, numeric account-month price for a bridge-enabled study; random-balanced design generator; price→WTP bridge function (not yet wired to the value matrix). `maxdiff.ts` (design generator, best-worst counting, normalization incl. the degenerate branch, §4.10).
*Accept:* T-CNJ-01…07, T-MXD-01…02 pass. **Derivative gate (mandatory, precedes the recovery test):** with documented central-difference step `h = 1e-5·max(1, |β_i|)`, analytic derivatives must meet both an absolute `1e-7` and relative `1e-6` tolerance on a random β/design. This localizes a sign/index bug before T-CNJ-01's global recovery test.
*Review:* the analytic gradient/Hessian is the single highest-risk piece of engine math. Do not weaken the derivative gate. Negative-definiteness is asserted only on the full-rank, non-separated fixture; rank-deficient/separated data must return a named failure status.

**P7d-2 — Conjoint + MaxDiff views + bridge** · M · → v1.1 extension (after P7d-1)
Analyze — Research views: CBC — attribute/level definer, task designer, respondent-response entry (manual + CSV), part-worth bar chart with 90% CI whiskers, hit-rate badge, "apply to value matrix" bridge action (gated per §4.10); MaxDiff — item list, task designer, entry, importance bar chart. Provenance stamp `conjoint (N=…)` written to affected value-matrix cells.
*Accept:* E2E-08: run a 3-attr × 3-level CBC on the shipped synthetic dataset → recover part-worths within 3·SE; apply bridge → value matrix updates with provenance tag; disabled-bridge tooltip appears when price attribute is removed; MaxDiff normalized scores displayed correctly.
*Review:* bridge messaging honest — "conjoint-inferred WTP under the modeled attribute levels," not "true WTP." If schedule slips, the bridge *action* here is the cut point (the P7d-1 estimator + chart still ship); see the scope posture below.

**P7e — Positioning: competitive map + competitor-in-envelope** · M · → v1.1 extension (after v1.0)
`competitive.ts` (Pareto point set, selected-segment break-even rays, competitor-offer synthesis into segment offer sets), Positioning view: competitor entry (name/price/per-segment value with an "overall" default), selected-segment 2D map with rays + points + tier dots + direct-dominance verdict, per-segment competitor-share readouts wired into the wind-tunnel dot panel and KPI header (competitor-loss share).
*Accept:* T-CMP-01…07 pass; E2E-09: add 3 competitors, select a segment, map renders in both themes, per-segment competitor-share appears in wind-tunnel dot panel and KPI header; toggle "include competitors in simulation" cleanly on/off with all KPIs restoring identically (T-CMP-06).
*Review:* competitor share does not corrupt Revenue/Potential accounting (competitors contribute to CompetitorLoss, never to OwnBuyerSurplus); map never interpolates a fake competitor offer; competitor-loss KPI only shown when at least one competitor exists.

**P8 — Pricing Decision Record** · M · → v0.9.0
Pricing Decision Record generator: assumptions + provenance, design summary, KPIs + waterfall, sensitivity summary + validation to-dos, linter findings, alternatives (other designs' KPIs), seed + date; optional research/positioning sections appear only when their extension is present. Export as Markdown download and print-CSS PDF.
*Accept:* E2E-10: generate record → markdown file downloads, contains the scenario's actual numbers (asserted); print stylesheet produces sane single-doc layout (manual check + snapshot of print CSS applied).
*Review:* the record reads like a strategy artifact; no invented claims — everything traceable to engine output; sections gracefully omit when their source extension is absent.

**P8b — Pricing-page mock** · M · → v1.1 extension (after v1.0)
Pricing-page mock rendered from the active design (clean, copy-ready, theme-aware).
*Accept:* mock renders all tiers/fences of templates correctly in both themes.
*Review:* it remains a presentation layer over the active design, never a second source of pricing truth.

**P9 — Hardening & v1.0.0** · M · → v1.0.0
Full axe pass on all views in both themes; cross-browser (Chromium/Firefox/WebKit smoke); perf budgets re-verified; content/copy edit pass; README rewrite with screenshots + live Vercel URL; MODEL-SPEC final read-through against implementation; template polish; release checklist.
*Accept:* all suites green incl. full e2e matrix; spec-coverage = 100% of MODEL-SPEC sections; bundle ≤ 300 KB gz; axe 0 serious/critical anywhere in either theme; a stranger can go URL → defensible decision record in < 15 minutes using only in-app guidance (owner user-test).
*Review:* end-to-end coherence — does the whole thing feel like one product; is anything shippable-embarrassing left.

**Release contract (not a fallback).** The **v1.0 required core** is P0–P6a, P7a, P7b, and P8's Pricing Decision Record. It is built end-to-end before any extension. **v1.1 extensions** are P6b, the pricing-page mock, P7c's bundling view, P7d-2's bridge/view polish, and P7e's positioning map. Their engines may land earlier only when they have a tested dependency in the core (for example, competitor offers support E7), but an engine without its view is not marketed as shipped. This release contract supersedes the earlier broad "in v1" wording in D-06/D-07.

**Extension order.** After v1.0 is green, add the pricing-page mock, P6b mechanism view, P7c bundling view, P7d-2 bridge/view, and P7e positioning map only as individually reviewed v1.1 increments. The core loop is not cuttable.

---

## 7. Test & verification spine

1. **L0 — numeric primitives:** reference-value tests for Φ/Φ⁻¹/partial expectations (T-STAT-*). Nothing downstream is trusted until these pass.
2. **L1 — closed-form unit tests:** every §4 module vs hand-derived or literature results (T-ENV/ECON/SCRN/SWP/BND/VW/MC/CNJ/MXD/CMP/LNT-*). Test IDs are stable and cited in code comments.
3. **L2 — property/invariant tests (fast-check):** conservation identity (including fenced free tiers and competitors), share simplex, own-price monotonicity, envelope-vs-brute-force agreement, dominated-offer nullity, MNL concavity/status handling (negative-definite Hessian only on identified fixtures), and Pareto-set monotonicity. These catch what example-based tests can't.
4. **L3 — golden scenarios:** the 3 templates + adversarial fixtures + a canonical CBC fixture + a canonical Positioning fixture snapshot full engine output (JSON, seeded). Any diff is a reviewable event, not noise.
5. **L4 — UI tests:** Testing Library for stateful components (matrix editing, undo, part-worth-bar apply-bridge flow); Playwright smoke flows E2E-01…10 (< 3 min total, chromium in CI, tri-browser at P9).
6. **L5 — CI gates:** typecheck, lint (incl. engine purity + `eslint-config-next` rules), all tests, bundle size, axe (serious+, both themes), spec-coverage report.

**Spec-citation discipline:** every engine test carries `@spec §x.y`; `scripts/spec-coverage.mjs` fails CI if a MODEL-SPEC section has zero citing tests (from P2 on). Coverage target: `lib/engine` ≥ 90% branches — but the named T-tests, not the percentage, are the real bar.

**7.5 Phase gate protocol:** agent runs the global DoD commands + the phase's acceptance list verbatim → records results in the session log (§13) → updates ledger/backlog → **proposes exact version + commit message per §0 rule 4** → stops. Opus reviews against the acceptance list + review-focus line. Owner gives the commit ask (§8.3). Any acceptance failure = phase stays open.

**7.6 Non-functional gates:** perf numbers, bundle size, and axe results (both themes) recorded per phase in the session log; regressions block the gate.

---

## 8. Repo conventions

### 8.1 Structure (target once P0 lands)

```
README.md          public-facing: what/why/run/deploy/stack (outline: §8.2)
CONTEXT.md         this doc: plan, spec pointer, ledger, backlog, protocol, log
LICENSE            MIT — Sidakpreet Singh
package.json       Next.js app; version renders in footer
next.config.mjs    output: 'export', no image loaders, no remote origins
tailwind.config.ts tokens (light/dark), Okabe–Ito palette per theme
tsconfig.json      strict
docs/              MODEL-SPEC.md (P1+), screenshots (P9)
app/               Next.js App Router (layout.tsx, page.tsx, globals.css)
components/        UI components, views, charts (SVG)
lib/engine|state/  per §3.3        content/  glossary, linter copy, templates
e2e/               Playwright        public/  favicon, static assets
.github/workflows/ ci.yml            scripts/ spec-coverage.mjs
```

### 8.2 README-vs-CONTEXT split

README = the product's public story: one-liner, thesis paragraph, screenshots, feature tour, quickstart (`npm i && npm run dev`), stack + architecture in 10 lines, deploy-your-own, docs map, license. Status-honest at every stage (currently: planning). No process/agent material.
CONTEXT (this doc) = everything about *how the work happens*: the plan, decisions, protocols, session log. A README reader should never need this file; an agent should rarely need more than this file.

### 8.3 Versioning & commits

- Semver-per-commit, message format: **`Commit vX.Y.Z: <imperative summary>`**. Version also lives in `package.json` and renders in the app footer.
- Mapping: v0.0.1 = this init; minor bump per completed phase (P0→v0.1.0 … P8→v0.9.0); patches for increments within a phase; **v1.0.0 = P9 sign-off**.
- **Commit protocol (owner-owned, agents never skip):** the agent proposes the exact version number and the exact commit message, lists the staged files, and waits for the owner's explicit go-ahead on both before running `git commit`. The owner may amend either the version or the message before authorizing; the agent respects the amendment verbatim. Trunk-based on `main`; short-lived branches only for risky spikes. `git log` is the changelog (the message format makes it one).
- **v0.0.1 commit timing (from S1):** when the owner authorizes this init commit — whenever that is — run it post-dated: `GIT_AUTHOR_DATE="2026-07-13T15:00:00+05:30" GIT_COMMITTER_DATE="2026-07-13T15:00:00+05:30" git commit -m "…"` so the timestamp lands on July 13 at 3 PM local. One command, no scheduling machinery (owner's call, S1b). The log entry reads `2026-07-13 15:00`.

---

## 9. Decisions ledger

| # | Decision | Options considered | Why |
|---|---|---|---|
| D-01 | **Next.js (App Router, `output: 'export'`) + React + TS strict + Tailwind, static export, Vercel deploy** | Vite+React (prior pick); vanilla ESM | Owner fluency + Vercel's zero-config repo-connect; App Router gives clean file-based structure without SSR while `output: 'export'` keeps the bundle a plain static SPA (no server, no data). §3.1–3.2. Flipped from Vite in S1. |
| D-02 | One primitive: utility upper envelope over offer sets | Per-feature ad-hoc rules; MC-only simulation | Concentrates correctness in one tested function; tiers/free/add-ons/bundling/competitors all unify; closed form → instant UI |
| D-03 | Within-segment heterogeneity analytic (lognormal ε), MC only for assumption uncertainty | MC for everything | Two conceptually different uncertainties; analytic layer is exact, fast, and testable against closed forms; MC reserved for what genuinely needs it |
| D-04 | Lognormal for WTP scale & bands | Normal, triangular, uniform | Positive support, right skew (matches WTP reality), closed-form partial expectations, clean P10/P90 fitting |
| D-05 | Hero = simulation wind tunnel; methods live in Analyze | Methods-led (PSM/conjoint centerpiece) | The live self-selection reveal is the differentiated thesis and the demo moment; methods-led reads as a stats toolkit. Methods feed the model rather than star |
| D-06 | **v1.0 ships the Decision Record; the pricing-page mock is a v1.1 extension** (Markdown + print-CSS PDF) | Defer the record entirely; PNG/PPTX export | The record is the strategy artifact and cannot be cut from v1.0. The mock is valuable but not allowed to delay a correct core; its status now matches §2 and §6. |
| D-07 | **CBC/MNL, MaxDiff-lite, and competitive positioning are v1.1 extensions built to the stated standard; competitor-as-alternative is core engine capability** | Shallow versions; quietly omit methods | Owner directive S1 was depth, not a license to ship a fragile UI before the core. v1.1 retains analytic MNL, a valid WTP contract, MaxDiff, and a segment-scoped map; competitor loss remains a core simulation/linter capability. |
| D-08 | Behavioral effects as cited linter checks, not numeric multipliers | Model decoy uplift quantitatively | Invented uplift coefficients would violate the correctness bar the product stands on; deterministic detectors with honest citations are defensible and more useful. (Disagrees with the brief's hint — deliberately.) |
| D-09 | Metrics: flat + per-seat (seats as segment multiplier); usage-based deferred | Full usage engine in v1 | Per-seat is one multiplication and covers most SaaS; usage needs consumption distributions — a second engine that would dilute v1 correctness focus |
| D-10 | Free tier = price-0 offer in the engine; trials deferred | Model trials | Free falls out of the envelope naturally (and powers leakage analysis); trials are temporal dynamics (v2 with retention) |
| D-11 | PSM is survey-first; model-generated curves are explicitly "illustrative" | Synthetic PSM as a headline feature | Synthetic survey results dressed as evidence = exactly the fake-rigor slop to avoid |
| D-12 | Charts hand-rolled SVG; no chart/router libs; deps capped at 7 | Recharts/visx; react-router | Bespoke viz (dots, waterfall, envelope, PSM crossings, part-worths, positioning map, tornado) has no library anyway; bundle, control, craft. Tabs are state; hash reserved for scenario payload |
| D-13 | **Light + dark themes both audited AA; palette re-toned per theme; theme is a scenario setting with header override** | Light only (prior pick); dark only | Flipped in S1: full theme parity signals design maturity for a portfolio piece, and Next.js + Tailwind make it cheap. Zero-network claim unchanged (no analytics, self-hosted font) |
| D-14 | v0.0.1 = docs + license + .gitignore only; toolchain lands in P0 | Scaffold now | Keeps the first commit purely reviewable intent; tool versions get chosen fresh at scaffold time, recorded in D-19b |
| D-15 | **Vercel deploy via repo-connect (Next static export)** | GitHub Pages (prior pick), Cloudflare Pages, Netlify | Owner-confirmed S1 given the Next.js pick; Vercel is the first-class path for Next; free tier trivially covers a portfolio SPA; still no lock-in (the built `out/` folder is portable to any static host) |
| D-16 | Ties break to cheaper offer; documented everywhere | Break to higher tier | Conservative revenue reporting; determinism in tests (see T-BND-04 for why it matters) |
| D-17 | CBC v1 = pooled MNL with random-balanced design; HB and D-efficient designs deferred to v1.1 | Full HB + D-efficient in v1 | Pooled MNL with SEs is enough to bridge to the value matrix honestly and enough for the pedagogical arc; HB is a step-change in complexity (per-respondent priors, sampler tuning) that would dilute correctness focus. D-efficient search is a refinement, not a foundation |
| D-18 | Competitor implemented as BOTH a chart point (Positioning map) AND a per-segment offer in the envelope engine | Chart-only competitor entry | Dual role means the same competitor data drives a picture AND a number; the "what % of enterprise leaves for Snowflake at this price?" answer emerges from the same envelope math the tiers use — no second engine, no fake integration |
| D-19 | Production dep budget bumped 6 → 7 to accommodate `next` | Stay at 6 by picking Vite; drop lz-string and store scenarios by base64+JSON | 7 is the minimum honest count for the stack chosen in D-01; the accent isn't dependency-count-as-vanity, it's *no chart libs, no router libs, no state megaframeworks*, which still holds. lz-string earns its slot: URL-hash sharing is a key product feature and JSON is not trivially short |
| D-20 | **Dual tie mode:** the simulator uses `'conservative'` (D-16); the closed-form screening/bundling oracles use `'seller-favorable'` at the σ→0 point path (participate + highest-value on ties) | One global tie rule; perturb fixture prices below binding values; recompute oracle numbers under the lognormal model | Review S2 (verified by simulating the spec) found D-16 alone makes T-SCRN-* return ≈2000 not 8000 and T-BND-01 return 0 not 20·n — the textbook menus price served buyers exactly at binding values, so every served buyer is on a tie. Seller-favorable is the δ→0 limit and recovers the exact textbook numbers; conservative stays the product default. Guard added to §4.2/§4.5/§4.6 so a weak executor can't "fix" the engine to force the wrong number; T-SCRN-01 moved σ=1e-3 → σ=0 (1e-3 drops the below-median half regardless of tie rule) |
| D-21 | Competitive value frontier is the **Pareto min-price-at-value staircase**, not a convex hull | Convex hull under (V, −P) | Review S2: the prose said "convex hull" while the construction algorithm computed the Pareto set — a contradiction that would give a weak model two different frontiers. Competitors are discrete and unmixable, so a Pareto-efficient point interior to the hull must stay on the frontier; the staircase is the correct object. Purged "hull"; T-CMP-01 pinned with a hull-vs-staircase fixture |
| D-22 | MaxDiff normalization gets an explicit **all-equal degenerate branch** → uniform 100/k | Leave the shift-then-normalize formula as-is | Review S2: on the symmetric fixture (exactly T-MXD-01's input) shift-then-normalize divides by zero → NaN. The degenerate branch makes the asserted "all scores equal" well-defined |
| D-23 | **P7d split** into P7d-1 (estimator) / P7d-2 (views) + a mandatory analytic-vs-finite-difference **derivative gate** before the MNL recovery test | Keep P7d as one L phase; rely on the concavity tests only | Review S2: the analytic MNL gradient/Hessian is the highest-risk engine math for a lower-capability executor, and the recovery test is a global pass/fail that doesn't localize a sign/index bug. The derivative gate is a local correctness signal; the split de-risks sizing and makes the bridge UI a clean cut point |
| D-24 | **Scope posture:** the non-cuttable core (P0–P6a, P7a, P7b, P8 record) is v1.0, built first; research-method *view layers* (P7c / P7d-2 / P7e) are optional-by-default, folded in only when the core is green | Treat all phases as equally required, cutting only under schedule pressure | Review S2: v1 is 2–3 products' worth of surface for a weaker executor. Making the research-method *views* optional-by-default (the engines still ship — they share the primitive) front-loads correctness onto the core without dropping capability. Reframes the existing cut lines from fallback into the plan |
| D-25 | **Correct the value waterfall and price units** | Preserve the earlier formulas and account/seat ambiguity | `OwnBuyerSurplus` excludes competitors, free fenced offers contribute to `FencingGap`, and the identity now holds. WTP is account-level at the segment's seat count; per-seat sweeps use per-seat bounds. |
| D-26 | **Positioning is segment-scoped and discrete; CBC bridge requires identified numeric price** | Interpolated frontier gaps; categorical-price pseudo-WTP | Zero-utility rays have slope `ε`; a discrete competitor map cannot interpolate unavailable offers. MNL is concave only under stated conditions, and a single price coefficient exists only for numeric price. |
| D-27 | **Bound sharing and verify runtime privacy** | Put all survey records in URLs; grep-only network checks | Compact URLs exclude respondent records and have byte/decompression guards; complete transfers use JSON. A browser request audit and deployment CSP substantiate the zero-network claim. |
| D-28 | **v1.0/v1.1 release contract resolves prior scope contradictions** | Keep mandatory-v1 ledger entries alongside optional cut lines | Lower-capability executors now have one source of truth: Decision Record/core first, then reviewed extensions. |

**D-19b (Phase 0):** exact toolchain versions to be recorded here at scaffold time — `next@X.Y.Z`, `react@X.Y.Z`, `typescript@X.Y.Z`, `tailwindcss@X.Y.Z`, `vitest@X.Y.Z`, `fast-check@X.Y.Z`, `playwright@X.Y.Z`, etc. Pin exact, don't caret.

## 10. Decisions I'm making for you — veto before build

V-01 **Stack: Next.js (App Router, `output: 'export'`) + React + TS + Tailwind, client-only static, Vercel deploy** (D-01, D-15). *S1: confirmed by owner.*
V-02 Hero framing: wind tunnel leads; PSM/Conjoint/Bundling live in Analyze — Research; competitive map in Analyze — Positioning (D-05).
V-03 Roster: **Conjoint (with a valid numeric-price bridge), MaxDiff-lite, and the competitive map are reviewed v1.1 extensions; competitor-as-alternative is core** (D-07). *S1 depth directive preserved.*
V-04 Behavioral effects: linter with citations, **no** numeric decoy multipliers — this overrides the brief's "model behavioral effects" hint on correctness grounds (D-08).
V-05 **Communicate in v1.0:** Decision Record, Markdown + print-PDF. The pricing-page mock follows as a reviewed v1.1 extension (D-06).
V-06 Templates in v1: exactly 3, data-only presets (§2.1).
V-07 Metrics: flat + per-seat; usage-based, trials, annual-term discounts deferred (D-09, D-10).
V-08 Buyer model: lognormal ε within segments (analytic), MC only for assumption bands (D-03, D-04).
V-09 **Light + dark themes both audited**; zero analytics/telemetry, self-hosted font (D-13). *S1: dark mode added.*
V-10 Dependency cap: **7** production deps as listed (§3.4, D-19).
V-11 **Commit protocol**: agent stops and proposes exact version + message; owner must approve both before `git commit` runs (§0 rule 4, §8.3). *S1: owner-stated.*

Silence = consent for any item; an overruled item gets a ledger entry and plan patch before P0 starts.

## 11. Open questions & owner actions

1. ~~**Repo:**~~ **Resolved:** `origin` is configured and `main` tracks `origin/main`. **Deploy:** owner still connects the existing repository to Vercel — needed for P0's live URL; CI/config can land first.
2. ~~**Product name.**~~ → **Resolved S1: "Product Studio" for now**, revisit later. Placeholder is fine through P4; the first-run copy uses this name.
3. ~~**Veto pass on §10**, then explicit go-ahead to commit v0.0.1.~~ → **Resolved S1c:** owner gave the go-ahead 2026-07-12; version `v0.0.1` and message `Commit v0.0.1: Init + Readme + Context` used as proposed. (§10 items remain overridable until P0 starts.)
4. **v0.0.1 commit timestamp:** owner asked (S1) that this commit land dated **2026-07-13 15:00 local**. When the go-ahead arrives, the agent post-dates the commit with `GIT_AUTHOR_DATE="2026-07-13T15:00:00+05:30"` and `GIT_COMMITTER_DATE="2026-07-13T15:00:00+05:30"` — no scheduled task (owner's simplification, S1b). → **Done (S1c):** committed post-dated as specified.
5. FYI: local `main` tracks `origin/main`; no deployment is implied until the Vercel connection in item 1 exists.

## 12. Backlog (v1.1 / v2 parking lot)

Usage-based metric engine · hierarchical Bayes for Conjoint (per-respondent part-worths) · D-efficient design generation for Conjoint · MNL best-worst estimation for MaxDiff · annual/term discounts & trial dynamics (retention model) · joint price optimizer with honesty framing · elasticity + cross-tier substitution matrix readout (analytic derivatives of §4.2 shares) · non-additive feature values (complements/substitutes) · segment-to-competitor value mapping via a survey shortcut · currency/i18n · PNG/slide exports · scenario version history & diffing · in-app worked tutorial ("defend this decision" walkthrough).

## 13. Session log

**S0 — 2026-07-12 (Fable, planning).** Repo initialized (`main`). Authored CONTEXT.md, README.md, LICENSE (MIT, Sidakpreet Singh), .gitignore. Staged for `Commit v0.0.1: Init + Readme + Context` — **not committed** (awaiting owner preview + explicit ask). Next: owner veto pass (§10), answers to §11, then commit and start P0.

**S1 — 2026-07-12 (Fable, revising per owner).** Owner delivered directives on the initial plan (via chat, this session):
- **Stack:** switch to Next.js (App Router, `output: 'export'`) + Vercel deploy → D-01 and D-15 flipped; §3 and P0 rewritten; dep budget bumped 6→7 (D-19); bundle gate raised 250→300 KB gz to account for Next runtime overhead (still hand-audited at P0).
- **Roster:** D-07 flipped — Conjoint (CBC) with proper MNL estimation, MaxDiff-lite, and Competitive price-value map are **in v1, done properly**. Added §4.10 Conjoint/MaxDiff-lite (MNL log-likelihood, Newton-Raphson, effects coding, price-bridge to WTP with significance gate) and §4.11 Competitive map & competitor-as-alternative (frontier hull, iso-utility rays, competitor-offer synthesis into the envelope engine with a new `CompetitorLoss` term in the conservation identity). Added phases P7d Conjoint/MaxDiff and P7e Positioning; new tests T-CNJ-01…06, T-MXD-01…02, T-CMP-01…06; added linter rule E7 (competitor loss); renumbered §4.10→§4.12.
- **Communicate:** D-06 reinforced — not deferred, no cuts.
- **Dark mode:** D-13 flipped — light + dark, both AA-audited; theme is a scenario setting with header override; palette re-toned per theme.
- **Product name:** kept "Product Studio" for now (§11 item 2 resolved).
- **Commit protocol:** owner formalized — agent must stop and propose exact version + commit message before every commit, owner approves both. Encoded in §0 rule 4, §8.3, and new veto V-11.
- **v0.0.1 timestamp:** owner asked for commit dated 2026-07-13 15:00 local (§8.3, §11 item 4) — handled by post-dating the commit with git date env vars at whatever time the go-ahead arrives.

Files updated: CONTEXT.md (this revision), README.md (stack + roadmap patch). All staged, **still not committed** (per §0 rule 4 and the timestamp ask).

**S1b — 2026-07-12 (Fable, continuation).** S1's session was interrupted mid-README patch; a fresh session resumed from this doc and completed the remainder: README stack line (Next.js static export, Vercel, light + dark) and P7 roadmap row (now lists Conjoint/MaxDiff + competitive positioning); .gitignore rewritten for Next.js (`.next/`, `out/`, `next-env.d.ts`, `*.tsbuildinfo`, `.vercel`). Owner simplified the timestamp mechanism: no scheduled reminder — just post-date the commit via git date env vars when the go-ahead arrives (§8.3 and §11.4 updated; the briefly-created scheduled task was deleted). All four files re-staged; **still not committed**, awaiting the owner's explicit approval of version + message.

**S1c — 2026-07-12 (Fable).** Owner asked for a pickup-completeness check, then gave the conditional go. Reverify pass clean: index = worktree (no drift), LICENSE verified (MIT, Sidakpreet Singh), no stale stack references outside intentional ledger records. `Commit v0.0.1: Init + Readme + Context` executed, post-dated `2026-07-13 15:00 +05:30` per §8.3 — repo history begins. Nothing pushed (no remote). Next: owner creates GitHub repo + Vercel connection (§11.1); P0 scaffold → v0.1.0 on the next explicit ask.

**S2 — 2026-07-13 (Opus 4.8, fresh-eyes structural review).** Owner commissioned an all-angles soundness review of the plan before execution (sibling repo gt-sandbox was reviewed in the same pass and cleared on core correctness — its six game oracles recompute exactly and its 2×2 classifier is provably complete). Findings here were verified by direct recomputation, not judgment: the game-theory oracles by exact rationals, and the PS economic engine by simulating it against its own §4 spec. One critical defect plus several smaller fixes, all applied to this doc:
- **Tie-rule / oracle contradiction (critical, D-20).** Simulating the engine under D-16 ("break to cheaper / outside") returns screening revenue ≈2000 (σ=0) or 5000 (σ=1e-3), never 8000; a pure bundle priced at value returns 0 (σ=0) or 10·n, never 20·n. Root cause: the textbook menus price served buyers at exact binding values (all ties), and D-16 sends the indifferent buyer to the outside option. Fix: a dual tie mode — `'seller-favorable'` (participate + highest-value on ties, σ→0) for the closed-form oracles, `'conservative'` staying the simulator default — plus guard notes in §4.2/§4.5/§4.6 so the engine is not "fixed" to force the number, and T-SCRN-01 moved σ=1e-3 → σ=0. Verified: the corrected convention reproduces T-SCRN-01…04 (8000 / 6000 / 5000>4400) and T-BND-01 (20·n) exactly.
- **Competitive frontier hull-vs-staircase (D-21):** purged "convex hull," pinned the Pareto staircase.
- **MaxDiff divide-by-zero (D-22):** added the all-equal degenerate branch → uniform 100/k.
- **Conjoint hardening (D-23):** split P7d, added a mandatory analytic-vs-finite-difference derivative gate before the recovery test.
- **Scope posture (D-24):** core-first v1.0; research-method view layers optional-by-default.
- **Minor:** reconciled the §3.2 "sub-10 ms" vs §3.5 "<150 ms" MC figures (raw compute vs end-to-end ceiling); removed the stale "vs. Vite budget" phrasing in §3.5; made P1's competitor-offer synthesis functional (not a stub) so E7 is genuinely testable in P5.
Ledger D-20…D-24 added; §3.5, §4.2, §4.5, §4.6, §4.10, §4.11, §4.12, §6 (P1, P5, P7d→P7d-1/-2, scope posture) revised; README status bumped to v0.0.2. Version → v0.0.2. Owner authorized `Commit v0.0.2: Key Implementation Fixes /O8`, post-dated 2026-07-13 16:00 (+05:30) — one hour after v0.0.1. Nothing pushed (no remote yet, §11.1).

**S2b — 2026-07-13 (fresh live-repo re-review).** The prior S2 fixes remain valid, but a fresh audit found additional genuine contract defects that would make a faithful implementation wrong: the §4.3 identity double-counted competitor utility and omitted free-tier fencing; per-seat value/sweep units were ambiguous; the Positioning map used the reciprocal ray slope and interpolated discrete competitors; the CBC bridge used an undefined categorical `β_price` and assumed strict concavity; URLs could silently exceed practical limits with respondent records; and v1.0/v1.1 scope statements conflicted. Applied D-25…D-28: corrected the formulas and tests, made Positioning segment-scoped/discrete, hardened CBC estimator/bridge statuses, bounded compact sharing and added runtime request verification, and made the release contract authoritative. Updated the stale remote/deploy handoff facts. This amendment is folded into `Commit v0.0.2: Key Implementation Fixes /O8` at the same post-dated time; no application code exists or was changed.
