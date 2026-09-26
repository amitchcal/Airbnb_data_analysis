# Airbnb Antwerp Pricing Dashboard — Presenter's Script

Written to be read almost verbatim. Purpose: bridge the 13–22 September gap (personal leave) — you built this on 12 September, but you're presenting after 9 days away. Don't rely on memory; rely on this script. See Guide Section 25.4.

**Status of this draft:** fully filled in overnight 25→26 Sep, after all 3 core pages were built and every number verified live in Power BI (including an independent pandas cross-check of the baseline revenue figure against the raw CSV). No `[FILL IN]` placeholders remain. Before presenting: do one read-through and swap in real screenshots for the two dashboard-walkthrough slides (5 and 6), and double-check the worked-example math in Slide 8 against the live dashboard (it's a calculated estimate, not a screenshotted value — flagged inline).

---

## Cold-open checklist (read this first if you're rushed)

If you remember nothing else, remember these five things:

1. **1,749 listings**, **1,111 hosts**, Antwerp, 26 Dec 2021 – 25 Dec 2022.
2. **Availability rate: 53.52%** — just over half of listing-nights are open or blocked; this is *not* a booking rate.
3. **Average price EUR 109.92, median EUR 79** — the average is pulled up by a long tail of expensive listings (max EUR 5,800).
4. **Baseline scenario = 70% occupancy × 1.00x price multiplier** — a neutral assumption I chose, not something in the data. Every other scenario is a delta off this line.
5. **My strongest insight:** Moving occupancy from 70% to 80% (multiplier held at 1.00x) lifts projected revenue by exactly +14.29%, from €24.55M to €28.06M — a clean, mechanical relationship, because occupancy is a linear multiplier in this model. That linearity is itself worth saying out loud: it means the dashboard shows you the *arithmetic* of a scenario, not a *forecast* of whether that occupancy is achievable.

---

## Slide 1 — Title

> "This is the Airbnb Antwerp Price & Scenario Analysis Dashboard — a self-service Power BI tool I built to let property owners and analysts explore pricing and revenue scenarios without any machine learning. The data covers 1,749 listings across Antwerp, over the calendar year from 26 December 2021 to 25 December 2022."

*If asked why no ML:* "That was a requirement of the brief — the goal was a transparent, self-service tool a non-technical stakeholder could drive themselves, not a black-box model."

---

## Slide 2 — Objective & dataset (Context)

> "The objective was to let stakeholders analyze listing performance and explore revenue scenarios interactively. The dataset has four tables: Listings, Calendar, Hosts, and Reviews — 1,749 listings, 319,192 calendar rows, 62,987 reviews, and 1,111 hosts. One thing worth flagging up front: the calendar only covers 151 to 209 days per listing, not a full year — so anything I say about 'revenue' today is a projection over the days we actually have, not a full-year actual."

---

## Slide 3 — Data preparation & star schema

> "Behind the dashboard is a star schema: a listings dimension, a hosts dimension, a date dimension, and two fact tables — one for the daily calendar, one for reviews — plus a bridge table for amenities, since each listing has a variable list of amenities, things like Wifi, Kitchen, or Smoke alarm, that needed to be broken out into their own rows rather than left as one long text string. Two cleaning decisions mattered most: amenities had to be JSON-parsed and case-normalized — 'AEG refrigerator' and 'aeg refrigerator' looked like the same amenity to a person but were technically two different values until I standardized the casing — and bathroom counts had to be parsed out of free text like '2.5 shared baths' into an actual number. But the two decisions that most affect every number downstream are: I never zero-filled a missing price — that would have silently dragged the average down — and I treat 'unavailable' strictly as blocked or unbooked, never as a confirmed booking."

---

## Slide 4 — What-if scenario design

**Context:** "Stakeholders wanted to ask 'what if occupancy were higher' or 'what if we raised prices in peak season' without touching a single formula."
**Insight:** "So I built two adjustable parameters — Occupancy Rate and a Seasonal Multiplier — and one baseline: 70% occupancy, 1.00x multiplier."
**Implication:** "That baseline isn't in the data or the brief — I picked it as a neutral midpoint so every scenario reads as a clear delta off a fixed reference line, not a moving target."
**Action:** "If your institution specifies a different baseline occupancy, it's a one-line change in the DAX — I'll show where."

*If asked why 70%/1.00x specifically:* "No baseline was specified anywhere in the brief or data description — I needed *a* fixed reference point, and 70% is a commonly used mid-range occupancy assumption in short-term rental analysis, not a value implied by this dataset."

---

## Slide 5 — Dashboard walkthrough: Overview

> "This is the Overview page — the landing page. Five KPI cards across the top: average price at €109.92, availability rate at 53.52%, total listings at 1,749, and projected revenue at €24.55 million under the default scenario, plus lifetime review count and average host tenure for extra context. Below that, a trend line shows average price declining from about €116 in late 2021 down to roughly €108 by late 2022 — and a listing-mix chart shows Entire Home/Apartment dominating the market, with Private Room a distant second, and Hotel Room and Shared Room both very small categories. Five slicers across the top — date, property type, room type, and the two What-If sliders — filter every visual on this page at once."

---

## Slide 6 — Dashboard walkthrough: Listing Analysis & Scenario Insights

> "On the Listing Analysis page: a geographic view of every listing colored by room type and sized by projected revenue, a price distribution histogram, and a top-20 listings table sorted by projected revenue. One note on the geographic view — I originally built this as a proper street-map visual, but Power BI's Map and Azure Maps visuals are both blocked in this environment by a tenant-level admin restriction, not something I could fix from Desktop settings. So I substituted a scatter chart plotting longitude against latitude directly — it doesn't have street-map tiles underneath, but it correctly traces Antwerp's actual geographic shape and shows the same clustering a real map would, so the analytical value is preserved even though the visual polish differs slightly from a typical map."

**Context:** "On the Scenario Insights page, let's actually move a slider."
**Insight:** "Moving occupancy from 70% to 80%, multiplier held at 1.00x, takes projected revenue from €24,553,642 to €28,061,306 — a €3.51 million, +14.29% increase."
**Implication:** "For a portfolio this size, even a 10-point occupancy improvement is worth millions in projected revenue — which is exactly why hosts and platform operators care so much about occupancy-boosting levers like pricing strategy, review volume, and listing quality."
**Risk:** "That +14.29% isn't a forecast — it assumes the demand exists to fill those extra nights, which this dataset can't confirm one way or the other. Projected Revenue is `price × occupancy × multiplier`, an arithmetic scenario, not a prediction of actual future bookings."

---

## Slide 7 — Key insights

Each insight below: state what changed, why it matters, what to do about it. Fill in once the dashboard is built — do not present generic praise.

1. **The market is overwhelmingly whole-unit, not shared.** Entire Home/Apartment dominates the listing mix, Private Room is a distant second, and Hotel Room and Shared Room are both negligible slivers. This means Antwerp's Airbnb supply behaves more like a distributed hotel alternative than a budget shared-living market — pricing and marketing strategy should be benchmarked against short-term rental competitors, not hostels.
2. **The price distribution is sharply right-skewed.** Average price is €109.92 but the median is only €79 — a relatively small number of premium listings (up to €5,800/night at the extreme) pull the average well above what a typical guest actually pays. Anyone using "average price" alone to judge market positioning is overestimating what most listings actually charge.
3. **Availability at 53.52% is a supply signal, not a demand signal.** Just over half of all listing-nights across the year are open or blocked — this measures how much inventory exists, not how many nights are booked. It should never be read as an occupancy or booking rate; conflating the two would overstate how "busy" the market actually is.
4. **224 of 1,749 listings — 12.8% — have zero reviews.** That's a meaningful visibility and trust gap: a guest comparing options has no social proof for roughly one in eight listings, regardless of how competitively they're priced. This is a segment worth targeted attention rather than assuming price alone drives bookings.
5. **Every number described as "revenue" here is explicitly partial-year.** No listing has more than 209 of 365 days loaded in this extract, so nothing in this dashboard claims to be a full annual actual — every projection is scoped to the loaded, priced calendar rows and labeled that way throughout.

---

## Slide 8 — Worked example (one listing or room type)

**Assumptions:** "Let's take the single highest-projected-revenue listing in the whole dataset: 'Zanzibar bedroom in an authentic mansion,' a Private Room in a Townhouse priced at €5,800 a night. At the baseline 70% occupancy and 1.00x multiplier, its projected revenue is €702,380. Now let's ask: what if this host raised occupancy to 80% and applied a 10% seasonal premium — 1.10x?"
**Impact:** "Because Projected Revenue scales linearly with both occupancy and multiplier, that combination takes this one listing from €702,380 to approximately €882,992 — an increase of about €180,600, or +25.7%. [Verify this exact figure live on the Listing Analysis or drill-through page by filtering to this listing and moving both sliders — the math should match since Projected Revenue is a direct multiplication.]"
**Implication:** "For a single premium listing, that's a six-figure swing from two slider moves — which is exactly the point of building this as an interactive tool rather than a static report: a host or analyst can immediately see how sensitive their revenue is to occupancy and pricing assumptions, for any listing or filtered segment they choose."
**Risk:** "This is a scenario, not a prediction — it tells you what *would* happen if 80% occupancy and a 10% premium both held for this listing, not that they will. A €5,800/night listing achieving 80% occupancy is a strong assumption that would need real demand evidence to back it up."

---

## Slide 9 — Recommendations

Each of these must be a concrete action: owner + move + timeframe + measure. Not "optimize pricing."

1. **Property managers should benchmark new listings against the €79 median, not the €109.92 average**, when setting an initial asking price for a standard Entire Home/Apartment unit — and revisit that baseline quarterly as more calendar data loads, to avoid anchoring to a figure skewed by a handful of luxury outliers.
2. **Hosts of the 224 zero-review listings should be enrolled in a review-solicitation nudge within their next 2 completed stays**, with a follow-up check at the 90-day mark to confirm review count has moved off zero — treating this as a trust-gap problem to close, not just a byproduct of being new.
3. **Before running any occupancy-based marketing campaign, the platform should validate 3-5 real booking data points against this dashboard's availability figures**, since Projected Revenue and Availability Rate here are scenario/supply constructs, not observed booking rates — a campaign built on an unvalidated conflation of the two risks overpromising achievable revenue.
4. **Hosts in the Entire Home/Apartment segment should trial a seasonal multiplier of 1.05-1.10x for one identified peak month**, tracking week-over-week occupancy for 4 weeks before deciding whether to extend the premium into the following month — using the Scenario Insights page to pre-estimate the revenue upside before committing.
5. **Whoever owns this dataset should prioritize extending the calendar extract to a full 365 days per listing within the next data refresh cycle**, since every revenue figure in this dashboard is currently scoped to 151-209 days per listing — a full year would let "Projected Revenue" become an actual annualized figure instead of a labeled partial-year projection.

---

## Slide 10 — Limitations, assumptions, conclusion & sources

> "A few things this dashboard deliberately does not claim. Availability isn't the same as bookings — I never label a blocked night as a confirmed booking. The calendar is partial, so any full-year number is an explicitly labeled estimate, never the headline figure. Correlation-style patterns here describe association, not causation — I'm not claiming price *causes* the patterns we see, just that they co-occur.
>
> Putting it together: use this dashboard to explore *how* revenue responds to occupancy and pricing assumptions you supply, not to read off a single "correct" number — the value here is in the sensitivity, not in any one scenario being the forecast.
>
> Sources: Airbnb Antwerp listings/calendar/hosts/reviews extract, provided as part of the Hero Vired capstone brief."

---

## Appendix — anticipated Q&A

- **"Why isn't this using machine learning?"** → Brief requirement; transparency and self-service over black-box prediction.
- **"How do you know the baseline is reasonable?"** → It isn't derived from the data; it's a stated assumption (Guide Section 0, Assumption 3), changeable in one place.
- **"Is Projected Revenue real revenue?"** → No — it's `price × occupancy × multiplier` summed over loaded, priced calendar rows. It is explicitly not observed booking revenue (Assumption 5).
- **"What would you do with more time/data?"** → A full 365-day calendar per listing instead of the current 151-209 days, and actual booking/transaction data instead of availability, so "Projected Revenue" could become an observed-revenue comparison rather than a pure scenario construct.
- **"Why does the geographic view look like a scatter plot instead of a real map?"** → Power BI's Map and Azure Maps visuals are both blocked in this environment by a tenant-level administrative restriction (confirmed via an explicit "contact your admin" message from Azure Maps) — not a data or build issue. The scatter chart substitute plots the same coordinates and still correctly traces Antwerp's shape and listing clusters.
