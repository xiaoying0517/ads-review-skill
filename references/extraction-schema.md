
# Extraction Schema

A field-by-field guide for extracting **observable** restaurant advertising and local visibility data from screenshots. This public schema is platform-neutral and uses no private dashboard access or merchant data.

Read this file before Step 1 in `SKILL.md`. Map the extracted values to the Intermediate JSON defined there. Do not generate findings or HTML before completing the confidence gate.

## 1. General extraction rules

1. **Classify each screenshot** as Campaign Performance, GBP, AI Search, Local Map Search, Reviews, or Unknown. A screenshot may contain more than one module.
2. **Use visible evidence only.** Preserve labels, units, dates, and the reporting period. Do not fill cropped or hidden fields from assumptions.
3. **Separate zero from missing.** Use `0` only when the source explicitly shows zero; use `null` for missing/unreadable numeric fields, `""` for unknown text, and `[]` for absent lists. For an optional module not supplied, use `available: false`.
4. **Normalize units consistently.** Store counts as numbers without separators, monetary amounts as numeric values without currency symbols, and percentages as percentage-point numbers (e.g., `12.5` means 12.5%, not 0.125). Preserve the original currency and measurement definition in extraction notes or `warnings` when relevant; do not assume USD.
5. **Normalize dates** to `YYYY-MM-DD` only when the date is unambiguous. Record the source date range separately in notes when the Intermediate JSON lacks a dedicated field. Never invent a reporting cutoff.
6. **Distinguish observed from derived.** A computed CTR, CPC, duration, or difference must identify its inputs and formula. Do not replace a visible platform value silently with a recalculated one.
7. **Do not infer causality.** Paid clicks, map interactions, AI mentions, and review sentiment are not verified sales or incremental lift.
8. **Avoid exposing confidential information.** Public demos must use synthetic or explicitly authorized screenshots and merchant details.

## 2. Campaign Performance screenshots

### Identity and reporting context

| Target field | Look for | Rule |
|---|---|---|
| `restaurant.name` | Merchant/business heading | Prefer user-confirmed full name when screenshot is truncated |
| `restaurant.address` | Address in header/details | Leave blank if not visible or supplied |
| `restaurant.city_state` | City and region | Do not infer from the business name |
| `restaurant.start_date` | Campaign start/launch date, e.g. `Started on: MM/DD/YYYY` | Parse the displayed format; normalize to `YYYY-MM-DD` only when unambiguous |
| Reporting window | Date picker, filter, caption | Record in notes; may differ from campaign lifetime |
| Campaign status | Active/paused/ended label | Context only; not an existing Intermediate JSON key |
| `campaign_metrics.ad_spend` | Spend/cost field | Record the numeric value and source currency |

### Detect KPI mode before mapping cards

- **`standard`:** The **Reach** tab is selected. Extract impressions, clicks, CTR, CPC, and ad spend from its visible fields, alongside any visible action metrics. If the interface does not use named tabs, select this mode only when the equivalent reach KPI set is clearly present.
- **`restructured`:** The **Results** tab is selected. In the original dashboard, the Revenue/ROAS range and ad-spend fields may remain visible regardless of the selected tab; the Google Maps action metrics are also present in either tab. The Results view does not supply the Reach metrics. Extract ad spend and all visible action metrics, but **never use Revenue or ROAS ranges** in the report. Do not invent impressions, clicks, CTR, or CPC.
- **Ambiguous:** If the selected view cannot be established and this changes the report layout, flag `campaign_metrics.kpi_mode` as low confidence and ask for a clearer screenshot.

The mode is a report-layout choice, not proof that absent metrics equal zero. Store **one** actual value (`"standard"` or `"restructured"`) in the final JSON, never the literal placeholder `"standard|restructured"`.

### Campaign metrics

| JSON field | Possible source label | Value rule |
|---|---|---|
| `impressions` | Impressions | Count |
| `clicks` | Clicks | Count |
| `ctr` | CTR / Click-through rate | Percent number, e.g. `3.2` for 3.2% |
| `cpc` | CPC / Cost per click | Currency amount |
| `orders` | Orders | Count of attributed actions, not independently verified sales |
| `menu_views` | Menu Views | Count |
| `directions` | Directions / Direction Views | Count; retain the platform label in notes if its meaning differs |
| `phone_calls` | Phone Calls | Count |
| `website_visits` | Website Visits | Count |
| `other_actions` | Other Actions | Count |

A screenshot may display **Total Revenue range** or **ROAS range**. The original workflow explicitly excludes both: **NEVER use either in the report, including narrative, KPI cards, or conclusions.** Store Visits and Cost per Store Visit are **not KPI cards**; they may appear in narrative only with clear source/estimation caveats. The six supported Google Maps action metrics are Orders, Direction Views, Phone Calls, Website Visits, Menu Views, and Other Actions. Do not invent JSON keys for excluded metrics without revising `SKILL.md` and the other references together.

### Search Insights: campaign keyword tags

