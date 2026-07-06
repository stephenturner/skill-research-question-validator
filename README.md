[![MseeP.ai Security Assessment Badge](https://mseep.net/pr/stephenturner-skill-research-question-validator-badge.png)](https://mseep.ai/app/stephenturner-skill-research-question-validator)

# Research Question Validator

A lightweight skill that checks whether a proposed research question is novel, saturated, or
somewhere in between — before you spend hours on a literature review or grant application.

It runs 4-5 targeted Consensus searches and returns a structured one-page assessment in chat:
novelty signal, evidence density, what the literature actually shows, direct gap quotes from
published papers, and a recommended next step.

## Installation

**Option 1: Download ZIP**

Click the green **Code** button at the top of this repo, then **Download ZIP**. Extract the ZIP and add the folder to your Claude skills directory.

**Option 2: Releases**

Go to the [Releases](https://github.com/stephenturner/skill-research-question-validator/releases) page and download the latest `.skill` file. Add it to your Claude skills in [customize/skills](https://claude.ai/customize/skills) on the web, or double-click it if you have Claude Desktop installed.

**Option 3: Build it yourself**

Build a `.skill` file from the source code, then add it to your Claude skills as described above.

```sh
git clone https://github.com/stephenturner/skill-research-question-validator.git
cd skill-research-question-validator
zip -r research-question-validator.skill SKILL.md references/
```

## Usage

Just describe your research question in plain language. The skill parses the question,
runs the searches, and delivers an assessment. 

**Trigger phrases:**

- "Validate this research question: ..."
- "Has this already been done? ..."
- "Is there a gap in the literature on ...?"
- "Is [X] a saturated research area?"
- "Is this worth pursuing?"
- "Check for prior work on ..."

---

## Example queries

**1. AI-assisted diagnosis**
> "Validate this research question: Can multimodal foundation models trained on EHR data,
> imaging, and genomics outperform unimodal models for early diagnosis of rare autoimmune diseases?"

*What this checks:* Whether multimodal approaches to rare disease diagnosis are covered in the
literature, or whether the rare disease application is a genuine gap compared to more common
conditions where this has been studied.

---

**2. Antimicrobial resistance prediction**
> "Is there already substantial work on using protein language models to predict antimicrobial
> resistance phenotypes directly from genomic sequence? I want to know if this is a saturated
> space before I start writing aims."

*What this checks:* Evidence density in the intersection of protein LLMs and AMR prediction,
whether systematic reviews exist, and what researchers in this field say is missing.

---

**3. Microbiome and immunotherapy**
> "Has anyone studied whether gut microbiome composition at baseline predicts response to
> PD-1 checkpoint inhibitors in non-small cell lung cancer specifically? I know there's
> colorectal cancer work but I'm not sure about NSCLC."

*What this checks:* Whether the colorectal cancer work extends to NSCLC, or whether the
lung cancer application is a genuine gap worth pursuing.

---

**4. Graph neural networks for drug discovery**
> "Check if there's a real gap in applying graph neural networks to predict synergistic
> drug combinations in glioblastoma. I've seen GNN work on leukemia but not solid tumors."

*What this checks:* Whether the solid tumor application is unstudied, what gap language
appears in existing GNN-drug synergy papers, and whether a review exists that would
cover this question.

---

**5. AI biosurveillance**
> "Is AI-powered pathogen early warning using wastewater metagenomics a novel enough area
> to build a research program around, or is it already well-established post-COVID?"

*What this checks:* Whether the field matured rapidly during/after COVID and is now
saturated, or whether methodological and application gaps remain that could anchor a
research program.

---

## Output

The assessment comes back in chat as a structured one-pager:

- **Novelty signal**: Saturated / Active with gaps / Genuine gap, with confidence level
- **Evidence density**: how many papers exist, whether systematic reviews are present, how recent the most active work is
- **What the literature shows**: 3-4 sentence synthesis with inline citations
- **Gap evidence**: direct quotes from published papers where researchers identify what's missing
- **Your question's position**: how your specific angle maps to what was found
- **Recommended next step**: whether to narrow the question, run a deeper review, or proceed to grant positioning
- **Search log**: full transparency on what was queried and how many results came back

---

## Where this fits in the workflow

Get additional skills like the literature review helper and the grant finder that both use Consensus at [consensus.app/home/mcp/](https://consensus.app/home/mcp/)

```
Research Question Validator   ← you are here
         ↓
Literature Review Helper      (if Active with gaps — map the full evidence base)
         ↓
Consensus Grant Finder        (if pursuing NIH funding)
```

If the validator returns **Genuine gap**, you can skip straight to the grant finder.
If it returns **Saturated**, refine the question and re-run before going further.
