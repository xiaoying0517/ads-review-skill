
# HTML Report Template Reference

A presentation and rendering reference for generating a single-file HTML report from the Intermediate JSON defined in `SKILL.md`.

Read `SKILL.md`, `references/extraction-schema.md`, and `references/analysis-rules.md` before using this template.

This is a **public, platform-neutral reference**. All merchant names, measurements, dates, and narrative findings must come from authorized inputs or clearly labeled synthetic test data.

## 1. Rendering contract

Every final report must be a self-contained `.html` file.

Requirements:

1. Include all CSS inside a `<style>` element.
2. Do not require external CSS, JavaScript, fonts, images, or chart libraries.
3. Use semantic HTML wherever practical.
4. Generate chart geometry from extracted data rather than hardcoded example values.
5. Escape merchant names, search terms, review text, and other source-provided strings before inserting them into HTML.
6. Render missing values as an em dash (`—`) or omit the affected component. Never display a missing value as zero.
7. Render numeric zero only when the source explicitly supports zero.
8. Use the currency specified in `campaign_metrics.currency`. Do not assume USD.
9. Do not show estimated Revenue or ROAS ranges anywhere.
10. Do not place Store Visits or Cost per Store Visit in KPI cards.
11. Do not imply causality between advertising, organic rankings, AI visibility, and customer reviews.
12. Do not expose private service pricing, proprietary benchmarks, internal responsibilities, or unapproved merchant information.
13. Respect the selected `language`: `"chinese"` or `"english"`.
14. Include optional modules only when supported by the supplied evidence.
15. Do not invent chart points, review counts, competitors, reporting periods, or business outcomes.

The examples below use `{{field_path}}` placeholders to explain data binding. They are **not literal values** and must be resolved or omitted before delivering the final HTML.

The generated report must contain no unresolved placeholders.

---

## 2. Data-to-presentation mapping

| Report element | Intermediate JSON source | Display rule |
|---|---|---|
| Restaurant title | `restaurant.name` | Escape text; do not invent |
| Address | `restaurant.address` | Omit if unknown |
| City / region | `restaurant.city_state` | Omit if unknown |
| Campaign start | `restaurant.start_date` | Display only if known |
| Approximate duration | `restaurant.duration_days` | Display only if valid |
| Reporting period | `campaign_metrics.reporting_window` | Preserve source scope |
| Campaign status | `campaign_metrics.campaign_status` | Use observed status only |
| Ad spend | `campaign_metrics.ad_spend` | Format with source currency |
| Currency | `campaign_metrics.currency` | Do not silently default to `$` |
| KPI layout | `campaign_metrics.kpi_mode` | Standard or restructured |
| Campaign search terms | `campaign_metrics.search_terms` | Array of strings |
| GBP total interactions | `gbp_metrics.total_interactions` | Display if supplied |
| GBP chart mode | `gbp_metrics.chart_type` | Dual or red-only |
| GBP chart period | `gbp_metrics.view` | Weekly or monthly |
| GBP chart points | `gbp_metrics.data_points` | Exact/estimated distinction |
| GBP chart labels | `gbp_metrics.series_labels` | Use actual visible labels |
| GBP search terms | `gbp_metrics.search_terms` | Term/count records |
| AI visibility | `ai_search.visibility_score` | Respect source-defined scale |
| AI sentiment | `ai_search.sentiment` | Preserve visible classification |
| AI Top-3 metric | `ai_search.top3_ranking` | Respect source definition |
| AI prompts | `ai_search.prompts` | Prompt-level metrics |
| AI Share of Voice | `ai_search.sov_your`, `ai_search.sov_competitors` | Same measurement set |
| AI Daily Trend | `ai_search.daily_trend` | Direction and dated points |
| AI-origin traffic | `ai_search.ai_origin_traffic` | Page views and sources |
| Local Map ranking | `local_map.avg_rank` | Source-defined ranking |
| Local Map visibility | `local_map.visibility_pct` | Source-defined percentage |
| Local Map keywords | `local_map.keywords` | Visible rows only |
| Local Map competitors | `local_map.competitors` | Include only supplied rows |
| Local Map context | `local_map.measurement_context` | Query, geography, period |
| Review tags | `review_data.tags_positive`, `tags_neutral`, `tags_negative` | Visible strings only |
| Review summary | `review_data.summary_bullets` | Source-supported themes |
| Review scope | `review_data.review_scope` | Preserve sample description |
| Highlights | `insights`, `diagnostic_summary` | Evidence-supported only |
| Warnings | `warnings`, `diagnostic_summary.priority_issue` | Do not force a P0 |
| Recommendation | `diagnostic_summary.recommended_action` | Actionable and supported |

All report sections must use these field names consistently.

---

## 3. Full CSS block

Use the following as the base CSS for the generated HTML.

For Chinese reports, retain the Chinese font stack.

For English reports, use:

`-apple-system, BlinkMacSystemFont, 'Segoe UI', Arial, sans-serif`

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'PingFang SC', 'Microsoft YaHei',
    -apple-system, sans-serif;
  font-size: 13px;
  color: #1a1a1a;
  background: #ffffff;
  padding: 40px 48px;
  line-height: 1.75;
}

.report-header {
  text-align: center;
  margin-bottom: 28px;
}

.report-header h1 {
  font-size: 26px;
  font-weight: 700;
  color: #1a1a2e;
  margin-bottom: 4px;
}

.report-header .subtitle {
  font-size: 14px;
  color: #E91E8C;
  font-weight: 600;
  margin-bottom: 4px;
}

