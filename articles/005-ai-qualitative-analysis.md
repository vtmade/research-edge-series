# #005: AI and the Qualitative Analysis Problem

**Why Most AI Tools Flatten Qualitative Data, and What Rigorous Analysis Actually Requires**

*Research Edge Series · By Vinay Thakur*

---

## The Problem in One Sentence

Most attempts to use AI for qualitative analysis produce a tidy summary that looks useful, reads cleanly, and falls apart the moment a researcher needs to defend it.

This article explains why that happens, what proper qualitative analysis actually involves, and how to use AI in a way that respects the method instead of replacing it with a confident-sounding paragraph.

---

## Why This Matters

Qualitative research has always carried a particular kind of weight. Unlike survey statistics, where the math is the math, qualitative findings depend on the analyst's discipline. A theme is only as trustworthy as the coding that produced it. A quote is only as defensible as the segment it was drawn from. Strip away that traceability and you're left with assertion.

When researchers began experimenting with large language models on transcripts, focus group recordings, and open-ended survey data, the appeal was obvious. The work is slow. The cognitive load is heavy. A tool that could read forty interviews in seconds and surface "the main themes" felt like a long-overdue gift.

The output, however, has rarely held up to scrutiny. And the reason is not that AI is incapable of qualitative reasoning — it is that the way most people use it skips every safeguard the method was built around.

---

## What "Qualitative Analysis" Actually Means

Before we can discuss where AI fails, we need a shared definition of what qualitative analysis actually is.

Qualitative analysis is the disciplined process of making sense of unstructured human language — interviews, focus group exchanges, open-ended survey responses, online discussions, observational notes. The goal is not to summarise what people said. The goal is to surface the patterns, structures, and contradictions inside what they said, in a way that another researcher could audit, challenge, or extend.

The field has spent decades developing methods for this. Two of the most widely used:

- **Grounded Theory** (Glaser & Strauss, 1967), which builds analytical categories inductively from the data itself, rather than imposing a pre-existing framework.
- **Thematic Analysis** (Braun & Clarke, 2006), which provides a structured six-phase process for identifying, organising, and reporting themes across a dataset.

Both methods share a core commitment: every claim must trace back to evidence, and every interpretation must be defensible against the raw text.

---

## What Goes Wrong by Default

When someone pastes a transcript into a generic chatbot and asks for "the key themes," the response usually looks like this:

> The participants expressed concerns about pricing, valued reliability, and emphasised the importance of customer service. There was a recurring theme of frustration with onboarding...

This reads well. It is also analytically useless. Here's why.

**It produces a summary, not an analysis.** A summary collapses information. Analysis decomposes it, traces it, and reorganises it into patterns. A summary tells you what was roughly there. Analysis tells you what the data actually shows when you look closely.

**It cannot be traced.** There is no way to know which segments produced which claim. If a stakeholder asks "where did this theme come from?", there is no answer.

**It cannot be audited.** If a finding feels wrong, there is no path back through the reasoning. The model's interpretation is opaque.

**It cannot feed the next step.** Real analysis is iterative. You take the codes, regroup them, check them against patterns, write the report. A summary paragraph terminates the workflow. There is nothing downstream to do with it.

**It looks clean. It isn't useful.**

---

## The Method That Actually Works

Rigorous qualitative analysis follows a sequence. Each stage has its own rules, its own outputs, and its own quality checks. The shape of the work is roughly:

**Code → Theme → Pattern → Cross-case**

Skip a stage and the findings collapse. Compress two stages into one and you lose the discipline that makes the result defensible.

### Step 1 — Coding and Grouping

Coding is the foundational act. It means tagging every small chunk of text — a sentence, a turn, a comment — with what it is about. Codes are usually written hierarchically, in the form `DOMAIN/CATEGORY/SUBCATEGORY`. For example: `EXPERIENCE/ONBOARDING/FRICTION`.

The discipline of good coding is to tag what was *said*, not what you assume it means.

> ✓ "Describes pricing as confusing"  
> ✗ "Frustrated with pricing"

The second is interpretation dressed as evidence. The participant did not say they were frustrated. An analyst inferred it. That inference may be correct, but it does not belong in the code itself.

Once a corpus has been coded, similar codes are grouped into **themes** — broader analytical categories that show up across multiple participants. Each theme requires a clear *boundary*: what it includes, and just as importantly, what it deliberately excludes. Loose boundaries are where most analyses go soft.

### Step 2 — Patterns and Cross-Checks

Once themes exist, the analyst reads the data through several pattern lenses. Six are commonly used:

1. **Similarity** — what do participants share, especially across different contexts?
2. **Difference** — where do views diverge, and what moderating factors might explain it?
3. **Frequency** — how often does a theme appear, distinguishing breadth (how many participants) from depth (how often each one returns to it)?
4. **Sequence** — what follows what? Journey stages, escalation, decision pathways.
5. **Co-occurrence** — which themes appear together, and is that pairing surprising?
6. **Cause** — what appears to drive what? Critically, every causal claim must be labelled as either *participant-reported* (they said it) or *analyst-inferred* (you concluded it). That distinction separates research from storytelling.

Cross-case analysis then examines how themes distribute across participants, which respondents fit the dominant pattern, and — most importantly — which ones don't. The outliers are often the most analytically valuable people in the dataset, because they reveal what actually drives the pattern when it's present.

---

## Where Most AI Tools Break

When measured against this method, almost all general-purpose AI tools fail in four predictable ways.

**Quotes are not verified against the source.** The model paraphrases, smooths, or fabricates quotations that sound like the original but never existed in it.

**There are no confidence levels.** A weakly supported claim and a strongly supported one are presented with identical certainty.

**Outliers get ignored.** The handful of participants who don't fit the pattern — the most analytically valuable people in the dataset — vanish into the averaging.

**Nothing is traceable.** There is no path from a theme back to the segments that produced it, and no way to audit how a finding was reached.

If you cannot audit a finding, you cannot defend it. And if you cannot defend it, you should not ship it.

---

## A Different Approach: Structured Analytical Output

The fix is not to abandon AI for qualitative work. It is to use AI the way the method requires: as a disciplined coding and pattern-detection layer, with structured, auditable output.

I have built a Claude Skill that does exactly this. It runs the full sequence — open coding, theme construction, quote selection, the six pattern checks, and quality flags — across whatever text data you give it. Interviews, focus groups, open-ended survey responses, online discussions.

The critical design choice is what it produces. The output is **not** a written summary. It is structured data — every coded segment, every theme definition, every quote, every pattern, with complete traceability back to the original text. Each finding carries a confidence label. Outliers are preserved, not averaged away.

Why structured output instead of a polished report? Because the analysis is the *raw material*, not the finished thing. Once you have it, you can do whatever you need with it:

- Write the report yourself, using the structured findings as evidence
- Hand the structured data to another AI layer to draft a stakeholder deck or executive summary
- Compare findings across studies, because the structure is consistent
- Audit any claim, because every claim points back to its source

One rigorous pass. Many downstream uses. The structure stays yours.

---

## How to Use the Skill

The Skill is open source and free on my GitHub. To install it in Claude:

1. Open Claude
2. Go to **Settings → Capabilities → Skills**
3. Upload the Skill file from the repository
4. In a new conversation, provide your data (a transcript, a CSV of survey responses, a focus group document) and ask for qualitative analysis

The Skill will read the data, detect its structure, run the full pipeline, and write structured output files to your working directory. It will also produce a short summary of what it found and where the outputs live.

Repository: [github.com/vtmade](https://github.com/vtmade)

---

## What This Means for Your Work

If you do qualitative research — academically, commercially, or as part of broader product or strategy work — the practical takeaways from this article are these.

**Stop accepting summaries as analysis.** A paragraph that lists "key themes" is not a finding. It is a sketch of one. Real findings carry their evidence with them.

**Ask where every claim came from.** If a researcher (or a tool) cannot point you back to the segments that produced a theme, treat the theme as a hypothesis, not a result.

**Preserve the outliers.** The participants who don't fit are usually the ones who tell you what actually drives the pattern.

**Label your causes honestly.** Distinguish what participants told you from what you inferred. That single discipline raises the credibility of your work more than almost anything else.

**Use AI as a structured coding layer, not a summary generator.** When the output is structured, traceable, and auditable, AI becomes an extraordinary accelerant. When it isn't, AI becomes a confident-sounding way to lose information.

The method has not changed. What has changed is that we now have tools capable of executing it at speed — if we ask them to.

---

## References

- Braun, V., & Clarke, V. (2006). Using thematic analysis in psychology. *Qualitative Research in Psychology*, 3(2), 77–101.
- Glaser, B. G., & Strauss, A. L. (1967). *The Discovery of Grounded Theory: Strategies for Qualitative Research*. Aldine.
- Saldaña, J. (2021). *The Coding Manual for Qualitative Researchers* (4th ed.). SAGE Publications.

---

**Research Edge Series #005**  
Vinay Thakur · #vtmade  
[github.com/vtmade/research-edge-series](https://github.com/vtmade/research-edge-series)
