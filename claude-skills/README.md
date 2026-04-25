# Claude Skills

This directory contains Claude Skills developed as part of the Research Edge Series.

## Available Skills

### Qualitative Analysis Skill
**File:** `qual-analysis.zip`
**Referenced in:** [Article #005 — AI and the Qualitative Analysis Problem](../articles/005-ai-qualitative-analysis.md)

A structured qualitative analysis pipeline that runs open coding, theme construction, quote selection, pattern detection, and quality checks — with full traceability back to source text.

**What it produces:**
- Coded segments with hierarchical tags
- Theme definitions with inclusion/exclusion boundaries
- Verified quotes mapped to source locations
- Six-lens pattern analysis (similarity, difference, frequency, sequence, co-occurrence, cause)
- Confidence labels and outlier preservation

**How to install:**
1. Open Claude
2. Go to **Settings → Capabilities → Skills**
3. Upload the Skill file from this directory
4. Provide your qualitative data and ask for analysis

### Correspondence Analysis Skill
**File:** `correspondence-analysis.zip`

Companion skill for the browser-based Correspondence Analysis Map. Helps Claude reason about brand × attribute biplots, choose appropriate standardisation, and interpret the resulting clusters and white-space.

---

## GenAI on Quantitative Survey Data — workflow skills

Three composable Claude Code skills that go with [Article #006 — A Working Framework for Using GenAI on Quantitative Survey Data](../articles/006-genai-quant-survey-workflow.md). Together they cover prep through analysis: the LLM interprets, Python computes, the analyst verifies.

### flatten-crosstab
**File:** `flatten-crosstab.zip`
**Pairs with:** Path A in the article

Converts agency crosstab deliverables (XLSX, XLS, CSV) into a flat, self-describing format. The skill inspects the file, confirms the structure with you, then runs Python to flatten deterministically. Outputs long and wide flat files, a data dictionary, and a four-check validation report.

*Use this when a crosstab has already arrived and you need to get it into a shape an LLM can read without losing meaning.*

### extract-crosstabs
**File:** `extract-crosstabs.zip`
**Pairs with:** Path C in the article

Generates flat-format crosstabs directly from raw respondent-level data (`.sav`, `.csv`, `.xlsx`). Supports all five question types, analyst-driven weighting, conditional bases, custom NETs, and optional significance testing. Outputs the flat file plus a formatted crosstab Excel — chainable into `flatten-crosstab` if you need both forms.

*Use this when you only have raw data and need to produce the standard cross-cut tables yourself.*

### tidy-data-analysis
**File:** `tidy-data-analysis.zip`
**Pairs with:** the analysis stage of the article

Picks up where the prep stack ends. Works through research objectives interactively — proposes analytical moves, runs them deterministically, and helps you pin each finding to its supporting evidence. Auto-runs four sanity checks per finding. Exports findings together with the tables behind them, and sessions resume across sittings.

*Use this when the data is clean and you need help going from a flat file to a defensible set of findings.*

**Companion specification:** the [flat-format DP delivery spec](../assets/006-dp-flat-format-spec.md) defines the same flat output shape that data processing teams can produce at source — useful when you want the format upstream rather than after the fact.

**Install (Claude Code skills):**
1. Download the relevant `.zip` from this directory
2. Locate your `~/.claude/skills/` directory
3. Unzip the package into that folder so the skill is registered
4. Hand Claude your input file and invoke the skill

---

*Skills are open source and free to use. See [github.com/vtmade](https://github.com/vtmade) for more.*