.report-header .date-range {
  font-size: 15px;
  font-weight: 700;
  color: #1a1a2e;
  margin-bottom: 10px;
}

.report-header .meta-bar {
  background: #f3f4f6;
  border-radius: 6px;
  padding: 7px 16px;
  display: inline-block;
  font-size: 11px;
  color: #6b7280;
}

.report-header .meta-bar span {
  margin: 0 10px;
}

.status-active {
  display: inline-block;
  background: #f0fdf4;
  color: #166534;
  border: 1px solid #86efac;
  border-radius: 4px;
  font-size: 11px;
  font-weight: 700;
  padding: 2px 8px;
  margin-left: 8px;
}

.status-neutral {
  display: inline-block;
  background: #f3f4f6;
  color: #4b5563;
  border: 1px solid #d1d5db;
  border-radius: 4px;
  font-size: 11px;
  font-weight: 600;
  padding: 2px 8px;
  margin-left: 8px;
}

.sec-header {
  background: #fce4ec;
  border-radius: 6px;
  padding: 8px 14px;
  margin: 22px 0 12px;
  font-size: 13.5px;
  font-weight: 700;
  color: #1a1a2e;
}

p {
  margin-bottom: 8px;
  color: #374151;
}

.kpi-grid {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 10px;
  margin-bottom: 10px;
}

.kpi-card {
  background: #ffffff;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  padding: 12px 14px;
  min-width: 0;
}

.kpi-card .kpi-label {
  font-size: 10px;
  color: #6b7280;
  margin-bottom: 2px;
}

.kpi-card .kpi-val {
  font-size: 22px;
  font-weight: 700;
  color: #E91E8C;
  line-height: 1.2;
  overflow-wrap: anywhere;
}

.kpi-card .kpi-sub {
  font-size: 10px;
  color: #6b7280;
  margin-top: 4px;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 10px;
  font-size: 12px;
}

thead tr th {
  background: #E91E8C;
  color: #ffffff;
  font-weight: 600;
  padding: 7px 10px;
  text-align: left;
}

tbody tr td {
  padding: 7px 10px;
  border-bottom: 1px solid #f3f4f6;
  color: #374151;
  vertical-align: top;
  overflow-wrap: anywhere;
}

tbody tr:last-child td {
  border-bottom: none;
}

tbody tr.sum-row td {
  font-weight: 700;
  background: #f9fafb;
}

.highlight {
  background: #fdf0f7;
  border-left: 3px solid #E91E8C;
  border-radius: 0 6px 6px 0;
  padding: 10px 14px;
  margin: 10px 0;
  font-size: 12px;
  color: #374151;
  line-height: 1.7;
}

.highlight strong {
  color: #1a1a2e;
}

.warning {
  background: #fff7ed;
  border-left: 3px solid #f97316;
  border-radius: 0 6px 6px 0;
  padding: 10px 14px;
  margin: 10px 0;
  font-size: 12px;
  color: #374151;
  line-height: 1.7;
}

.warning strong {
  color: #c2410c;
}

.two-col-box {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 14px;
  margin-bottom: 6px;
}

.box-good {
  background: #f0fdf4;
  border: 1px solid #86efac;
  border-radius: 8px;
  padding: 12px 14px;
}

.box-attn {
  background: #fafafa;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  padding: 12px 14px;
}

.box-title {
  font-weight: 700;
  font-size: 12px;
  margin-bottom: 8px;
}

.box-title.good {
  color: #166534;
}

.box-title.attn {
  color: #374151;
}

ul {
  padding-left: 16px;
}

ul li {
  font-size: 11.5px;
  color: #4b5563;
  margin-bottom: 4px;
  line-height: 1.55;
}

.tag-row {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin: 6px 0;
}

.tag {
  display: inline-block;
  border-radius: 20px;
  padding: 3px 10px;
  font-size: 10.5px;
  font-weight: 500;
}

.tag-pos {
  background: #f0fdf4;
  color: #166534;
  border: 1px solid #86efac;
}

.tag-neu {
  background: #f9fafb;
  color: #6b7280;
  border: 1px solid #e5e7eb;
}

.tag-neg {
  background: #fff0f0;
  color: #b91c1c;
  border: 1px solid #fca5a5;
}

.ai-score-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 10px;
  margin-bottom: 14px;
}

.ai-score-card {
  background: #f8faff;
  border: 1px solid #dbeafe;
  border-radius: 8px;
  padding: 12px 14px;
  text-align: center;
}

.ai-score-card .ai-val {
  font-size: 24px;
  font-weight: 700;
  color: #1d4ed8;
  line-height: 1.2;
}

.ai-score-card .ai-label {
  font-size: 10px;
  color: #6b7280;
  margin-top: 3px;
}

.presence-bar-wrap {
  display: flex;
  align-items: center;
  gap: 8px;
}

.presence-bar-bg {
  flex: 1;
  background: #f3f4f6;
  border-radius: 4px;
  height: 7px;
  overflow: hidden;
}

.presence-bar-fill {
  background: #2563eb;
  border-radius: 4px;
  height: 7px;
}

.sov-row {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 6px 0;
  border-bottom: 1px solid #f3f4f6;
}

.sov-row:last-child {
  border-bottom: none;
}

.sov-name {
  font-size: 12px;
  color: #374151;
  width: 180px;
  flex-shrink: 0;
  overflow-wrap: anywhere;
}

