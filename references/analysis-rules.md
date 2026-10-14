
# Analysis Rules

Business judgment rules for interpreting extracted restaurant advertising and local visibility data and framing the report narrative.

These rules are independent of output format. They apply to HTML reports, presentation decks, executive summaries, and customer-facing talking points.

Read `SKILL.md` and `references/extraction-schema.md` before applying these rules. All analysis must use the Intermediate JSON data contract defined in `SKILL.md`.

The purpose of this document is to turn observable metrics into defensible business insights, not to manufacture positive conclusions.

---

## 1. Absolute constraints

### 1.1 Never use platform-estimated Revenue or ROAS ranges

Some advertising dashboards display estimated Total Revenue or ROAS ranges that may include modeled offline conversions.

These ranges must never appear in the final report:

- Not in KPI cards
- Not in narrative
- Not in highlights
- Not in diagnostic summaries
- Not in conclusions

Do not infer merchant revenue from attributed orders, ad clicks, store visits, or other engagement metrics.

`campaign_metrics.orders` represents the visible platform-attributed order count. It is not independently verified merchant revenue.

### 1.2 Never put Store Visits or Cost per Store Visit in the KPI grid

Store Visits and Cost per Store Visit may be modeled or estimated rather than directly observed.

They are not supported KPI cards.

They may be mentioned briefly in narrative only when the source and estimation limitations are explicit.

Do not present modeled store visits as confirmed restaurant visits.

### 1.3 Never predict future KPI quantities

Do not write statements such as:

- "The campaign will generate 500 orders next month."
- "AI visibility will increase by 20%."
- "The restaurant is guaranteed to gain more customers."

Report observed performance and evidence-based improvement opportunities.

Recommendations may describe intended outcomes, but must not promise specific future results.

### 1.4 Never turn missing data into negative performance

A missing screenshot, unavailable module, or unreadable metric is not evidence of poor performance.

Examples:

- `ai_search.available: false` means AI Search evidence was not supplied.
- `local_map.available: false` means Local Map evidence was not supplied.
- `gbp_metrics.total_interactions: null` means the value is unknown.
- `campaign_metrics.clicks: null` does not mean the campaign received zero clicks.

Use `null` and `[]` according to the data contract.

Do not use missing values to support a negative finding.

### 1.5 Never infer causality from cross-channel correlation

Paid media, GBP interactions, local map rankings, AI visibility, and review sentiment describe different aspects of customer discovery.

They do not establish a causal chain by themselves.

Do not claim that:

- Advertising caused organic map rankings to improve.
- AI mentions caused website visits or restaurant orders.
- A higher review count caused better local rankings.
- Increased GBP interactions prove incremental customer acquisition.

Use qualified language such as:

- "The data suggests..."
- "This may indicate..."
- "The channels show a similar pattern..."
- "Additional attribution data would be required to confirm..."

### 1.6 Never expose private business rules

Public reports and repository examples must not disclose:

- Confidential service prices or subscription tiers
- Internal operational responsibilities
- Proprietary campaign benchmarks
- Private vendor or customer arrangements
- Internal attribution formulas
- Unapproved merchant data

Use only public, synthetic, or explicitly authorized information.

---

## 2. KPI grid selection

Select the KPI layout using `campaign_metrics.kpi_mode`.

Do not choose a layout based on assumed performance.

### 2.1 Standard grid

Condition:

`campaign_metrics.kpi_mode = "standard"`

This corresponds to a Reach view where impressions, clicks, CTR, and CPC are available.

Preferred layout:

```text
Row 1:
Ad Spend | Impressions | Clicks | Online Orders

Row 2:
Menu Views | Direction Requests | CPC | Website Visits
```

Rules:

