# Flat-format crosstab delivery spec

A specification for data processing teams delivering quantitative survey crosstabs in flat format, alongside or instead of traditional layout-heavy crosstabs.

Version 1.0 · April 2026

---

## Why this spec exists

Traditional crosstab deliverables are built for human reading — banner rows at the top, questions and response options stacked in column A, merged headers, base sizes tucked into footers. They work for people but break for almost everything else: LLMs misread the banner structure, scripts need custom parsing for every template, and the same file can't easily be used twice for different analyses.

This spec describes a second deliverable — a **flat file** — that sits alongside the traditional crosstab. The flat file carries the same data in a shape that is unambiguous for machine reading, consistent across studies, and reusable across analyses. Most DP platforms (SPSS, Q, Quantum, Wincross, Askia, custom pipelines) can export this format with a small amount of configuration work. Once set up, it adds minutes to a delivery rather than hours.

The spec is intentionally neutral on tools, naming conventions, and internal workflows. What matters is the shape of the data and the semantics of each column — not what you call things inside your system.

---

## What this spec applies to

Aggregate survey crosstabs: tables where cells contain percentages, means, counts, or indices computed across a banner.

It does not apply to:

- Respondent-level raw data (SPSS, SAV, CSV with one row per respondent)
- Qualitative data
- Tracking dashboards or BI exports

For those, separate specifications apply.

---

## The flat format — conceptually

A flat file represents every cell of a crosstab as a self-describing row. Every row carries enough information to stand alone without reference to layout, headers, or footers.

A cell in a traditional crosstab sits at the intersection of:

- A question (e.g., "Awareness of Brand X")
- A response option (e.g., "Top of mind")
- A banner cut (e.g., Gender × Male)

In the flat format, those three dimensions become columns, and the cell value is its own column. One cell becomes one row.

Two output shapes are expected, both derived from the same data:

- **Long format** — one row per cell (question × response × banner cut). Preferred for LLM reasoning, database loading, and any downstream code-based analysis.
- **Wide format** — banner cuts as columns, question × response as rows. Preferred for human inspection and wave-on-wave comparison.

Both files contain the same information. The DP team produces both; the analyst picks the one that fits the task.

---

## Required semantics

The following fields must be present in the long format. Column names can follow your existing conventions — what matters is that each piece of information is present and unambiguous.

Any column you already produce that carries the same semantic content (e.g., you call `question_id` something else) is acceptable. The column-name mapping should be documented once, in the methodology note.

### Fields required in every row

| Semantic content | Purpose |
|---|---|
| **Sheet / table source** | Which sheet or table in the original deliverable this row came from. Enables traceability back to the source document. |
| **Question identifier** | A short code for the question (`Q1`, `Q2a`, `S3_Q4`, whatever convention is in use). If the study does not use identifiers, auto-generate them sequentially (`Q1`, `Q2`, ...). |
| **Question text** | The verbatim question as it appeared to the respondent or as labelled in the crosstab. |
| **Row type** | Classification of what kind of row this is. Strict semantic set (see Row type values below); your labels can differ as long as the mapping is consistent and documented. |
| **Response option** | The row label as it appeared in the crosstab (e.g., "Top of mind", "Very satisfied", "NET Top 2", "Mean"). |
| **Banner group** | The banner family this cut belongs to (Total, Gender, Age, Market, Segment, etc.). |
| **Banner value** | The specific cut within the group (Total, Male, Female, 18–24, 25–34, UAE, KSA, etc.). |
| **Value** | The numeric value in the cell. Strictly numeric — no `%` symbols, no significance letters, no footnote markers. |
| **Value type** | What kind of measure the value represents. Strict semantic set: `percent`, `mean`, `median`, `count`, `index`. |
| **Base n** | The base size for this banner value. Required in every row, even though it repeats — the redundancy enables row-level audit without secondary lookups. |

### Fields recommended but optional

| Semantic content | When to include |
|---|---|
| **Significance markers** | If significance testing is applied, preserve the marker letters (`A`, `AB`, `ab`) in a separate column. The `value` column stays purely numeric. |
| **Base label** | A short description of the base (e.g., "All respondents", "Aware of brand"), useful when multiple bases apply within the same study. |
| **Wave** | For tracking studies, the wave identifier. |

---

## Row type values

The row type distinguishes what kind of aggregation the row represents. This is critical: treating a NET row as a response row will break downstream analysis.

The required semantic set:

| Semantic | Meaning |
|---|---|
| `response` | An individual response option (e.g., "Very satisfied", "Top of mind") |
| `net` | A NET (e.g., "NET Top 2", "NET Aware", "NET Agreement") |
| `subtotal` | A grouped sub-total that is not a NET (e.g., a "Total aware" row that sums components) |
| `mean` | A mean or average row |
| `median` | A median row |

Your internal labels can differ (you may call them `summary_type`, `cell_category`, or something else). What must not differ is the meaning — every row must be classifiable into exactly one of these five categories, and the mapping must be documented.

If a row does not fit any of these categories, flag it in the methodology note rather than force-fitting.

---

## Value conventions

### Numeric format

- Percentages expressed as numbers in the range 0–100 (14 for 14%), not as decimals (0.14). Pick one convention and apply it across the study.
- Means, medians, indices: expressed in their natural scale (e.g., 6.8 on a 10-point scale, not 0.68).
- Counts: integer.
- No currency symbols, percent signs, or thousands separators embedded in the value.

### Null handling

- Empty cells in the source → null in the flat file (not 0).
- `-`, `*`, `N/A`, `–` → null.
- Never use sentinel values like `-999` to represent missing.

### Significance markers

- Strip from the value column.
- If preserved, store in a separate significance column (e.g., `sig_markers`).
- Never leave letter suffixes attached to numeric values (`14A` is not a number).

---

## Output deliverables

### Required files

Two files per study deliverable:

1. **Flat file — long format**
   - One row per cell
   - Contains all fields described above
   - Format: **CSV** (UTF-8 with BOM for Excel compatibility) **and Excel (.xlsx)**