.sov-bar-bg {
  flex: 1;
  background: #f3f4f6;
  border-radius: 4px;
  height: 8px;
  overflow: hidden;
}

.sov-bar-fill {
  background: #9ca3af;
  border-radius: 4px;
  height: 8px;
}

.sov-pct {
  font-size: 12px;
  font-weight: 700;
  width: 52px;
  text-align: right;
}

.rank-badge {
  display: inline-block;
  padding: 2px 9px;
  border-radius: 10px;
  font-size: 11px;
  font-weight: 700;
}

.rank-good {
  background: #dcfce7;
  color: #15803d;
}

.rank-mid {
  background: #fef9c3;
  color: #854d0e;
}

.rank-bad {
  background: #fee2e2;
  color: #dc2626;
}

.rank-neutral {
  background: #f3f4f6;
  color: #374151;
}

.context-note {
  font-size: 10.5px;
  color: #6b7280;
  margin: 6px 0 10px;
}

.chart-container {
  margin: 12px 0;
  overflow-x: auto;
}

.chart-container svg {
  width: 100%;
  height: auto;
  display: block;
}

.footer {
  margin-top: 30px;
  padding-top: 10px;
  border-top: 1px solid #e5e7eb;
  font-size: 10px;
  color: #9ca3af;
  text-align: center;
}

@media (max-width: 760px) {
  body {
    padding: 20px 16px;
  }

  .kpi-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .ai-score-grid {
    grid-template-columns: 1fr;
  }

  .two-col-box {
    grid-template-columns: 1fr;
  }

  .sov-name {
    width: 110px;
  }
}

@media print {
  body {
    padding: 20px 28px;
  }

  .kpi-card,
  .ai-score-card,
  .highlight,
  .warning,
  .box-good,
  .box-attn {
    break-inside: avoid;
  }

  .sec-header {
    break-after: avoid;
  }
}
```

The CSS classes preserve the original report's general visual identity while removing proprietary labels and unsupported benchmark-based color rules.

---

## 4. Report structure and section numbering

Use this logical sequence:

1. Executive Summary
2. KPI Overview
3. Engagement Actions
4. Keyword Insights
5. GBP Trends
6. AI Search Visibility, if available
7. Local Map Search, if available
8. Customer Reviews, if available
9. Highlights & Watch Points
10. Conclusion & Next Actions

Section numbers are **display numbers**, not fixed module IDs.

Generate consecutive section numbers after optional sections are included or omitted.

For example, if AI Search and Local Map are unavailable, do not leave unexplained gaps in section numbering.

Do not generate empty sections merely to preserve a fixed count.

---

## 5. Report header

Bind the header to:

- `restaurant.name`
- `restaurant.city_state`
- `restaurant.address`
- `restaurant.start_date`
- `restaurant.duration_days`
- `campaign_metrics.reporting_window`
- `campaign_metrics.campaign_status`

Illustrative structure:

```html
<div class="report-header">
  <h1>{{restaurant.name}}</h1>

  <div class="subtitle">
    Advertising &amp; Local Visibility Report
  </div>

  <div class="date-range">
    {{campaign_start_display}}
    {{optional_status_badge}}
  </div>

  <div class="meta-bar">
    {{optional_location}}
    {{optional_reporting_window}}
    {{optional_duration}}
    {{report_generation_date}}
  </div>
</div>
```

Rendering rules:

- Escape all user-provided text.
- Display city and region only when supplied.
- Show the campaign status badge only when status is visible and unambiguous.
- Do not assume a campaign is active.
- Use `status-active` only for a confirmed active state.
- Use `status-neutral` for other confirmed states.
- Do not display an invented campaign launch date.
- Do not substitute report generation date for campaign reporting cutoff.
- Do not display approximate duration when `restaurant.duration_days` is `null`.
- Keep campaign reporting period distinct from campaign lifetime.

For Chinese reports, translate the title and labels consistently.

---

## 6. Section header

Use:

```html
<div class="sec-header">
  {{section_number}} &nbsp;{{section_title}}
</div>
```

Generate the section number dynamically after determining which optional modules are included.

---

## 7. KPI Grid — Standard

Condition:

`campaign_metrics.kpi_mode = "standard"`

Preferred layout:

```text
Row 1:
Ad Spend | Impressions | Clicks | Online Orders

Row 2:
Menu Views | Direction Requests | CPC | Website Visits
```

Data mapping:

| Card | JSON source |
|---|---|
| Ad Spend | `campaign_metrics.ad_spend` |
| Impressions | `campaign_metrics.impressions` |
| Clicks | `campaign_metrics.clicks` |
| CTR subtitle | `campaign_metrics.ctr` |
| Online Orders | `campaign_metrics.orders` |
| Menu Views | `campaign_metrics.menu_views` |
| Direction Requests | `campaign_metrics.directions` |
| CPC | `campaign_metrics.cpc` |
| Website Visits | `campaign_metrics.website_visits` |
| Optional Phone Calls | `campaign_metrics.phone_calls` |

Card structure:

```html
<div class="kpi-grid">
  <div class="kpi-card">
    <div class="kpi-label">广告支出</div>
    <div class="kpi-val">{{formatted_ad_spend}}</div>
    <div class="kpi-sub">{{spend_reporting_context}}</div>
  </div>

  <div class="kpi-card">
    <div class="kpi-label">展示次数</div>
    <div class="kpi-val">{{formatted_impressions}}</div>
    <div class="kpi-sub">{{impressions_context}}</div>
  </div>

  <div class="kpi-card">
    <div class="kpi-label">点击次数</div>
    <div class="kpi-val">{{formatted_clicks}}</div>
    <div class="kpi-sub">{{ctr_if_available}}</div>
  </div>

  <div class="kpi-card">
    <div class="kpi-label">归因订单数</div>
    <div class="kpi-val">{{formatted_orders}}</div>
    <div class="kpi-sub">平台归因数据</div>
  </div>
