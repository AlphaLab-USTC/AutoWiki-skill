---
name: autowiki
description: "Use when maintaining an LLM-driven knowledge base wiki, ingesting research papers/sources, querying wiki knowledge, or running wiki lint checks. Activates for: paper ingest, wiki query, wiki maintenance, knowledge compilation, source clustering, or any task involving structured Obsidian-based knowledge management."
---

# AutoWiki — LLM-Maintained Knowledge Base

You are the sole maintainer of this wiki. You read raw sources, compile knowledge, maintain cross-references, and keep everything consistent. The human curates sources, directs analysis, and provides cognitive insights.

## IDE

The human reads and edits the wiki in **Obsidian**. All wiki pages must be valid Obsidian-compatible markdown.

### Wikilink Rules

| Location | Format |
|----------|--------|
| Body text, table cells, bullet items | `[[slug]]` |
| YAML `milestone` field | `"[[slug]]"` (quoted) |
| Other YAML values | bare slug |
| Fenced code blocks | exempt |

Use `[[slug]]` wherever a human might want to click through. Obsidian's backlinks panel automatically surfaces reverse links.

### Properties Conventions

- `tags` must include: a `year/YYYY-MM` tag (publication date from arXiv/conference/journal) and a `venue/<name>` tag (short canonical: `venue/NeurIPS`, `venue/ICML`, `venue/arXiv-preprint`, etc.)
- `milestone` in YAML must be quoted wikilink: `"[[slug]]"` (Obsidian Properties UI renders as clickable link)
- All other YAML conventions are documented in the `paper.md` template itself

## Architecture

```
raw/                → Write-once source archive
  ├── new/          → Uncompiled sources (human drops files here)
  └── compiled/     → Ingested sources, organized by topic path
      └── <topic-path>/
          ├── <source>.pdf
          └── <source-slug>_figures/
kb/                 → Wiki (agent-owned)
  ├── sources/      → Literature — grouped by milestone under <topic-path>/
  ├── topics/       → Milestone nodes — conceptual breakthroughs
  ├── journal/      → Cognitive change timeline (one file per month)
  ├── index.md      → Milestone tree navigation
  └── log.md        → Chronological operation record
output/             → Human-readable views (on demand)
```

Source files are organized under `sources/<topic-path>/` subdirectories, where `<topic-path>` mirrors the topic file's directory path relative to `topics/` (may be nested, e.g., `agent-self-evolution/memory-evolution/self-evolving-memory-architectures/`). When creating a new source, place it in the directory matching the topic file that the source's `milestone:` YAML references (create the directory if needed). Extracted figures live in `raw/compiled/<topic-path>/<source-slug>_figures/` alongside the source PDF. Obsidian resolves `[[wikilinks]]` by filename alone, so directory depth does not affect links.

### Three-Tree Mirroring Invariant

Three directory trees MUST maintain identical structure at all times:

| Tree | Root | Purpose |
|------|------|---------|
| `topics/` | `kb/topics/` | Milestone definition files |
| `sources/` | `kb/sources/` | Source (paper) pages |
| `raw/compiled/` | `raw/compiled/` | PDFs + extracted figures |

**Rule**: For any topic at path `topics/<path>/<slug>.md`, the corresponding source directory is `sources/<path>/<slug>/` and the raw directory is `raw/compiled/<path>/<slug>/`. The `<path>` includes all ancestor directories (e.g., `agent-self-evolution/memory-evolution/`).

**Corollary**: Moving a topic file to a new directory requires moving both the corresponding source directory and raw/compiled directory, AND updating `raw_path` in all affected source files.

Example:

| Topic file | Source dir | Raw dir |
|---|---|---|
| `topics/foo.md` | `sources/foo/` | `raw/compiled/foo/` |
| `topics/foo/bar/baz.md` | `sources/foo/bar/baz/` | `raw/compiled/foo/bar/baz/` |

### Truth Hierarchy (reference upward, never restate)

```
sources/ → topics/ (standalone | merged parent | split parent + leaves) → journal/
```

- **Sources** are atomic fact pages — each declares a `milestone:` parent in YAML pointing to an existing topic file.
- **Standalone topics** own sources directly via `## Source Cluster` and represent specific conceptual breakthroughs.
- **Merged parent topics** (`subtopics: [...]` in YAML) contain inline H3 subtopic sections when total papers < 5. Sources point their `milestone:` to the parent file. The parent owns `## Source Cluster` directly.
- **Split parent topics** (`children: [...]` in YAML) aggregate child milestones that have their own files (when a subtopic grows to ≥ 5 papers). Sources point to the child leaf file.
- **Bidirectional linking**: source → topic via `## Feeds`; topic → source via `## Source Cluster`; child → parent via `parent_milestone` YAML; parent → children via `children` YAML. Obsidian backlinks show all reverse links automatically.
- Higher layers reference lower layer page IDs. Never copy content between layers.

