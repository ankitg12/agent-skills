---
name: typst-it
description: "Generate a polished, print-quality Typst technical document (with diagrams, tables, quoted evidence, and hyperlinked references) when the user asks for a detailed writeup, investigation report, or architecture doc — a strong alternative to Word/.docx."
---

## When to use this

Use **Typst** over Word/.docx for any detailed technical writeup — investigation reports, architecture summaries, root-cause docs, migration explainers. Target documents that:
- Read like a real technical report: title page, executive summary, numbered sections
- Look good in print (LaTeX-like typesetting: `#set page`, `#set heading(numbering: "1.1")`, justified body text)
- Include diagrams (architecture/flow diagrams, not just prose)
- Include tables (comparison tables, evidence tables, status tables)
- Include clickable links to sources (chat threads, GitHub PRs, wiki pages)
- Cite primary sources with direct quotes (messages, PR bodies) attributed by name/date

Trigger phrases: "write this up", "create a doc", "typst it", "make a report on this" — especially after a multi-source investigation (chat search, GitHub PR review, live system evidence) that needs synthesizing into a shareable artifact.

**Before compiling, keep and consult a personal Typst gotchas file** (e.g. `~/notes/typst.md`) for known syntax traps (footnote placement, `#N` collisions, fletcher import, table-column sizing, etc.) — and append to it whenever a new compile-error class is discovered.

**Critical Syntax Rules (Pre-flight checklist before `typst compile`):**
1. **Bold markup is single `*word*`, NEVER `**word**`**: Markdown double-stars emit dozens of compiler warnings (`warning: no text within stars; using multiple consecutive stars has no additional effect`).
2. **Math mode strings require quotes**: In `$ ... $`, multi-letter identifiers like `$Count=65$` or `$RTO$` fail compilation (`unknown variable`). Write `$"Count"=65$` or place code in backticks (`` `Count=65` ``).
3. **No LaTeX backslash operators in math**: Writing `$\approx$` fails (`unknown variable: pprox`). Typst uses bare symbols: `$approx$` without a backslash.

## Go deep on content; be resource-conscious about *presentation*

Two standing directives that shape every report:

1. **Go deep on genuine technical content.** Aim for excruciating technical depth, not a shallow summary — more raw console output, more architecture/state-machine diagrams, register/bitfield or digital-circuit-level detail when the subject is hardware-adjacent, CPU-instruction/assembly-level detail when the subject is software-execution-adjacent. When a draft feels "done" at 5-6 pages, that is a signal to dig one layer deeper (an extra `redis-cli`/`systemctl`/register dump, an extra architecture diagram, an extra industry-comparison angle), not to stop. See "Depth expectation" below for the concrete section-by-section breakdown.
2. **But be resource-conscious about presentation, not content.** Depth of *content* and resource-consciousness of *styling* are separate axes — go deep on facts/evidence/diagrams, but keep the visual presentation itself ink-light: light-tint backgrounds + colored borders/text instead of solid color fills, and color used only where it carries meaning (see Color scheme below). Ink-light governs *page body* ink, NOT the title block — the title IS boxed (see Color scheme item 1). More pages of real technical substance is the goal; more solid-color ink per page is not.

## Workflow

1. Gather ALL evidence first (chat searches, GitHub PR reads, live command output, issue-tracker/chat prior-art search) before writing — don't start the `.typ` file mid-research; synthesize once.
2. Write the full `.typ` source in one or two `write`/`eval` calls, using the structure template above (single-accent boxed title, accent headings with a thin rule, light-tint table headers, semantic callouts at 2-4 key moments picked by meaning not novelty, light console blocks, fletcher diagrams wrapped in `#figure` for node graphs, no TOC by default). If using fletcher, ALWAYS start the file with `#import "@preview/fletcher:0.5.7": diagram, node, edge` — omitting this import is the single most common first-compile error.
3. Compile: `typst compile file.typ` (from a working directory where the file lives — check `typst --version` once per session to confirm the binary is available; if using fletcher, a throwaway `/tmp` test compile confirms network/package resolution before wiring it into the real doc). Hitting a compile error? Check your gotchas notes file before debugging from scratch.
4. **Always visually verify before declaring done** — render every page to PNG (`typst compile --format png --ppi 100 file.typ "page-{p}.png"` — quote the output pattern) and `read` each PNG to inspect layout/spacing/rendering, especially any page with a diagram, table, or callout. Diagrams with 3+ edges or bidirectional edge pairs need particular scrutiny for label collisions.
5. **Then do a separate CONTENT read pass — layout verification is not content verification.** Step 4 proves nothing renders wrong; it does NOT prove anything is *true*. `read` every rendered page again, this time reading the prose against the source material, and specifically hunt: (a) hand-typed section cross-references ("see §9.1.3") that went stale when a chapter was inserted; (b) table cells whose claim is inverted or transposed relative to the source; (c) numbers that were transcribed second-hand rather than from the primary artifact — mark those `[INFERENCE]` or go verify them. Confirmed necessary in practice: a document that passed the full layout pass still contained a dangling `§` reference AND a factually inverted table cell, both caught only by reading. A 40 KB document written in one pass has NOT been proofread just because it compiled.
6. Move the final `.typ` + `.pdf` into the project's docs location (e.g. `docs/generated/`) and commit both, then delete any intermediate PNG check files — keep the source alongside the compiled output so it's re-editable later.

