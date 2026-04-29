# 🗳️ Peru 2026 — Electoral Analytics Dashboard

**Live dashboard:** [catoralonso.github.io/peru-election-dashboard-2026](https://catoralonso.github.io/peru-election-dashboard-2026)

Real-time electoral analytics system built during the April 2026 Peruvian general election. Processes partial ONPE results across 26 regions and projects a bicameral congressional composition and second-round outcomes using empirical statistical modeling.

> **Model validation in progress.** Second-round predictions will be compared against actual runoff results (expected July 2026). Historical prediction accuracy will be tracked here.

---

## Features

- **Interactive SVG map** — regional heatmaps with 2021/2026 historical toggle and Lima/Callao zoom inset
- **D'Hondt seat allocator** — real-time congressional composition across Senate (60 seats) and Chamber of Deputies (130 seats)
- **JEE contested ballots tracker** — monitors unprocessed ballots per region with criticality analysis for 2nd/3rd place race
- **Vote evolution chart** — time-series with backcasting, forward projection to 100%, and ONPE cutoff line
- **Second-round projection model** — dual-ceiling empirical model with regional uncertainty bands
- **Sankey vote transfer diagram** — canvas-rendered flow of eliminated candidates' votes to runoff finalists

---

## Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla JS, HTML5, CSS3 |
| Charts | Chart.js 4.4 |
| Map | Hand-coded SVG (543×792 viewport, 26 regions) |
| Sankey | Custom Canvas API implementation |
| Seat allocation | D'Hondt algorithm (from scratch) |
| Data | Static JSON — updated manually from ONPE |

No build step. No framework. No backend. Single `index.html`.

---

## Data Sources

| Source | Usage |
|---|---|
| [ONPE](https://www.onpe.gob.pe) | Official partial vote counts (actas contabilizadas) |
| [JNE](https://www.jne.gob.pe) | Seat distribution by region (Res. 0053-2025-JNE) |
| [IEP](https://iep.org.pe) | Second-round polling — Feb 2026 |
| [DATUM](https://datum.pe) | Second-round polling — Feb 2026 |
| ONPE 2021 historical | Empirical regional transfer coefficients |

---

## Methodology

### First Round — Congressional Seat Allocation

Seats are allocated using the **D'Hondt method** applied independently to each of the three electoral lists:

- **Senado Nacional** (30 seats) — national single district
- **Senado Regional** (30 seats) — 1 seat per region, 4 for Lima Metropolitana; winner-takes-all by plurality
- **Cámara de Diputados** (130 seats) — regional districts per JNE official allocation

```
D'Hondt quotient: V(p) / s
where V(p) = votes for party p, s = seats already assigned + 1
```

Congressional composition updates dynamically as ONPE partial counts change.

---

### Second Round — Projection Model

The model projects a Keiko Fujimori (FP) vs. José Williams Palomino (JP) runoff using five components:

#### 1. Empirical Regional Coefficients

Base transfer rates derived from actual ONPE 2021 data: fraction of third-party votes captured by Keiko/Castillo in each region during the actual 2021 runoff.

#### 2. 2026 Candidate Ideological Profiles

Each eliminated first-round candidate is assigned a transfer coefficient toward Keiko:

| Candidate | Party | Keiko transfer |
|---|---|---|
| Aliaga (RLA) | Center-right | +0.65 |
| Nieto (BG) | Center-right | +0.62 |
| Belmont (OBRAS) | Populist | +0.45 |
| Álvarez (PPT) | Left-leaning | +0.30 |
| Chau (AN) | Far-right | +0.32 |

15% abstention rate applied to all transfers.

#### 3. Dual Ceiling — IEP + DATUM (Feb 2026)

Projected Keiko vote share is capped at the average of two polling firms per macrozone, with 70% elasticity applied:

```
ceiling_k = (IEP_ceiling + DATUM_ceiling) / 2
projected_k = min(projected_k, ceiling_k × 0.70 + projected_k × 0.30)
```

| Macrozone | IEP ceiling | DATUM ceiling |
|---|---|---|
| Nacional | 54% | 50% |
| Lima/Callao | 52.9% | 41% |
| Norte | 50.2% | 47% |
| Sur | 58.4% | 66% |

Lima uses ±8pp uncertainty (vs. ±6.2pp nationally) due to higher Aliaga vote volatility.

#### 4. Antivoto Sánchez Palomino (Cerrón proxy — DATUM)

JP's anti-vote factor is applied as an external band reducing JP's projected ceiling:

| Zone | Antivoto factor |
|---|---|
| Nacional | 10% |
| Lima | 14% |
| Sur | 6% |

Applied with 50% elasticity as an outer uncertainty band, not a hard deduction.

#### 5. Uncertainty Quantification

```
RMSE: ±6.2pp (national), ±8pp (Lima)
Regions with margin < RMSE × 1.5 classified as "uncertain"
Outer band: RMSE + antivoto_band
Inner band: RMSE only
```

#### National Projection (as of April 23, 2026)

```
Keiko Fujimori (FP):  ~54%
José Williams (JP):   ~46%
```

Weighted by first-round valid votes per region.

---

### JEE Contested Ballots Analysis

Actas JEE are ballots flagged for formal review — they contain procedural observations, not necessarily fraud. Historically ~85% are eventually counted (ONPE 2021 baseline).

The dashboard estimates votes contained in JEE ballots as:

```
estimated_jee_votes(region) = actas_jee × (valid_votes / actas_counted)
```

Critical regions for the 2nd/3rd place race (Aliaga vs. Williams):
- **Lima Metropolitana**: 1,311 JEE ballots, RP avg. 19.9%
- **Extranjero**: 509 JEE ballots, RP avg. 25.5%

---

## Model Validation Plan

This dashboard will serve as a public prediction record. After the second round:

| Metric | Prediction | Actual | Error |
|---|---|---|---|
| Keiko national % | ~54% | TBD | TBD |
| JP national % | ~46% | TBD | TBD |
| Keiko wins regions | TBD | TBD | TBD |
| JP wins regions | TBD | TBD | TBD |

*Table will be updated after official JNE results.*

---

## Repo Structure

```
peru-election-dashboard-2026/
├── index.html          # Full dashboard (self-contained)
├── README.md           # This file
└── data/
    └── snapshots/      # Historical ONPE data points used for trend analysis
```

---

## Why this project exists

Built as a portfolio demonstration of applied data analytics on real, high-stakes, time-sensitive data. Electoral systems combine statistical modeling, algorithmic implementation, data visualization, and domain knowledge — all under the constraint of a live event with no second chances.

The second-round model is not a poll aggregator. It is a transfer model built on behavioral assumptions derived from empirical data, designed to be falsifiable when the runoff occurs.

---

## Author

**Alonso Arredondo Rodríguez**
Data Analyst · NLP · GCP · Python

[linkedin.com/in/alonso-arredondo](https://linkedin.com/in/alonso-arredondo) · [github.com/catoralonso](https://github.com/catoralonso)

---

*Data updates: manual, sourced directly from ONPE official releases. Last update: April 23, 2026 — 94.4% count.*