- Extract **every visible** keyword or search term, preserving its displayed text and order.
- Do not infer hidden rows or claim that a cropped list is complete.
- Campaign search terms may be kept in extraction notes until the public Intermediate JSON has a dedicated campaign keyword field. Do **not** place campaign terms into `gbp_metrics.search_terms` unless they are actually GBP search terms.
- If keyword data is absent, the Keyword Insights report section is omitted.

## 3. Google Business Profile (GBP) screenshots

| JSON field | Look for | Rule |
|---|---|---|
| `gbp_metrics.total_interactions` | Total / All Interactions | Count for the visible period |
| `gbp_metrics.view` | Weekly / Monthly selection | Store `"weekly"` or `"monthly"` |
| `gbp_metrics.chart_type` | Chart legend and visible lines | Store `"dual"` or `"red-only"` |
| `gbp_metrics.data_points` | Chart dates and values | Capture visible points, flag estimates |
| `gbp_metrics.search_terms` | GBP search impressions / terms | Extract visible terms and counts |

### Chart style and data points

- **`dual`:** The source interface shows two series (originally labeled `Before [provider]` and `With [provider]`, often gray and red). Record the actual visible legend text for each; do not interpret them as a causal before/after experiment.
- **`red-only`:** A single red series is visible (originally the `With [provider]` series). This is a layout label, **not** proof of merchant tenure or service relationship.
- If the chart does not fit either style, flag the mismatch and request clarification rather than forcing a mode.
- Read exact values from tooltips or labels when possible. If a value is visually estimated from a line or axis, mark it approximate and add its JSON path to `confidence_flags.approximated_fields`.
- Do not treat pending/current incomplete periods as final observations.

Suggested array item shapes (examples of **structure**, not performance data):

```json
{
  "data_points": [
    {
      "date": "YYYY-MM-DD",
      "series_a": null,
      "series_b": null,
      "estimated": false,
      "pending": false
    }
  ],
  "search_terms": [
    {
      "term": "visible term",
      "count": null
    }
  ]
}
```

For `red-only`, use `series_a` for the one visible series and `series_b: null`; for `dual`, use both. Keep actual legend labels in extraction notes so the HTML does not invent series meanings. `null` dates or omitted points are preferable to invented timestamps.

### GBP-absent fallback

If no GBP screenshot is supplied, do not invent `gbp_metrics.total_interactions`. If Local Map visibility is available, the report may substitute a clearly labeled **Local Map Visibility %** card for the original GBP Total KPI card. If neither is available, omit that card. This follows the original optional-GBP rule.

## 4. AI Search visibility screenshots (optional)

Set `ai_search.available: true` **only** when relevant AI Search evidence is provided. Otherwise keep it `false` and do not assess AI visibility.

| JSON field | Look for | Rule |
|---|---|---|
| `date_range` | Selected measurement window | Preserve the displayed range; do not infer |
| `visibility_score` | Visibility / presence score | Numeric percentage or source-defined score; record its scale |
| `sentiment` | Sentiment label | Preserve visible label, not inferred tone |
| `top3_ranking` | Top-3 ranking / appearance rate | Numeric percentage only if definition is visible or documented |
| `prompts` | Prompt-level results | Read visible prompts and associated metrics |
| `sov_your` | Merchant Share of Voice | Numeric percentage if shown |
| `sov_competitors` | Competitor Share of Voice | Visible competitor rows only |

Suggested `prompts` item:

```json
{
  "prompt": "visible text",
  "presence_pct": null,
  "sov_pct": null,
  "sentiment": "",
  "query_type": "brand|category|unknown"
}
```

Suggested `sov_competitors` item:

```json
{
  "name": "visible competitor",
  "sov_pct": null
}
```

- Mark `query_type` as `brand` only if the visible query explicitly contains the business name or clear brand reference; otherwise use `category` for a generic category query or `unknown`.
- Do not expand truncated prompt text or infer omitted competitors.
- **Daily Trend:** Note whether the displayed AI visibility trend is rising, flat, or declining over the shown date range; flag visual estimates.
- **AI-Origin Traffic:** If visible, note total AI page views and the displayed traffic-source breakdown, without guessing missing sources or counts.
- These observations have no dedicated Intermediate JSON field yet; preserve them in extraction notes rather than silently dropping them or inventing new keys.
- Visibility results are **sample-dependent**: queries, model, geography, date, and measurement method can change the outcome. Do not label a score as a universal probability of recommendation unless the source defines it that way.

## 5. Local Map Search screenshots (optional)

Set `local_map.available: true` only when a relevant local-ranking/map-visibility screenshot is supplied.

| JSON field | Look for | Rule |
|---|---|---|
| `avg_rank` | Average Rank | Numeric value; lower typically means better ranking when defined that way |
| `visibility_pct` | Visibility % | Percentage number, preserving source definition |
| `keywords` | Keyword-level ranking table | Visible rows only |
| `competitors` | Competitor comparison table | Visible rows only; distinguish merchant's own row |

