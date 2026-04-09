---
type: source
id: <slug>
raw_path: raw/compiled/<topic-path>/<source>.pdf  # Full nested path mirroring topic directory (e.g., raw/compiled/a/b/c/source.pdf)
url: <https://arxiv.org/abs/XXXX.XXXXX>  # Permanent external URL (arXiv, conference, DOI)
tags:
  - <domain>
  - year/<YYYY-MM>  # Publication date from arXiv/conference/journal (e.g. year/2025-03)
  - venue/<venue-name>  # Publishing venue (e.g. venue/NeurIPS, venue/ICML, venue/Nature, venue/arXiv-preprint)
milestone: "[[<topic-slug>]]" # Must reference an existing topic file (standalone, merged parent, or split-out leaf)
created: <YYYY-MM-DD>
last_updated: <YYYY-MM-DD>
authors:
  - <author name>
aliases:
  - <human-readable paper title>
---

<!-- FORMATTING CONVENTIONS (apply to all wiki body content):
     Structural: **Bold key labels** in structured entries, *italic sub-field labels* as nested bullets.
     Relations: single-line — "- **[[slug]]** (*type*): delta text"
     Figure interpretations: blockquote below image (> interpretation)
     
     INLINE BOLDING (most important rule):
     Within every content field, **bold the core phrase** a scanning reader should catch.
     One bold span per sentence/bullet — target key finding, mechanism, or shift.
     Applies to: Essence, Factors, Critical Analysis, Relations, Figures, Inspirations. -->

## Essence

> [!abstract]
> **One-Sentence Summarization**: ""
> **Contribution**: "" (**bold** the core novelty within the sentence)
> (Derived from comparison — what specifically is new given the field's prior art?)

## Factors

<!-- (Author Claims from Introduction)
     Reflects ONLY what the authors claim in their Introduction — not our analysis.
     FORMATTING: Bold key terms, system names, and technical concepts for scannability. -->

### Context
{2-3 sentences. **Bold** the core research problem and key technical terms.}

### Related Work
<!-- Bold the system/paper name, then describe. -->
- **Name**: description
- **Name**: description

### Gap
{2-3 sentences. **Bold** the specific limitation that motivates this work.}

### Proposal
{2-3 sentences. **Bold** the method name and **bold** the key insight/mechanism.}

## Figures

<!-- Run paper_extract_figures.py first. Read figures_manifest.json to review captions (text),
     then select the most informative figures based on captions and paper content.
     Figure numbering does NOT imply a fixed role (e.g., Fig 1 is not always a teaser).
     Write a one-line interpretation as a blockquote below the image for visual separation. -->

![<descriptive-alt>](../../../raw/compiled/<topic-path>/<slug>_figures/figure_<N>.png)
> {One-line interpretation — **bold** the key takeaway from this figure}

> [!note]
> Figure images live in `raw/compiled/<topic-path>/<slug>_figures/`.
> Paths are relative from `kb/sources/<topic-path>/<source>.md`.
> `<topic-path>` is the full nested path mirroring the topic's directory (e.g., `a/b/c/`).
> Include as many or as few figures as needed to convey the paper's key visual information.

## Critical Analysis

### Novel Insight
<!-- What we didn't know before this paper. Each entry MUST be contrastive:
     state what the wiki's prior understanding was, then what this paper changes.
     Test: "Would a senior researcher cite this in their own paper's motivation?"
     Anti-pattern: restating the paper's claims. This must be YOUR derived insight
     from comparing this paper against what the wiki already knows. -->

- **Insight**: <sentence with **core finding bolded** for scanning>
  - *Prior*: <what wiki sources believed before> ([[source-slug]])
  - *Update*: <how understanding changes — **bold the specific shift**>

### Fundamental Limitations
<!-- NOT "the paper didn't test enough models." These are limitations of the APPROACH
     or PROBLEM FORMULATION that affect the entire research direction.
     Each must identify the root cause and cross-reference other affected work.
     Anti-patterns: "only tested on X", "evaluation uses Y", "no comparison with Z"
     — those are reviewer TODOs, not research insights.
     Test: "Would solving this limitation be a publishable contribution?" -->

- **Limitation**: <sentence with **core difficulty bolded**>
  - *Root cause*: <why this is hard — **bold the mechanism**>
  - *Also affects*: [[source1]], [[source2]]
  - *Implication*: <consequence — **bold the key takeaway**>

### Research Frontier
<!-- Concrete next-step problems that this paper makes tractable or newly visible.
     Each must specify what prerequisite would need to exist and what the closest
     existing attempt is. This is YOUR future-work radar, not the authors'.
     Anti-pattern: "future work should test on more models/domains."
     Test: "Could someone write a paper abstract starting from this direction?" -->

- **Direction**: <sentence with **the opportunity bolded**>
  - *Prerequisite*: <what needs to exist first>
  - *Closest attempt*: [[source]] tried <X> but **<gap remains>**

## Feeds

(Which topic does this source feed? Use [[wikilinks]]. Note subtopic if applicable.)
- [[<topic-slug>]]: integrated (subtopic: <name> if applicable)

## Relations

<!-- Temporal Context: 1-3 sentences positioning this paper in its evolutionary chain(s).
     Identify: (1) direct predecessors this paper builds on, (2) direct successors it enables,
     (3) cross-domain origins if method was adapted from another field,
     (4) cross-topic impact (papers in other topics affected by this one).
     Reference specific papers with [[wikilinks]] and dates (YYYY-MM). -->

**Temporal context:** <1-3 sentences positioning this paper in evolutionary chain(s).
Include chain notation where applicable: predecessor(date) → **this-paper(date)** → successor(date).
Note cross-domain origins and cross-topic impact.>

<!-- Relation entries: one line per relation.
     Bold the target wikilink, italicize the relation type, then describe the delta inline.
     Types: builds_on, extends, supersedes, adapts, contrasts_with, complements, challenges -->

- **[[<source-slug>]]** (*builds_on*): <delta — **bold the key difference**>

## Transferable Inspirations

(What we learn from this for our own thinking / future research directions)
- 

## Cognitive Shifts

- [YYYY-MM-DD] <shift description> (source: <conversation/ingest/query context>)

## Open Questions

- 
