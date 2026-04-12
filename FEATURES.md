# LocaLegacy — Feature Roadmap

## Status: 3 high-value features planned

---

## LG-F1: Price Staleness Badges

**Summary:** Every asset card in the Assets tab shows a badge indicating how many days since the price was last updated. Assets not updated in 7+ days show amber, 30+ days show red.

**Value:** Manual price tracking is only useful if users keep prices current. Visual staleness nudges fix the silent data decay problem that plagues all self-tracked portfolios.

**Data:** `asset.updatedAt` (timestamp, already stored). Fall back to `asset.createdAt` if `updatedAt` is missing.

**Badge rules:**
- < 7 days: no badge (fresh)
- 7–29 days: amber badge "7d ago" style
- 30+ days: rose badge with ⚠ icon

**UI:** Badge rendered inline in the asset card's tag row (next to SHORT-TERM / ILLIQUID chips).

**TODO:**
- [ ] Add `staleDays(asset)` helper: `Math.floor((Date.now() - (asset.updatedAt || asset.createdAt)) / 86400000)`
- [ ] Add `stalenessBadge(asset)` returning HTML string (badge or empty string)
- [ ] Insert badge into each asset card in `renderAssets()`
- [ ] Add `data-testid="stale-badge-${id}"` for tests

---

## LG-F2: Diversification Score

**Summary:** A dashboard widget showing a letter-grade diversification score (A–F) based on how concentrated the portfolio is by asset class.

**Value:** Most casual investors are over-concentrated in 1–2 asset classes without realizing it. A red "D" grade next to "92% in crypto" is a wake-up call they'll act on.

**Calculation:**
- Compute each class's % of total portfolio value
- Apply Herfindahl-Hirschman Index (HHI): `sum(pct_i^2)`  
- HHI of a perfectly even 5-class portfolio ≈ 0.20
- Score: A < 0.30, B 0.30–0.45, C 0.45–0.60, D 0.60–0.75, F > 0.75
- (Higher HHI = more concentrated = worse grade)

**UI:** Card in the Dashboard tab below the asset class breakdown. Shows:
- Letter grade (large, colored)
- Top line: "Portfolio is {grade description}" 
- Warning if any single class > 60%: "⚠ {class} is {pct}% of portfolio"

**TODO:**
- [ ] Add `calcDiversification(assets)` returning `{ score, grade, dominant, dominantPct }`
- [ ] Render diversification card in `renderDashboard()` below the by-class section
- [ ] Add `data-testid="div-score"` wrapper and `data-testid="div-grade"` text for tests

---

## LG-F3: Tax Lot Summary Widget

**Summary:** A dashboard card showing the total unrealized short-term vs long-term gains broken out separately, with a callout for the upcoming short-term-to-long-term conversion dates.

**Value:** Tax planning is the #1 reason sophisticated investors track positions manually. Knowing you have $8,400 in short-term gains vs $22,000 in long-term is actionable before year-end.

**Calculation:**
- Short-term: assets where `isShortTerm(asset)` is true (opened < 1 year ago)
- Long-term: assets where `isShortTerm(asset)` is false
- Sum unrealized gain/loss per bucket using existing `gainLoss(asset)` function
- "Converts to LT in X days": for each short-term asset with a gain, show days until the 1-year anniversary

**UI:** Card in Dashboard tab below the top/worst performer row. Shows:
- Two columns: Short-Term | Long-Term with total gain/loss each
- A mini list of "→ converts in X days" for short-term gains (top 3 by value)

**TODO:**
- [ ] Add `calcTaxLots(assets)` returning `{ shortTermGL, longTermGL, converting }` 
- [ ] `converting` = short-term assets with gain, sorted by soonest conversion, top 3
- [ ] Render tax lot card in `renderDashboard()` (show only when assets exist)
- [ ] Add `data-testid="taxlot-card"`, `data-testid="taxlot-st"`, `data-testid="taxlot-lt"` for tests
