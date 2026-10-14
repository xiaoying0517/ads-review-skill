

---
name: ads-review-report
description: >
  Create or update restaurant advertising and local visibility HTML reports from dashboard screenshots.
  Use for Google Ads, Google Business Profile, local map visibility, AI search, and customer reviews.
  Extract metrics, validate confidence, and synthesize cross-channel insights.
---

# Advertising & Local Visibility Review Skill

Generate an evidence-based, single-file HTML performance review for a restaurant. The skill is designed around **dashboard screenshots**, not privileged
access to an advertising platform or internal business system.

**Public-version scope:** This is a generalized portfolio implementation. It must not
require proprietary dashboards, unpublished benchmarks, merchant records, or internal
service packages. Demonstrations should use synthetic or explicitly authorized data.

## Four-layer diagnostic framework

Treat the report as a connected customer-discovery journey, not as unrelated KPI blocks.
Analyze only layers supported by the provided evidence.

| Layer | Inputs | Diagnostic question |
|---|---|---|
| **1. Paid Media Performance** | Google Ads campaign data | What measurable engagement or actions followed paid exposure? |
| **2. Local Discovery** | Google Business Profile (GBP), local map results | How visible is the business in local discovery surfaces? |
| **3. AI Discovery** | AI search visibility observations | Does the business appear in the sampled AI-search scenarios? |
| **4. Reputation / Conversion Support** | Customer reviews and response patterns | Do reputation signals support or weaken the discovery-to-decision journey? |

Use the four layers to organize the Executive Summary, Highlights & Watch Points, and
Conclusion. **Do not equate visibility, clicks, map actions, or review sentiment with
verified sales or causal lift.** A layer with no supporting screenshot is **not assessed**,
not automatically classified as weak.

## Operating modes and dependencies

Determine the mode before extracting data:

- **Create mode:** The user supplies new screenshots without an existing report. Run Steps 1–5.
- **Update mode:** The user supplies new screenshots and identifies a previous report to
  revise. Run Step 0, then Steps 1–5. Record what changed.

Read the companion files at the relevant steps. They must be reviewed and generalized
separately before this public skill can be considered end-to-end runnable.

| Reference | Purpose | Read before |
|---|---|---|
| `references/extraction-schema.md` | Screenshot types, field mapping, KPI variants | Step 1 |
| `references/analysis-rules.md` | Evidence constraints, interpretation and synthesis | Step 3 |
| `references/html-template.md` | Single-file HTML, charts and layout conventions | Step 4 |

---

## Step 0 — Update mode only: inspect the existing report

*Skip this step for a new report.*

1. **Locate the baseline.** Read the previously generated HTML report supplied by the user
   or present in the current workspace. If there are multiple plausible reports, ask which
   one to update. Never assume an earlier report is accessible.
2. **Capture existing values.** Record the reporting window, campaign duration, spend,
   displayed campaign KPIs, GBP series, optional AI/local/review sections, and chart data.
3. **Extract the new screenshots using Step 1**, then produce a concise change summary:

   ```text
   CHANGE SUMMARY
   Reporting window: [old] → [new]
   Ad spend: [old] → [new] ([difference, if comparable])
   Campaign duration: [old] → [new]
   KPI changes: [material changes and their definitions]
   New sections: [newly supported sections]
   Chart updates: [new or revised data points]
   Data updated through: [date]
   ```

4. **Check comparability.** Compare a metric only when its definition, unit, and time
   window are compatible. Do not describe an increase in cumulative totals as an
   improvement in performance without a valid comparison basis.
5. **Choose the update method.** Prefer targeted editing for updated figures, chart
   points, or a single new section. Regenerate the report if the KPI variant changes,
   the narrative structure is no longer valid, or more than half of the report requires
   replacement. Preserve still-valid content when possible.
6. **Update the date label** (for example, `Data updated through [Month Day, Year]`)
   and ensure the summary and conclusion reflect the latest evidence.

---

## Step 1 — Extract observed data from screenshots

Read `references/extraction-schema.md`. Identify each screenshot's module and extract
**visible** values only. Distinguish clearly between reported values, visual estimates,
missing fields, and derived calculations.

- Identify the business using the name/location supplied by the user; do not invent
  missing names, addresses, campaign dates, or metrics.
- Detect the campaign KPI layout (`standard` vs. `restructured`) before selecting cards.
- Detect the GBP chart style (`dual` vs. `red-only`) from what is actually visible;
  never infer a vendor relationship or before/after causality from color alone.
- Extract AI Search, Local Map, and Reviews **only if their evidence is supplied**.
- Keep screenshots with ambiguous numbers or overlapping reporting periods flagged.
- If the user supplies a file format not yet covered by the extraction reference (such
  as CSV/XLSX), do not imply the import path is validated; ask to inspect the fields or
  obtain supported screenshots before proceeding.

### Pipeline