</div>
```

Generate the second row using the same `.kpi-card` structure.

Rules:

- Use the original two-row, four-column design when all eight metrics are available.
- Do not force eight cards when some metrics are missing.
- Phone Calls may replace Website Visits according to `analysis-rules.md`.
- Do not describe attributed orders as independently confirmed revenue.
- Do not include internal CPC benchmark subtitles.
- Format monetary values using `campaign_metrics.currency`.
- If the currency is unknown, do not silently insert `$`.
- If the selected KPI mode is unresolved, do not guess a layout.

---

## 8. KPI Grid — Restructured

Condition:

`campaign_metrics.kpi_mode = "restructured"`

Preferred layout:

```text
Row 1:
Ad Spend | Online Orders | Menu Views | Direction Requests

Row 2:
Phone Calls | Website Visits | Other Actions | GBP Total Interactions
```

Data mapping:

| Card | JSON source |
|---|---|
| Ad Spend | `campaign_metrics.ad_spend` |
| Online Orders | `campaign_metrics.orders` |
| Menu Views | `campaign_metrics.menu_views` |
| Direction Requests | `campaign_metrics.directions` |
| Phone Calls | `campaign_metrics.phone_calls` |
| Website Visits | `campaign_metrics.website_visits` |
| Other Actions | `campaign_metrics.other_actions` |
| GBP Total Interactions | `gbp_metrics.total_interactions` |

Use the same `.kpi-grid` and `.kpi-card` components defined above.

Rules:

- Do not add Impressions, Clicks, CTR, or CPC when the Results view does not supply them.
- Do not use Revenue or ROAS ranges.
- Do not display missing metrics as zero.
- Do not describe Other Actions as a specific action type unless the source provides that breakdown.

### GBP-absent fallback

If `gbp_metrics.total_interactions` is unavailable and `local_map.visibility_pct` is supported, substitute:

```text
Label: Local Map Visibility
Value: local_map.visibility_pct
Subtitle: Local Map Search · measured period
```

Read the measured period from:

`local_map.measurement_context.date_range`

If Local Map visibility is also unavailable, omit the card.

Never hardcode a seven-day period without source evidence.

---

## 9. Engagement actions table

This section summarizes visible customer actions.

Use:

- `campaign_metrics.orders`
- `campaign_metrics.menu_views`
- `campaign_metrics.directions`
- `campaign_metrics.phone_calls`
- `campaign_metrics.website_visits`
- `campaign_metrics.other_actions`

Structure:

```html
<table>
  <thead>
    <tr>
      <th>行为类型</th>
      <th>数量</th>
      <th>日均</th>
      <th>说明</th>
    </tr>
  </thead>

  <tbody>
    {{engagement_action_rows}}
  </tbody>
</table>
```

Each row must use a supported action label and value.

Daily average rule:

```text
Daily average = action count / applicable reporting days
```

Calculate a daily average only when the action's reporting window and denominator are compatible.

`restaurant.duration_days` must not be used automatically when the displayed action metric covers a shorter selected period.

If the denominator is unavailable, render `—` or omit the daily-average column.

Do not treat this table as a verified customer conversion funnel.

Action categories may overlap or represent different user behaviors. Do not calculate a combined unique-customer total unless the source explicitly supports it.

Do not sum categories into a "core interactions" total when overlap or metric definitions are unresolved.

---

## 10. Highlight and warning boxes

Use the original pink highlight and orange warning visual conventions.

```html
<div class="highlight">
  <strong>{{highlight_title}}</strong>
  {{evidence_based_finding}}
</div>
```

```html
<div class="warning">
  <strong>{{priority_issue_title}}</strong>
  {{supported_issue_and_action}}
</div>
```

Rules:

- Every highlight must be supported by extracted evidence.
- Do not invent favorable comparisons or industry benchmarks.
- Use at most one primary P0 warning box.
- Omit the P0 warning box when no P0 issue is supported.
- Do not interpret missing module data as a warning.
- Do not render raw JSON strings without HTML escaping.

Use `diagnostic_summary.priority_issue` and `diagnostic_summary.recommended_action` for the primary issue and next action when supported.

---

## 11. Keyword Insights tables

Keep Campaign Search Insights and GBP Search Insights separate.

### Campaign search terms

Source:

`campaign_metrics.search_terms`

This is an array of strings.

```html
<p style="font-weight:600;font-size:12px;margin-bottom:6px;">
  广告搜索词
</p>

<table>
  <thead>
    <tr>
      <th>类别</th>
      <th>关键词</th>
      <th>说明</th>
    </tr>
  </thead>

  <tbody>
    {{campaign_search_term_rows}}
  </tbody>
</table>
```

Classify terms as brand, category, competitor-related, or unknown only when evidence supports the classification.

Do not invent keyword impression counts for campaign keyword tags.

### GBP search terms

Source:

`gbp_metrics.search_terms`

Suggested item structure:

```json
{
  "term": "visible term",
  "count": null
}
```

```html
<p style="font-weight:600;font-size:12px;margin-bottom:6px;">
  GBP 搜索词
