# #006: A Working Framework for Using GenAI on Quantitative Survey Data

## Why quantitative survey data is hard for LLMs

The usual complaints are context and scale. The model doesn't know enough about the study. The dataset is too large for a prompt. Both are real, and both are solvable — context can be added through supporting documents, and data can be handled in chunks.

The harder problem is the shape of the data itself. There are two versions of this problem, one at each end of the spectrum.

**Raw respondent files don't tell the model what it's looking at.**

A variable labelled `q4_1` could be any of the following:

- A mean score on a rating scale
- A frequency count from a single-response question
- A rank position from a sequencing exercise
- A screening flag

An experienced analyst would open the questionnaire and check. An LLM won't. It produces a plausible table and gives no indication of which interpretation it used. Bases get applied to the wrong universe. Multi-response questions get collapsed into single-response distributions. Ranked items get treated as categorical.

**Crosstabs look safer because they already resemble tables.**

But a crosstab's meaning is carried by its *layout*. Merged headers get flattened into confusion. Base sizes sit in footers and disappear. The value in any cell depends on which column sits above it and which question sits to its left — and the model doesn't reliably track either.

Quantitative work needs three properties from any workflow:

> → **Verifiability** — any number can be checked against its source
> → **Validity** — the computation is correct
> → **Consistency** — same input, same output

Pasting raw data or a crosstab into an LLM gives you none of them. This isn't a general problem with LLMs — they handle summarisation and qualitative coding well. It's specific to aggregate quantitative work, which is exactly where most teams are most eager to apply them.

---

## The fix is a workflow, not a tool

<figure class="article-aside"><span class="article-aside-label">Roles</span><h4 class="article-aside-title">LLM interprets, Python computes, the analyst verifies</h4><svg viewBox="0 0 240 250" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Workflow diagram"><rect x="50" y="8" width="140" height="50" rx="6" fill="#ffffff" stroke="#c4703f" stroke-width="1.5"/><text x="120" y="30" text-anchor="middle" font-family="Inter,sans-serif" font-size="13" font-weight="600" fill="#191919">LLM</text><text x="120" y="46" text-anchor="middle" font-family="Inter,sans-serif" font-size="10" fill="#555">interprets &amp; drafts</text><line x1="120" y1="62" x2="120" y2="88" stroke="#888" stroke-width="1.2"/><polygon points="120,93 115,86 125,86" fill="#888"/><rect x="50" y="96" width="140" height="50" rx="6" fill="#ffffff" stroke="#c4703f" stroke-width="1.5"/><text x="120" y="118" text-anchor="middle" font-family="Inter,sans-serif" font-size="13" font-weight="600" fill="#191919">Python</text><text x="120" y="134" text-anchor="middle" font-family="Inter,sans-serif" font-size="10" fill="#555">executes deterministically</text><line x1="120" y1="150" x2="120" y2="176" stroke="#888" stroke-width="1.2"/><polygon points="120,181 115,174 125,174" fill="#888"/><rect x="50" y="184" width="140" height="50" rx="6" fill="#191919"/><text x="120" y="206" text-anchor="middle" font-family="Inter,sans-serif" font-size="13" font-weight="600" fill="#ffffff">Analyst</text><text x="120" y="222" text-anchor="middle" font-family="Inter,sans-serif" font-size="10" fill="#bbbbbb">verifies &amp; judges</text></svg><p class="article-aside-caption">The order matters. Plan and draft first, run the code second, hold the judgement at the end.</p></figure>

When this starts going wrong, the instinct is to pick one tool. Either hand the whole problem to the LLM and hope better prompting solves it, or work only in Python.

Both fail. The LLM on its own is inconsistent from run to run. Python on its own needs new code every time a crosstab template changes.

The better approach treats the two as complementary.

> **The LLM interprets. Python computes.**

The LLM reads the file's structure, drafts the transformation code, and helps articulate findings in plain language. Python runs the code, produces the numbers, and holds the data in a form that can be inspected.

The order matters: the LLM plans and drafts → Python executes → the analyst verifies.

Commercial tools are starting to package this pattern into platforms. Whether you build or buy, understand the mechanics first — it's easier to evaluate a product when you know what it should be doing underneath.

---

## Flat data: the shape that makes this work

Flat data is not the whole solution, but it is the shape that makes the rest of the workflow possible.

In the context of quantitative survey analysis, "flat" has a specific meaning:

- **Columns** hold the banner cuts — the profile dimensions, segments, and independent variables being compared across
- **Rows** hold each question paired with each response option
- **Cells** hold the value — a percentage, a mean, or an index

Every row describes itself. There are no merged cells, no indented sub-rows, and no layout the reader has to interpret.

The contrast is easiest to see with an example.

**Before — a typical crosstab fragment:**

