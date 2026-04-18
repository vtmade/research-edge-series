---
name: correspondence-analysis
description: Prepare data for correspondence analysis and interpret the resulting perceptual map. Validates your contingency table, recommends a standardisation mode, then turns map positioning into plain-language findings and strategic narrative. Designed to work alongside the Research Edge Correspondence Analysis browser tool.
author: Vinay Thakur
github: vtmade
---

# Correspondence Analysis Skill

## What This Skill Does

Correspondence analysis (CA) converts a grid of numbers into a perceptual map — a 2D plot that shows which row items (attributes, drivers, barriers) are most strongly associated with which column items (brands, segments, markets).

The **Correspondence Analysis browser tool** (included in this repository under `toolkit/correspondence_map.html`) does the SVD computation and renders the interactive map. This skill wraps around it — handling data preparation before you upload, and strategic interpretation after you see the map.

**Use this skill in three ways:**
1. Paste your table → get a validation report and standardisation recommendation before uploading
2. Describe or paste your map output → get a structured interpretation of the positioning
3. Both in sequence → full end-to-end analytical workflow

---

## How to Read the Outputs

This skill produces three plain-text files. Start with `preparation-notes.txt` before uploading your data. Return to `interpretation.txt` and `narrative.txt` once you have the map.

- **preparation-notes.txt** — data validation findings, profile tables, standardisation recommendation with reason
- **interpretation.txt** — structured map reading: clusters, oppositions, centre vs periphery, white space
- **narrative.txt** — strategic narrative: key findings, per-item positioning statements, opportunities

---

## Execution Mode: AUTOPILOT

Run all stages in sequence without pausing unless:
- The data has a structural problem that requires a decision (e.g. negative values, empty rows)
- You reach Stage 3 and the user has not yet described or provided map output — pause and ask

---

## Stage 1: Receive & Validate Data

### Accepted input formats

Accept the contingency table in any of these forms:
- Pasted CSV (comma or tab delimited)
- Markdown table
- Plain description of the grid (e.g. "3 brands × 8 attributes, here are the values...")

The table must have:
- A header row naming the column items (brands, segments, markets, etc.)
- A label column naming the row items (attributes, drivers, barriers, etc.)
- All remaining cells must be numeric (counts, scores, ratings, or frequencies)

### Validation checklist

For each item below, report Pass, Warning, or Error:

| Check | Rule |
|-------|------|
| Structure | Has header row + label column |
| Cell type | All cells numeric |
| Negatives | No negative values (CA requires non-negative input) |
| Zeros | Flag if >25% of cells are zero (sparse data, map may be unreliable) |
| Size | At least 3 rows and 3 columns; warn if >30 rows or >10 columns (browser tool limit) |
| Empty | No fully empty rows or columns |

**If negatives are found:** CA cannot run on negative data. Ask the user whether to (a) shift all values by the minimum so the smallest becomes zero, or (b) take absolute values, or (c) they will recode manually. Do not proceed until resolved.

**If sparse (>25% zeros):** Warn but proceed. Note in `preparation-notes.txt`.

### Row and column profiles

Compute and display:

**Column profiles (% of each column's total):**
For each column, show each row value as a percentage of that column's sum.
Format as a table. This shows what each column item is *most associated with*.

**Column marginals:**
Show the grand total of each column as a % of the overall total.
Flag any column whose marginal exceeds 40% — that is the "big brand effect" signal.

**Row marginals:**
Show the grand total of each row as a % of the overall total.

---

## Stage 2: Standardisation Recommendation

Based on the column marginals, recommend one of three standardisation modes:

### Decision rules

**Raw (Standard CA):**
- Use when no single column dominates the marginals (all columns within 10–30% range)
- Best for: clean contingency tables where scale differences are small
- What it does: computes standard row and column profiles — each row and column is weighted by its total

**Double-centered (Equalised):**
- Use when one or more columns have marginals >40% — the "big brand effect"
- Best for: brand tracking where big brands have more responses just because they're bigger
- What it does: subtracts the row average and column average from each cell, removing volume effects so the map shows *distinctiveness* rather than *popularity*

**Z-score (Fully normalised):**
- Use when columns represent very different response pools or scales (e.g. one column is a niche segment with 50 respondents, another is mainstream with 5,000)
- Best for: multi-market or multi-segment data where volume differences are extreme
- What it does: standardises each row to zero mean and unit variance across columns, giving every row equal spread

### Output format for Stage 2 (in `preparation-notes.txt`)

```
STANDARDISATION RECOMMENDATION
================================
Recommended mode: [Raw / Double-centered / Z-score]
Reason: [1–2 sentences linking to the marginal data above]

In the browser tool, set the Standardisation dropdown to: [Raw / Equalised / Fully normalised]
```

---

## Stage 3: Interpret the Map

**Trigger:** User describes the map, pastes coordinate output, or provides a screenshot description.

If the user has not yet described the map after Stage 2 is complete, output:

```
Upload your data to the browser tool using the settings above. Once you have the map, 
describe what you see — which items appear close together, which are on opposite sides, 
which are near the centre vs. at the edges — and I'll interpret the positioning.

Or paste the inertia line (e.g. "Dim 1 = 52% | Dim 2 = 19% | Total = 71%") 
and any coordinates if available.
```

### Interpretation framework

When the user provides map information, work through these five lenses:

**1. Inertia (explanatory power)**
- If total inertia (Dim 1 + Dim 2) ≥ 70%: the 2D map is a reliable representation
- 50–70%: adequate, but note that ~30–50% of variation is not shown
- <50%: caution — the map may be misleading, a 3D analysis may be needed

**2. Clusters (what goes together)**
- Identify groups of row items that appear near the same column items
- Name each cluster by its dominant theme
- State what that cluster implies about the column items within it

**3. Oppositions (what contrasts with what)**
- Identify pairs of column items on opposite sides of the origin
- Identify row items that sit between two opposing columns — these are the "battleground" attributes
- State what the opposition axis represents

**4. Centre vs. periphery**
- Column items near the centre: close to average — not strongly differentiated on any attribute
- Column items at the edges: strongly distinctive — tightly associated with specific attributes
- Row items near the centre: apply broadly across all column items
- Row items at the edges: highly discriminating — strong signal for specific column items

**5. White space**
- Quadrants or areas of the map with no column items
- These represent attribute clusters that no current column item owns
- Flag as potential positioning opportunities

### Output format for `interpretation.txt`

```
MAP INTERPRETATION
==================

Inertia: [value]% — [reliable / adequate / caution note]

CLUSTERS
--------
[Cluster name]: [list of column items] are positioned near [list of row items]
Implication: [1 sentence]

[repeat for each cluster]

OPPOSITIONS
-----------
[Column A] vs [Column B]: separated along [describe axis]
Battleground attributes: [row items between them]
What this axis represents: [1 sentence]

CENTRE / PERIPHERY
------------------
Most central (average, undifferentiated): [items]
Most peripheral (strongly distinctive): [items] — associated with [attributes]

WHITE SPACE
-----------
[Describe unoccupied areas and the attributes that cluster there]
Opportunity: [1 sentence on what type of positioning could own this space]
```

---

## Stage 4: Strategic Narrative

Write `narrative.txt` in plain English, as if briefing a senior stakeholder who has not seen the map.

### Structure

```
STRATEGIC NARRATIVE
===================

KEY FINDINGS
------------
1. [Most important insight from the map — what the map is really saying at the top level]
2. [Second most important — usually the key opposition or dominant cluster]
3. [Third — white space, risk, or unexpected finding]
[4–5 optional if clearly warranted]

POSITIONING SUMMARY
-------------------
[Column item 1]: [1–2 sentences — where they sit, what they own, what they lack]
[Column item 2]: [1–2 sentences]
[repeat for each column item]

STRATEGIC IMPLICATIONS
----------------------
Opportunities: [What unowned positioning space exists and who could take it]
Risks: [Any column items that are undifferentiated or occupying contested space]
Next steps: [What analysis or research would sharpen the picture further]
```

---

## Quality Standards

- Never invent coordinates or positions not described or provided by the user
- If the user's description is ambiguous (e.g. "they seem close"), reflect the uncertainty in the interpretation ("appears to be associated with..." rather than "is strongly associated with...")
- Keep findings grounded in the data — every claim in the narrative should trace back to a specific map feature
- Flag if the map has fewer than 3 meaningful dimensions (e.g. all items cluster in one quadrant) — the tool may need richer data
- Preserve outlier findings — if one item is far from all others, note it explicitly rather than ignoring it

---

## About This Skill

Part of the **Research Edge Series** — rigorous methodology education for social scientists and market researchers.

Browser tool: [github.com/vtmade/research-edge-series](https://github.com/vtmade/research-edge-series) → `toolkit/correspondence_map.html`  
Series: [github.com/vtmade/research-edge-series](https://github.com/vtmade/research-edge-series)  
Author: Vinay Thakur — [@vtmade](https://github.com/vtmade)