Suggested `keywords` item:

```json
{
  "keyword": "visible term",
  "avg_rank": null,
  "visibility_pct": null
}
```

Suggested `competitors` item:

```json
{
  "name": "visible business",
  "is_self": false,
  "avg_rank": null,
  "visibility_pct": null,
  "rating": null,
  "review_count": null
}
```

- Use a visible `You`/merchant indicator to set `is_self: true`; do not rely on row order.
- The original dashboard used a color guide of **green = top 3**, **yellow = 4–10**, and **red = 10+** for average rank. Preserve that mapping only when the displayed legend or source documentation confirms it; do not infer rank from color in an unrelated interface. The original `10` boundary is ambiguous across the latter two bands, so prefer numeric rank values when visible.
- Capture the query set, location/grid, and date when shown. Local ranking is geographically and query dependent.
- A listed competitor is a **comparison candidate**, not necessarily a true substitute; relevance is assessed later in `analysis-rules.md`.

## 6. Customer Reviews screenshots (optional)

Set `review_data.available: true` only when a review-summary or review-tag screenshot is supplied.

| JSON field | Source | Rule |
|---|---|---|
| `tags_positive` | Positive tags | Transcribe visible tag text |
| `tags_neutral` | Neutral tags | Transcribe visible tag text |
| `tags_negative` | Negative tags | Transcribe visible tag text |
| `summary_bullets` | Review summary | Preserve visible claims; flag truncation |

- The original tag styling was **positive: green / ↗ / `tag-pos`**, **neutral: gray-white / — / `tag-neu`**, **negative: pink-red / ↘ / `tag-neg`**. Preserve these generic CSS-class conventions if the HTML template uses them, but classify screenshots by explicit category labels or a verified legend rather than color alone.
- Keep tags as strings, e.g. `"friendly staff"`; do not fabricate frequencies, sentiment shares, or sample size.
- The original panel was labeled **“Summary of the last 50 reviews.”** When that wording is visible, retain the stated 50-review scope; for other panels, preserve their actual N. Never treat the panel scope as a verified count of all reviews.
- Do not infer who owns or manages review replies unless supplied by the user.

## 7. Campaign duration and cross-screenshot reconciliation

Calculate `restaurant.duration_days` only when **both** campaign start date and a valid campaign data cutoff are known:

```text
Duration (days) = campaign reporting cutoff date - campaign start date
```

- Do not substitute the latest GBP chart date for the **campaign** cutoff without evidence that their reporting periods align.
- If only a month, a pending period, or an ambiguous date is visible, leave `duration_days: null` and request clarification if duration is material.
- The original workflow used the **latest non-pending GBP data point** as an approximate cutoff. This is permitted **only if the reporting periods demonstrably align**; otherwise do not infer a campaign cutoff from GBP alone.
- Preserve the original narrative display convention for approximate duration: **`约 X 天` if fewer than 60 days; `约 X 个月` if 60 days or more**. For months, explain that the figure is approximate. Store a numeric estimate with an approximation flag.
- In Update mode, compare values only when definitions, units, attribution windows, and reporting windows are compatible. A cumulative total increasing is not necessarily improved performance.
- If screenshots disagree on the merchant, date range, or units, stop and resolve the conflict before combining them.

## 8. Confidence scoring and JSON mapping

| Level | Definition | Required handling |
|---|---|---|
| **HIGH** | Clearly visible value, unambiguous label and unit | Extract directly |
| **MEDIUM** | Legible but approximate chart reading, or a noncritical ambiguity | Mark `~` / `约`; add to `approximated_fields` |
| **LOW** | Obscured, truncated, conflicting, or materially inferred | Add to `low_confidence_fields`; ask if material |

Use stable JSON paths such as `campaign_metrics.ad_spend`, `gbp_metrics.data_points[0].series_a`, or `local_map.avg_rank` in `confidence_flags`. Add a short reason in `warnings` or extraction notes.

### Final consistency checks

- Every number maps to a visible source or an explicitly documented formula.
- Each optional module's `available` flag matches the supplied evidence.
- `kpi_mode`, `chart_type`, `view`, and `language` contain **one selected value**, not a pipe-separated list of alternatives. If unresolved, record `null` and follow the confidence gate.
- Missing values remain missing; never turn `null` into `0` to fill KPI cards.
- All arrays use consistent item shapes, and percentage units are consistent.
- Do not produce analysis or HTML until Step 1.5 in `SKILL.md` passes.

**Compatibility note:** This schema keeps the existing top-level Intermediate JSON keys in `SKILL.md`. Some original screenshot details (such as campaign Search Insights tags, AI Daily Trend and traffic-source breakdown, chart legend names, merchant star rating/review count, or data cutoff) have no dedicated field there; preserve them as extraction notes for now. This is a **known data-contract limitation** and should be fixed before claiming complete automated handoff into HTML. Add new fields only as a coordinated versioned change across `SKILL.md`, `analysis-rules.md`, and `html-template.md`.