```
                    Total    Male    Female   18-24   25-34
Awareness of Brand
  Top of mind         12      14       10       8      13
  Unaided             34      36       32      28      35
```

**After — flattened:**

```
question              response      banner_group   banner_value   value
Awareness of Brand    Top of mind   Total          Total          12
Awareness of Brand    Top of mind   Gender         Male           14
Awareness of Brand    Top of mind   Gender         Female         10
Awareness of Brand    Top of mind   Age            18-24           8
Awareness of Brand    Top of mind   Age            25-34          13
Awareness of Brand    Unaided       Total          Total          34
```

The flattened version is longer, but every row is unambiguous — both to the LLM reading it and to Python computing on it. Everything downstream becomes more straightforward.

---

## The framework: Prep → Validation → Analysis

Three stages, in order. The LLM supports each stage but does not drive any of them.

### Prep — where most of the work sits

<figure class="article-aside"><span class="article-aside-label">Decision</span><h4 class="article-aside-title">Three prep paths, one per starting point</h4><svg viewBox="0 0 260 380" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Decision tree of three prep paths"><rect x="50" y="6" width="160" height="44" rx="6" fill="#191919"/><text x="130" y="26" text-anchor="middle" font-family="Inter,sans-serif" font-size="11" font-weight="600" fill="#ffffff">What are you</text><text x="130" y="40" text-anchor="middle" font-family="Inter,sans-serif" font-size="11" font-weight="600" fill="#ffffff">starting with?</text><line x1="130" y1="50" x2="130" y2="78" stroke="#888" stroke-width="1.2"/><polygon points="130,82 126,76 134,76" fill="#888"/><rect x="20" y="85" width="220" height="78" rx="6" fill="#ffffff" stroke="#c4703f" stroke-width="1.5"/><text x="130" y="105" text-anchor="middle" font-family="Inter,sans-serif" font-size="10" fill="#555">A crosstab from the agency</text><text x="130" y="119" text-anchor="middle" font-family="Inter,sans-serif" font-size="10" fill="#555">or DP team</text><line x1="40" y1="129" x2="220" y2="129" stroke="#efefef"/><text x="130" y="146" text-anchor="middle" font-family="Inter,sans-serif" font-size="12" font-weight="700" fill="#c4703f">Path A &middot; Flatten</text><text x="130" y="158" text-anchor="middle" font-family="Inter,sans-serif" font-size="9" fill="#191919">convert layout into flat format</text><line x1="130" y1="170" x2="130" y2="186" stroke="#888" stroke-width="1.2"/><polygon points="130,190 126,184 134,184" fill="#888"/><rect x="20" y="193" width="220" height="78" rx="6" fill="#ffffff" stroke="#c4703f" stroke-width="1.5"/><text x="130" y="213" text-anchor="middle" font-family="Inter,sans-serif" font-size="10" fill="#555">Raw data &amp; multivariate</text><text x="130" y="227" text-anchor="middle" font-family="Inter,sans-serif" font-size="10" fill="#555">analysis ahead</text><line x1="40" y1="237" x2="220" y2="237" stroke="#efefef"/><text x="130" y="254" text-anchor="middle" font-family="Inter,sans-serif" font-size="12" font-weight="700" fill="#c4703f">Path B &middot; Block extract</text><text x="130" y="266" text-anchor="middle" font-family="Inter,sans-serif" font-size="9" fill="#191919">minimum viable dataset</text><line x1="130" y1="278" x2="130" y2="294" stroke="#888" stroke-width="1.2"/><polygon points="130,298 126,292 134,292" fill="#888"/><rect x="20" y="301" width="220" height="78" rx="6" fill="#ffffff" stroke="#c4703f" stroke-width="1.5"/><text x="130" y="321" text-anchor="middle" font-family="Inter,sans-serif" font-size="10" fill="#555">Raw data &amp; need to produce</text><text x="130" y="335" text-anchor="middle" font-family="Inter,sans-serif" font-size="10" fill="#555">cross-cut tables yourself</text><line x1="40" y1="345" x2="220" y2="345" stroke="#efefef"/><text x="130" y="362" text-anchor="middle" font-family="Inter,sans-serif" font-size="12" font-weight="700" fill="#c4703f">Path C &middot; Generate</text><text x="130" y="374" text-anchor="middle" font-family="Inter,sans-serif" font-size="9" fill="#191919">tabulate to flat format</text></svg><p class="article-aside-caption">Pick the path that matches the input. Crosstab in hand &rarr; A. Multivariate work &rarr; B. Need the tables yourself &rarr; C.</p></figure>

Preparation is where the craft lives, and it splits into three paths depending on what you're starting with and what the analysis needs.

**Path A → Flattened crosstabs.**

This is the default for most quantitative work, *assuming you are working with crosstabs to begin with.* The crosstab arrives from the agency or the DP team. You convert it into the flat format shown above — either by running a script yourself or by specifying the format at source. From that point on, the flat file is the single source for the study: the LLM reads it, Python computes on it, and the analyst can trace any number back to a cell.