</p>

<table>
  <thead>
    <tr>
      <th>搜索词</th>
      <th>展示次数</th>
      <th>说明</th>
    </tr>
  </thead>

  <tbody>
    {{gbp_search_term_rows}}
  </tbody>
</table>
```

If `count` is missing, display `—` rather than inventing a value.

If either list is empty, omit its unsupported table.

Do not mix campaign search terms with GBP search impressions.

---

## 12. GBP SVG chart

Source fields:

- `gbp_metrics.chart_type`
- `gbp_metrics.view`
- `gbp_metrics.data_points`
- `gbp_metrics.series_labels`
- `restaurant.start_date`

Use a self-contained SVG.

Recommended viewBox:

`0 0 680 200`

Recommended chart area:

```text
Left:   x = 54
Right:  x = 634
Top:    y = 10
Bottom: y = 155
```

### 12.1 Dual-series chart

Condition:

`gbp_metrics.chart_type = "dual"`

Use two line series, with neutral gray and accent pink.

The chart must use the actual legend labels from:

- `gbp_metrics.series_labels.series_a`
- `gbp_metrics.series_labels.series_b`

Basic structure:

```html
<div class="chart-container">
  <svg
    viewBox="0 0 680 200"
    xmlns="http://www.w3.org/2000/svg"
    role="img"
    aria-label="{{accessible_chart_description}}"
  >
    <line
      x1="54" y1="10"
      x2="54" y2="155"
      stroke="#e5e7eb"
    />

    <line
      x1="54" y1="155"
      x2="634" y2="155"
      stroke="#e5e7eb"
    />

    <polyline
      points="{{series_a_points}}"
      fill="none"
      stroke="#9ca3af"
      stroke-width="2"
      stroke-linejoin="round"
    />

    <polyline
      points="{{series_b_points}}"
      fill="none"
      stroke="#E91E8C"
      stroke-width="2.5"
      stroke-linejoin="round"
    />

    {{generated_axis_labels}}
    {{generated_point_markers}}
    {{generated_series_legend}}
  </svg>
</div>
```

Rules:

- Generate coordinates from actual visible data.
- Do not hardcode the legend as "before service" and "after service."
- Do not interpret the dual chart as a causal experiment.
- Use the actual source labels.
- Do not automatically place a campaign launch marker.
- A launch marker may be included only when its date is known, falls within the plotted time range, and is relevant to the source's timeline.
- A launch marker must not imply that subsequent changes were caused by the campaign.

### 12.2 Red-only chart

Condition:

`gbp_metrics.chart_type = "red-only"`

Render only `series_a`, using the accent pink line.

Use:

`gbp_metrics.series_labels.series_a`

for the legend.

Do not assume the merchant is an existing or long-term customer.

Do not label the entire chart as a period "after service" unless explicitly supported by the source.

### 12.3 Pending and estimated points

Suggested point structure:

```json
{
  "date": "YYYY-MM-DD",
  "series_a": null,
  "series_b": null,
  "estimated": false,
  "pending": false
}
```

Rules:

- Exclude pending points from definitive trend conclusions.
- Visually distinguish pending or estimated observations when displayed.
- Use `~` or an appropriate note for estimated values.
- Do not silently interpolate missing observations.
- Do not fabricate a continuous line across unknown gaps.
- Do not display unresolved chart labels as factual descriptions.

### 12.4 SVG coordinate math

For a chart with a nonnegative value scale:

```text
chart_top = 10
chart_bottom = 155
chart_height = 145

y = chart_bottom - (value / max_value * chart_height)
```

Choose a positive `max_value` that is at least as large as the largest visible plotted value.

If all visible values are zero, use a positive display scale rather than dividing by zero.

For equally spaced observations:

```text
x = left + index * (right - left) / (point_count - 1)
```

When only one point exists, place it at a defined position without dividing by zero.

Use actual date spacing if the chart is intended to represent elapsed time rather than evenly spaced categories.

For weekly charts, use the dates in `gbp_metrics.data_points`.

For monthly charts, use the source's monthly observations.

Do not assume a fixed number of weeks or months.

---

## 13. AI Search section

Include this section only when:

`ai_search.available = true`

Use:

- `ai_search.date_range`
- `ai_search.visibility_score`
- `ai_search.sentiment`
- `ai_search.top3_ranking`
- `ai_search.prompts`
- `ai_search.sov_your`
- `ai_search.sov_competitors`
- `ai_search.daily_trend`
- `ai_search.ai_origin_traffic`

### 13.1 AI scorecards

Keep the original three-column scorecard design.

```html
<div class="ai-score-grid">
  <div class="ai-score-card">
    <div class="ai-val">
      {{visibility_score_display}}
    </div>
    <div class="ai-label">
      AI Visibility Score
    </div>
  </div>

  <div class="ai-score-card">
    <div class="ai-val">
      {{sentiment_display}}
    </div>
    <div class="ai-label">
      AI Sentiment
    </div>
  </div>

  <div class="ai-score-card">
    <div class="ai-val">
      {{top3_ranking_display}}
    </div>
    <div class="ai-label">
      Top-3 Ranking Metric
    </div>
  </div>