### Milestone Hierarchy

Milestones form a tree with three topic modes:

| Mode | YAML | Owns sources? | Subtopics |
|------|------|---------------|-----------|
| **Standalone** | `subtopics: [], children: []` | Yes, via `## Source Cluster` | None |
| **Merged parent** | `subtopics: [a, b, ...]` | Yes, unified `## Source Cluster` | Inline H3 sections |
| **Split parent** | `children: [a, b, ...]` | No, children own sources | Separate files |

**Invariants:**
- A source's `milestone:` field must reference an existing topic file (standalone, merged parent, or split-out leaf).
- Bidirectional: parent `children`/`subtopics` ↔ child `parent_milestone`.
- Maximum depth: 3 levels.
- Topic files: flat in `topics/` by default. When a parent has split-out children, child files go to `topics/<parent-slug>/`; parent stays at `topics/<parent-slug>.md`.
- Source directories: `sources/<topic-path>/` where `<topic-path>` mirrors the topic file's directory path relative to `topics/`.
- **Three-tree mirroring**: `topics/`, `sources/`, `raw/compiled/` directory trees must be structurally identical (see Architecture > Three-Tree Mirroring Invariant).
- Source page `raw_path` must use the full nested path: `raw/compiled/<topic-path>/<source-slug>.pdf`.

**Consolidation Rule (< 5 papers):**

When a parent milestone has **fewer than 5 total source papers** across all subtopics, the subtopics are **inline H3 sections** within the parent file (no separate leaf topic files). The parent uses `subtopics: [...]` in YAML.

Each inline subtopic section format:
```markdown
### <Subtopic Name>
> One-line milestone definition
- key property (source: [[<source-slug>]])
```

**Promotion Rule (≥ 5 papers):**

When a subtopic accumulates ≥ 5 papers, it splits out to its own file at `topics/<parent-slug>/<subtopic-slug>.md`. Move the subtopic from `subtopics` to `children` in parent YAML. The child file uses standalone/leaf mode. This is a Confirm-tier operation.

**Granularity Heuristic (topic source count):**

| Count | Action |
|-------|--------|
| > 8 | **Must split** — topic is too coarse; identify sub-clusters |
| 3–8 | **Judgment call** — split only if distinct conceptual sub-clusters exist |
| < 3 | **Don't split** — granularity is appropriate |

**Tag-to-Parent Promotion:** When 3+ standalone topics share a tag, consider creating a merged parent matching that tag name. If total papers < 5, use merged parent mode (inline subtopics). This is a Confirm-tier operation.

**Classification Fitness Check:** Before assigning a source, verify the source's core research question genuinely falls within the topic's milestone definition. Shared keywords (e.g., "safety") do NOT imply shared research questions. If the connection is indirect — create a new standalone topic rather than force-fitting.

## Page Formats

All pages use structured markdown: YAML frontmatter + fixed section headings. See `references/` for exact templates:
- `references/source-template/paper.md` — Paper source page (includes formatting conventions as comments)
- `references/topic-template.md` — Milestone topic template
- `references/journal-template.md` — Journal entry template

## Operations

### Ingest (new source added to raw/new)

When the human adds a new source, think through this checklist:

**Phase 1 — Analyze (read from `raw/new/`, decide milestone):**

1. **What do we already know?** — Scan index.md → read related wiki pages → understand our existing knowledge in this area
2. **Temporal positioning** — Identify this paper's position in the field's timeline:
   a. **Direct predecessors**: Which specific papers does this one build on, extend, or supersede? Check the paper's Related Work section + wiki's existing Relations sections.
   b. **Evolutionary chain**: Which named chain(s) does this paper belong to? Check topic page's Chronological Evolution if it exists. Classify as: new chain origin, intermediate node, terminal node, or chain-splitting fork.
   c. **Cross-domain origins**: Was the core method adapted from another field? Note source domain, original paper, and time gap.
   d. **Cross-topic impact**: Does this paper affect papers/topics outside its own milestone? (e.g., evaluation paper retroactively questions claims in an attack topic)
   e. **Temporal tensions**: Does this paper contradict earlier findings? Note the specific contradiction and affected papers.
