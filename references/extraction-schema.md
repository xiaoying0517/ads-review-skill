
# Extraction Schema

A field-by-field guide for extracting **observable** restaurant advertising and local visibility data from screenshots. This public schema is platform-neutral and uses no private dashboard access or merchant data.

Read this file before Step 1 in `SKILL.md`. Map the extracted values to the Intermediate JSON defined there. Do not generate findings or HTML before completing the confidence gate.

## 1. General extraction rules

1. **Classify each screenshot** as Campaign Performance, GBP, AI Search, Local Map Search, Reviews, or Unknown. A screenshot may contain more than one module.
2. **Use visible evidence only.** Preserve labels, units, dates, and reporting periods. Do not fill cropped or hidden fields from assumptions.
3. **Separate zero from missing.** Use `0` only when the source explicitly shows zero; use `null` for missing or unreadable numeric fields, `""` for unknown text, and `[]` for absent lists. For an optional module not supplied, use `available: false`.
4. **Normalize units consistently.** Store counts as numbers without separators, monetary amounts as numeric values without currency symbols, and percentages as percentage-point numbers (e.g., `12.5` means 12.5%, not 0.125). Store the source currency in `campaign_metrics.currency` when identifiable; do not assume USD.
5. **Normalize dates** to `YYYY-MM-DD` only when unambiguous. Preserve reporting windows in their corresponding fields. Never invent a reporting cutoff.
6. **Distinguish observed from derived.** A computed CTR, CPC, duration, or difference must identify its inputs and formula. Do not silently replace a visible platform value with a recalculated one.
7. **Do not infer causality.** Paid clicks, map interactions, AI mentions, and review sentiment are not verified sales or incremental lift.
8. **Avoid exposing confidential information.** Public demos must use synthetic or explicitly authorized screenshots and merchant details.
9. **Follow the Intermediate JSON contract.** Use the exact field names defined in `SKILL.md`. Do not create undocumented fields or mix metrics from different modules.
10. **Preserve measurement context.** Record dates, geography, query sets, currency, and source labels whenever they affect interpretation. If no dedicated JSON field exists, explain the limitation in `warnings` rather than inventing data.

## 2. Campaign Performance screenshots

### Identity and reporting context

| Target field | Look for | Rule |
|---|---|---|
| `restaurant.name` | Merchant/business heading | Prefer user-confirmed full name when screenshot is truncated |
| `restaurant.address` | Address in header/details | Leave blank if not visible or supplied |
| `restaurant.city_state` | City and region | Do not infer from the business name |
| `restaurant.start_date` | Campaign start/launch date | Normalize to `YYYY-MM-DD` only when unambiguous |
| `campaign_metrics.reporting_window` | Date picker, reporting filter, caption | Preserve the displayed campaign reporting period; do not confuse it with campaign lifetime |
| `campaign_metrics.campaign_status` | Active, paused, ended, or equivalent label | Preserve the visible status; otherwise use `""` |
| `campaign_metrics.ad_spend` | Spend/cost field | Record the numeric value |
| `campaign_metrics.currency` | Currency symbol, code, or dashboard setting | Store an identifiable currency code or label; otherwise use `""` |

The campaign start date and the selected reporting window are different concepts. A dashboard showing the last 30 days does not establish that the campaign started 30 days ago.

Do not infer the currency from the restaurant's country alone. If a currency symbol is ambiguous, retain the displayed symbol and flag the uncertainty if it materially affects the report.

### Detect KPI mode before mapping cards

- **`standard`:** The **Reach** tab is selected. Extract impressions, clicks, CTR, CPC, and ad spend from visible fields, alongside any visible action metrics. If the interface does not use named tabs, select this mode only when the equivalent Reach KPI set is clearly present.
- **`restructured`:** The **Results** tab is selected. In the original dashboard, the Revenue/ROAS range and ad-spend fields may remain visible regardless of the selected tab; Google Maps action metrics are also present in either tab. The Results view does not supply the Reach metrics. Extract ad spend and all visible action metrics, but **never use Revenue or ROAS ranges** in the report. Do not invent impressions, clicks, CTR, or CPC.
- **Ambiguous:** If the selected view cannot be established and this changes the report layout, set `campaign_metrics.kpi_mode` to `null`, flag it as low confidence, and ask for a clearer screenshot.

The mode is a report-layout choice, not proof that absent metrics equal zero.

Store exactly one of:

- `"standard"`
- `"restructured"`
- `null` when unresolved

