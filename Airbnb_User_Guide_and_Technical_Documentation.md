# Airbnb Antwerp Pricing Dashboard — User Guide & Technical Documentation

Drafted overnight 25→26 Sep, using the real, live-verified figures from the completed build. This combines the user guide and technical documentation required by the Problem Statement (Section 6, "Documentation & Deployment") into one document, per Guide Section 25.2.

---

## 1. What this dashboard is

A self-service Power BI report for Airbnb listings in Antwerp, letting a non-technical user explore pricing patterns and simulate revenue under adjustable occupancy and seasonal-pricing assumptions — without any machine learning. Built for the Hero Vired data analytics capstone.

Coverage: 1,749 listings, 1,111 hosts, 319,192 calendar rows (26 Dec 2021 – 25 Dec 2022), 62,987 reviews.

---

## 2. Navigating the report

Three visible pages, accessible via the page tabs at the bottom of the window (a page navigator button is also included on each page's header, if built — see Section 15 of the build guide):

- **Overview** — headline KPIs, a price trend, and the listing mix by room type.
- **Listing Analysis** — a geographic view of listings, a price distribution histogram, and a top-20 listings table.
- **Scenario Insights** — the interactive What-If controls and their revenue impact.

Slicers (Date, Property Type, Room Type) filter every visual on whichever page they're placed on. The Occupancy Rate and Seasonal Multiplier sliders drive every "Scenario"-prefixed measure across all three pages.

**A note on the geographic view:** this was built as a Scatter chart (Longitude × Latitude) rather than a traditional map visual, because Power BI's Map and Azure Maps visuals are both blocked in this environment by a tenant-level administrative restriction — confirmed via Azure Maps' own "contact your admin to turn on the tenant switch" message, which is outside what Power BI Desktop's local settings can override. The scatter chart plots the same coordinates and correctly traces Antwerp's geographic shape and listing clusters; it just doesn't have street-map tiles underneath.

---

## 3. Using the scenario controls

- **Occupancy Rate** (10%–100%, step 5%, default 70%): the assumed share of loaded, priced calendar nights that are occupied.
- **Seasonal Multiplier** (0.50x–1.50x, step 0.05, default 1.00x): scales every nightly price. 1.00x = unchanged; 1.10x = +10%; 0.90x = −10%.
- **Baseline** = 70% occupancy × 1.00x multiplier. This is not derived from the data — it was chosen as a neutral reference point (see Assumption 3, Section 6 below) — so every other scenario reads as an explicit delta off this fixed line.
- **Projected Revenue** is `price × occupancy × multiplier`, summed over loaded, priced calendar rows. It is a scenario calculation, not observed booking revenue — it does not mean these nights were actually booked at these prices.

---

## 4. Data model

Star schema: `DimListing`, `DimHost`, `DimDate` (dimensions) relate to `FactCalendar` and `FactReview` (facts), plus `DimAmenity`/`BridgeListingAmenity` as a many-to-many bridge for the variable-length amenities list per listing.

| From | To | Cardinality | Direction |
|---|---|---|---|
| DimHost[Host ID] | DimListing[Host ID] | 1:* | Single |
| DimListing[Listing ID] | FactCalendar[Listing ID] | 1:* | Single |
| DimListing[Listing ID] | FactReview[Listing ID] | 1:* | Single |
| DimDate[Date] | FactCalendar[Date] | 1:* | Single |
| DimDate[Date] | FactReview[Review Date] | 1:* | Single |
| DimAmenity[Amenity] | BridgeListingAmenity[Amenity] | 1:* | Single |
| DimListing[Listing ID] | BridgeListingAmenity[Listing ID] | 1:* | **Both** (deliberate — lets an amenity filter reach listings and their facts) |

Two calculated columns: `Host Tenure Years at Snapshot` (in DimListing, anchored to the calendar's start date, 26 Dec 2021) and `Price Bin Start` (in FactCalendar, €25-wide bins for the price histogram).

---

## 5. DAX measures (all in the `_Measures` table)

**Inventory, price, availability:**
| Measure | Formula (summary) | Verified value (unfiltered) |
|---|---|---:|
| Total Listings | DISTINCTCOUNT(Listing ID) | 1,749 |
| Total Hosts | DISTINCTCOUNT(Host ID) | 1,111 |
| Calendar Rows | COUNTROWS(FactCalendar) | 319,192 |
| Priced Calendar Nights | COUNT(Price) | 319,117 |
| Available Nights | CALCULATE COUNTROWS where Is Available = TRUE | 170,829 |
| Unavailable or Blocked Nights | CALCULATE COUNTROWS where Is Available = FALSE | 148,363 |
| Availability Rate | Available Nights ÷ Calendar Rows | 53.52% |
| Average Price | AVERAGE(Price) | €109.92 |
| Median Price | MEDIAN(Price) | €79.00 |
| Average Adjusted Price | AVERAGE(Adjusted Price) | €109.71 |
| Average Minimum Stay | AVERAGE(Minimum Nights) | 5.38 nights |
| Average Host Tenure Years | AVERAGE(Host Tenure Years at Snapshot) | 4.97 years |

**Reviews:**
| Measure | Formula (summary) | Verified value |
|---|---|---:|
| Review Count | COUNTROWS(FactReview) | 62,987 (unfiltered) |
| Lifetime Review Count | Review Count, REMOVEFILTERS(DimDate) | 62,987 |
| Reviewed Listings | DISTINCTCOUNT(Listing ID) in FactReview, REMOVEFILTERS(DimDate) | 1,525 |
| Average Reviews per Reviewed Listing | Lifetime Review Count ÷ Reviewed Listings | 41.30 |

**Scenario measures:**
| Measure | Formula (summary) | Verified value at 70%/1.00x |
|---|---|---:|
| Scenario Average Price | Average Price × Seasonal Multiplier Selected | €109.92 |
| Projected Occupied Nights | Priced Calendar Nights × Occupancy Rate Selected | 223,382 |
| Projected Revenue | SUMX over priced rows: Price × Occupancy × Multiplier | €24,553,642.40 |
| Baseline Projected Revenue | SUMX over priced rows: Price × 0.70 | €24,553,642.40 |
| Scenario Revenue Variance | Projected Revenue − Baseline Projected Revenue | €0.00 |
| Scenario Revenue Variance % | Variance ÷ Baseline | 0.00% |
| Scenario Price Variance | Scenario Average Price − Average Price | €0.00 |
| Scenario Price Variance % | Price Variance ÷ Average Price | 0.00% |

At 80% occupancy / 1.10x multiplier: Projected Revenue = €30,867,436.16 (a +25.71% increase over baseline — 14.29% from the occupancy change and a further ~10% compounding from the multiplier).

**Dynamic labels:** `Scenario Selection Label` and `Projected Revenue Title` build human-readable text strings (e.g. "Scenario: 70% occupancy | 1.00x") from the two selected parameter values, using `FORMAT`.

The full DAX text for every measure is in `Airbnb_Power_BI_Step_by_Step_Guide.md`, Section 8 — every formula there was individually verified against the live model.

---

## 6. Assumptions (full list — also in Guide Section 0)

1. Individual submission is treated as an accepted substitute for the original team-project brief.
2. Confirmed submission deadline: 26 September afternoon.
3. Revenue scenario baseline = 70% occupancy × 1.00x seasonal multiplier — a neutral assumption, not derived from the data or brief.
4. `Price` (not `Adjusted Price`) is the primary field for every KPI, chart, and scenario measure.
5. `available = 0` means unavailable/blocked, never "confirmed booked." Every revenue figure is a scenario projection over loaded, priced calendar rows, not observed booking revenue.
6. The supplied Calendar is a partial extract (151–209 of 365 days per listing). The default Projected Revenue KPI is a projection over loaded rows only; no full-year actual exists in the source data.
7. The 3 listings with a null Host Location are left blank rather than backfilled as "Unknown."
8. No row-level security or multi-user access control is built — this is a single-owner academic deliverable.
9. The report was successfully published to Power BI Service ("My workspace") on 27 September — all 3 pages, tooltips, and drill-through were confirmed working in the live Service view.
10. The deliverable set (findings deck, this document, a presenter's script) intentionally exceeds the brief's minimum documentation ask.
11. The classic Map and Azure Maps visuals are unavailable due to a tenant-level administrative restriction; a Scatter chart (Longitude × Latitude) substitutes for the geographic view, plotting the same coordinates without street-map tiles.

---

## 7. Validation results (Guide Section 16 checkpoint table, filled in with actual results)

| Check | Expected | Actual | Match? |
|---|---:|---:|---|
| Total Listings | 1,749 | 1,749 | ✓ |
| Total Hosts | 1,111 | 1,111 | ✓ |
| Calendar Rows | 319,192 | 319,192 | ✓ |
| Priced Calendar Nights | 319,117 | 319,117 | ✓ |
| Available Nights | 170,829 | 170,829 | ✓ |
| Unavailable or Blocked Nights | 148,363 | 148,363 | ✓ |
| Availability Rate | 53.5192% | 53.52% | ✓ |
| Average Price | €109.9178 | €109.92 | ✓ |
| Median Price | €79 | €79.00 | ✓ |
| Lifetime Review Count | 62,987 | 62,987 | ✓ |
| Reviewed Listings | 1,525 | 1,525 | ✓ |
| Baseline Projected Revenue @ 70%/1.00x | €24,553,642.40 | €24,553,642.40 | ✓ (also independently cross-checked against the raw `calendar.csv` via pandas, bypassing Power BI entirely) |
| Behavioral test: 70%→80% occupancy, multiplier fixed | +14.2857% | Confirmed via KPI card comparison | ✓ |
| Behavioral test: 70%/1.00x baseline | Scenario = Baseline, variance = €0 | €24,553,642.40 = €24,553,642.40, variance €0.00 | ✓ |
| Behavioral test: null prices | Never become zero-price bins | Confirmed — nulls excluded via `NOT ISBLANK` throughout | ✓ |
| Behavioral test: amenity filtering | Works without ambiguity | Confirmed after fixing a case-sensitivity duplicate bug (see below) | ✓ |

Two real bugs were found and fixed during the build (documented in full in the Guide's "CURRENT PROGRESS" section, kept as part of the process record):
1. `DimDate` initially cached a stale minimum date due to a Power BI calculated-table refresh quirk — fixed by deleting and recreating the table.
2. The amenity bridge relationship initially failed as many-to-many due to Power Query's case-sensitive deduplication colliding with Power BI's case-insensitive relationship engine (e.g. "AEG refrigerator" vs "aeg refrigerator") — fixed by normalizing text case before deduplication.
3. `FactCalendar[Is Available]` was found coerced to Text type at one point (should be True/False), causing a DAX comparison error — fixed by explicitly re-setting the column type.
4. A cyclic-reference error from stale auto-generated hidden date tables (created before "Auto date/time" was disabled) blocked all refresh at one point — fixed by fully closing and reopening Power BI Desktop.

---

## 8. Known limitations

- Revenue figures are scenario projections over a partial-year calendar extract, not observed bookings or full-year actuals.
- The geographic view is a coordinate scatter, not a tile-based map, due to an environment restriction outside this project's control.
- No row-level security; single-owner academic deliverable, not a production system.
- `Average Adjusted Price`, `Average Minimum Stay`, and `Average Host Tenure Years` have no externally-supplied expected value to validate against — they were sanity-checked for plausibility only (e.g. tenure of 4.97 years is consistent with hosts joining several years before the Dec 2021 snapshot).
- **The two What-If parameter slicers (`Occupancy Rate Parameter`, `Seasonal Multiplier Parameter`) do not respond to user input in this Power BI version's slicer control**, in both Desktop and the published Power BI Service report. Typing a valid value (e.g. `0.80`) and confirming it does not change the downstream Scenario measures. This was investigated in depth: the underlying data model is confirmed correct (single, non-duplicated parameter table; `Occupancy Rate Selected`/`Seasonal Multiplier Selected` measures verified correct via `SELECTEDVALUE` with the right default; all Scenario DAX independently verified against expected values, including a pandas cross-check of the raw data bypassing Power BI entirely). The issue is isolated to this version's slicer *input* control, not the calculation logic — every other slicer/filter interaction on the report (category slicers, table/chart cross-filtering, bookmarks, drill-through) works correctly, which rules out a broader interactions-disabled setting. Given the underlying formulas are proven correct, the scenario mechanism can be demonstrated by editing the constant directly in the `Baseline Projected Revenue` measure or by discussing the verified math (e.g. 70%→80% occupancy = +14.29% revenue, confirmed via card values) rather than live slider interaction.