2. **Flat file — wide format**
   - Banner cuts pivoted to columns
   - Each banner cut becomes one column (naming convention at DP team's discretion)
   - Format: **CSV and Excel**

Both files carry the same underlying data. The wide format is derived from the long format — they should reconcile exactly.

### Optional companion artefacts

On request only:

- **Data dictionary** — column names, semantics, allowed values
- **Banner plan** — expected banner groups, banner values, and base sizes

These are useful for large studies or when the study is being handed to an external analytics team. For most routine deliveries, they're not required.

### Methodology note

A short plain-text note accompanying the flat files, covering:

- Weighting scheme (if applied) and what the weights represent
- Base definitions (total, screened, conditional) and how they map to the banner cuts
- Significance testing scheme (if applied) — what the letters mean, what tests, what thresholds
- Any row-type taxonomy your team uses that differs from the standard set above
- Any study-specific conventions worth noting

This note is where all the context that doesn't belong in the data itself should live.

---

## File naming

Flexible, but a consistent pattern helps. A workable convention:

```
<study_id>_<wave>_flat_long.csv
<study_id>_<wave>_flat_wide.csv
<study_id>_<wave>_methodology.md
```

Example:

```
BrandTracker_Q3_2025_flat_long.csv
BrandTracker_Q3_2025_flat_wide.csv
BrandTracker_Q3_2025_methodology.md
```

---

## Example — how a crosstab fragment becomes flat

### Source crosstab

```
                         Total    Male    Female   18-24   25-34
                        (n=1000) (n=500) (n=500)  (n=200) (n=250)
Q1. Awareness of Brand X
  Top of mind              12      14      10       8      13
  Unaided                  34      36      32      28      35
  Aided                    71      73      69      65      72
  NET Aware                71      73      69      65      72
```

### Long format

```csv
source_sheet,question_id,question_text,row_type,response_option,banner_group,banner_value,value,value_type,base_n
Brand_Awareness,Q1,Q1. Awareness of Brand X,response,Top of mind,Total,Total,12,percent,1000
Brand_Awareness,Q1,Q1. Awareness of Brand X,response,Top of mind,Gender,Male,14,percent,500
Brand_Awareness,Q1,Q1. Awareness of Brand X,response,Top of mind,Gender,Female,10,percent,500
Brand_Awareness,Q1,Q1. Awareness of Brand X,response,Top of mind,Age,18-24,8,percent,200
Brand_Awareness,Q1,Q1. Awareness of Brand X,response,Top of mind,Age,25-34,13,percent,250
Brand_Awareness,Q1,Q1. Awareness of Brand X,response,Unaided,Total,Total,34,percent,1000
Brand_Awareness,Q1,Q1. Awareness of Brand X,response,Unaided,Gender,Male,36,percent,500
Brand_Awareness,Q1,Q1. Awareness of Brand X,response,Unaided,Gender,Female,32,percent,500
Brand_Awareness,Q1,Q1. Awareness of Brand X,response,Unaided,Age,18-24,28,percent,200
Brand_Awareness,Q1,Q1. Awareness of Brand X,response,Unaided,Age,25-34,35,percent,250
Brand_Awareness,Q1,Q1. Awareness of Brand X,response,Aided,Total,Total,71,percent,1000
...
Brand_Awareness,Q1,Q1. Awareness of Brand X,net,NET Aware,Total,Total,71,percent,1000
Brand_Awareness,Q1,Q1. Awareness of Brand X,net,NET Aware,Gender,Male,73,percent,500
...
```

Every row is self-describing. The banner structure is no longer carried by layout — it's carried by two columns (`banner_group` and `banner_value`). The NET row is flagged with `row_type = net`, not treated as a response.

### Wide format

```csv
source_sheet,question_id,question_text,row_type,response_option,value_type,Total_Total,Gender_Male,Gender_Female,Age_18-24,Age_25-34,Total_Total_base_n,Gender_Male_base_n,Gender_Female_base_n,Age_18-24_base_n,Age_25-34_base_n
Brand_Awareness,Q1,Q1. Awareness of Brand X,response,Top of mind,percent,12,14,10,8,13,1000,500,500,200,250
Brand_Awareness,Q1,Q1. Awareness of Brand X,response,Unaided,percent,34,36,32,28,35,1000,500,500,200,250
Brand_Awareness,Q1,Q1. Awareness of Brand X,response,Aided,percent,71,73,69,65,72,1000,500,500,200,250
Brand_Awareness,Q1,Q1. Awareness of Brand X,net,NET Aware,percent,71,73,69,65,72,1000,500,500,200,250
```

Same data. More compact for reading. The column-naming pattern here uses `<banner_group>_<banner_value>` and `<banner_group>_<banner_value>_base_n`, but any consistent pattern the DP team prefers is acceptable.

---

## Frequently asked questions

### Does this replace the traditional crosstab deliverable?

No. Teams can continue producing the traditional layout-heavy crosstab for stakeholders who read it that way. The flat file sits alongside as a second deliverable, derived from the same underlying tabulation.

### Why require base_n in every row if it repeats?

Because the primary consumers of the flat file — LLMs and downstream code — work row by row. Putting `base_n` on every row means any row can be audited or filtered without a secondary lookup. The storage cost is trivial; the robustness gain is substantial.

### What about multi-response questions where percentages don't sum to 100?

No special handling required in the flat file itself — each response row carries its own value. Downstream validation logic (e.g., "this question is multi-response, skip the sum-to-100 check") uses the question text, which is preserved in every row.

### What if our platform can't easily produce both long and wide?

Long format is the priority. If producing both is genuinely difficult, long format alone satisfies the core requirement — wide can be derived from long trivially with a single pandas pivot or Excel PivotTable. The analyst can do this themselves if needed.

### How do we handle conditional bases (questions asked to sub-samples)?

The `base_n` in each row reflects the *actual* base for that banner cut, not the total sample. If Q5 was asked only to respondents who said "yes" to Q4, the `base_n` for Q5's rows is the count of "yes" respondents within each banner cut. Document this in the methodology note.

### What about tables that aren't survey questions — significance tables, sample composition summaries, etc.?

Exclude them from the flat file. They're part of the documentation deliverable, not the data. The methodology note can reference them.

### Can we use different row type labels than the five listed?

The semantics (five categories) are fixed. The labels are flexible. If your system uses `summary_kind` with values `pct / NET / avg / med / sub`, that's fine — as long as the mapping to the five semantic categories is documented once in the methodology note.

### What encoding should the CSVs use?

UTF-8 with BOM. The BOM ensures Excel opens the file correctly with non-Latin characters preserved. Without it, Excel on some platforms will mis-decode Arabic, Cyrillic, and other non-Latin scripts.

### How do we handle decimals — period or comma?

Period (`.`). Thousands separators should not be embedded in the value column. Regional formatting is a display concern, not a data concern.

### What if the analyst finds an error in the flat file?

Treat it the same way you'd treat an error in the traditional crosstab. The flat file is derived from the same tabulation, so an error there typically reflects an error upstream. Fix at source, regenerate both deliverables.

---

## Version history

| Version | Date | Changes |
|---|---|---|
| 1.0 | April 2026 | Initial spec |

---

## Contact and feedback

This spec is a working standard. Refinements welcome. If a requirement here is genuinely hard to meet in a specific platform or workflow, the most useful thing is a concrete example — which part breaks, for which study, with what data — so the spec can evolve to accommodate it without losing rigour.