3. **What does the field already have?** — Read the paper's related work / baselines → understand the scope's prior art
4. **What is specifically new?** — From the two comparisons above, derive the core contribution (not stated independently — DERIVED from comparison)
5. **For each related wiki page** — What's the delta between this paper and existing wiki knowledge? These deltas feed directly into Critical Analysis: Novel Insight uses them as `prior:` references, Fundamental Limitations cross-references shared problems via `also_affects:`, Research Frontier references closest existing attempts via `closest_attempt:`.
6. **Assign milestone** — First, apply the **Classification Fitness Check** (see Milestone Hierarchy section): does this source's core research question genuinely match an existing topic's milestone definition? A paper about general LLM jailbreaking does NOT fit an "agent safety" topic just because both involve "safety." If no existing topic is a genuine conceptual match → create a new standalone topic first (Confirm tier, but misclassification is worse than a small new topic). Only after confirming fit (or creating a new topic) set `milestone:` in source YAML.
7. **Granularity check** — After assignment, count the topic's total sources. If a merged parent's subtopic reaches ≥ 5 papers, flag for split-out (Confirm tier). If a standalone topic reaches > 8, flag for sub-clustering.
8. **Post-assignment checks**: New topic needed? Human cognitive insights to record? Cross-source synthesis triggered? (If yes to any, dual-write per Proactive Write-back rules.)

**Phase 2 — Report to human before compiling:**

1. Three-sentence lead: scope, method, insight
2. Positioning: state which domain/category this paper belongs to, and where it sits relative to the field's prior art (baselines, related work from the paper itself). Use wiki knowledge internally to sharpen positioning, but do NOT enumerate wiki page statuses in the report.
3. Contribution: derived from the delta between what the field already has and what this paper adds — never stated independently
4. Critical analysis: novel insights (contrastive with wiki knowledge), fundamental limitations (of the approach/direction, not the paper), research frontier opened

Wiki page management (which pages to create, update, link) is your responsibility — do NOT report proposed page changes to the human.

**Phase 3 — Finalize raw/ (milestone is now known):**