Never store `"standard|restructured"` as a final value.

### Campaign metrics

The following fields belong inside `campaign_metrics`.

| JSON field | Possible source label | Value rule |
|---|---|---|
| `impressions` | Impressions | Count |
| `clicks` | Clicks | Count |
| `ctr` | CTR / Click-through rate | Percent number, e.g., `3.2` for 3.2% |
| `cpc` | CPC / Cost per click | Monetary amount in the source currency |
| `orders` | Orders | Count of attributed actions, not independently verified sales |
| `menu_views` | Menu Views | Count |
| `directions` | Directions / Direction Views | Count |
| `phone_calls` | Phone Calls | Count |
| `website_visits` | Website Visits | Count |
| `other_actions` | Other Actions | Count |

A screenshot may display **Total Revenue range** or **ROAS range**. The original workflow explicitly excludes both:

**NEVER use either in the report, including narrative, KPI cards, analysis, or conclusions.**

Store Visits and Cost per Store Visit are **not KPI cards**. They may appear in narrative only with clear source and estimation caveats.

The six supported Google Maps action metrics are:

1. Orders
2. Direction Views
3. Phone Calls
4. Website Visits
5. Menu Views
6. Other Actions

Do not invent JSON keys for excluded metrics without revising `SKILL.md` and the other references together.

### Search Insights: campaign keyword tags

Map campaign keywords directly to:

`campaign_metrics.search_terms`

- Extract **every visible** campaign keyword or search term.
- Preserve the displayed text and order.
- Store the terms as an array of strings.
- Do not infer hidden rows or claim that a cropped list is complete.
- Do not place campaign search terms into `gbp_metrics.search_terms` unless they are actually GBP search terms.
- If keyword data is absent, use `[]` and omit the unsupported Keyword Insights content.

Example structure:

```json
{
  "search_terms": [
    "visible campaign keyword",
    "another visible campaign keyword"
  ]
}
```

These are structural examples, not actual merchant search terms.

## 3. Google Business Profile (GBP) screenshots

| JSON field | Look for | Rule |
|---|---|---|
| `gbp_metrics.total_interactions` | Total / All Interactions | Count for the visible period |
| `gbp_metrics.view` | Weekly / Monthly selection | Store `"weekly"`, `"monthly"`, or `null` |
| `gbp_metrics.chart_type` | Chart legend and visible lines | Store `"dual"`, `"red-only"`, or `null` |
| `gbp_metrics.data_points` | Chart dates and values | Capture visible points and flag estimates |
| `gbp_metrics.series_labels` | Actual chart legend labels | Preserve the visible labels for each series |
| `gbp_metrics.search_terms` | GBP search impressions / terms | Extract visible terms and counts |

### Chart style and data points

- **`dual`:** The source interface shows two series, originally labeled `Before [provider]` and `With [provider]`, often gray and red. Record the actual visible legend text for each; do not interpret them as a causal before/after experiment.
- **`red-only`:** A single red series is visible, originally the `With [provider]` series. This is a layout label, **not** proof of merchant tenure or service relationship.
- If the chart does not fit either style, set `chart_type` to `null`, flag the mismatch, and request clarification if the chart is material to the report.
- Read exact values from tooltips or labels when possible.
- If a value is visually estimated from a line or axis, mark it approximate and add its JSON path to `confidence_flags.approximated_fields`.
- Do not treat pending or incomplete periods as final observations.

### Series labels

Map visible chart legend labels to:

`gbp_metrics.series_labels`

Use:

```json
{
  "series_labels": {
    "series_a": "",
    "series_b": ""
  }
}
```

Rules:

- For a `red-only` chart, use `series_a` for the visible series and leave `series_b` empty.
- For a `dual` chart, map each actual visible legend label consistently to `series_a` and `series_b`.
- Keep the same mapping across all chart data points.
- Do not replace actual labels with assumed service relationships.
- If a label is unreadable, leave it empty and flag the ambiguity when material.

### GBP data point structure

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

For `red-only`, use `series_a` for the one visible series and `series_b: null`; for `dual`, use both.

Keep exact and estimated values distinguishable. Do not fabricate timestamps, missing chart points, or unreadable counts.

### GBP-absent fallback

If no GBP screenshot is supplied, do not invent `gbp_metrics.total_interactions`.

If Local Map visibility is available, the report may substitute a clearly labeled **Local Map Visibility %** card for the original GBP Total KPI card.

If neither is available, omit that card.