</div>
```

Rules:

- Use the actual score scale and definition.
- Do not describe Visibility Score as a universal recommendation probability.
- Do not invent sentiment when absent.
- Do not interpret Top-3 Ranking as a universal search ranking.
- If an individual score is missing, omit its card or display `—` with clear context.
- Show the source measurement period when available.

### 13.2 Prompt table with presence bars

Source:

`ai_search.prompts`

Suggested prompt item:

```json
{
  "prompt": "visible prompt",
  "presence_pct": null,
  "sov_pct": null,
  "sentiment": "",
  "query_type": "unknown"
}
```

```html
<table>
  <thead>
    <tr>
      <th>问题类型</th>
      <th>Prompt</th>
      <th>出现率</th>
      <th>AI SoV</th>
    </tr>
  </thead>

  <tbody>
    {{ai_prompt_rows}}
  </tbody>
</table>
```

Presence bar structure:

```html
<div class="presence-bar-wrap">
  <div class="presence-bar-bg">
    <div
      class="presence-bar-fill"
      style="width:{{validated_presence_width}}%;"
    ></div>
  </div>

  <span>{{presence_pct_display}}</span>
</div>
```

Rules:

- Use the actual source percentage.
- Clamp the visual bar width to 0–100 only after validating that the source metric is a percentage.
- Preserve the source value separately if its scale is not a percentage.
- Do not apply unpublished High/Mid/Low benchmark classes.
- Do not equate a high presence percentage with guaranteed recommendation.
- Do not display a presence bar when the source metric is missing or its scale is unresolved.
- Preserve prompt text without inventing truncated content.

### 13.3 Share of Voice chart

Sources:

- `ai_search.sov_your`
- `ai_search.sov_competitors`

```html
<div class="sov-row">
  <div class="sov-name">
    {{merchant_name}}
  </div>

  <div class="sov-bar-bg">
    <div
      class="sov-bar-fill"
      style="width:{{merchant_sov_bar_width}}%;"
    ></div>
  </div>

  <div class="sov-pct">
    {{merchant_sov_display}}
  </div>
</div>
```

Repeat the same structure for visible competitors.

Bar scaling:

```text
max_sov = maximum of visible, valid SoV values

bar_width_pct = value / max_sov * 100
```

If `max_sov` is zero, use zero-width bars.

If a value is missing, do not generate a numeric bar for it.

Do not automatically set the merchant's bar to a fixed percentage of the chart width.

Label comparisons as applying to the **measured competitor set**, not the entire market.

### 13.4 AI Daily Trend

Source:

`ai_search.daily_trend`

Expected structure:

```json
{
  "direction": null,
  "data_points": []
}
```

Each data point:

```json
{
  "date": "YYYY-MM-DD",
  "value": null,
  "estimated": false
}
```

Use the same SVG rendering principles as the GBP chart.

Suggested heading:

```html
<div class="sec-header">
  {{section_number}} &nbsp;AI Visibility Trend
</div>

<p class="context-note">
  {{ai_search_date_range_display}}
</p>

<div class="chart-container">
  {{generated_ai_trend_svg}}
</div>
```

Rules:

- Plot only visible dated measurements.
- Use `ai_search.date_range` for the measurement context.
- Do not invent daily data to fill missing dates.
- Visually distinguish estimated points.
- Do not label the trend rising, flat, or declining unless supported by `ai_search.daily_trend.direction`.
- Do not forecast future values.
- If no reliable points exist, omit the chart and avoid unsupported trend claims.

### 13.5 AI-Origin Traffic

Source:

`ai_search.ai_origin_traffic`

Expected structure:

```json
{
  "total_page_views": null,
  "sources": []
}
```

Each source:

```json
{
  "source": "visible source label",
  "page_views": null
}
```

Suggested structure:

```html
<div class="kpi-card">
  <div class="kpi-label">
    AI-Origin Page Views
  </div>

  <div class="kpi-val">
    {{ai_origin_total_page_views}}
  </div>

  <div class="kpi-sub">
    Directly attributed AI referral traffic
  </div>
</div>

<table>
  <thead>
    <tr>
      <th>AI 来源</th>
      <th>页面浏览量</th>
    </tr>
  </thead>

  <tbody>
    {{ai_origin_source_rows}}
  </tbody>
</table>
```

Rules:

- Show the total only when available.
- Show source rows only when supplied.
- Do not assume visible sources sum to the displayed total.
- Do not convert page views into orders or restaurant visits.
- Do not describe low AI-origin traffic as proof of low AI influence.
- Include measurement caveats when the source's attribution definition is limited.

---

## 14. Local Map Search section

Include this section only when:

`local_map.available = true`

Use:

- `local_map.avg_rank`
- `local_map.visibility_pct`
- `local_map.keywords`
- `local_map.competitors`
- `local_map.measurement_context`

### 14.1 Measurement context

Show the available context before presenting rankings.

```html
<p class="context-note">
  查询范围：{{local_map_query_set_display}}
  · 地理范围：{{local_map_location_display}}
  · 测量周期：{{local_map_date_range_display}}
</p>
```

Omit unknown context components rather than displaying invented labels or values.

Do not claim citywide ranking from a limited query set or grid.

### 14.2 Overview cards

Keep the original two-column layout.

```html
<div class="two-col-box">
  <div class="box-good">
    <div class="box-title good">
      综合平均排名
    </div>

    <div class="kpi-val">
      {{local_map_avg_rank_display}}
    </div>

    <p class="context-note">
      {{rank_measurement_definition}}
    </p>
  </div>

  <div class="box-good">
    <div class="box-title good">
      综合可见度
    </div>

    <div class="kpi-val">
      {{local_map_visibility_display}}
    </div>

    <p class="context-note">
      {{visibility_measurement_definition}}
    </p>
  </div>
