---
name: research-question-validator
description: >
  Fast research question validity check using Consensus. Runs 4-5 targeted searches to assess
  whether a proposed research question is novel, saturated, or somewhere in between — with evidence.
  Returns a structured Word document (.docx): novelty signal, evidence density, what the literature
  shows, gap quotes, and a recommended next step.

  Trigger when the user asks: "Is this research question novel?", "Has this been done before?",
  "Is there a gap in the literature on X?", "Is [X] already saturated?", "Validate my research
  question", "Check for prior work on X", "Is this worth pursuing?", or any phrasing where the
  user wants a go/no-go answer before committing to a full review or grant application.

  Do NOT trigger for full literature reviews (use literature-review-helper), grant opportunities
  (use consensus-grant-finder), or simple one-off paper searches.
---

# Research Question Validator

## Overview

This skill takes a proposed research question and gives the researcher a fast, evidence-grounded
answer to: "Is this worth pursuing?"

It runs 4-5 targeted Consensus searches, synthesizes what they return, and delivers a concise
assessment as a Word document (.docx) the researcher can save, annotate, and share. The goal
is a decision-grade answer before committing hours to a full literature review or grant application.

The output answers three things:
1. **Novelty signal** — is this saturated, active with gaps, or a genuine gap?
2. **Evidence summary** — what does the existing literature actually show?
3. **Next step** — what should the researcher do with this information?

---

## Data Integrity Principles

Everything in the assessment must come from what Consensus returned in this session.

**Only cite what Consensus returned.** Never supplement with papers from training knowledge.
If you know of a relevant paper from training data but Consensus didn't return it, do not include
it in the assessment. If you mention it at all, label it `[Not from Consensus — model knowledge]`
and keep it out of counts and gap evidence.

**Confirm before proceeding.** A search is not complete until the result is received and inspected.
Never mark a search done based on intent — only on a confirmed response with a result count.

**Detect and report plan-tier caps.** After the first search, check how many papers were returned.
If the response says "showing top 10" (out of more found) or includes an upgrade prompt, the user
is on a capped plan. Note the cap and use it to calibrate confidence in the assessment:
- Unauthenticated: ~3 results per search
- Free tier: ~10 results per search
- Pro: ~20 results per search

If the assessment is based on capped results, say so: "These searches returned a maximum of [N]
results per query. Sparse results on a dimension may reflect a plan-tier ceiling rather than a
genuine gap in the literature."

**Surface gaps, don't fill them.** If a search returns zero results, say so explicitly. Do not
silently substitute training knowledge. A dimension with no results is a useful signal on its own
— either the question is genuinely novel, or the terminology was off.

---

## Error Handling

1. On any search failure: wait 3 seconds, retry once.
2. Log every failure — which search, what error, whether the retry succeeded.
3. After 3 consecutive failures: stop and alert the user. Report what was collected before the failures.
4. Never silently skip a failed search. Note it in the assessment with `[Search failed — not included]`.

---

## Workflow

### Phase 1: Parse the Research Question

Read the user's question and extract:
- **Core concept(s)**: the central subject matter (e.g., "transformer models", "sepsis prediction")
- **Population or domain**: who or what is being studied (e.g., "ICU patients", "protein sequences")
- **Proposed approach or intervention**: the method or factor being examined (e.g., "graph neural networks", "CRISPR editing")
- **Outcome**: what would be measured or demonstrated (e.g., "antimicrobial resistance", "drug-drug interaction prediction")
- **Study type implied**: mechanistic, clinical/translational, computational/AI, observational

If the question is ambiguous, briefly note the interpretation you're using and proceed — don't
ask for clarification unless the question is genuinely unworkable.

Distill the question into **three search angles**:
- A direct angle: the question as stated
- A methods angle: the approach applied to this domain
- A gap angle: what researchers say is missing in this space

---

### Phase 2: Run 4-5 Consensus Searches Sequentially