- Display CTR as the subtitle of the Clicks card when available.
- Use the source currency from `campaign_metrics.currency`.
- Show a daily average only when `restaurant.duration_days` is known and applicable to the same measurement period.
- Do not calculate daily averages from a campaign lifetime duration when the KPI covers a shorter reporting window.
- Website Visits is the default final card.
- Phone Calls may replace Website Visits when it is more relevant to the merchant's documented business objective and supported by the observed data.
- Do not invent a benchmark to justify a card replacement.

If a preferred metric is missing, do not display zero. Use a supported alternative metric or omit the card, subject to the HTML template's layout rules.

### 2.2 Restructured grid

Condition:

`campaign_metrics.kpi_mode = "restructured"`

This corresponds to a Results view where the Reach metrics are not supplied.

Preferred layout:

```text
Row 1:
Ad Spend | Online Orders | Menu Views | Direction Requests

Row 2:
Phone Calls | Website Visits | Other Actions | GBP Total Interactions
```

Rules:

- Do not reconstruct impressions, clicks, CTR, or CPC from unrelated metrics.
- Do not include Revenue or ROAS ranges.
- Display only observed metrics.
- Use the GBP fallback rule when GBP Total Interactions is unavailable.
- Do not force an eight-card grid when the evidence does not support eight cards.

### 2.3 GBP-absent KPI fallback

If `gbp_metrics.total_interactions` is unavailable and `local_map.available = true`, the eighth KPI position may use:

```text
Label:
Local Map Visibility

Value:
local_map.visibility_pct

Subtitle:
Local Map Search · visible measurement period
```

Use the actual period from:

`local_map.measurement_context.date_range`

Do not hardcode "Last 7 days" unless that period is explicitly supported by the source.

If Local Map visibility is also unavailable, omit the card.

### 2.4 KPI subtitle rules

Subtitles should add meaningful context rather than repeat the headline number.

Suitable context includes:

- Observed CTR
- A supported daily average
- The displayed reporting window
- A visible source label
- An attribution or estimation caveat

Avoid unsupported claims such as "above industry average" or "exceptional performance."

---

## 3. GBP trend framing

Use:

- `gbp_metrics.total_interactions`
- `gbp_metrics.view`
- `gbp_metrics.chart_type`
- `gbp_metrics.data_points`
- `gbp_metrics.series_labels`

### 3.1 Establish the actual trend

Before writing a trend statement:

1. Confirm whether the chart is weekly or monthly.
2. Identify the visible series using `series_labels`.
3. Exclude pending or incomplete periods from definitive trend conclusions.
4. Distinguish exact data points from visually estimated values.
5. Check whether the observation period is long enough to support the conclusion.

Do not interpret a dual-series chart as proof that a service caused an improvement.

### 3.2 Rising trend

When the latest completed data points show a clear increase, describe the observed trajectory.

Example:

"GBP interactions increased across the most recent completed reporting periods. Additional observations are needed to determine whether this pattern will continue."

Do not write "more growth is expected."

### 3.3 Flat trend

When completed data points remain relatively stable, describe the stability without forcing a positive interpretation.

Example:

"GBP interactions remained broadly stable during the observed period, with no clear sustained increase or decrease."

### 3.4 Declining trend

When completed data points show a decline, report it directly.

Example:

"Weekly GBP interactions declined from the earlier observed level. The available data does not establish the cause."

A decline may have several possible explanations, including incomplete data, seasonality, changes in search demand, or operational changes.

However, do not assert any explanation without supporting evidence.

### 3.5 Structural explanations

Before attributing a trend to seasonality, campaign maturity, or business conditions, check whether the explanation is supported by:

- The visible reporting period
- The restaurant category
- Known local events or operating changes
- Comparable historical data
- User-supplied business context

Do not automatically frame declining performance as healthy normalization.

### 3.6 Estimated chart values

If a chart value is estimated, use qualified wording and respect:

`confidence_flags.approximated_fields`

If the uncertainty materially affects the trend conclusion, flag it and request clarification.

---

## 4. Analysis checklist

Run this checklist before writing highlights, detailed analysis, or the conclusion.