Discover, extract figures, move PDF, and verify — all targeting `raw/compiled/<topic-path>/` (where `<topic-path>` mirrors the assigned topic's directory path, e.g., `agent-self-evolution/memory-evolution/self-evolving-memory-architectures/`):

```bash
# 1. Discover actual PDF path (handles flat or nested drops)
#    Expected: raw/new/<source>.pdf
#    Also handles: raw/new/<subdir>/<source>.pdf
find raw/new/ -name "*.pdf" -type f

# 2. Extract figures (if source is a PDF)
python scripts/paper_extract_figures.py <actual-pdf-path> \
    -o raw/compiled/<topic-path>/<source-slug>_figures

# 3. Move PDF from new/ to compiled/
mv <actual-pdf-path> raw/compiled/<topic-path>/<source-slug>.pdf

# 4. VERIFY: PDF must no longer exist under raw/new/
#    If this finds the file, the move failed — retry or report error.
! find raw/new/ -name "<source>*.pdf" -type f | grep -q .
```

**Hard gate:** Do NOT proceed to Phase 4 until the PDF is confirmed absent from `raw/new/`. If the move fails (wrong path, permission error), fix and retry. The `raw/new/` inbox must be empty of this source before writing wiki pages.

**Phase 4 — Write wiki pages:**

- **Always create**: New source page using `paper.md` template (essence + CRGP factors + figure references + critical analysis + feeds + cognitive shifts). Set `raw_path` to `raw/compiled/<topic-path>/<source-slug>.pdf` (full nested path).
- **Always update**: index.md, log.md
- **As needed**: Related milestone topics (Source Cluster, Key Properties, Arrival/Departure)
- **As needed**: New milestone topics (if new conceptual breakthrough emerged)
- **As needed**: Update topic's ## Chronological Evolution if:
  (a) topic now has ≥3 integrated sources and section doesn't exist yet, or
  (b) new paper changes the evolutionary chain structure (new node, new chain, or supersession)
- **As needed**: Journal entry (if cognitive shifts occurred)

Strategy: **Immediate cascading update** — wiki must be fully consistent after every ingest.

**End-of-ingest gate:** After all wiki writes, run a final sanity check:
```bash
# No PDFs for just-ingested source(s) should remain in raw/new/
find raw/new/ -name "*.pdf" -type f
```
If any ingested source's PDF is still found under `raw/new/` (at any depth), complete the move now before reporting success. An ingest is not done until `raw/new/` contains only un-ingested sources.

### Reorganize (topic hierarchy change)

When topics are moved, merged, split, or restructured:

**Checklist (all steps mandatory, in order):**

1. **Topic files**: Create/move/edit topic `.md` files. Update YAML (`parent_milestone`, `children`, `subtopics`).
2. **Source directories**: Move `sources/` directories to mirror new `topics/` tree.
3. **Raw directories**: Move `raw/compiled/` directories to mirror new tree.
4. **Source path update**: In ALL affected source `.md` files, update:
   - `raw_path:` YAML field (e.g., `raw/compiled/<new-topic-path>/<slug>.pdf`)
   - `![alt](...)` image embeds — relative paths to `_figures/` (e.g., `../../../raw/compiled/<new-topic-path>/<slug>_figures/figure_N.png`)
   - Figure directory references in comments/blockquotes (e.g., `raw/compiled/<new-topic-path>/<slug>_figures/`)
5. **Index**: Rewrite `index.md` to reflect new hierarchy.
6. **Log + Journal**: Record the reorganization.
7. **Verify three-tree mirroring**:

```bash
# Trees must match (excluding _figures dirs)
diff <(find kb/sources/ -type d | sed 's|kb/sources/||' | sort) \
     <(find raw/compiled/ -type d ! -name "*_figures" | sed 's|raw/compiled/||' | sort)

# All raw_path references must resolve
grep -r "^raw_path:" kb/sources/ | while IFS=: read -r f v; do
  p=$(echo "$v" | sed 's/^raw_path: *//'); [ ! -f "$p" ] && echo "BROKEN raw_path: $f → $p"
done

# All figure image embeds must resolve
grep -rn '!\[.*\](.*_figures.*\.png)' kb/sources/ | while IFS= read -r line; do
  f=$(echo "$line" | cut -d: -f1)
  img=$(echo "$line" | grep -o '(.*_figures[^)]*' | tr -d '(')
  # Resolve relative path from source file's directory
  dir=$(dirname "$f")
  resolved=$(cd "$dir" && realpath -q "$img" 2>/dev/null || echo "")
  [ ! -f "$resolved" ] && echo "BROKEN image: $f → $img"
done
```

This is a **Confirm-tier** operation — always requires user approval before execution.

### Query (human asks a question)

1. Read index.md → find relevant pages
2. Read relevant pages, synthesize answer
3. Did the answer produce new value? Apply the Proactive Write-back decision boundary:
   - New cross-source insight (novel connection, shared limitation, or research opportunity) → dual-write (Notify tier)
   - Cognitive shift from discussion → dual-write (Notify tier)
   - Open Question resolved → dual-write (Notify tier)
   - Pure factual answer with no new synthesis → don't write
4. If any writes occurred, print one-line summary to terminal.

### Lint (health check)

Periodically check for:
- Contradictions between pages
- Stale claims superseded by newer sources
- Orphan pages with no inbound links
- Missing pages (frequently referenced but nonexistent topics)
- Missing cross-references
- Orphan sources (no `milestone:` field)
- Feeds ↔ Source Cluster mismatch (source says `integrated` but milestone's Source Cluster doesn't list it, or vice versa)
- Broken `[[wikilinks]]` (link target has no corresponding page)
- Hollow milestones (leaf topic with empty Source Cluster)
- **Hierarchy consistency:**
  - Source `milestone:` references a non-existent topic file
  - Merged parent (`subtopics: [...]`) missing H3 sections for listed subtopics
  - Split parent (`children: [...]`) missing child files in `topics/<parent-slug>/`
  - Topic has both non-empty `subtopics` and `children` for the same slug (pick one mode per subtopic)
  - `parent_milestone` value in a child that doesn't match any existing topic
  - Hierarchy depth > 3 levels
  - Merged parent subtopic with ≥ 5 papers (should split out to own file)
  - Standalone topic source count > 8 (granularity violation — flag for split)
- **raw/ consistency:**
  - Orphan `_figures/` directory in `raw/compiled/` with no corresponding PDF (interrupted ingest)
  - PDF anywhere under `raw/new/` (including subdirectories) that has a corresponding source page in `kb/` (forgot to move) — use `find raw/new/ -name "*.pdf" -type f` to discover at any depth
  - Source page `raw_path` points to nonexistent file
  - `![alt](...)` image embed path does not resolve to an existing file (stale path after reorganize)
  - Nested subdirectories inside `raw/new/` (human may have dropped a folder instead of flat files) — flatten or flag
  - Three-tree mirroring violation (see Architecture > Three-Tree Mirroring Invariant)
- **Temporal consistency:**
  - Source with ≥1 Relations entry but missing **Temporal context** paragraph at top of ## Relations section
  - Topic with ≥3 integrated sources but missing ## Chronological Evolution section
  - Temporal context references a [[source]] that doesn't exist as a wiki page (broken predecessor/successor link)
  - Evolutionary chain in Chronological Evolution lists a source not in the topic's Source Cluster (Integrated, Stubs, or Mentioned)
  - Cross-topic timeline references a source under the wrong topic (source's milestone doesn't match the topic cited in the timeline)

When structural issues are detected during any operation (not only explicit lint):
- Silent-tier issues → fix immediately, journal entry only
- If a fix would change semantic content (e.g., resolving a contradiction) → escalate to Confirm tier

## Proactive Write-back

The agent autonomously writes to the wiki when valuable observations arise — not only when explicitly instructed. Every proactive write follows the **dual-write rule**: update the relevant page in-place AND append an atomic audit entry in `journal/`.

### Autonomy Tiers

| Tier | Scope | Agent behavior |
|------|-------|---------------|
| **Silent** | Structural fixes: broken wikilinks, Feeds↔Cluster sync, missing cross-references, index.md sync | Fix immediately, log in journal. No terminal notification. |
| **Notify** | New observations: cognitive shifts, cross-source synthesis, open-question resolution | Write to wiki + journal, then print one-line summary to terminal. |
| **Confirm** | Structural changes: new milestone topic, milestone reassignment, delete/merge pages, contradiction resolution, **milestone split/merge, promote leaf to parent, tag-to-parent promotion** | Describe proposed change in terminal, wait for human approval before writing. |

### Trigger → Action Table

| Trigger | Detection | In-place target | Journal entry type |
|---------|-----------|-----------------|-------------------|
| **Conversation insight** | During discussion, a new understanding/analogy/contradiction emerges that is not from any paper but from the dialogue itself | Source `## Cognitive Shifts` or Topic `## Open Questions` | `insight` |
| **Cross-source synthesis** | While answering a query or during ingest, discover an unrecorded connection or tension between existing sources | Source `## Critical Analysis` (add to Novel Insight, Fundamental Limitations, or Research Frontier as appropriate) + Topic `## Departure` or `## Source Cluster > Mentioned` | `synthesis` |
| **Lint auto-fix** | Detected during any read operation (not only explicit lint command) | The broken page itself | `lint-fix` |
| **Open Question evolution** | New source partially/fully answers an existing Open Question on any topic | Topic `## Open Questions` (mark resolved/partially-resolved with source ref) | `oq-update` |

### Atomic Journal Entry Format

See `references/journal-template.md` for the exact entry format. Types: `insight`, `synthesis`, `lint-fix`, `oq-update`.

### Decision Boundary: Write or Not?

Before writing, the agent applies this filter:
1. **Is it already recorded?** → Don't write (avoid duplication)
2. **Is it specific enough to be actionable?** → Vague feelings don't qualify; concrete observations do
3. **Would a future reader (human or agent) benefit from finding this?** → If yes, write

## Conventions

- **Wiki language**: Configured in `kb/index.md` frontmatter as `wiki_language` (e.g., `en`, `zh-CN`, `zh-TW`, `ja`). All wiki body content (Essence, Factors, Critical Analysis, Relations, Topic prose, Journal entries, etc.) MUST be written in the configured language. Language-invariant elements remain in English: YAML field names, section headings (`## Essence`, `## Factors`, etc.), formatting labels (`**Insight**:`, `*Prior*:`, etc.), kebab-case slugs, wikilinks, and tag names.
- Page filenames use kebab-case slugs: `attention-is-all-you-need.md`
- All dates use ISO format: `2026-04-06`
- Domain names use kebab-case: `nlp-architectures`
- Log entries: `## [YYYY-MM-DD] <operation> | <title>`
- Journal files: one per month `YYYY-MM.md`

## Bootstrapping a New Wiki

To initialize a fresh AutoWiki project:

1. **Ask the user** which language the wiki content should use (e.g., `en`, `zh-CN`, `zh-TW`, `ja`). Default to `en` if not specified.

2. **Create directory structure**:
```bash
mkdir -p raw/new raw/compiled kb/{sources,topics,journal} output
```

3. **Create `kb/index.md`** with language configuration in frontmatter:
```yaml
---
type: index
wiki_language: <chosen-language>  # en | zh-CN | zh-TW | ja | ...
last_updated: <YYYY-MM-DD>
---
```

4. **Create `kb/log.md`** for the chronological operation record.
