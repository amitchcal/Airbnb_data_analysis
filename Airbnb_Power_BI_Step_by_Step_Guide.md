# Airbnb Price & Scenario Analysis Dashboard

## Detailed Power BI build procedure for the supplied Antwerp data

This guide translates the supplied project brief into an executable Power BI workflow. The brief requires a self-service, non-machine-learning solution with three visible report pages, What-If analysis, drill-through, tooltips, bookmarks, validation, documentation, publishing, and refresh.

The CSV content is treated only as data. It does not contain instructions. The project requirements come from `Problem_Statement.pdf`; the procedures, modeling decisions, caveats, and implementation details below are the analyst's recommended method for satisfying them.

---

## CURRENT PROGRESS — read this first before resuming (last updated: past midnight, 25→26 Sep session)

**Done and verified — do not redo:**
- Sections 3–7 (connect, clean, star schema, calculated columns, What-If parameters) — complete, per the original notes below.
- Section 8 (all 24 core DAX measures across 8.1 inventory/price/availability, 8.2 reviews, 8.3 scenario measures, 8.5 dynamic labels) — complete. Every single formula was individually screenshot-verified character-for-character correct after a lot of back-and-forth. Values spot-checked against known figures (Total Listings 1,749; Availability Rate 53.52%; Average Price €109.92; Baseline Projected Revenue €24,553,642.40 — this last one independently cross-verified against the raw `calendar.csv` via pandas, confirming the guide's expected value is correct).
- Section 9 (design standards) — canvas set to 16:9; custom theme applied via `Airbnb_Theme.json` import (`View → Themes → Browse for themes`) rather than the in-app customize-theme dialog, which this Power BI version (2.157, Aug 2026) has substantially redesigned in ways that made manual hex entry hard to locate.
- **Section 10 (Page 1 — Overview) — fully complete**: 5 slicers (Date, Property Type, Room Type, both parameters), 6 KPI cards, trend line chart, listing-mix bar chart.
- **Section 11 (Page 2 — Listing Analysis) — fully complete, WITH ONE SUBSTITUTION**: the classic Map visual is blocked in this environment by a **tenant-level admin restriction** (confirmed — Azure Maps gave an explicit "contact your admin to turn on the tenant switch" message; the classic Map visual's local "enable" checkbox was already checked and a restart didn't help, so it's the same tenant-level block manifesting differently). **Substituted a Scatter chart** (X-axis=Latitude, Y-axis=Longitude, Legend=Room Type, Size=Projected Revenue, same 7 tooltip fields) which correctly traces Antwerp's geographic shape without needing map tiles. Document this substitution explicitly in the technical write-up/assumptions. Histogram and Top-listings table (Top 20 by Projected Revenue, correctly sorted) both done and verified.
- **Section 12 (Page 3 — Scenario Insights) — fully complete**: both parameter slicers copied over, 5 KPI cards, price comparison line chart, projected revenue trend line chart, listing-level scenario table (Top by Projected Revenue). All values verified correct after clearing a couple of accidental cross-filter selections (clicking a chart bar/table row filters the whole page — always check the KPI cards still show full-dataset values, not a filtered subset, before trusting a screenshot).

**This means all 3 core visible pages (the Problem Statement's headline "three-page report" requirement) are done.**

**Bugs hit and fixed this session — don't re-diagnose from scratch if any resurface:**
1. `DimDate` stale-cache and `DimAmenity`/`BridgeListingAmenity` case-collision bugs — see original notes retained below.
2. `FactCalendar[Is Available]` got auto-coerced to **Text** type at some point (should be True/False) — DAX comparison errors resulted ("cannot compare Text with True/False"). Fix: in Power Query, explicitly set the column's type to True/False again (don't rely on the custom-column formula alone to hold the type).
3. Newly-created measures sometimes silently overwrote an existing one instead of creating a new one, if you clicked into an existing measure's formula bar instead of clicking `_Measures` table name → New measure fresh each time. Always click the **table name**, not an existing measure, before "New measure."
4. A **cyclic reference** error ("FactReview... waiting for LocalDateTable/DateTableTemplate...") blocked all refresh at one point. Cause: hidden auto-generated date tables that existed from before "Auto date/time" was disabled (disabling the setting only stops *new* ones; it doesn't delete already-created ones). Fix: fully close and reopen Power BI Desktop (a fresh model load cleans these up) — don't just try in-session Refresh, it stays blocked.
5. Dragging a single measure onto blank canvas often defaults to a **KPI** visual (needs Trend axis/Target to render) instead of a plain **Card** — fix is clicking the `123` icon in the "Suggest a visual" panel's suggestion row instead of the first (colored) icon.
6. Building charts with a category field + measure(s): watch for fields landing in the wrong axis well (e.g. a category field defaulting into the value/X-axis slot, auto-aggregating as "Count of..." instead of using the intended measure). Always check the Build pane's field wells explicitly rather than trusting the auto-suggestion blindly.

**⚠ DEADLINE IS TODAY (26 Sep) AFTERNOON.** This is the last work session — not one of several remaining days. Everything below is prioritized with that in mind.

**Documentation/presentation materials are already done — overnight, while you slept:**
- **`Airbnb_Presentation_Script.md`** — fully filled in with real, verified numbers (no `[FILL IN]` tags remain). Do one read-through before presenting.
- **`Airbnb_User_Guide_and_Technical_Documentation.md`** — combined user guide + technical doc, drafted using the actual verified DAX values, model diagram, and assumptions. Ready to submit as-is, or convert to PDF if the rubric wants a PDF specifically.
- **`Airbnb_Findings_Presentation.pptx`** — the actual 10-slide deck has been built (not just outlined), using `pptxgenjs`, styled to the same color palette as the dashboard (`#FF385C`/`#6C5CE7`/`#00875A`/`#D97706`). It passed structural validation and a content check (no placeholder text). Open it in PowerPoint first thing this morning and skim it — it hasn't had a full visual proofread yet (font sizing, spacing) since the rendering tool needed for that wasn't available overnight.

This means **all of today's remaining time can go toward the Power BI work**, which is the part that actually needs live Desktop interaction. Realistic priority order, given the compressed timeline:
1. Section 13 (drill-through) — explicit Problem Statement requirement, ~30-45 min.
2. Section 15 (bookmarks: Reset + Typical/All-price) — explicit requirement, ~20 min.
3. Section 14 (tooltip page) — explicit requirement, ~15-20 min.
4. Quick header pass (title, page navigator, reset button, refresh timestamp) — ~20-30 min.
5. Final packaging per Section 25.3's checklist and the GitHub check-in steps — copy the `.pbix` and all 3 documentation files above into `Submissions`, commit, and submit.

If something has to be cut, cut in this order (least costly first): header/navigator polish, then the tooltip page, then the bookmarks — try hardest to keep the drill-through, since "detailed listing drill-through" is named explicitly in the Problem Statement.

---

## 0. Scope, ownership, and assumptions (individual submission)

This capstone was originally scoped as a 5-learner team project (see `Airbnb_PowerBI_Project_Plan.xlsx`, roles L1–L5). Due to team time-coordination constraints, it is being **executed and submitted individually**. Section 23 remaps the team plan to a single owner; Section 24 gives a solo timeline; Section 25 defines the presentation/documentation deliverable. `Airbnb_Solo_Submission_Checklist.xlsx` is the working tracker for the actual build; the original `Airbnb_PowerBI_Project_Plan.xlsx` is retained and submitted as-is as evidence of the originally intended team plan.

**The following assumptions are part of the final deliverable** — copy this list (or a summary of it) into the Technical Documentation / user guide (Section 20) and into the closing slide of the presentation deck (Section 25), so a grader can see exactly what was assumed versus what was specified.

1. **Individual submission is treated as an accepted substitute for the team deliverable.** The team project-plan Excel is submitted unmodified as a reference/checklist artifact per the brief; if the course requires an explicit "individual contribution" statement or academic-integrity declaration for solo submission of a team assignment, that is outside this guide's scope and should be confirmed with the instructor separately.
2. **Confirmed deadline: 20 September, with an instructor-granted extension to 27 September** (some groups requested and received this extension). Separately, the learner is on personal leave from **13 September 11:00 IST to 22 September**, with no work possible in that window. The professor has agreed that submitting a work-in-progress (WIP) version on the morning of 13 September, before departure, satisfies the interim checkpoint and avoids being marked a defaulter — the final, polished version follows during 23–26 September, inside the extended window. Section 24 reflects this real schedule rather than a generic multi-week estimate.
3. **Revenue scenario baseline = 70% occupancy × 1.00x seasonal multiplier** (Section 8.3). This is not stated in the Problem Statement or Data Description; it was chosen as a neutral midpoint. If the instructor specifies a different baseline, change the two constants in `Baseline Projected Revenue` once and note the change in the documentation.
4. **`Price` (not `Adjusted Price`) is the primary field** for every KPI, chart, and scenario measure (Section 4.3), because the Problem Statement does not distinguish the two. `Adjusted Price` is retained in the model for reference/drill-through only.
5. **`available = 0` means unavailable/blocked, never "confirmed booked."** No booking or actual-revenue metric is derived from it. Every revenue figure in the report is explicitly labeled as a scenario projection over the *loaded, priced* calendar rows — not observed booking revenue (Section 2, Section 20).
6. **The supplied Calendar is a partial extract** (151–209 of 365 days per listing; no listing has a full year). The default Projected Revenue KPI is therefore a projection over loaded rows only. Any full-year/annualized figure (Section 8.4) is treated as a clearly labeled estimate, never the headline KPI, because it assumes loaded days are representative of the unloaded ones.
7. **The 3 listings with a null `Host Location`** are left blank rather than back-filled with "Unknown" (Section 4.4) — chosen to avoid manufacturing a false category. Log this choice explicitly in the technical documentation so it is a stated decision, not a silent gap.
8. **No row-level security or multi-user access control is built.** This is a single-owner academic deliverable, not a production multi-tenant deployment (Section 5.3, Section 19 are followed only up to what a single learner's Power BI account requires).
9. **Publishing to Power BI Service (Section 19) assumes a Power BI account/license is available.** If the grading rubric accepts a Desktop-only `.pbix` plus refresh/publish screenshots as evidence (rather than a live, gateway-refreshed Service report), publishing is best-effort rather than a hard blocker — confirm against the actual rubric if one exists beyond `Problem_Statement.pdf`.
10. **The deliverable set intentionally goes beyond the Problem Statement's minimum ask.** Alongside the `.pbix`, a findings presentation deck and a written user guide/technical document are produced to a materially higher documentation and storytelling standard than the team baseline plan describes — using the "Rainfall vs Agriculture" submission as the internal quality benchmark (Section 25).

---

## 1. What the finished solution should contain

### Three visible report pages

1. **Overview**
   - KPI cards: average price, availability rate, total listings, projected revenue.
   - Slicers: calendar date, property type, room type, occupancy rate, seasonal multiplier.
   - Supporting price/revenue trend and listing-mix visuals.

2. **Listing Analysis**
   - Map using latitude and longitude.
   - Price distribution histogram.
   - Top-listings table.

3. **Scenario Insights**
   - Base-price versus scenario-price comparison.
   - Projected-revenue trend.
   - Scenario variance KPIs and listing-level comparison.

### Technical support pages

- **Listing Detail**: a hidden drill-through page filtered by Listing ID. It is not part of the three-page navigation, so the report still has three visible pages.
- **Listing Tooltip**: a hidden report-page tooltip.

### Recommended output file

Save the Desktop file as `Airbnb_Antwerp_Pricing_Dashboard.pbix`.

---

## 2. Facts established from the supplied files

Use these values later as reconciliation checkpoints.

| Table | Grain | Rows | Key/date observations |
|---|---|---:|---|
| Listings | One row per listing | 1,749 | `listing_id` is unique; 1,111 distinct hosts |
| Hosts | One row per host | 1,111 | `host_id` is unique |
| Calendar | One row per listing-date | 319,192 | No duplicate listing-date pairs; dates run 26 Dec 2021 to 25 Dec 2022 |
| Reviews | One row per review | 62,987 | `review_id` is unique; dates run 19 Sep 2011 to 26 Dec 2021 |

Additional checks:

- All listing IDs in Calendar and Reviews match Listings.
- All host IDs in Listings match Hosts.
- Calendar contains 170,829 available rows and 148,363 unavailable rows.
- Overall availability rate is **53.5192%**.
- Price is populated for 319,117 calendar rows; 75 prices are null, and all 75 occur on unavailable rows.
- Average valid nightly price is **EUR 109.9178**; median is EUR 79; maximum is EUR 5,800.
- The 99th percentile of price is EUR 650, so the histogram needs explicit outlier handling.
- Each listing has only 151-209 calendar dates in this extract. No listing has a complete 365-day calendar.
- 224 listings have no reviews.

### Two essential interpretation rules

1. `available = 0` means unavailable or blocked. It does **not** prove that the night was booked. Do not label unavailable nights as bookings and do not calculate actual revenue from them.
2. The supplied Calendar is partial at the individual-listing level. The default projected-revenue measure therefore describes only the loaded, priced calendar rows in the current filter context. Any annualized result must be labeled as an estimate based on an extrapolation assumption.

---

## 3. Create the project and import the sources

1. Open Power BI Desktop.
2. Select **File > New**.
3. Save the file immediately as `Airbnb_Antwerp_Pricing_Dashboard.pbix`.
4. Select **Home > Get data > Text/CSV**.
5. Import each file separately because their schemas differ:
   - `listings.csv`
   - `calendar.csv`
   - `hosts.csv`
   - `reviews.csv`
6. For every import, select **Transform Data**, not Load.
7. In Power Query, rename the original source queries:
   - `Stg_Listings`
   - `Stg_Calendar`
   - `Stg_Hosts`
   - `Stg_Reviews`
8. Right-click each staging query and clear **Enable load**. Keep them as reusable source/staging queries.
9. Right-click each staging query and choose **Reference** to create the model tables:
   - `Stg_Listings` -> `DimListing`
   - `Stg_Calendar` -> `FactCalendar`
   - `Stg_Hosts` -> `DimHost`
   - `Stg_Reviews` -> `FactReview`

Using references avoids reading the same file independently for every derived table.

---

## 4. Clean and shape the data in Power Query

Do not replace null prices with zero. Zero would be interpreted as a real free night and would lower the average price and projected revenue incorrectly.

### 4.1 Clean `DimListing`

1. Confirm these data types:

| Column | Type |
|---|---|
| listing_id | Whole number |
| listing_url | Text |
| name | Text |
| description | Text |
| latitude | Decimal number |
| longitude | Decimal number |
| property_type | Text |
| room_type | Text |
| accomodates | Whole number |
| bathrooms_text | Text |
| bedrooms | Decimal number |
| beds | Decimal number |
| amenities | Text |
| host_id | Whole number |

2. Rename the misspelled `accomodates` column to `Accommodates`.
3. Rename other columns to readable names, such as `listing_id` -> `Listing ID`, `host_id` -> `Host ID`, and `room_type` -> `Room Type`.
4. Select Listing Name, Property Type, Room Type, and Bathrooms Text. Apply **Transform > Format > Trim** and then **Clean**.
5. Standardize Property Type and Room Type consistently. Using Proper Case is acceptable if applied to every row.
6. Add a numeric bathroom field with **Add Column > Custom Column**:

```powerquery
let
    t = Text.Lower([Bathrooms Text]),
    firstToken = try Text.BeforeDelimiter(t, " ") otherwise t,
    parsed = try Number.FromText(firstToken) otherwise null
in
    if parsed <> null then parsed
    else if Text.Contains(t, "half") then 0.5
    else null
```

Name the column `Bathrooms` and set it to Decimal Number.

7. Keep the Description column only if you plan to show it on the drill-through page. Otherwise remove it to reduce model size. Long text should not be placed in ordinary visuals.
8. Do not fill null Bedrooms or Beds with zero; “unknown” is different from none.
9. Remove `Amenities` from `DimListing` after creating the bridge query in the next section.
10. Verify that Listing ID remains unique: select Listing ID and use **Keep Rows > Keep Duplicates** temporarily. The result must be empty; remove this diagnostic step afterward.

### 4.2 Normalize Amenities

The amenities field is a JSON-like list. Create one row per listing-amenity instead of leaving it as one long text string.

1. Right-click `Stg_Listings` and select **Reference**.
2. Name the query `BridgeListingAmenity`.
3. Keep only `listing_id` and `amenities`.
4. Open **Home > Advanced Editor** and use this pattern, adjusting names if your staging query was renamed differently:

```powerquery
let
    Source = Stg_Listings,
    KeepColumns = Table.SelectColumns(Source, {"listing_id", "amenities"}),
    ParseJson = Table.TransformColumns(
        KeepColumns,
        {{"amenities", each try Json.Document(Text.ToBinary(_)) otherwise {}, type list}}
    ),
    ExpandList = Table.ExpandListColumn(ParseJson, "amenities"),
    RenameColumns = Table.RenameColumns(
        ExpandList,
        {{"listing_id", "Listing ID"}, {"amenities", "Amenity"}}
    ),
    CleanText = Table.TransformColumns(
        RenameColumns,
        {{"Amenity", each Text.Trim(Text.Clean(Text.From(_))), type text}}
    ),
    RemoveBlank = Table.SelectRows(CleanText, each [Amenity] <> null and [Amenity] <> ""),
    RemoveDuplicates = Table.Distinct(RemoveBlank, {"Listing ID", "Amenity"})
in
    RemoveDuplicates
```

5. Right-click `BridgeListingAmenity` and select **Reference**.
6. Name the new query `DimAmenity`.
7. Keep only Amenity and remove duplicates.
8. If amenities are not used in any slicer or visual, keep the normalized tables documented but hide their fields from report view to reduce clutter.

### 4.3 Clean `FactCalendar`

1. Rename `calender_id` to `Calendar ID` and `listing_id` to `Listing ID`.
2. Set types:

| Column | Type |
|---|---|
| Calendar ID | Whole number |
| Listing ID | Whole number |
| date | Date |
| available | Whole number initially |
| price | Fixed decimal number |
| adjusted_price | Fixed decimal number |
| minimum_nights | Whole number |
| maximum_nights | Whole number |

3. Rename the remaining columns to `Date`, `Available Flag`, `Price`, `Adjusted Price`, `Minimum Nights`, and `Maximum Nights`.
4. Add a logical column named `Is Available`:

```powerquery
[Available Flag] = 1
```

5. Set `Is Available` to True/False, then remove `Available Flag` if it is no longer needed.
6. Do not remove the 75 null-price rows. Availability metrics can still use them, while price/revenue measures will explicitly count only priced rows.
7. Validate the grain:
   - Select Listing ID and Date.
   - Use **Keep Rows > Keep Duplicates**.
   - The result should contain zero rows.
   - Delete the diagnostic step after validation.
8. Keep both Price and Adjusted Price. Use Price as the primary baseline unless the course instructor specifies that Adjusted Price is authoritative.

### 4.4 Clean `DimHost`

1. Rename columns to `Host ID`, `Host Name`, `Host Since`, `Host Location`, and `Host About`.
2. Set Host ID to Whole Number and Host Since to Date.
3. Apply Trim and Clean to Host Name and Host Location.
4. Leave the three missing Host Locations as null or replace them with the text `Unknown`; document whichever decision you use.
5. Leave Host About nulls unchanged. The field is descriptive and 621 rows are null.
6. Confirm Host ID is unique.

### 4.5 Clean `FactReview`

1. Rename columns to `Review ID`, `Listing ID`, `Review Date`, `Reviewer ID`, `Reviewer Name`, and `Comments`.
2. Set Review ID, Listing ID, and Reviewer ID to Whole Number.
3. Set Review Date to Date.
4. Confirm Review ID is unique.
5. Do not deduplicate by Listing ID plus Review Date. There are 96 repeated listing-date combinations, but they can represent different legitimate reviews; Review ID is the correct key.
6. If no text analysis is required, remove Reviewer Name and Comments from the loaded model. This improves compression and avoids exposing personal names unnecessarily.

### 4.6 Apply the queries

1. Confirm only these queries have **Enable load** selected:
   - DimListing
   - DimHost
   - FactCalendar
   - FactReview
   - BridgeListingAmenity
   - DimAmenity
2. Select **Home > Close & Apply**.
3. Save the PBIX.

---

## 5. Build the star schema

Microsoft recommends a star-schema pattern in which dimension tables filter/group and fact tables summarize. Use single-direction, one-to-many relationships wherever possible.

### 5.1 Create a proper date table

Select **Modeling > New table** and enter:

```DAX
DimDate =
VAR AllFactDates =
    UNION (
        SELECTCOLUMNS ( FactCalendar, "DateValue", FactCalendar[Date] ),
        SELECTCOLUMNS ( FactReview, "DateValue", FactReview[Review Date] )
    )
RETURN
    ADDCOLUMNS (
        CALENDAR (
            MINX ( AllFactDates, [DateValue] ),
            MAXX ( AllFactDates, [DateValue] )
        ),
        "Year", YEAR ( [Date] ),
        "Quarter", "Q" & FORMAT ( [Date], "Q" ),
        "Month No", MONTH ( [Date] ),
        "Month", FORMAT ( [Date], "MMM" ),
        "Year Month", FORMAT ( [Date], "yyyy-MM" ),
        "Year Month Sort", YEAR ( [Date] ) * 100 + MONTH ( [Date] ),
        "Day of Week No", WEEKDAY ( [Date], 2 ),
        "Day of Week", FORMAT ( [Date], "ddd" )
    )
```

Then:

1. Select DimDate[Month] and choose **Column tools > Sort by column > Month No**.
2. Select DimDate[Year Month] and sort by Year Month Sort.
3. Select DimDate[Day of Week] and sort by Day of Week No.
4. Right-click DimDate and select **Mark as date table > Mark as date table**, then choose DimDate[Date].
5. Turn off automatic date/time for this file under **File > Options and settings > Options > Current File > Data Load > Auto date/time**.

### 5.2 Create relationships

In Model view, create the following:

| From (one side) | To (many side) | Cardinality | Filter direction | Active |
|---|---|---|---|---|
| DimHost[Host ID] | DimListing[Host ID] | 1:* | Single | Yes |
| DimListing[Listing ID] | FactCalendar[Listing ID] | 1:* | Single | Yes |
| DimListing[Listing ID] | FactReview[Listing ID] | 1:* | Single | Yes |
| DimDate[Date] | FactCalendar[Date] | 1:* | Single | Yes |
| DimDate[Date] | FactReview[Review Date] | 1:* | Single | Yes |
| DimAmenity[Amenity] | BridgeListingAmenity[Amenity] | 1:* | Single | Yes |
| DimListing[Listing ID] | BridgeListingAmenity[Listing ID] | 1:* | Both | Yes |

Use bidirectional filtering only on the listing-amenity bridge so that an Amenity slicer can filter listings and their facts. Do not turn every relationship to Both; that creates ambiguity and hurts performance.

### 5.3 Model layout and housekeeping

1. Place dimensions across the top and facts below them in Model view.
2. Create a blank table named `_Measures` using **Home > Enter data**, add one dummy column/row, and hide the dummy column. Store all measures in this table.
3. Hide technical keys from report view except Listing ID, which is needed for drill-through.
4. Set DimListing[Listing URL] data category to **Web URL**.
5. Set Latitude data category to **Latitude** and Longitude to **Longitude**; set both to **Do not summarize**.
6. Add table and measure descriptions in Model view. This is part of project documentation.

---

## 6. Add supporting calculated columns

### 6.1 Host tenure at the dataset snapshot

The calendar begins on 26 Dec 2021, which is the appropriate supplied-data snapshot anchor. Create this calculated column in DimListing so it responds naturally to listing filters:

```DAX
Host Tenure Years at Snapshot =
DIVIDE (
    DATEDIFF (
        RELATED ( DimHost[Host Since] ),
        DATE ( 2021, 12, 26 ),
        DAY
    ),
    365.25
)
```

Format it as Decimal Number with one decimal place. Label it as tenure “at snapshot,” not current age.

### 6.2 Price bins for the histogram

Create in FactCalendar:

```DAX
Price Bin Start =
VAR BinSize = 25
RETURN
    IF (
        NOT ISBLANK ( FactCalendar[Price] ),
        INT ( FactCalendar[Price] / BinSize ) * BinSize
    )
```

Use the numeric Price Bin Start on the X-axis. A EUR 25 bin width provides a readable initial distribution; adjust only after inspecting the visual.

---

## 7. Create the What-If parameters

Power BI currently creates numeric-range parameters from **Modeling > New parameter > Numeric range** and can add the slicer automatically.

### 7.1 Occupancy Rate parameter

1. Open **Modeling > New parameter > Numeric range**.
2. Enter:
   - Name: `Occupancy Rate Parameter`
   - Data type: Decimal number
   - Minimum: 0.10
   - Maximum: 1.00
   - Increment: 0.05
   - Default: 0.70
   - Add slicer to this page: selected
3. Format the parameter field and generated value measure as Percentage with zero decimals.
4. Rename the generated measure, if necessary, to `Occupancy Rate Selected`.

If you create it manually, use:

```DAX
Occupancy Rate Parameter = GENERATESERIES ( 0.10, 1.00, 0.05 )

Occupancy Rate Selected =
SELECTEDVALUE ( 'Occupancy Rate Parameter'[Value], 0.70 )
```

### 7.2 Seasonal Multiplier parameter

1. Create a second Numeric range parameter:
   - Name: `Seasonal Multiplier Parameter`
   - Data type: Decimal number
   - Minimum: 0.50
   - Maximum: 1.50
   - Increment: 0.05
   - Default: 1.00
   - Add slicer to this page: selected
2. Format it as 0.00x using a custom format string if desired.
3. Rename the generated measure to `Seasonal Multiplier Selected`.

Manual alternative:

```DAX
Seasonal Multiplier Parameter = GENERATESERIES ( 0.50, 1.50, 0.05 )

Seasonal Multiplier Selected =
SELECTEDVALUE ( 'Seasonal Multiplier Parameter'[Value], 1.00 )
```

These parameter tables must remain disconnected from the rest of the model.

---

## 8. Create the core DAX measures

Create these measures in `_Measures`.

### 8.1 Inventory, price, and availability

```DAX
Total Listings =
DISTINCTCOUNT ( DimListing[Listing ID] )

Total Hosts =
DISTINCTCOUNT ( DimHost[Host ID] )

Calendar Rows =
COUNTROWS ( FactCalendar )

Priced Calendar Nights =
COUNT ( FactCalendar[Price] )

Available Nights =
CALCULATE (
    COUNTROWS ( FactCalendar ),
    FactCalendar[Is Available] = TRUE ()
)

Unavailable or Blocked Nights =
CALCULATE (
    COUNTROWS ( FactCalendar ),
    FactCalendar[Is Available] = FALSE ()
)

Availability Rate =
DIVIDE ( [Available Nights], [Calendar Rows] )

Average Price =
AVERAGE ( FactCalendar[Price] )

Median Price =
MEDIAN ( FactCalendar[Price] )

Average Adjusted Price =
AVERAGE ( FactCalendar[Adjusted Price] )

Average Minimum Stay =
AVERAGE ( FactCalendar[Minimum Nights] )

Average Host Tenure Years =
AVERAGE ( DimListing[Host Tenure Years at Snapshot] )
```

Format price measures as Currency with EUR, availability as Percentage, and counts as Whole Number.

### 8.2 Reviews

```DAX
Review Count =
COUNTROWS ( FactReview )

Lifetime Review Count =
CALCULATE (
    [Review Count],
    REMOVEFILTERS ( DimDate )
)

Reviewed Listings =
CALCULATE (
    DISTINCTCOUNT ( FactReview[Listing ID] ),
    REMOVEFILTERS ( DimDate )
)

Average Reviews per Reviewed Listing =
DIVIDE ( [Lifetime Review Count], [Reviewed Listings] )
```

Use Lifetime Review Count in current/future calendar views. A 2022 calendar slicer would otherwise remove all historical reviews from a date-filtered Review Count.

### 8.3 Scenario calculations

```DAX
Scenario Average Price =
[Average Price] * [Seasonal Multiplier Selected]

Projected Occupied Nights =
[Priced Calendar Nights] * [Occupancy Rate Selected]

Projected Revenue =
VAR Occupancy = [Occupancy Rate Selected]
VAR Multiplier = [Seasonal Multiplier Selected]
RETURN
    SUMX (
        FILTER ( FactCalendar, NOT ISBLANK ( FactCalendar[Price] ) ),
        FactCalendar[Price] * Occupancy * Multiplier
    )

Baseline Projected Revenue =
SUMX (
    FILTER ( FactCalendar, NOT ISBLANK ( FactCalendar[Price] ) ),
    FactCalendar[Price] * 0.70
)

Scenario Revenue Variance =
[Projected Revenue] - [Baseline Projected Revenue]

Scenario Revenue Variance % =
DIVIDE ( [Scenario Revenue Variance], [Baseline Projected Revenue] )

Scenario Price Variance =
[Scenario Average Price] - [Average Price]

Scenario Price Variance % =
DIVIDE ( [Scenario Price Variance], [Average Price] )
```

The baseline is explicitly 70% occupancy and 1.00x price. If the course expects a different baseline, change the constant once and document it.

### 8.4 Optional annualized estimate

Do not use this as the default KPI because every listing has only 151-209 loaded calendar days. If the evaluator requests a full-year estimate, create:

```DAX
Annualized Projected Revenue Estimate =
SUMX (
    VALUES ( DimListing[Listing ID] ),
    VAR LoadedNights = CALCULATE ( [Priced Calendar Nights] )
    VAR LoadedRevenue = CALCULATE ( [Projected Revenue] )
    RETURN
        DIVIDE ( LoadedRevenue, LoadedNights ) * 365
)
```

Display a visible note: “Annualized from the average loaded priced days per listing; assumes loaded dates are representative.”

### 8.5 Dynamic labels

```DAX
Scenario Selection Label =
"Scenario: "
    & FORMAT ( [Occupancy Rate Selected], "0%" )
    & " occupancy | "
    & FORMAT ( [Seasonal Multiplier Selected], "0.00x" )

Projected Revenue Title =
"Projected revenue at "
    & FORMAT ( [Occupancy Rate Selected], "0%" )
    & " occupancy and "
    & FORMAT ( [Seasonal Multiplier Selected], "0.00x" )
```

Apply the title measure with **Format visual > Title > fx > Field value**.

---

## 9. Design standards before building visuals

1. Use a 16:9 canvas, approximately 1280 x 720.
2. Use a light neutral background, dark text, one Airbnb-style accent color, one scenario color, and one warning color.
3. Suggested palette:
   - Primary: `#FF385C`
   - Scenario: `#6C5CE7`
   - Positive: `#00875A`
   - Warning: `#D97706`
   - Dark text: `#222222`
   - Background: `#F7F7F7`
4. Keep a consistent header on all three visible pages:
   - Report title on the left.
   - Page navigator in the center or left below title.
   - Last refresh timestamp and Reset Filters button on the right.
5. Use **Insert > Buttons > Navigator > Page navigator** for the three visible pages.
6. Keep slicers in the same position across pages.
7. Use dropdown slicers for Property Type and Room Type, a between-date slicer for Calendar Date, and slider-style slicers for the two parameters.
8. Use **View > Sync slicers** to synchronize the global slicers across the three visible pages.
9. Do not synchronize Listing ID to all pages; keep it for drill-through or detailed analysis.
10. Add alt text to every meaningful visual and maintain a logical tab order in the Selection pane.

### 9.1 Dashboard quality checklist (instructor guidance)

The following comes from the course's recorded expectation-setting sessions (chart-critique and dashboard-review segments) and functions as a de facto grading checklist even where no formal rubric was stated. Apply it to every page before considering it done:

- **Match chart type to data shape**: trend over time → line chart; category vs. a number → bar chart (or pie only for a true 100% breakdown); category with sub-groups → stacked bar chart; numeric-vs-numeric relationship → scatter plot. A chart type that doesn't match the question being asked was explicitly flagged as a mistake.
- **Limit the color palette.** Too many colors on one page was called out as distracting — stick to the Section 9 palette above; don't add extra colors per visual.
- **Visually emphasize good/bad, not just report numbers.** Use the Positive/Warning colors purposefully (e.g. positive vs. negative scenario variance) rather than leaving every KPI the same neutral color.
- **No generic titles.** "Sales Dashboard" (or here, "Airbnb Dashboard") was explicitly called a mistake — every page and chart title should state what it's for (e.g. "Average nightly price: base vs scenario", already used in Section 10.4), not a category label.
- **Scale large numbers.** Use Power BI's Display Units formatting (K/M) instead of showing raw multi-digit numbers on KPI cards and axes.
- **No ALL CAPS.** Titles and labels should be normal sentence/title case, not all-uppercase.
- **Never show the same metric twice on one page** — duplicated KPIs (e.g. Average Price appearing on two different visuals on the same page) were explicitly flagged in a critique exercise.
- **Every visual needs a visible data label** — a chart with no labels was flagged as incomplete.
- **Never expose raw column names to the viewer.** This is already covered by Section 5.3's field renaming, but re-confirm nothing like `price_bin_start` or `is_available` leaks into a visual title or tooltip — use the human-readable names (Price Bin Start already reads fine; double-check tooltips too).
- **Keep consistent alignment and clear separation between sections** of a page (cards row, slicer row, chart area) rather than a cluttered single block.

---

## 10. Build Page 1 - Overview

### 10.1 Layout

- Top strip: title, page navigator, reset button.
- Second strip: global slicers.
- Third strip: KPI cards.
- Bottom section: price/revenue trend and listing mix.

### 10.2 Slicers

Add:

1. DimDate[Date] as a Between slicer.
2. DimListing[Property Type] as Dropdown.
3. DimListing[Room Type] as Dropdown.
4. Occupancy Rate Parameter value as Slider.
5. Seasonal Multiplier Parameter value as Slider.

Set the two parameter slicers to Single Select where applicable.

### 10.3 KPI cards

Create cards for:

- Average Price
- Availability Rate
- Total Listings
- Projected Revenue

Optional supporting cards:

- Lifetime Review Count
- Average Host Tenure Years

Place a small information icon beside Projected Revenue with tooltip text explaining that it is a scenario estimate over loaded priced rows, not observed booking revenue.

### 10.4 Trend visual

Use a Line chart:

- X-axis: DimDate[Year Month]
- Y-axis: Average Price and Scenario Average Price
- Title: `Average nightly price: base vs scenario`

The chart responds to the seasonal multiplier and demonstrates the scenario effect over time.

### 10.5 Listing mix

Use a horizontal bar chart:

- Y-axis: DimListing[Room Type]
- X-axis: Total Listings
- Sort descending.

If space allows, add a second bar chart by Property Type, limited to the top 10 categories.

---

## 11. Build Page 2 - Listing Analysis

### 11.1 Map

Use Azure Maps or the available Power BI map visual:

- Latitude: DimListing[Latitude]
- Longitude: DimListing[Longitude]
- Legend: DimListing[Room Type]
- Size: Projected Revenue or Average Price
- Tooltips: Listing Name, Property Type, Room Type, Average Price, Availability Rate, Lifetime Review Count, Projected Revenue

Set Latitude and Longitude to Do not summarize. Do not rely on geocoding listing names.

### 11.2 Price distribution histogram

Use a column chart:

- X-axis: FactCalendar[Price Bin Start]
- Y-axis: Calendar Rows
- Visual-level filter: Price is not blank.

Because the maximum is EUR 5,800 and the 99th percentile is EUR 650, create two bookmarks:

- `Typical Price Range`: visual filter Price <= 650.
- `All Prices`: no upper-price filter.

Use a bookmark navigator to toggle the views. Never silently remove outliers; the user should be able to display them.

### 11.3 Top listings table

Add a Table visual with:

- Listing ID
- Listing Name
- Property Type
- Room Type
- Accommodates
- Average Price
- Availability Rate
- Lifetime Review Count
- Projected Revenue

Sort by Projected Revenue descending. Apply a Top N visual filter, such as Top 20 by Projected Revenue. Add conditional formatting to Projected Revenue and Availability Rate.

### 11.4 Interactions

Use **Format > Edit interactions**:

- Selecting a room type should filter the map, histogram, and table.
- Selecting a map point should filter the table.
- Avoid interactions that turn the histogram into an unreadable single bar; set them to None where necessary.

---

## 12. Build Page 3 - Scenario Insights

### 12.1 Scenario controls and KPIs

Keep the Occupancy and Seasonal Multiplier slicers at the top. Add:

- Scenario Average Price
- Projected Occupied Nights
- Projected Revenue
- Scenario Revenue Variance
- Scenario Revenue Variance %

Use green for positive variance and orange/red for negative variance, but always show the signed number so meaning does not depend on color.

### 12.2 Price comparison chart

Use a clustered column chart or line chart:

- X-axis: DimDate[Year Month]
- Values: Average Price and Scenario Average Price

Use Base as dark gray and Scenario as purple. Keep both measures on the same currency scale.

### 12.3 Projected revenue trend

Use a line chart:

- X-axis: DimDate[Year Month]
- Y-axis: Projected Revenue and Baseline Projected Revenue
- Dynamic title: Projected Revenue Title

### 12.4 Listing-level scenario table

Add:

- Listing ID
- Listing Name
- Average Price
- Scenario Average Price
- Projected Occupied Nights
- Baseline Projected Revenue
- Projected Revenue
- Scenario Revenue Variance %

Sort by Projected Revenue. This table is the main source for drill-through.

---

## 13. Add the hidden Listing Detail drill-through page

1. Add a new page and rename it `Listing Detail`.
2. In the Drill-through field well, add DimListing[Listing ID].
3. Keep all filters selected only if that is desired; normally retain the current date, property, room, and scenario context.
4. Power BI adds a Back button automatically; format it clearly.
5. Add cards for Listing Name, Property Type, Room Type, Accommodates, Bedrooms, Beds, Average Price, Availability Rate, Lifetime Review Count, and Projected Revenue.
6. Add a line chart of Base Price and Scenario Price by Date.
7. Add a line chart of Projected Revenue by Year Month.
8. Add a table of Date, Price, Adjusted Price, Is Available, Minimum Nights, and Maximum Nights.
9. Add Listing URL as a Web URL button if desired.
10. Right-click the page tab and select **Hide page**.
11. Test from the Page 3 table: right-click one listing -> **Drill through > Listing Detail**.

---

## 14. Add a report-page tooltip

1. Create a page named `Listing Tooltip`.
2. In page formatting, set **Canvas Settings > Page size template = Tooltip**.
3. Set **Page Information > Tooltip = On**.
4. Add DimListing[Listing ID] to Tooltip fields.
5. Add compact cards for Listing Name, Average Price, Availability Rate, Lifetime Review Count, and Projected Revenue.
6. Add a small price trend if it remains readable.
7. Hide the page.
8. Assign it manually to the map and top-listings visuals under **Format visual > Tooltip > Report page** if automatic matching does not work.

---

## 15. Create bookmarks and navigation

### Reset Filters bookmark

1. Put each visible page into its intended default state.
2. Open **View > Bookmarks**.
3. Select Add and rename the bookmark `Reset - Overview`.
4. Keep Data selected so slicer/filter state is captured.
5. Insert a Reset button and set **Action > Type = Bookmark**.
6. Repeat for the other visible pages or use carefully scoped selected-visual bookmarks.

### Typical versus all-price bookmarks

1. Open **View > Selection** and **View > Bookmarks**.
2. Configure the histogram with Price <= 650 and add `Typical Price Range`.
3. Remove the upper limit and add `All Prices`.
4. If bookmarks should affect only the histogram, choose **Selected visuals** and clear Current page as appropriate.
5. Add **Insert > Buttons > Navigator > Bookmark navigator** and point it to a bookmark group containing these two states.

---

## 16. Validate the model and measures

Create a temporary page named `QA` and compare Power BI results with these expected unfiltered values:

| Check | Expected value |
|---|---:|
| Total Listings | 1,749 |
| Total Hosts | 1,111 |
| Calendar Rows | 319,192 |
| Priced Calendar Nights | 319,117 |
| Available Nights | 170,829 |
| Unavailable or Blocked Nights | 148,363 |
| Availability Rate | 53.5192% |
| Average Price | EUR 109.9178 |
| Review Count with no date filter | 62,987 |
| Reviewed Listings | 1,525 |
| Baseline Projected Revenue at 70% and 1.00x | EUR 24,553,642.40 |
| Projected Revenue at 80% and 1.10x | EUR 30,867,436.16 |

The revenue checkpoints are over the supplied priced calendar rows, not a full 365 days per listing.

### Required behavioral tests

1. Change occupancy from 70% to 80% with multiplier fixed at 1.00x. Projected Revenue should increase by 14.2857%.
2. Change multiplier from 1.00x to 1.10x with occupancy fixed. Scenario Price and Projected Revenue should increase by 10%.
3. At 70% occupancy and 1.00x multiplier, Projected Revenue must equal Baseline Projected Revenue and variance must be zero.
4. Filter one Listing ID and manually verify `sum(price) x occupancy x multiplier` for that listing.
5. Check that date, property type, and room type slicers filter every intended visual.
6. Verify that Lifetime Review Count remains available when the Calendar date slicer is set to 2022.
7. Verify map coordinates fall around Antwerp and are not aggregated.
8. Confirm null prices do not become zero-price bins.
9. Confirm Amenity filtering works. If it causes ambiguity, revisit the single controlled bidirectional bridge relationship.
10. Test drill-through from multiple source visuals and test the Back button.
11. Test tooltips after publishing, not only in Desktop.

After validation, hide the QA page rather than deleting it, or retain the expected-value table in project documentation.

---

## 17. Optimize the model and report

1. Use Import mode for these CSV sizes.
2. Disable load for staging queries.
3. Remove unused long-text fields, especially Comments, Reviewer Name, Host About, and Description, unless they appear in a required visual.
4. Hide technical keys and parameter-support fields.
5. Prefer explicit measures over dragging numeric columns into visuals as implicit sums.
6. Keep relationship direction Single except the deliberate amenity bridge.
7. Avoid calculated columns on the 319,192-row Calendar table unless necessary. The one Price Bin column is justified by the histogram.
8. Limit the top-listings table to a useful number of rows.
9. Avoid too many visuals on one page; approximately 6-10 meaningful visuals is usually enough.
10. Open **Optimize > Performance analyzer** or **View > Performance analyzer**, start recording, refresh visuals, and inspect slow DAX or rendering.
11. Target ordinary interactions below roughly 2-3 seconds on the development machine.
12. Save, close, reopen, and perform a full refresh to catch broken file paths or type-conversion errors.

---

## 18. Add a last-refresh timestamp

In Power Query, create a blank query named `RefreshInfo`:

```powerquery
let
    Source = #table(
        type table [LastRefreshUTC = datetimezone],
        {{DateTimeZone.FixedUtcNow()}}
    )
in
    Source
```

Add a measure:

```DAX
Last Refresh Label =
"Data refreshed: "
    & FORMAT ( MAX ( RefreshInfo[LastRefreshUTC] ), "dd MMM yyyy HH:mm" )
    & " UTC"
```

Display it in a small card in the report header.

---

## 19. Publish and configure refresh

### Preferred approach for a course project

Move the four CSV files into a stable SharePoint or OneDrive for Business folder, update the Power Query source paths, refresh successfully in Desktop, and then publish. This avoids dependence on a personal computer being online.

### If the CSV files remain on the local drive

Power BI Service cannot directly refresh a file from your PC unless a gateway is configured and running.

1. Install and sign in to an on-premises data gateway; personal mode is acceptable for an individual course project, while Microsoft recommends an enterprise gateway for managed organizational use.
2. Keep the CSV folder path stable.
3. In Power BI Desktop, select **Home > Refresh** and confirm success.
4. Select **Home > Publish**, choose the correct workspace, and publish.
5. In Power BI Service, open the workspace and locate the semantic model.
6. Open **Settings** for the semantic model.
7. Configure **Gateway and cloud connections** and map the model to the gateway/data source.
8. Confirm **Data source credentials**.
9. Open **Refresh > Schedule refresh**, turn it on, choose a suitable frequency/time zone, and enable refresh failure notifications.
10. Select **Refresh now** once and check Refresh history.
11. Open the published report and retest slicers, bookmarks, drill-through, tooltips, map, and parameter sliders.
12. Record the workspace, report owner, refresh frequency, source location, and gateway owner in the documentation.

---

## 20. User guide text to include in the submission

### Navigation

- Use the page navigator to move among Overview, Listing Analysis, and Scenario Insights.
- Use Date, Property Type, and Room Type to filter the report.
- Use Reset Filters to restore the designed default state.

### Scenario controls

- Occupancy Rate changes the assumed share of loaded priced nights that are occupied.
- Seasonal Multiplier scales each nightly price: 1.00x leaves price unchanged, 1.10x adds 10%, and 0.90x reduces it by 10%.
- Projected Revenue is a scenario result, not observed booking revenue.

### Detailed analysis

- Hover over listings to see the custom tooltip.
- Right-click a listing and select Drill through > Listing Detail for the listing-level view.
- On the histogram, switch between Typical Price Range and All Prices to inspect high-price outliers.

### Data limitations

- Unavailable nights can include bookings, owner blocks, and other restrictions; they are not labeled as confirmed bookings.
- Calendar coverage is incomplete per listing, so the default projection covers loaded priced rows only.
- Review count is a volume metric; the data does not contain a rating score.
- The solution performs descriptive and scenario analysis only and does not use machine learning.

---

## 21. Submission checklist

- [ ] Four sources imported through Power Query.
- [ ] Staging queries have load disabled.
- [ ] Price types and date types validated.
- [ ] Amenities normalized.
- [ ] Star schema created with correct cardinalities.
- [ ] Date table created, sorted, and marked as the date table.
- [ ] Occupancy and seasonal What-If parameters created.
- [ ] Core KPIs and scenario measures created and formatted.
- [ ] Three visible pages completed.
- [ ] Hidden Listing Detail drill-through page completed.
- [ ] Hidden report-page tooltip completed.
- [ ] Reset and price-range bookmarks completed.
- [ ] Slicer synchronization and visual interactions tested.
- [ ] Raw-data reconciliation values matched.
- [ ] Performance Analyzer run and slow visuals addressed.
- [ ] Refresh timestamp displayed.
- [ ] PBIX published and Service refresh tested.
- [ ] Navigation, metric definitions, assumptions, and refresh steps documented.

---

## 22. Official Power BI references

- [Star schema guidance](https://learn.microsoft.com/en-us/power-bi/guidance/star-schema)
- [Create and use numeric range parameters](https://learn.microsoft.com/en-us/power-bi/transform-model/desktop-what-if)
- [Set and use date tables](https://learn.microsoft.com/en-us/power-bi/transform-model/desktop-date-tables)
- [Create drill-through pages](https://learn.microsoft.com/en-us/power-bi/create-reports/desktop-drillthrough)
- [Create report-page tooltips](https://learn.microsoft.com/en-us/power-bi/create-reports/desktop-tooltips)
- [Create report bookmarks](https://learn.microsoft.com/en-us/power-bi/create-reports/desktop-bookmarks?tabs=powerbi-desktop)
- [Use Performance Analyzer](https://learn.microsoft.com/en-us/power-bi/create-reports/desktop-performance-analyzer)
- [Refresh semantic models created from local PBIX files](https://learn.microsoft.com/en-us/power-bi/connect-data/refresh-desktop-file-local-drive)
- [Configure scheduled refresh](https://learn.microsoft.com/en-us/power-bi/connect-data/refresh-scheduled-refresh)

---

## 23. Individual submission adaptation (team plan → solo ownership)

The team plan in `Airbnb_PowerBI_Project_Plan.xlsx` assigns 14 tasks (phases A–I) across 5 learners (L1–L5), with several tasks deliberately duplicated across learners for peer review (e.g. `C.1 Data Model` and `D.1 Basic DAX Measures` each appear under 3 different learners; `F.1 Design Core Pages` and `G.1 Data Validation` appear under 2–3). Executed solo, those duplications collapse into one pass plus a documented self-review step, since there is no second learner to cross-check the work.

| Phase | Task | Original team owner(s) | Solo owner | Deduplication note |
|---|---|---|---|---|
| A | A.1 Dataset Connectivity | L1 | You | No change — single pass |
| B | B.1 Calendar cleaning | L1 | You | No change |
| B | B.2 Listings/Hosts/Reviews cleaning | L2, L5 | You | One pass; L5's 10h folded into review time |
| C | C.1 Data model & relationships | L1, L2, L3 | You | One build pass + a self-review pass using Section 16's QA page, replacing peer cross-check |
| C | C.2 Feature aggregation | L3 | You | No change |
| D | D.1 Basic DAX measures | L1, L2, L3 | You | One authoring pass; the other two learners' time becomes your own reconciliation against Section 16 expected values |
| E | E.1 What-if scenario modeling | L3, L4 | You | No change in scope, single build |
| F | F.1 Core report pages | L2, L4, L5 | You | Build once; use the "behavioral tests" in Section 16 in place of a second learner's UAT |
| F | F.2 Interactivity & tooltips | L4, L5 | You | No change |
| G | G.1 Data validation & DAX testing | L2, L3, L4, L5 | You | Formal self-QA pass against Section 16's checkpoint table — do this on a different day than the build, with fresh eyes |
| G | G.2 Performance tuning | L4 | You | No change |
| H | H.1 User guide & technical doc | L1, L2, L5 | You | Scope increased — see Section 25 |
| H | H.2 Publish & share | (unassigned in source plan) | You | No change |
| I | I.1 Coordination & meetings | All | You (reduced) | Team stand-ups/retro replaced by a short self-check-in at the end of each work session — see the `Task_Checklist` sheet in the solo workbook |

Two Excel files travel with this submission and serve different purposes — do not merge them:

- **`Airbnb_PowerBI_Project_Plan.xlsx`** (unmodified, original) — evidence of the plan as designed for a 5-person team.
- **`Airbnb_Solo_Submission_Checklist.xlsx`** (new) — the actual tracker used to execute and QA the solo build, referenced throughout Sections 23–25.

---

## 24. Solo execution timeline (actuals, against the confirmed 20/27 September deadline)

This replaces the generic 3-week estimate with the real calendar the work is running against: a build window ending tonight, a personal-leave gap with no possible work, and a short polish window inside the instructor-granted extension. Deduplicating the team plan's overlapping assignments (Section 23) gives roughly 100–110 hours of genuinely distinct work; spread across the dates below, with the heaviest concentration in the final push before departure.

| Date | Day | Phase | Guide sections | Exit criteria / deliverable |
|---|---|---|---|---|
| 5–6 Sep | Sat–Sun | Kickoff, dataset connectivity, start Calendar cleaning | 3, 4.3 | Sources connected; staging queries created; Calendar price/date cleaning underway |
| 7–8 Sep | Mon–Tue | Finish cleaning (Listings/Hosts/Reviews, amenities bridge); build star schema & relationships | 4.1–4.5, 5, 6 | Star schema built; relationships validated; `DimDate` marked as date table |
| 9–10 Sep | Wed–Thu | Core DAX measures; What-If parameters & scenario DAX | 8.1–8.2, 7, 8.3–8.5 | QA page shows Total Listings = 1,749, Total Hosts = 1,111, Calendar Rows = 319,192; both What-If parameters work |
| 11 Sep | Fri | Core report pages — Overview and Listing Analysis | 10, 11 | Both pages substantially built with KPI cards, slicers, map, histogram, top-listings table |
| **12 Sep (tonight)** | **Sat** | **Scenario Insights page, drill-through/tooltip/bookmarks, QA against Section 16, performance pass, publish, first documentation draft** | **12–19** | **All 3 visible pages + hidden drill-through/tooltip pages done; Section 16 checkpoints and behavioral tests run; refresh timestamp visible; `.pbix` published or screenshot evidence captured** |
| 13 Sep (before 11:00 IST) | Sun | Final review pass; package and submit **WIP version** | — | WIP submitted to professor as the agreed interim checkpoint — not a default, per prior agreement |
| 13 Sep 11:00 – 22 Sep | — | **Personal leave — no work planned or possible** | — | — |
| 23–25 Sep | Wed–Fri | Resume: address any professor feedback on the WIP; finalize user guide/technical doc; build the findings deck and presenter's script (Section 25) | 20, 25 | Documentation complete; findings deck drafted with real KPI values; presenter's script (Section 25.4) finalized so the 10-day gap doesn't cost re-derivation time |
| 26 Sep | Sat | Final packaging, rehearsal using the presenter's script, submit final version | 25.3 | Full submission package complete and submitted — one day inside the 27 Sep extended deadline as buffer |

Adjust the `Timeline` sheet in `Airbnb_Solo_Submission_Checklist.xlsx` if any of these dates shift — the task list itself does not change.

---

## 25. Presentation & research report deliverable (target: clearly stronger than the team baseline)

The Problem Statement's minimum ask (Section 6, "Documentation & Deployment") is a user guide plus technical documentation. To meet the "10x better documented" target, this submission adds a short findings presentation, modeled on the structure and tone of the prior "Rainfall vs Agriculture" submission (title → objective/dataset → method → dashboard walkthrough → insights → one worked example → recommendations → limitations/conclusion → sources), adapted to the Airbnb Antwerp dataset and this dashboard's actual KPI values.

**Narrative framework (instructor guidance):** the course's recorded expectation-setting sessions stressed that technical correctness (a clean model, working DAX) is "necessary but insufficient" — what gets rewarded is a decision-ready narrative. Two tools from those sessions structure the deck below:

- **CIIA** — every insight slide should move through **C**ontext (what decision this supports) → **I**nsight (what went up/down/stayed flat) → **I**mplication (why that matters to an owner/host) → **A**ction (a concrete recommendation with an owner, a move, a timeframe, and a measure — never a vague "improve X").
- **The three-question rule** — anywhere you present a number, answer: *What changed? Why does it matter? What should we do?* A number with no framing ("availability rate is 53.52%") is an anti-pattern; a framed one ("just over half of nights are booked or blocked, and it varies by room type — that's a pricing lever, not a fixed constraint") is the target.
- **Scenario narrative template** — for the What-If slides specifically, present each scenario as **assumptions → impact → implication → risk**, not just a number: what you assumed (occupancy %, multiplier), what it changes (revenue), why that matters (implication for the host), and what could go wrong (risk that the assumption doesn't hold).

### 25.1 Findings deck outline (`Airbnb_Findings_Presentation.pptx`, ~10 slides)

1. **Title** — project name, your name, Antwerp geography, 26 Dec 2021–25 Dec 2022 window, 1,749 listings.
2. **Objective & dataset** — restate the Problem Statement's objective in one line (this is the Context for everything that follows); a coverage table (rows, grain, date range) drawn from Section 2's facts table.
3. **Data preparation & star schema** — a simple diagram of DimListing / DimHost / DimDate / FactCalendar / FactReview / the amenity bridge (Section 5), plus the 2–3 cleaning decisions that most affect the numbers (null prices never zero-filled; `available=0` ≠ booked; partial calendar).
4. **What-if scenario design** — the two parameters, the baseline (70%/1.00x), and the `Projected Revenue` formula in plain English, not raw DAX. Frame the baseline itself using the scenario template: assumption (70% occupancy, no seasonal adjustment) → what it's for (a neutral reference point) → implication (every other scenario is read as a delta off this line).
5. **Dashboard walkthrough — Overview** — a screenshot plus a 2–3 line narrative of what it shows.
6. **Dashboard walkthrough — Listing Analysis & Scenario Insights** — screenshots plus narrative, including one scenario slider change presented with the full assumptions → impact → implication → risk template (e.g. assumption: occupancy 70%→80%; impact: revenue +14.29%; implication: a realistic marketing/pricing push could capture this; risk: assumes demand exists to fill the extra nights, which the data cannot confirm).
7. **Key insights** — 5–7 findings, each run through CIIA and the three-question rule, grounded in actual computed numbers (e.g. overall availability rate 53.52%, average price EUR 109.92 vs median EUR 79 showing right-skew, room-type or property-type price/availability differences once computed) — not generic praise, matching the rainfall deck's "what this does and doesn't tell us" discipline.
8. **Worked example** — pick one listing or one room type, walk it through both occupancy and multiplier changes using the scenario template, and state plainly what the resulting variance does and doesn't imply (echo the rainfall deck's "What It Does Not Tell Us" framing, adapted: e.g. "higher projected revenue at 80% occupancy is a scenario assumption, not evidence the listing will actually achieve that occupancy").
9. **Recommendations** — 5–7 recommendations, each written as a CIIA Action (owner + move + timeframe + measure, e.g. "Hosts in [room type] should test a +10% seasonal multiplier for Q3 2022 and track occupancy weekly to confirm demand holds" — not "optimize seasonal pricing").
10. **Limitations, assumptions & conclusion + sources** — pull directly from Section 0's assumption list plus the data-limitation bullets in Section 20; close with one plain-language conclusion sentence answering "what should the reader do with this"; list the source files.

Keep the Section 9 palette (`#FF385C` primary, `#6C5CE7` scenario, `#00875A` positive, `#D97706` warning) consistent between the dashboard and the deck so the two artifacts read as one system, and apply the Section 9.1 dashboard-quality checklist to every chart/screenshot reused in the deck (scaled numbers, data labels, no duplicate metrics, meaningful titles).

### 25.2 Written user guide & technical documentation (`Airbnb_User_Guide_and_Technical_Documentation.pdf`)

Combine Section 20's navigation/scenario/limitations copy with: the star-schema diagram, the full DAX measure list with one-line descriptions (Section 8), the What-If parameter setup steps (Section 7), the Section 16 validation table with your actual results filled in, and the Section 0 assumptions list. This is the artifact a grader or another analyst would use to understand *why* a number is what it is, not just *what* the dashboard shows.

### 25.4 Presenter's script (`Airbnb_Presentation_Script.md`) — bridging the 10-day memory gap

Section 24's schedule has a hard 9-day gap (13–22 September, personal leave) between when the deck's content is fresh in your head and when you actually have to stand up and present it on 23–26 September. A slide deck alone is not enough to present cold after that gap — you need the words, not just the bullet points. This script is a full, near-verbatim speaker script, one block per slide, written during the 23–25 September session (once the dashboard's real KPI values exist) so that on presentation day you are reading/rehearsing a script, not reconstructing reasoning from memory.

Structure — one block per slide from Section 25.1, each written to be spoken aloud in 30–60 seconds:

1. For each slide, write 3–5 sentences of actual spoken narration (not bullet fragments) that a presenter can read almost verbatim.
2. For every insight or scenario slide, make the CIIA structure explicit *in the script itself* — literally write a Context sentence, an Insight sentence, an Implication sentence, and an Action sentence in that order, so the three-question rule ("what changed / why it matters / what to do") is answered without having to improvise it live.
3. Add a one-line "if asked" note under any slide likely to draw a question (e.g. under the scenario slides: "if asked why 70%/1.00x was chosen as baseline — no baseline was specified in the brief, so a neutral midpoint was picked; see Assumption 3").
4. Keep a short "cold-open checklist" at the top of the script: the 5 numbers you must not forget (Total Listings, Availability Rate, Average Price, the baseline assumption, and your single strongest insight) — so even a rushed re-read the morning of the 26th restores the essentials in under 2 minutes.

Draft this script's skeleton now, before the 12 September build session, with `[FILL IN]` placeholders wherever a real computed number is needed; fill in the actual figures once the dashboard is built tonight, and finalize the wording during the 23–25 September session per Section 24.

### 25.3 Final submission package

- [ ] `Airbnb_Antwerp_Pricing_Dashboard.pbix` — the built report
- [ ] `Airbnb_Findings_Presentation.pptx` — the 10-slide deck (Section 25.1)
- [ ] `Airbnb_User_Guide_and_Technical_Documentation.pdf` — combined user guide + technical doc (Section 25.2)
- [ ] `Airbnb_Presentation_Script.md` — presenter's script for the live presentation (Section 25.4)
- [ ] `Airbnb_PowerBI_Project_Plan.xlsx` — original team plan, submitted unmodified as required reference material
- [ ] `Airbnb_Solo_Submission_Checklist.xlsx` — actual individual execution/QA tracker (Section 23)
- [ ] `Airbnb_Power_BI_Step_by_Step_Guide.md` — this guide (optional to submit; demonstrates process rigor if included, e.g. exported to PDF)

---

## 26. Offline execution notes (for working without internet access)

This entire guide — Sections 3 through 18 — was written to be executable in Power BI Desktop with **no internet connection**: all data sources are local CSVs, all transformations are Power Query (M) running locally, all DAX/relationships/visuals are local computation, and Performance Analyzer (Section 17) is fully local. You can work through all of that offline.

**Two specific exceptions that need internet:**

1. **The map visual (Section 11.1)** — both Power BI's built-in Map visual and Azure Maps render their base imagery (the actual map tiles behind your listing dots) by fetching them live from a mapping service. Offline, the visual will still accept your `Latitude`/`Longitude`/`Legend`/`Tooltip` field assignments and the listing points will be positioned correctly in the underlying data, but the map background will likely appear blank, gray, or fail to render. **Recommendation:** build and wire up the map visual fully offline (Steps in Section 11.1 don't require internet to configure), but treat its visual appearance as unverified until you're back online — do a quick visual check then, before considering Section 11 "done."
2. **Publishing to Power BI Service (Section 19)** — requires an active connection by definition. This was already flagged as best-effort in Assumption 9 (Section 0); simply defer all of Section 19 to whenever connectivity returns (the 23–26 September window works fine for this).

Everything else — Power Query cleaning, the star schema and `DimDate` table, What-If parameters, all DAX measures, the 3 report pages' non-map visuals, drill-through, the report-page tooltip, and bookmarks — works exactly the same with Wi-Fi off. If Power BI Desktop shows an account/sign-in prompt at launch, it can normally be dismissed or skipped ("Continue without signing in" style option) without blocking any of the local authoring work above.

**Suggested offline workflow for this session:** continue sequentially from wherever this guide left off, save frequently (`Ctrl+S`), and don't attempt Section 11.1's visual verification or Section 19's publish step until online again — everything else can be completed and is self-contained within this document.