## 4. AI Search visibility screenshots (optional)

Set `ai_search.available: true` **only** when relevant AI Search evidence is provided. Otherwise keep it `false` and do not assess AI visibility.

### Core AI Search fields

| JSON field | Look for | Rule |
|---|---|---|
| `ai_search.date_range` | Selected measurement window | Preserve the displayed range |
| `ai_search.visibility_score` | Visibility / presence score | Numeric percentage or source-defined score; preserve its scale |
| `ai_search.sentiment` | Sentiment label | Preserve visible label, not inferred tone |
| `ai_search.top3_ranking` | Top-3 ranking / appearance rate | Numeric percentage only if definition is visible or documented |
| `ai_search.prompts` | Prompt-level results | Read visible prompts and associated metrics |
| `ai_search.sov_your` | Merchant Share of Voice | Numeric percentage if shown |
| `ai_search.sov_competitors` | Competitor Share of Voice | Visible competitor rows only |
| `ai_search.daily_trend` | AI visibility trend chart | Record trend direction and visible dated values |
| `ai_search.ai_origin_traffic` | AI-origin traffic dashboard | Record total page views and visible source breakdown |

### Prompt-level data

Suggested `prompts` item:

```json
{
  "prompt": "visible text",
  "presence_pct": null,
  "sov_pct": null,
  "sentiment": "",
  "query_type": "unknown"
}
```

Allowed `query_type` values:

- `"brand"` — the query explicitly contains the business name or a clear brand reference.
- `"category"` — the query is a generic category or discovery query.
- `"unknown"` — the query type cannot be established.

Do not expand truncated prompt text or infer omitted prompts.

### Competitor Share of Voice

Suggested `sov_competitors` item:

```json
{
  "name": "visible competitor",
  "sov_pct": null
}
```

Extract visible competitors only. Do not assume that the displayed competitor list represents the entire market.

### Daily Trend

Map AI visibility trends to:

`ai_search.daily_trend`

Structure:

```json
{
  "daily_trend": {
    "direction": null,
    "data_points": []
  }
}
```

Allowed `direction` values:

- `"rising"`
- `"flat"`
- `"declining"`
- `null`

Extraction rules:

- Determine direction from the visible trend over the displayed measurement period.
- Do not describe normal short-term fluctuations as a sustained trend without sufficient evidence.
- Extract exact dated values when labels or tooltips are available.
- If chart values are visually estimated, mark `estimated: true` and add the corresponding JSON paths to `confidence_flags.approximated_fields`.
- If the chart does not support a reliable direction, use `null`.

Suggested data point structure:

```json
{
  "date": "YYYY-MM-DD",
  "value": null,
  "estimated": false
}
```

Do not fabricate daily values to create a smooth line or fill missing dates.

### AI-Origin Traffic

Map AI-origin traffic to:

`ai_search.ai_origin_traffic`

Structure:

```json
{
  "ai_origin_traffic": {
    "total_page_views": null,
    "sources": []
  }
}
```

Suggested source item:

```json
{
  "source": "visible source label",
  "page_views": null
}
```

Rules:

- Extract total AI-origin page views only when visible.
- Preserve the displayed source labels.
- Record visible source-level page views without inventing missing values.
- Do not assume the listed sources sum to the total if the screenshot is cropped or incomplete.
- Do not equate AI-origin page views with conversions or sales.
- If the source dashboard uses a different measurement definition, preserve that distinction in `warnings`.

### AI visibility interpretation constraints

AI visibility results are **sample-dependent**. Queries, models, geography, dates, and measurement methods can change the outcome.

Do not label a visibility score as a universal probability of recommendation unless the source explicitly defines it that way.

Do not claim that an observed AI mention caused website traffic, restaurant visits, or sales without supporting attribution evidence.

## 5. Local Map Search screenshots (optional)

Set `local_map.available: true` only when a relevant local-ranking or map-visibility screenshot is supplied.

### Core Local Map fields

| JSON field | Look for | Rule |
|---|---|---|
| `local_map.avg_rank` | Average Rank | Numeric value; lower typically means better ranking when defined that way |
| `local_map.visibility_pct` | Visibility % | Percentage number, preserving source definition |
| `local_map.keywords` | Keyword-level ranking table | Visible rows only |
| `local_map.competitors` | Competitor comparison table | Visible rows only; distinguish merchant's own row |
| `local_map.measurement_context` | Query set, location/grid, measurement period | Preserve the visible context for interpreting rankings |