```text
Dashboard screenshots
    → Step 1: Field extraction
    → Intermediate JSON
    → Step 1.5: Confidence gate
    → Step 2: Report settings
    → Step 3: Cross-layer diagnosis
    → Step 4: Single-file HTML
    → Step 4.5: QA
    → Step 5: Save / deliver
```

### Intermediate JSON

Use the following as a **working data contract**. Keep the keys stable for the
companion references. Values below are illustrative placeholders, **not sample
performance results**. Use `null` for a missing or unreadable numeric value; use `0`
only when the evidence explicitly shows zero. `available: false` means that the
module was not supplied, not that the business performed poorly.

```json
{
  "restaurant": {
    "name": "",
    "address": "",
    "city_state": "",
    "start_date": "",
    "duration_days": null
  },
  "campaign_metrics": {
    "ad_spend": null,
    "kpi_mode": "standard|restructured",
    "impressions": null,
    "clicks": null,
    "ctr": null,
    "cpc": null,
    "orders": null,
    "menu_views": null,
    "directions": null,
    "phone_calls": null,
    "website_visits": null,
    "other_actions": null
  },
  "gbp_metrics": {
    "total_interactions": null,
    "chart_type": "dual|red-only",
    "view": "weekly|monthly",
    "data_points": [],
    "search_terms": []
  },
  "ai_search": {
    "available": false,
    "date_range": "",
    "visibility_score": null,
    "sentiment": "",
    "top3_ranking": null,
    "prompts": [],
    "sov_your": null,
    "sov_competitors": []
  },
  "local_map": {
    "available": false,
    "avg_rank": null,
    "visibility_pct": null,
    "keywords": [],
    "competitors": []
  },
  "review_data": {
    "available": false,
    "tags_positive": [],
    "tags_neutral": [],
    "tags_negative": [],
    "summary_bullets": []
  },
  "confidence_flags": {
    "low_confidence_fields": [],
    "approximated_fields": []
  },
  "insights": [],
  "warnings": [],
  "language": "chinese|english",
  "diagnostic_summary": {
    "paid_media": "",
    "local_visibility": "",
    "ai_visibility": "",
    "reputation": "",
    "cross_module_findings": [],
    "priority_issue": "",
    "recommended_action": ""
  }
}
```

Complete this intermediate representation before drafting analysis or HTML. The
reference documents must use the same field names and missing-value conventions.

---

## Step 1.5 — Extraction confidence gate (required)

Before writing the report, prepare an internal extraction check:

```text
EXTRACTION CONFIDENCE SUMMARY
Business: [name / city]
Reporting period / start date: [value] — [HIGH / MEDIUM / LOW; reason]
Ad spend: [value] — [HIGH / MEDIUM / LOW]
KPI layout: [Standard / Restructured] — [confidence; reason]
GBP chart style: [Dual / Red-only / Not supplied] — [confidence; reason]
Chart data points: [count] — [observed / estimated]
Search terms: [count visible]
AI Search / Local Map / Reviews: [available / not supplied]
Missing or obscured fields: [list]
```

**Decision gate:**

- **Stop and ask** if a low-confidence field would materially alter KPI selection,
  trend direction, campaign duration, or the conclusion. Request a clearer screenshot
  or explicit confirmation.
- **Proceed with a visible qualifier** if a noncritical chart value is approximate.
  Prefix the figure with `~` or `约`, and retain the approximation flag.
- **Proceed with a noted gap** when a noncritical field is missing. Omit unsupported
  claims instead of substituting guesses.

Never silently infer missing metrics or claim exact precision from a visually estimated
chart point.

---

## Step 2 — Determine report settings

| Setting | Selection rule |
|---|---|
| **Language** | Follow the user's requested language; otherwise use the language of the request. |
| **KPI grid** | `standard` if supported reach metrics are visible; otherwise `restructured` when only results metrics are available. See extraction schema. |
| **GBP chart** | `dual` or `red-only`, based on visible series. Do not assert a causal before/after comparison without evidence. |
| **GBP section** | Include only when relevant screenshot data is provided. |
| **AI Search section** | Include only when relevant screenshot data is provided. |
| **Local Map section** | Include only when relevant screenshot data is provided. |
| **Reviews section** | Include only when relevant screenshot data is provided. |
| **Review response ownership** | Describe who handles reviews only if the user explicitly supplies that information; otherwise use neutral recommendations. No vendor-specific pricing or service tiers. |
| **Output name** | Use a neutral, sanitized name such as `restaurant-review-report.html`. |

If a screenshot lacks a metric required by the default layout, use a documented
fallback or omit that KPI. Do not invent a value to fill a visual card.

---

## Step 3 — Analyze and synthesize

Read `references/analysis-rules.md`. Apply evidence-first business reasoning across
available layers, distinguishing **observation**, **interpretation**, and **recommended
action**.

Before generating HTML, fill `diagnostic_summary`:

1. Write a one-sentence assessment for each evidenced layer; label missing layers
   as not assessed.
2. List meaningful cross-module findings (for example, paid traffic alongside
   weak local visibility), without claiming causation from correlation.