## Use Semantic markup
Per Typst's own accessibility guide: always use the real semantic element (`= Heading`, `#quote(...)`, `#table(...)`) rather than hand-styling plain text/prose to *look* like that element (e.g. `#text(size: 16pt, weight: "bold")[Heading]`). Never invent a parallel "looks like a heading" convention; always drive appearance off the real markup element.

## Color scheme

- **Body font: `New Computer Modern`.** The benchmark is "like LaTeX" — NCM
  is the LaTeX look and it ships with Typst, so it always resolves. NEVER pick a font that is
  not installed on the machine (verify with `typst fonts`): Typst only *warns* on a missing family
  and silently falls back, so the doc compiles "clean" while looking wrong. If an `unknown font
  family` warning appears, that is a defect to fix, not a warning to tolerate.
- **Semantic callout palette.**  green `#1a7f37`, yellow `#9a6700`, red `#cf222e`, blue `#0969da`, purple `#6639ba`, orange `#bc4c00`; console/table tints `#f2f6fb` / `#eaf1fb` with border `#9fb8d6` / `#c8d6e8`. Inline and block `raw` should use a monospace family that exists on the machine (e.g. `Consolas`, `JetBrains Mono`, `Menlo`).

**One structural accent color, used for chrome only — plus semantic callout blocks for meaning.** This is the settled pattern; apply it by default without re-deriving from scratch:
1. Title block: plain centered colored text with bordered box. Use `#box(fill: white, stroke: 1.5pt + accent, ...)` around the title.
2. Table headers: light tint fill + accent-colored text, NEVER a solid color fill.
3. Semantic callout blocks for meaningful findings — this is where extra color is earned, because it's functional, not decorative. Use whichever of these fit the content; don't force all of them into every doc — 2-4 total callouts per report is a good density, not one per paragraph.
   - **Verified** (green checkmark) — confirmed-good finding, one line.
   - **Anomaly** (yellow exclamation) — unexpected but not yet confirmed as a defect.
   - **Confirmed defect** (red X) — confirmed-bad finding, one line.
   - **Note** (blue info) — a non-critical observation worth highlighting.
   - **Tip** (purple lightbulb) — a helpful suggestion or best practice.
   - **Open question** (orange question mark) — a genuine unresolved discrepancy or inquiry.
4. Vary tint by document/section where it aids print legibility.
5. Diagram box colors remain role-based/semantic.
6. **No large solid dark-color fills anywhere** (e.g. a full dark-blue banner with white text). Section headings: colored text + a thin colored rule underneath, not a filled color band or box.
7. **Console/code-output blocks: light-tinted background, not grey.** A flat `#f4f4f4` grey reads as dull/photocopier-ish. Use a light tint with a subtle matching border.
8. **NEVER use `#pagebreak()` between chapters.** A break before every `=` heading leaves half a page of blank paper per chapter and wastes real sheets when printed. Let sections flow; a new chapter starting mid-page is CORRECT. The only acceptable `#pagebreak()` is the single one separating the title page from the body.

## Two default sections — include them unprompted

- **Competitive / industry context.** How do the alternatives solve this problem, and what does this architectural position structurally buy *and* cost? Frame each advantage as a corresponding testable risk. Label the section as general industry knowledge with `[INFERENCE]` unless it comes from a real competitive document.
- **The automation / test harness, concretely.** Exact invocation, runner host, venv and pinned deps, marker selection, and how to read collected vs deselected vs executed counts. "There is a suite" is not coverage.

## Figure placement: wrap diagrams/tables in `#figure(...)`, let Typst float them

Per Typst's own layout guidance, figures default to inline placement in the content flow, but wrapping a diagram or table in `#figure(..., caption: [...])` unlocks `placement: top`/`placement: bottom` to let Typst float it to the top/bottom of the page rather than awkwardly splitting content around it mid-paragraph. Always give diagrams a `#figure(...)` wrapper with a real caption (never a bare `#diagram(...)` call floating in prose) — this is also what makes the diagram cross-referenceable via `@sec-label` and gives screen readers/repurposing tools a proper semantic figure+caption pairing, not just an unlabeled image.