### Keyword-level data

Suggested `keywords` item:

```json
{
  "keyword": "visible term",
  "avg_rank": null,
  "visibility_pct": null
}
```

Extract only visible keyword rows. Do not infer ranking values for missing keywords.

### Competitor comparison data

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

- Use a visible `You` or merchant indicator to set `is_self: true`; do not rely on row order.
- If the merchant's own row cannot be identified confidently, flag the ambiguity.
- Ratings and review counts belong to the corresponding visible competitor row. Do not assign a competitor's values to the merchant.
- A listed competitor is a **comparison candidate**, not necessarily a direct substitute.

### Measurement context

Map the measurement context to:

`local_map.measurement_context`

Structure:

```json
{
  "measurement_context": {
    "query_set": [],
    "location": "",
    "date_range": ""
  }
}
```

Rules:

- `query_set`: Preserve visible search queries or keywords used in the map measurement. Use an array of strings.
- `location`: Preserve the displayed geographic context, such as a city, neighborhood, or grid description.
- `date_range`: Preserve the visible measurement period.
- If a context field is not supplied, leave it empty rather than inferring it.
- Do not treat rankings from different locations, query sets, or dates as directly comparable without checking their measurement definitions.

### Ranking color conventions

The original dashboard used the following average-rank color guide:

- **Green:** Top 3
- **Yellow:** Positions 4–10
- **Red:** Positions 10+

Preserve this mapping only when the displayed legend or source documentation confirms it.

The original boundary at rank 10 is ambiguous across the latter two bands. Prefer numeric rank values when visible.

Do not infer rankings from colors in an unrelated dashboard.

## 6. Customer Reviews screenshots (optional)

Set `review_data.available: true` only when a review-summary or review-tag screenshot is supplied.

### Review fields

| JSON field | Source | Rule |
|---|---|---|
| `review_data.tags_positive` | Positive tags | Transcribe visible tag text |
| `review_data.tags_neutral` | Neutral tags | Transcribe visible tag text |
| `review_data.tags_negative` | Negative tags | Transcribe visible tag text |
| `review_data.summary_bullets` | Review summary | Preserve visible claims; flag truncation |
| `review_data.review_scope` | Review-summary heading or filter | Preserve the stated sample scope |

### Review tag conventions

The original tag styling was:

- **Positive:** Green / ↗ / `tag-pos`
- **Neutral:** Gray-white / — / `tag-neu`
- **Negative:** Pink-red / ↘ / `tag-neg`

Preserve these generic CSS-class conventions if the HTML template uses them.

Classify screenshots by explicit category labels or a verified legend rather than color alone.

Keep tags as strings, such as `"friendly staff"`. Do not fabricate tag frequencies, sentiment shares, or sample sizes.

### Review scope

Map the displayed review-summary scope to:

`review_data.review_scope`

Examples of source labels include:

- `"Last 50 reviews"`
- `"Last 30 days"`
- `"Selected review period"`

These are examples of possible labels, not assumed values.

Rules:

- Preserve the actual scope shown in the screenshot.
- The original panel was labeled **“Summary of the last 50 reviews.”** Use that scope only when it is visible or otherwise confirmed.
- Do not assume that a sample of 50 reviews represents all reviews.
- Do not infer the number of reviews from the number of displayed summary bullets or tags.
- If no scope is shown, use `""`.
- Do not infer who owns or manages review replies unless supplied by the user.

## 7. Campaign duration and cross-screenshot reconciliation

Calculate `restaurant.duration_days` only when **both** campaign start date and a valid campaign data cutoff are known.

```text
Duration (days) = campaign reporting cutoff date - campaign start date
```

### Reporting-window reconciliation

Use the following fields to check reporting context:

- `restaurant.start_date`
- `campaign_metrics.reporting_window`
- `ai_search.date_range`
- `local_map.measurement_context.date_range`
- GBP chart dates from `gbp_metrics.data_points`

Rules:

1. The campaign reporting window is not necessarily the campaign lifetime.
2. Do not substitute the latest GBP chart date for the campaign cutoff without evidence that the reporting periods align.
3. If the campaign reporting window clearly provides a valid cutoff, use that date for the duration calculation.
4. If the reporting window is ambiguous, incomplete, or does not establish a valid cutoff, leave `duration_days: null`.
5. If the cutoff is material to the report, ask the user for clarification.
6. Do not use AI Search or Local Map dates as campaign cutoff dates without explicit supporting evidence.