3. Identify at most one highest-priority issue, based on the strength of evidence
   and potential business relevance.
4. Recommend one concrete next action tied to the observed issue. If evidence is
   insufficient to prioritize an action, say so.

Treat AI visibility and local ranking measurements as **scope-limited observations**:
results can vary by query, geography, time, sampling method, and platform. A nearby
business is not automatically a direct competitor; compare competitors only when
category, audience, and search intent support the comparison.

Do not publish or reuse confidential, unverified performance benchmarks. Any public
heuristic must be clearly identified as illustrative and configurable, not an
industry-wide norm.

---

## Step 4 — Generate the HTML report

Read `references/html-template.md`. Produce a **self-contained HTML file** using
supported components and inline styling. Do not assume the presence of internal
branding, private assets, or proprietary report templates.

| Order | Chinese section | English section | Include |
|---|---|---|---|
| 01 | 执行摘要 | Executive Summary | Always |
| 02 | 核心指标快照 | KPI Snapshot | Always, using available metrics |
| 03 | 广告驱动行为漏斗 | Ad-Driven Engagement Funnel | When supported campaign actions exist |
| 04 | 关键词洞察 | Keyword Insights | When keyword/search-term data exists |
| 05 | GBP 互动分析 | GBP Interaction Analysis | If GBP evidence is supplied |
| 06 | AI 搜索曝光分析 | AI Search Performance | If AI Search evidence is supplied |
| 07 | 本地地图搜索排名 | Local Map Search | If Local Map evidence is supplied |
| 08 | 评价分析 | Review Analysis | If review evidence is supplied |
| 09 | 阶段亮点 / 注意事项 | Highlights & Watch Points | Always; may state evidence limitations |
| 10 | 结论 | Conclusion | Always |

Renumber sections consecutively after omitting unsupported optional modules. If a
campaign screenshot does not support a keyword table or funnel, omit that section
rather than inventing content.

**KPI-card rule:** Use only directly supported or transparently derived measures.
Where the original layout expects an eighth card, GBP interactions or local-map
visibility may be used only if those values are present and clearly labeled.
Do not use estimated Store Visits as a headline KPI card; if supplied, describe
its limitations in the narrative.

**Action rule:** Do not append an unsupported multi-step implementation plan. The
Conclusion should identify one evidence-linked priority action when possible.

---

## Step 4.5 — Quality assurance before delivery (required)

| Check | Pass condition |
|---|---|
| Evidence traceability | Every displayed number is present in the intermediate data or has a transparent calculation. |
| Revenue attribution | No unsupported platform revenue, ROAS, or incremental-sales claim is presented as verified business impact. |
| KPI integrity | KPI variant and labels match available fields; missing values are not treated as zero. |
| Optional sections | A section appears only when supported by the supplied evidence. |
| Sequential numbering | Section numbers are consecutive after optional sections are removed. |
| Approximation labeling | Visually estimated values are marked `~` or `约`. |
| Benchmark integrity | No confidential thresholds or unverified claims of industry averages. |
| Highlight support | Positive claims are backed by specific evidence; avoid generic praise. |
| Warning priority | At most one primary/P0 warning is emphasized. |
| Recommendation linkage | The proposed action traces to an observed finding. |
| Business identity | Business name and location are consistent throughout the report. |
| Privacy and branding | No real client identifiers, private endpoints, internal names, or company branding unless explicitly authorized for this output. |

If any check fails, revise the HTML before delivery.

---

## Step 5 — Save and deliver

1. Save the result as a single HTML file in a user-accessible output location provided
   by the runtime or requested by the user.
2. Use the runtime's **available** file-delivery mechanism. Do not call a
   platform-specific tool that is not installed.
3. Provide the file or its accessible path, the reporting date, and a short note
   about missing modules or approximate values.
4. For Update mode, include the material changes from Step 0.

---

## Evaluation framework (work in progress)

Evaluate this public version against **authorized or synthetic** cases. Do not
publish confidential historical reports, screenshots, or reviewer comments.

- **Data extraction accuracy:** Agreement with verified values visible in inputs.
- **Material metric omission rate:** Whether important, available metrics were missed.
- **Business judgment error rate:** Unsupported trend claims, priorities, or conclusions.
- **Human edit time:** Manual corrections needed to make a draft acceptable.
- **Cross-module consistency:** Whether conclusions reconcile evidence across layers.

The original evaluation plan proposes comparison across **at least 20 cases**.
This is a **future target**, not evidence that such testing has been completed.
Until a sufficiently varied and authorized evaluation set is available, describe
results as preliminary and report sample sizes explicitly.

## Public release limitations

- This `SKILL.md` describes the workflow; its three `references/` files must be
  independently reviewed and aligned before claiming the public repo is runnable.
- CSV/XLSX input, automated connectors, and production integrations are **not**
  validated by this version.
- The repository is a generalized demonstration of a restaurant-focused workflow
  design, not an official tool or endorsed integration from any advertising or SaaS provider.
- Use for other local-business categories would require additional schema and rule validation.