</div>
```

Rules:

- Do not automatically color a rank as good or bad using an unpublished threshold.
- Preserve the source-defined meaning of visibility percentage.
- If one metric is unavailable, omit its card or show an appropriately labeled missing state.
- Do not call an average rank a Google-wide ranking.

### 14.3 Keyword ranking table

Source:

`local_map.keywords`

Suggested item:

```json
{
  "keyword": "visible term",
  "avg_rank": null,
  "visibility_pct": null
}
```

```html
<table>
  <thead>
    <tr>
      <th>关键词</th>
      <th>平均排名</th>
      <th>可见度</th>
      <th>说明</th>
    </tr>
  </thead>

  <tbody>
    {{local_map_keyword_rows}}
  </tbody>
</table>
```

Ranking badge rules:

- Use `.rank-good`, `.rank-mid`, and `.rank-bad` only when the relevant source legend or documented classification supports them.
- Otherwise use `.rank-neutral`.
- Do not treat an internal color band as an official Google ranking standard.
- Do not infer ranking from color alone.
- Render missing values as `—`.

### 14.4 Competitor comparison table

Source:

`local_map.competitors`

Suggested item:

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

```html
<table>
  <thead>
    <tr>
      <th>餐厅</th>
      <th>平均排名</th>
      <th>可见度</th>
      <th>评分</th>
      <th>评价数</th>
    </tr>
  </thead>

  <tbody>
    {{local_map_competitor_rows}}
  </tbody>
</table>
```

Rules:

- Highlight the merchant row only when `is_self: true` is supported.
- Do not invent a merchant row if none was extracted.
- Do not assign a competitor's rating or review count to the merchant.
- Show ratings and review counts only when visible.
- Do not claim a ranking difference was caused by review volume.
- Do not assume every listed business is a direct competitor.

---

## 15. Customer Reviews section

Include this section only when:

`review_data.available = true`

Use:

- `review_data.tags_positive`
- `review_data.tags_neutral`
- `review_data.tags_negative`
- `review_data.summary_bullets`
- `review_data.review_scope`

### 15.1 Review scope

Display the source's actual review sample description.

```html
<p class="context-note">
  {{review_scope_display}}
</p>
```

If the scope is unknown, do not invent a sample size.

### 15.2 Positive tags

```html
<p style="font-weight:600;font-size:12px;margin-bottom:6px;">
  好评亮点 ↗
</p>

<div class="tag-row">
  {{positive_review_tags}}
</div>
```

Generate each positive tag as:

```html
<span class="tag tag-pos">
  {{escaped_positive_tag}}
</span>
```

### 15.3 Neutral tags

```html
<p style="font-weight:600;font-size:12px;margin-bottom:6px;">
  中性标签 —
</p>

<div class="tag-row">
  {{neutral_review_tags}}
</div>
```

Generate each neutral tag using `.tag-neu`.

### 15.4 Negative tags

```html
<p style="font-weight:600;font-size:12px;margin-bottom:6px;">
  负面标签 ↘
</p>

<div class="tag-row">
  {{negative_review_tags}}
</div>
```

Generate each negative tag using `.tag-neg`.

### 15.5 Review summary

Render `review_data.summary_bullets` as a short evidence-based summary.

Rules:

- Preserve visible themes.
- Do not fabricate frequencies.
- Do not treat one tag as proof of a widespread problem.
- Do not assume the sample represents all reviews.
- Do not infer who manages review replies.
- Do not introduce confidential service tiers or operational responsibilities.
- Omit empty tag groups.

---

## 16. Highlights & Watch Points

Use the original two-column layout.

```html
<div class="two-col-box">
  <div class="box-good">
    <div class="box-title good">
      ✓ 阶段亮点
    </div>

    <ul>
      {{supported_highlight_items}}
    </ul>
  </div>

  <div class="box-attn">
    <div class="box-title attn">
      △ 需关注
    </div>

    <ul>
      {{supported_watch_point_items}}
    </ul>
  </div>
</div>
```

Sources:

- `insights`
- `warnings`
- `diagnostic_summary.cross_module_findings`
- `diagnostic_summary.priority_issue`

Rules:

- Each highlight must be evidence-supported.
- Each watch point must be actionable or clearly identified as a measurement limitation.
- Do not treat missing AI, map, or review screenshots as poor performance.
- If no meaningful watch point exists, omit the watch-point box.
- If no supported highlights exist, do not invent them.
- Do not imply causal relationships between different discovery channels.

---

## 17. Conclusion & Next Actions

Use:

- `diagnostic_summary.paid_media`
- `diagnostic_summary.local_visibility`
- `diagnostic_summary.ai_visibility`
- `diagnostic_summary.reputation`
- `diagnostic_summary.cross_module_findings`
- `diagnostic_summary.priority_issue`
- `diagnostic_summary.recommended_action`

Recommended structure:

**Paragraph 1 — Performance interpretation**

Summarize the strongest observed findings across the available modules.

**Paragraph 2 — Operational next action**

Identify the single most consequential supported issue or opportunity and recommend a specific next action.

Rules:

- Do not introduce new metrics not present elsewhere in the report.
- Do not promise future results.
- Do not present a hypothesis as fact.
- Do not claim that paid activity caused changes in GBP, AI Search, or Local Map.
- If no priority issue is supported, do not manufacture one.

---

## 18. Footer

Use a neutral footer.

```html
<div class="footer">
  Data sources: supplied advertising and local visibility
  dashboard screenshots
  &nbsp;·&nbsp;
  {{restaurant_name}}
  &nbsp;·&nbsp;
  {{report_generation_date}}