The original workflow used the **latest non-pending GBP data point** as an approximate cutoff. This is permitted **only if the reporting periods demonstrably align**.

### Duration display

Preserve the original narrative display convention:

- Fewer than 60 days: `约 X 天`
- 60 days or more: `约 X 个月`

For months, explain that the figure is approximate.

If the duration is derived from approximate evidence, add `restaurant.duration_days` to `confidence_flags.approximated_fields`.

### Cross-screenshot consistency

- In Update mode, compare values only when definitions, units, attribution windows, and reporting windows are compatible.
- A cumulative total increasing is not necessarily improved performance.
- If screenshots disagree on the merchant, date range, or units, stop and resolve the conflict before combining them.
- Do not combine AI visibility, paid advertising, GBP, and Local Map values into a single numeric performance score unless a separately documented and justified method exists.
- Treat each module's measurement scope as distinct unless the source establishes comparability.

## 8. Confidence scoring and JSON mapping

| Level | Definition | Required handling |
|---|---|---|
| **HIGH** | Clearly visible value, unambiguous label and unit | Extract directly |
| **MEDIUM** | Legible but approximate chart reading, or a noncritical ambiguity | Mark `~` / `约`; add to `approximated_fields` |
| **LOW** | Obscured, truncated, conflicting, or materially inferred | Add to `low_confidence_fields`; ask if material |

Use stable JSON paths in `confidence_flags`.

Examples:

- `campaign_metrics.ad_spend`
- `campaign_metrics.currency`
- `campaign_metrics.kpi_mode`
- `gbp_metrics.data_points[0].series_a`
- `gbp_metrics.series_labels.series_a`
- `ai_search.daily_trend.data_points[0].value`
- `ai_search.ai_origin_traffic.total_page_views`
- `local_map.avg_rank`
- `review_data.review_scope`

Add a short reason in `warnings` when a value is uncertain or a measurement limitation affects interpretation.

### Final consistency checks

- Every number maps to a visible source or an explicitly documented formula.
- Each optional module's `available` flag matches the supplied evidence.
- `kpi_mode`, `chart_type`, `view`, and `language` contain one selected value or `null`, not a pipe-separated list of alternatives.
- Missing values remain missing; never turn `null` into `0` to fill KPI cards.
- All arrays use consistent item shapes and percentage units.
- Campaign keywords map to `campaign_metrics.search_terms`, not GBP search terms.
- GBP legend names map to `gbp_metrics.series_labels`.
- AI Daily Trend maps to `ai_search.daily_trend`.
- AI-origin traffic maps to `ai_search.ai_origin_traffic`.
- Local Map query, location, and date context map to `local_map.measurement_context`.
- Review-summary scope maps to `review_data.review_scope`.
- Reporting windows are preserved and are not silently treated as campaign lifetime.
- Revenue and ROAS ranges are excluded from all report content.
- No unsupported metric, competitor claim, or causal conclusion is introduced.
- Do not produce analysis or HTML until Step 1.5 in `SKILL.md` passes.

### Data contract compatibility

This extraction schema is aligned with the **Intermediate JSON v0.2** defined in `SKILL.md`.

The following fields now have explicit destinations:

| Extracted information | Intermediate JSON field |
|---|---|
| Campaign reporting period | `campaign_metrics.reporting_window` |
| Campaign status | `campaign_metrics.campaign_status` |
| Campaign currency | `campaign_metrics.currency` |
| Campaign Search Insights | `campaign_metrics.search_terms` |
| GBP chart legend names | `gbp_metrics.series_labels` |
| AI Daily Trend | `ai_search.daily_trend` |
| AI-origin traffic | `ai_search.ai_origin_traffic` |
| Local Map measurement context | `local_map.measurement_context` |
| Review-summary sample scope | `review_data.review_scope` |

Some source details still require care:

- Merchant-level star ratings and review counts do not have separate top-level merchant fields in the current Intermediate JSON. They may be represented within a visible Local Map competitor row when applicable, but must not be silently assigned to the merchant.
- The campaign cutoff date does not have a dedicated standalone field. Use the visible reporting window to establish a cutoff only when unambiguous; otherwise leave duration unknown.
- Source-specific measurement definitions and unusual labels may require `warnings` or a future coordinated schema revision.

Do not add new fields independently to this document without updating the corresponding data contract in `SKILL.md`.

The remaining references, `analysis-rules.md` and `html-template.md`, must consume these same field names before the public skill can be considered end-to-end validated.