Run all searches one at a time with at least a 1-second pause between calls. Confirm each result
arrived before sending the next. Track: query sent, results received, results that will be cited.

**Search 1 — Direct evidence** (always run)
What has actually been published on this specific question?
```
query: "<core concept and outcome as a focused academic query>"
```
No year filter — get the full picture first.

**Search 2 — Systematic reviews and meta-analyses** (always run)
Has someone already synthesized this literature? A systematic review is the strongest signal
that a question has been settled — or that it hasn't.
```
query: "systematic review <core concept>" OR "meta-analysis <core concept>"
```
If the user's question is computational/AI (e.g., involves ML models, neural networks, LLMs),
also check for benchmark papers or survey articles, which play the same role in that literature.

**Search 3 — Recent activity** (always run)
Is this an active, growing area, or has interest plateaued?
```
query: "<core concept and approach>"
year_min: <current year - 2>
```
Compare the recency of results here to Search 1. A field with mostly older papers and few
recent ones may be mature or abandoned. A field with many recent papers is still being actively
worked.

**Search 4 — Gap language** (always run)
What do researchers in this field say is missing? This is the most valuable search for
identifying whether the user's specific angle is genuinely novel.
```
query: "<core concept> research gaps limitations unmet needs future directions"
```
Scan abstracts for language like: "remains unclear", "no studies have", "future research should",
"critical gap", "limited evidence", "poorly understood", "has not been applied to". Extract
2-4 such quotes with their citations — these are the backbone of the gap evidence.

**Search 5 — Methods or AI angle** (run if the question involves a specific computational or
methodological approach)
Has this specific method been applied to this domain, or only to adjacent ones?
```
query: "<method or model type> <domain or condition>"
```
If results show the method has been applied elsewhere but not to this exact domain, that's a
strong novelty signal. If results show it's already been applied here, that's important context.

After all searches complete, record the final tally: searches sent, results received per search,
and any failures.

---

### Phase 3: Synthesize and Score

Before writing the assessment, make two structural decisions:

**Novelty signal** — assign one of three levels based on what the searches returned:

| Signal | Criteria |
|--------|----------|
| Saturated | Systematic review(s) exist that directly address the question; many primary studies; question appears answered |
| Active with gaps | Papers exist but systematic reviews identify unresolved aspects; Search 4 returns clear gap language; the user's specific angle is not well-covered |
| Genuine gap | Few or no papers directly addressing the question; no systematic reviews; Search 4 returns researchers calling for exactly this type of work |

**Confidence** — note the confidence level in this signal:
- High: multiple searches returned consistent results across dimensions
- Moderate: results were mixed or some searches returned few papers
- Low: most searches returned few results (possible plan-tier cap or narrow terminology)

---

### Phase 4: Generate the .docx

Read the DOCX skill at `/mnt/skills/public/docx/SKILL.md` before generating. Write the
document as a Node.js script using the `docx` library and execute via bash_tool.

Install if needed:
```bash
npm install -g docx
```

Save to `/mnt/user-data/outputs/research-question-assessment-[slug].docx`.

#### Document structure