</div>
```

For Chinese reports, translate the source description.

Rules:

- Do not use an unapproved company or service-provider name.
- Do not imply that the report is officially endorsed by a platform.
- Do not claim that every data source was used when some modules were absent.
- List only the actual supplied source categories.
- Do not display private internal identifiers.
- Include merchant address only when authorized and appropriate.

---

## 19. Formatting and missing-value rules

### 19.1 Numeric values

- Use thousands separators for large counts.
- Use source-supported decimal precision.
- Do not manufacture precision.
- Use percentage symbols only when the source metric is defined as a percentage.
- Preserve approximate values using `~`, `约`, or an explanatory note.

### 19.2 Currency

Read:

`campaign_metrics.currency`

Examples of possible source labels include currency codes or displayed currency symbols.

Do not assume that `$` always means USD.

If the source currency is ambiguous, preserve the source label and flag the ambiguity when material.

### 19.3 Dates

- Normalize unambiguous machine-readable dates during extraction.
- Localize date display according to report language.
- Keep campaign start date, reporting window, and report generation date distinct.
- Do not invent a reporting cutoff.
- Do not display a duration derived from incompatible periods.

### 19.4 Missing data

Use:

- `null` for unknown numeric values.
- `""` for unknown text.
- `[]` for unavailable list items.
- `available: false` for optional modules not supplied.

Do not convert missing data into zero.

Do not use placeholder numbers in the final HTML.

### 19.5 Escaping and safe rendering

Before inserting source text into HTML:

- Escape `&`, `<`, `>`, `"`, and `'` as appropriate to the insertion context.
- Never insert raw screenshot-derived text as executable HTML.
- Validate numeric values before inserting them into CSS widths or SVG coordinates.
- Do not insert arbitrary source strings into CSS properties or HTML attributes without context-appropriate validation.

The final HTML must remain valid and self-contained.

---

## 20. Language selection

Read:

`language`

Allowed values:

- `"chinese"`
- `"english"`
- `null` before selection

If the user explicitly requests a language, follow that request.

Otherwise, use Chinese as the default report language after the language-selection step.

Do not infer language preference from the restaurant name, cuisine, or address alone.

For English reports, replace the body font stack with:

```css
font-family: -apple-system, BlinkMacSystemFont,
  'Segoe UI', Arial, sans-serif;
```

Translate all user-facing text consistently.

Keep source labels in their original language when translation would change the metric's meaning.

---

## 21. Final HTML quality gate

Before delivering the report, verify all of the following:

1. The HTML is a single self-contained file.
2. All CSS is embedded in `<style>`.
3. No external library or asset is required.
4. All merchant text is safely escaped.
5. No unresolved `{{placeholder}}` remains.
6. No real merchant data has been introduced into a synthetic public example.
7. No proprietary company names, service tiers, pricing, or internal benchmarks appear.
8. No platform-estimated Revenue or ROAS range appears.
9. Store Visits and Cost per Store Visit are excluded from KPI cards.
10. The KPI grid matches `campaign_metrics.kpi_mode`.
11. Monetary values use `campaign_metrics.currency`.
12. Reporting periods use the correct source fields.
13. Missing metrics are not displayed as zero.
14. Daily averages use compatible denominators.
15. Campaign search terms and GBP search terms are kept separate.
16. GBP chart legends use `gbp_metrics.series_labels`.
17. GBP charts do not imply service causality.
18. Pending or estimated chart points are handled correctly.
19. AI Search content is included only when `ai_search.available = true`.
20. AI Daily Trend uses `ai_search.daily_trend`.
21. AI-origin traffic uses `ai_search.ai_origin_traffic`.
22. Local Map content is included only when `local_map.available = true`.
23. Local Map findings respect `local_map.measurement_context`.
24. Merchant and competitor rows are not confused.
25. Review content is included only when `review_data.available = true`.
26. Review conclusions respect `review_data.review_scope`.
27. Section numbers remain consecutive after optional sections are omitted.
28. No unsupported benchmark, prediction, or causal claim is introduced.
29. The conclusion is consistent with `diagnostic_summary`.
30. The final file renders without broken tables, overflowing labels, or invalid SVG coordinates.

If a material check fails, correct the report before delivery.

---

## 22. Data contract compatibility

This reference is aligned with the **Intermediate JSON v0.2** defined in `SKILL.md` and the updated extraction and analysis references.

| Newly supported field | Rendering responsibility |
|---|---|
| `campaign_metrics.currency` | Currency-aware monetary values |
| `campaign_metrics.reporting_window` | Correct reporting-period labels |
| `campaign_metrics.campaign_status` | Evidence-based status badge |
| `campaign_metrics.search_terms` | Campaign keyword table |
| `gbp_metrics.series_labels` | Actual chart legend names |
| `ai_search.daily_trend` | AI visibility trend chart and description |
| `ai_search.ai_origin_traffic` | AI referral page-view card and source table |
| `local_map.measurement_context` | Map ranking measurement scope |
| `review_data.review_scope` | Review sample description |

This file specifies presentation behavior. It does not authorize inventing new metrics, field names, or business conclusions.

If the Intermediate JSON changes, update `SKILL.md`, `extraction-schema.md`, `analysis-rules.md`, and this file together.

End-to-end compatibility must be verified with a synthetic or explicitly authorized test case before the skill is described as fully validated.