## Fletcher for node/edge diagrams

Working pattern, confirmed compiling and rendering correctly (wrapped in `#figure` per the placement guidance above). **Remember the import** (see Workflow step 2) — this snippet assumes it's already at the top of the file:
```typst
#figure(
  diagram(
    node-stroke: 0.6pt,
    spacing: (3em, 2em),
    node((0,0), [Step 1], fill: rgb("#dbe9ff"), corner-radius: 4pt),
    node((0,1), [Step 2], fill: rgb("#dbe9ff"), corner-radius: 4pt),
    edge((0,0), (0,1), "-|>", bend: 10deg),
    edge((0,1), (0,0), "-|>", bend: 10deg),
  ),
  caption: [One-line caption explaining both directions/edges since inline labels are omitted here.],
)
```
Coordinates are `(column, row)` tuples; `node-stroke`/`fill`/`corner-radius` style individual nodes; `edge(from, to, "-|>")` draws an arrow (the mark string, not a keyword arg, selects the arrowhead — `"-|>"` and `"->"` both work; `"..|>"` for a dotted arrow). Multiple independent chains can share one `diagram(...)` call using different column indices per chain plus a header-row node with `stroke: none` per chain. Keep edge labels to one short word/phrase and move the full explanation into the figure's `caption` — inline edge labels longer than a word are the classic label-collision failure.

## Depth expectation: default to long-form, nerd-deep, not a summary

Unless explicitly asked for something short, target **10-12 rendered pages** for a full investigation writeup — a short, correct-but-thin report is a failure mode, not an efficient outcome. Concretely, "going deep" means including, wherever the subject matter supports it:
- **Full raw console/log output**, not truncated snippets — actual command + actual output, verbatim, for every claim that came from live system evidence.
- **At least one architecture or state-machine diagram** (via fletcher, see above) showing how the pieces actually connect, not just prose describing it.
- **Register/bitfield or digital-circuit-level detail** when the subject is hardware-adjacent (e.g. a specific config register's bit layout, a signal timing diagram, an ASIC block's port-mapping table, real struct/opcode definitions pulled from actual driver source) — don't stop at "the hardware does X," show the actual mechanism.
- **CPU-instruction/assembly or syscall-level detail** when the subject is software-execution-adjacent (e.g. what syscall a driver actually issues, what the disassembly of the hot path looks like) — don't stop at "the driver does X," show the actual code path.
- **A dedicated Industry/External Context section** (see below) — never present an internal mechanism in isolation.
- **Genuine prior-art/history search** — check the issue tracker, chat history, and git history for whether this exact question was asked before (even years earlier); a report that closes a multi-year-old open question is stronger evidence than one that treats a finding as brand-new.

Structure: (1) Title + Executive Summary, (2) Background, (3) Evidence Timeline/Methodology — one subsection per distinct finding with its own quoted evidence block, usually the longest section, (4) Disambiguation section if the subject's name overlaps with something else, (5) Comparative tables, (6) Industry/external context, (7) Architecture/Synthesis as a numbered end-to-end list paired with diagrams, (8) Open Questions with concrete follow-ups, (9) Tie-in to the originating investigation, (10) Appendix with full raw evidence trimmed from the body. When a draft feels done but short, do one more real round of investigation (another live check, another register dump, another web search, another diagram, another history search for prior art) rather than padding existing prose — depth must come from real content, never from restating the same finding in more words.

## Ground findings against the external/industry ecosystem

Never present an internal or found mechanism in isolation. Once it's understood, search the web for how the broader industry solves the same problem class (e.g. a custom daemon vs. `systemd`/kernel-standard equivalents, competing vendors' analogous tools, relevant IETF/OCP specs). Include a dedicated "Industry Context" section with a comparison table (subject vs. 2-3 named external equivalents) and real cited links/footnotes (using `#footnote[#link(...)]`, never `@footnote[...]`) — this is what elevates a report from an isolated note to grounded technical analysis.

## Content conventions

- Direct, attributed quotes from chat/PR sources (name, channel, date) rather than paraphrasing
- An "Open Questions" section at the end flagging genuine unresolved discrepancies rather than papering over them
- A "tie-in" section connecting the document's research back to whatever originating investigation prompted it
- Footnoted/linked sources so every non-obvious claim is traceable
- Honest confidence-labeling on anything that's the author's own synthesis rather than a directly-confirmed fact (e.g. "reviewed, pending upstream confirmation" rather than overclaiming "resolved")