If you are working from raw respondent data instead of a crosstab, skip to Path C before coming back to this stage.

**Path B → Block extraction from raw data.**

This path is reserved for analyses that a crosstab cannot carry — driver analysis, regression, segmentation, clustering, factor analysis, and CFA. Anything multivariate. Anything that needs respondent-level correlations. Anything that builds a derived measure.

The important move here is *not* to load the full dataset. You extract the specific block of questions the analysis requires, at respondent level, with labels and codes intact. A minimum viable dataset, shaped precisely for the question being asked.

**Path C → Cross-tab extraction from raw data.**

This path applies when the only thing you receive is raw respondent data and you need to produce the standard cross-cut tables yourself. For any study of real size, this is the most challenging preparation work — and the one most prone to silent error.

It requires rigour at every step:

- Identifying which variables belong in the banner and which in the rows
- Applying the correct base for each question (total, screened-in, conditional)
- Carrying weights through consistently
- Handling multi-response, ranked, and scale questions with the right aggregation method
- Producing a flat-format table at the end, not a layout-heavy crosstab

> The principle across all three paths: **shape the data for the question, not the question for the data.**

### Validation — light touch, non-negotiable

Validation lives inside the prep code as automated checks. The specific checks vary by study, but common examples include:

- Base size reconciliation against the expected n
- Percentages within each banner cut summing to 100 (±1)
- Type and range integrity on numeric fields

If a check fails, the prep is wrong and the analysis does not start. Which checks matter for which study is a judgement that stays with humans.

### Analysis — where the LLM earns its place

Once the input is clean and validated, the LLM becomes genuinely useful in three modes:

- **Writing code** for specific cuts and pivots the analyst specifies
- **Interpreting patterns** when pointed at a defined comparison
- **Drafting findings** in plain language for a deck or memo

The analyst sets the question and the frame. The LLM works inside it. Every number traces back to a cell, and every step can be re-run without producing different results the second time.

---

## Tools & solutions

Four artefacts make this framework runnable end to end. They are designed to compose: prep upstream, analysis downstream, with the flat format as the contract between them. Each links through to its full detail page.

### [flatten-crosstab](../claude-skills/flatten-crosstab.zip) — Claude Code skill

Converts agency crosstab deliverables (XLSX, XLS, CSV) into the flat format described above. The skill inspects the file, confirms the structure with you, then runs Python to flatten deterministically. Outputs long and wide flat files, a data dictionary, and a four-check validation report.

*Use this when a crosstab has already arrived and you need to get it into a shape an LLM can read without losing meaning. Pairs with Path A.*

[Get the skill →](../claude-skills/flatten-crosstab.zip)

### [Flat-format delivery spec](../assets/006-dp-flat-format-spec.md) — DP instruction set

A version-controlled markdown specification any data processing team can work from. It defines the flat output shape semantically — required columns, row-type taxonomy, value conventions, encoding — while leaving naming and internal workflow flexible. Includes worked examples and an FAQ covering common edge cases. Platform-agnostic and independently shareable.

*Use this when you want the flat format produced at source, before the file ever reaches your hands. Pairs with Path A.*

[Get the spec →](../assets/006-dp-flat-format-spec.md)

### [extract-crosstabs](../claude-skills/extract-crosstabs.zip) — Claude Code skill

Generates flat-format crosstabs directly from raw respondent-level data (`.sav`, `.csv`, `.xlsx`). Supports all five question types, analyst-driven weighting, conditional bases, custom NETs, and optional significance testing. Outputs the flat file plus a formatted crosstab Excel — chainable into `flatten-crosstab` if you need both forms.

*Use this when you only have raw data and need to produce the standard cross-cut tables yourself. Pairs with Path C.*

[Get the skill →](../claude-skills/extract-crosstabs.zip)

### [tidy-data-analysis](../claude-skills/tidy-data-analysis.zip) — Claude Code skill

Picks up where the prep stack ends. Works through research objectives interactively — proposes analytical moves, runs them deterministically, and helps you pin each finding to its supporting evidence. Auto-runs four sanity checks per finding. Exports findings together with the tables behind them, and sessions resume across sittings.

*Use this when the data is clean and you need help going from a flat file to a defensible set of findings.*

[Get the skill →](../claude-skills/tidy-data-analysis.zip)

---

Some teams will prefer the flattening skill for speed and control. Others will prefer the DP spec for scale and repeatability. Teams working from raw data will need the extraction skill regardless — it's the most demanding of the four and the one where rigour matters most. The analysis skill works on top of any of them, once the data is in shape.

All four artefacts, along with the supporting documentation and code, are available from the [claude-skills directory](../claude-skills/) of this site.