**Section 1 — Research Question and Novelty Signal**
- Restate the question as interpreted (one sentence)
- Novelty Signal as a styled callout box: background color keyed to signal level
  - Saturated: light red (#fde8e8) fill
  - Active with gaps: light amber (#fef3cd) fill
  - Genuine gap: light green (#e8f5e9) fill
- Confidence level (High / Moderate / Low) with a one-sentence explanation of why

**Section 2 — Evidence Density**
2-3 sentences: how many papers exist, whether a systematic review is present, how recent the
most active work is. Include a small summary table:

| Dimension | Finding |
|-----------|---------|
| Papers found (Search 1) | [N] |
| Systematic review exists | Yes / No |
| Most active period | [year range] |
| Consensus plan cap | [N results/query or "not detected"] |

**Section 3 — What the Literature Shows**
3-4 sentences synthesizing the direction of evidence. Where is there agreement? Where is there
conflict or uncertainty? Cite inline as Author et al. (Year) with the paper title hyperlinked
to its Consensus URL using ExternalHyperlink. Every paper cited here must have a URL from
this session.

**Section 4 — Gap Evidence**
The direct quotes from published papers where researchers identify what is missing. Present
as a bulleted list using LevelFormat.BULLET:
- "[Exact quote from abstract]" — Author et al. (Year), Journal [hyperlinked to Consensus URL]

Include 2-4 quotes. If Search 4 returned no gap language, include a note in this section:
"Search 4 returned no papers with explicit gap language — this may reflect a plan-tier cap
or may indicate the field does not commonly discuss limitations in abstracts."

**Section 5 — Your Question's Position**
2-3 sentences directly addressing how the user's specific angle maps to what was found. Is
the exact question covered by existing work? Is there a version of it that is novel? Be
specific — name the gap rather than describing it in general terms.

**Section 6 — Recommended Next Step**
One of:
- Saturated: "This question appears well-addressed in the literature. Consider narrowing
  the scope — a specific population, method variant, or application domain not yet covered —
  then re-validate."
- Active with gaps: "This area has active research and identified gaps. Run a deeper
  literature review (literature-review-helper) to map the full evidence base before committing
  to a study design, then use the grant finder to identify funding."
- Genuine gap: "Limited prior work on this specific question. This is a strong candidate for
  original research. Consider using the grant finder to identify NIH institutes and mechanisms
  funding work in this space."

**Section 7 — Search Log**
Full audit table:

| # | Query | Filters | Results Received | Notes |
|---|-------|---------|-----------------|-------|
| 1 | [query] | none | [N] | [e.g., "plan cap: showing 10 of 20"] |
| 2 | [query] | year_min: 2023 | [N] | |
| ... | | | | |

Include below the table: detected plan tier, any failed searches and retry outcomes, and a
one-sentence note that all cited papers were drawn only from Consensus results in this session.

#### Styling
- Arial 12pt body, navy (#1a3a5c) headings
- Novelty Signal callout: colored fill per signal level (see Section 1 above), bold text
- Table header rows: light blue (#e8f0f8) fill
- Keep the document to 2-3 pages including the search log — this is a decision tool, not a
  full report

---

### Phase 5: Deliver

Save the .docx and present with `present_files`.

Brief in-chat summary:

> "Assessment saved as [filename]. Bottom line:
> - **Novelty signal**: [Saturated / Active with gaps / Genuine gap] ([confidence] confidence)
> - **Evidence**: [one sentence on paper count and systematic review presence]
> - **Key gap quote**: "[most compelling quote]" — Author et al. (Year)
> - **Recommended next step**: [one sentence]
>
> All citations link to Consensus. The Search Log in the document shows every query and result count."

---

## Notes

- **Rate limit**: 1 Consensus query per second. Always sequential, never parallel.
- **Terminology matters**: if Search 1 returns sparse results, try a terminology variant
  before concluding the question is a gap. AI/ML fields especially have rapidly shifting
  vocabulary (e.g., "language model" vs. "large language model" vs. "foundation model").
  If you adjust terminology, note the adjustment in the search log.
- **Treat zero results carefully**: zero results on Search 1 could mean (a) genuine novelty,
  (b) plan-tier cap, or (c) wrong terminology. Check all three before declaring a genuine gap.
- **Cross-signal the searches**: a question that returns 0 systematic reviews, <5 primary
  papers on Search 1, and clear gap language on Search 4 is a much stronger genuine gap
  signal than any one of those alone.
- **Don't editorialize beyond the evidence**: the novelty signal is derived from what the
  searches returned — not from your training knowledge about the field's importance or promise.
  If training knowledge conflicts with search results, trust the search results and note any
  discrepancy honestly.