Only surface patterns supported by the available evidence.

| Pattern | Evidence to check | Appropriate interpretation |
|---|---|---|
| Brand search terms | Merchant name appears in `campaign_metrics.search_terms` or `gbp_metrics.search_terms` | The merchant is appearing in brand-related search activity; the data alone does not establish brand-awareness growth |
| Competitor-related terms | Visible campaign or GBP terms include a competitor name | The merchant is appearing in searches associated with another brand; verify context before describing relevance |
| Rising GBP interactions | Completed `gbp_metrics.data_points` show an upward pattern | Report the observed trend without forecasting continued growth |
| Category keyword mismatch | Campaign search terms conflict with the merchant's verified offering | Flag potentially mismatched search intent and recommend checking keyword targeting and business information |
| Multiple product categories | Verified merchant offering spans distinct categories and matching terms appear | Describe the observed breadth of search coverage |
| CTR performance | `campaign_metrics.ctr` is available | Interpret alongside clicks, spend, reporting period, and campaign objective; do not use unpublished benchmarks |
| AI category-query visibility | Generic discovery prompts have visible presence | Identify where the merchant appears in category discovery |
| Local Map ranking gaps | Specific keywords have weaker observed rankings | Identify location- and query-specific optimization opportunities |
| Review friction | Negative tags or summary themes are visible | Describe the reputation issue without inventing frequency or customer impact |

If a pattern is absent, skip it.

Do not create a highlight merely to fill space.

---

## 5. Highlight and warning selection

### 5.1 Highlights

Use highlights for meaningful, evidence-backed findings.

Suitable examples include:

- A clearly rising trend in completed GBP observations
- Relevant search terms covering multiple verified business categories
- Strong relative visibility within the measured competitor set
- A useful combination of paid, map, and AI discovery signals
- A positive review theme supported by the supplied review summary

Every highlight should include:

1. The observed evidence
2. Why it matters
3. Any important limitation

Avoid unsupported superlatives.

### 5.2 Priority warnings

Use a P0 warning only for a material, actionable issue supported by clear evidence.

Examples:

- Verified business information conflicts with prominent search terms and is likely to attract unsuitable customer intent.
- Repeated food-quality concerns are explicitly documented in the supplied review evidence and require urgent merchant attention.
- A critical reporting inconsistency prevents reliable interpretation of a major performance claim.

Do not infer that one negative review tag represents a widespread problem.

### 5.3 P1 improvement opportunities

Use P1 for issues that warrant attention but do not establish an immediate critical risk.

Examples:

- Weak visibility for a relevant local search query
- Incomplete business information
- Limited evidence of category-query AI presence
- Review themes suggesting a possible operational improvement

### 5.4 Warning display rule

Use at most one primary P0 warning box per report.

If several critical issues are present, select the most consequential and explain other issues in the narrative or action plan.

If no P0 issue is supported, omit the box.

---

## 6. AI Search analysis rules

Apply this section only when:

`ai_search.available = true`

Relevant fields:

- `ai_search.date_range`
- `ai_search.visibility_score`
- `ai_search.sentiment`
- `ai_search.top3_ranking`
- `ai_search.prompts`
- `ai_search.sov_your`
- `ai_search.sov_competitors`
- `ai_search.daily_trend`
- `ai_search.ai_origin_traffic`

### 6.1 Prompt classification

Use the extracted `query_type` for each prompt.

Supported values:

- `"brand"`
- `"category"`
- `"unknown"`

Brand queries explicitly refer to the merchant or a recognizable brand identifier.

Category queries describe a cuisine, product, occasion, or location without explicitly naming the merchant.

Do not classify an ambiguous query as a brand query without supporting evidence.

### 6.2 Prompt presence interpretation

Use `presence_pct` only when its source definition is known.

Compare prompts within the same measurement context when possible.

Appropriate findings include:

- Brand queries show higher observed presence than category queries.
- A specific category query has no observed presence in the supplied sample.
- Presence varies substantially across different customer discovery scenarios.

Do not apply unpublished presence thresholds as universal standards.

A high observed presence percentage does not guarantee that an AI system will recommend the merchant to every user.

### 6.3 Share of Voice

Use:

- `ai_search.sov_your`
- `ai_search.sov_competitors`

Compare only values from the same measurement set.

If the merchant's SoV exceeds the displayed competitors, describe it as leading **within the measured set**.

If values are similar, describe them as comparable.

If the merchant trails a displayed competitor, identify the relative gap as a possible opportunity.

Do not claim market-wide dominance from a limited sample.

### 6.4 Sentiment

Use the visible source sentiment labels.

- **Positive:** Note the favorable representation when supported.
- **Neutral:** Describe neutrally; do not force a positive or negative conclusion.
- **Negative:** Identify the issue and compare with review evidence when available.

AI sentiment and customer review sentiment are not necessarily based on the same data.

Do not assume that one caused the other.

### 6.5 Scene coverage gaps

A prompt with observed zero presence may indicate an opportunity for better coverage of that specific customer discovery scenario.

Before recommending action, check whether the scenario is relevant to the merchant's verified offering.

Do not classify zero presence as P0 without clear material business impact.

### 6.6 AI Daily Trend

Use:

`ai_search.daily_trend.direction`

Allowed values:

- `"rising"`
- `"flat"`
- `"declining"`
- `null`

Interpret the direction only over the displayed `ai_search.date_range`.

Use `ai_search.daily_trend.data_points` for supporting observations.

Rules:

- Do not infer a sustained trend from a single data point.
- Do not turn visually estimated values into exact numbers.
- Do not forecast future AI visibility.
- If `direction` is `null`, state that the trend is unresolved rather than assigning a direction.
- Do not compare AI Daily Trend with GBP trend as though they measure the same behavior.

### 6.7 AI-Origin Traffic

Use:

- `ai_search.ai_origin_traffic.total_page_views`
- `ai_search.ai_origin_traffic.sources`

AI-Origin Traffic represents traffic attributed to identifiable AI referral sources according to the source dashboard.

It is a directional attribution signal, not a complete measure of AI-assisted discovery.

Rules:

- Do not interpret low directly attributed AI traffic as proof that AI has no influence.
- Do not interpret high AI visibility with low AI-origin traffic as an automatic contradiction.
- Do not assume all AI-influenced visits are tracked as AI referrals.
- Do not describe AI-origin page views as confirmed orders.
- If source-level values are shown, identify their contribution only when the denominator and measurement period are clear.
- Do not infer missing traffic sources.

Use the actual source labels and measurement definitions.

### 6.8 AI module unavailable

If `ai_search.available = false`, omit AI-specific performance conclusions.

Do not describe AI visibility as weak, absent, or declining.

A future AI measurement may be recommended as a data-coverage improvement, but not as a response to an unobserved performance failure.

---

## 7. Local Map Search analysis rules

Apply this section only when:

`local_map.available = true`

Relevant fields:

- `local_map.avg_rank`
- `local_map.visibility_pct`
- `local_map.keywords`
- `local_map.competitors`
- `local_map.measurement_context`

### 7.1 Rank interpretation

Interpret average rank using the source definition and measurement context.

Lower numeric rank generally indicates better placement when the source uses conventional ranking order.

Use specific observed values rather than unsupported performance labels.

For example:

"The restaurant's average position for this measured query set was X."

If a source-confirmed ranking legend is available, its categories may be used with an appropriate caveat.

Do not present internal reporting bands as official search-engine standards.

### 7.2 Keyword-level analysis

Review each visible keyword in:

`local_map.keywords`

Identify:

- Queries with comparatively stronger observed ranking
- Queries with comparatively weaker observed ranking
- Differences in visibility across relevant categories

Do not infer the ranking of unmeasured keywords.

### 7.3 Competitor comparison

