---
name: ads-review-report
description: >
  Use this skill to create a structured advertising and local visibility performance report
  for a restaurant or local business. Trigger when the user provides screenshots or exported
  performance data covering Google Ads, Google Business Profile (GBP), local map visibility,
  AI search visibility, or customer reviews, together with a merchant name or location.
  Also trigger when the user asks to update or regenerate a previously created report with
  new performance data. The skill extracts structured metrics, checks extraction confidence,
  synthesizes findings across multiple discovery channels, and produces a single-file HTML
  performance report.
---

# Ads Review Report Skill

An AI-assisted review skill for evaluating restaurant advertising performance and local
digital visibility across paid media, local search, AI discovery, and reputation signals.

## The four-layer framework

Every report evaluates a merchant's **complete digital visibility chain** across four layers.
This is not a collection of disconnected metrics — it is a diagnostic of how the merchant
performs across the customer journey from paid acquisition to reputation-supported conversion.

| Layer | Module | Core question |
|---|---|---|
| **1. Paid Media Performance** | Google Ads | Did the ad spend drive measurable downstream behavior? |
| **2. Local Discovery** | Google Business Profile + Local Map Search | Can organic/local users find the merchant in Google Search and Maps? |
| **3. AI Discovery** | AI Search | Is the merchant visible or recommended when users search through AI assistants? |
| **4. Reputation / Conversion Support** | Customer Reviews | Does the merchant's reputation reinforce or undermine conversion after discovery? |

When writing the Executive Summary and Conclusion, frame findings across these four layers.
When writing Highlights & Watch Points, note any layer where the merchant is particularly
strong, weak, or inconsistent.

---

You are creating or updating a performance report for a restaurant or local business.

The user may provide screenshots, exported metrics, or previously generated reports.

**First: determine the mode.**

- New screenshots or data with no existing report → **Create mode**
- New data referencing an existing report, or a request to update/regenerate → **Update mode**

This skill uses three reference files:

| File | Purpose | When to read |
|---|---|---|
| `references/extraction-schema.md` | Field-by-field extraction guide | Before Step 1 |
| `references/analysis-rules.md` | Business rules, framing logic, and diagnostic constraints | Before Step 3 |
| `references/html-template.md` | HTML structure, CSS, charts, and components | Before Step 4 |

---