Use:

`local_map.competitors`

Compare the merchant with competitors only when:

1. The merchant's own row is reliably identified using `is_self`.
2. The metrics use compatible measurement definitions.
3. The query set, geography, and period are comparable.
4. The comparison businesses are relevant to the question being analyzed.

Ratings and review counts may provide useful context, but they do not establish why a business ranks higher or lower.

Do not attribute ranking differences to review volume without additional evidence.

If the merchant's own row is unavailable, do not invent a merchant rating or review count.

### 7.4 Measurement context

Use:

`local_map.measurement_context.query_set`

`local_map.measurement_context.location`

`local_map.measurement_context.date_range`

Every Local Map conclusion should respect the measured query set, location, and period.

Avoid statements such as "the restaurant ranks first in the city" when the evidence covers only a limited keyword set or search grid.

### 7.5 Local Map module unavailable

If `local_map.available = false`, omit map-ranking conclusions.

Do not interpret the absence of a screenshot as poor organic map visibility.

---

## 8. Cross-module synthesis

Populate:

`diagnostic_summary.cross_module_findings`

Only when the required evidence is available.

The four diagnostic layers are:

1. Paid Media
2. Local Visibility
3. AI Visibility
4. Reputation

The objective is to identify meaningful relationships and gaps across the customer discovery journey.

### 8.1 Paid activity with limited measured AI presence

Required evidence:

- Relevant paid campaign metrics are visible.
- AI Search data is available.
- The measured AI visibility is comparatively limited.

Possible finding:

"Paid advertising is generating observable engagement, while the supplied AI Search sample shows more limited discovery visibility. This suggests an opportunity to examine AI discovery coverage separately."

Do not call this a proven acquisition-channel gap without compatible evidence.

Do not use this pattern when `ai_search.available = false`.

### 8.2 Paid engagement with weaker measured map visibility

Required evidence:

- Paid engagement actions are visible.
- Local Map data is available.
- Relevant map queries show comparatively weaker visibility.

Possible finding:

"Paid advertising is generating customer actions, while the measured organic map presence is less prominent for several relevant queries."

Do not claim that advertising is compensating for organic weakness unless incremental attribution evidence supports that claim.

### 8.3 AI visibility with reputation concerns

Required evidence:

- AI Search data is available.
- Review evidence shows a specific negative theme.

Possible finding:

"The merchant appears in the measured AI discovery results, while the supplied reviews identify a customer-experience concern worth addressing."

Do not claim that the concern has already reduced conversions.

### 8.4 Map visibility with limited review evidence

Required evidence:

- Local Map visibility is available.
- The merchant's review count or recent review activity is explicitly supported.

Possible finding:

"The merchant has observable local map presence, while the available review evidence suggests an opportunity to strengthen social proof."

Do not assume that review volume is low simply because the review screenshot is missing.

### 8.5 Search-intent mismatch across channels

Required evidence:

- Campaign search terms contain a verified mismatch with the merchant's offering.
- Separate business-profile evidence confirms a relevant information mismatch.

Possible finding:

"Paid search terms and business-profile information show a consistent mismatch with the merchant's verified offering, creating a potential customer-intent issue."

Recommend correcting the verified mismatch.

Do not assert a GBP description problem unless the actual description or equivalent business information has been supplied.

### 8.6 Multiple strong discovery signals

Required evidence:

- At least two relevant modules are available.
- Both show positive observations under their own measurement definitions.

Possible finding:

"The merchant shows favorable visibility signals in both local map search and the measured AI discovery sample."

Do not claim combined market dominance or causal reinforcement.

### 8.7 No supported cross-module pattern

If none of the patterns are supported, leave:

```json
{
  "cross_module_findings": []
}
```

Do not force a synthesis finding.

---

## 9. Review management framing

Apply this section only when:

`review_data.available = true`

Relevant fields:

- `review_data.tags_positive`
- `review_data.tags_neutral`
- `review_data.tags_negative`
- `review_data.summary_bullets`
- `review_data.review_scope`

### 9.1 Review theme interpretation

Identify recurring or material themes only when supported by the supplied evidence.

Examples:

- Food quality
- Service experience
- Waiting time
- Ordering experience
- Value perception

Do not invent review frequencies or claim a theme is widespread based on one isolated tag.

### 9.2 Review scope

Use:

`review_data.review_scope`

Preserve the actual sample description.

If the source says "Last 50 reviews," describe the findings as relating to that sample.

Do not treat the sample as the entire merchant review history.

If the scope is unknown, qualify the conclusion accordingly.

### 9.3 Review response responsibility

Do not infer who manages review replies.

The available screenshots do not establish whether replies are managed by:

- The merchant
- An external service provider
- A shared operations team
- An automated tool

Only describe ownership or responsibility when explicitly confirmed by the user or an authorized source.

Do not infer service entitlements from the merchant's advertising metrics.

### 9.4 Review improvement recommendations

Recommendations should address observed review themes and confirmed operating responsibilities.

Examples:

- Respond to specific negative feedback with factual and professional language.
- Investigate recurring operational issues when supported by the review sample.
- Maintain consistent review-response practices.
- Encourage authentic customer feedback through appropriate, platform-compliant methods.

Do not promise rating increases or a fixed number of new reviews.

### 9.5 Review evidence unavailable

If `review_data.available = false`, omit review-specific findings.

A recommendation to establish review monitoring may be appropriate, but it must be framed as a measurement or operational suggestion, not as proof of a reputation problem.

---

## 10. Diagnostic summary mapping

Populate the existing `diagnostic_summary` object after completing module-level analysis.

Use the following fields without renaming them:

| JSON field | Content |
|---|---|
| `paid_media` | Concise interpretation of available campaign metrics |
| `local_visibility` | GBP and/or Local Map findings, with measurement context |
| `ai_visibility` | AI Search findings when evidence is available |
| `reputation` | Review findings when evidence is available |
| `cross_module_findings` | Evidence-supported cross-channel observations |
| `priority_issue` | Single most consequential supported issue |
| `recommended_action` | One actionable recommendation tied to the priority issue |

### 10.1 Module summaries

Write concise, evidence-based summaries.

Do not populate a module summary with unsupported performance conclusions.

When a module is unavailable, use an appropriately neutral statement or leave its text empty according to the report requirements.

### 10.2 Priority issue

Select the most consequential issue based on:

- Evidence strength
- Potential business relevance
- Actionability
- Merchant context

Do not create an artificial priority issue when no meaningful issue is supported.

Use `""` if no priority issue can be established.

### 10.3 Recommended action

Every recommendation should identify:

1. The issue or opportunity
2. The proposed operational action
3. The evidence supporting the recommendation

Do not recommend actions that depend on unverified service entitlements or capabilities.

### 10.4 Insights and warnings

Use the existing top-level arrays:

- `insights`
- `warnings`

`insights` should contain supported observations suitable for the report narrative.

`warnings` should contain extraction limitations, material uncertainties, or business risks that require qualification.

Do not treat every missing field as a business warning.

Do not introduce new top-level JSON keys without updating `SKILL.md`.

---

## 11. Writing quality rules

### 11.1 Executive Summary

Open with campaign duration and total ad spend when both are available and comparable.

If duration is unknown, do not invent it.

Identify two or three meaningful observed metrics or findings.

Summarize the local visibility trajectory when supported.

Mention a material review issue only when review evidence is available.

Aim for three concise paragraphs.

### 11.2 KPI card subtitles

Use:

- Observed supporting metrics
- Clearly defined reporting context
- Valid daily averages
- Source-specific measurement caveats

Do not use confidential or unpublished performance benchmarks.

### 11.3 Highlights

Each highlight should be specific and defensible.

Preferred structure:

**Observation → Business relevance → Limitation or next step**

Avoid vague statements such as "performance is outstanding" without a supported basis for comparison.

### 11.4 Conclusion

Use two concise paragraphs when the report format allows.

Paragraph 1:

Summarize the strongest supported observations and overall measured trajectory.

Paragraph 2:

Recommend the single most impactful next action, grounded in the available data.

Do not introduce new unsupported claims in the conclusion.

### 11.5 Tone

Use a professional, data-forward, and appropriately cautious tone.

Prefer specific observations over promotional language.

Distinguish:

- Observed data
- Derived calculations
- Interpretive hypotheses
- Operational recommendations

Do not present a hypothesis as a verified fact.

---

## 12. Language selection

Use the existing field:

`language`

Allowed values:

- `"chinese"`
- `"english"`
- `null` before selection

### Selection rules

| Condition | Language |
|---|---|
| User explicitly requests Chinese | `"chinese"` |
| User explicitly requests English | `"english"` |
| No language preference is supplied | `"chinese"` by default |
| Language remains unresolved during extraction | `null` until selected |

Do not infer a merchant's preferred language solely from the merchant name, cuisine, or address.

For English reports, use an appropriate English-language font stack:

```css
font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Arial, sans-serif;
```

Translate all user-facing labels, section headings, table headers, review tags, and footer text consistently.

Preserve merchant names and source-specific terms when translation would distort their meaning.

---

## 13. Final analysis quality gate

Before passing the structured findings to the HTML template, verify:

1. The analysis uses only fields defined in the current Intermediate JSON.
2. The extraction confidence gate in `SKILL.md` has passed.
3. Revenue and ROAS ranges are absent from all report content.
4. Store Visits and Cost per Store Visit are excluded from KPI cards.
5. KPI mode matches the supplied Campaign Performance view.
6. Missing values are not treated as zero or poor performance.
7. GBP trend statements exclude incomplete periods and respect approximation flags.
8. Campaign search terms are not confused with GBP search terms.
9. AI Search findings are included only when `ai_search.available = true`.
10. AI Daily Trend and AI-Origin Traffic use their dedicated JSON fields.
11. Local Map conclusions respect query, geography, and date context.
12. Review conclusions respect `review_data.review_scope`.
13. No confidential service tiers, pricing, or internal benchmarks appear.
14. No causal or predictive claim exceeds the available evidence.
15. `diagnostic_summary` is populated consistently with the module-level analysis.
16. The priority issue and recommended action are supported and actionable.
17. Language selection uses one supported value.
18. The HTML template receives the same field names and missing-value conventions.

If a material issue fails this gate, correct the analysis or request clarification before generating the final report.

---

## Data contract compatibility

This document is aligned with the **Intermediate JSON v0.2** in `SKILL.md` and the updated `references/extraction-schema.md`.

It consumes the following newly supported fields:

| Data | JSON field | Analysis use |
|---|---|---|
| Campaign reporting period | `campaign_metrics.reporting_window` | Reporting context and valid comparisons |
| Campaign status | `campaign_metrics.campaign_status` | Interpretation of active, paused, or ended campaigns |
| Campaign currency | `campaign_metrics.currency` | Correct monetary display |
| Campaign Search Insights | `campaign_metrics.search_terms` | Brand, competitor, and category keyword analysis |
| GBP chart legend labels | `gbp_metrics.series_labels` | Accurate trend interpretation |
| AI Daily Trend | `ai_search.daily_trend` | Observed AI visibility trajectory |
| AI-origin traffic | `ai_search.ai_origin_traffic` | Directional referral-traffic analysis |
| Local Map measurement context | `local_map.measurement_context` | Query-, geography-, and period-aware ranking analysis |
| Review sample scope | `review_data.review_scope` | Properly scoped reputation findings |

The remaining `html-template.md` reference must use the same fields before end-to-end compatibility can be claimed.

Do not add or rename JSON fields independently in this document.

