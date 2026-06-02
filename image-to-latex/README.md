# image-to-latex

A [Claude Code skill](https://docs.claude.com/en/docs/claude-code/skills) that turns any image into compilable LaTeX. The skill is triggered automatically when a user shows Claude an image and asks for the LaTeX source — phrases like "recreate this in LaTeX", "reproduce this figure", "turn this screenshot into LaTeX", "用 LaTeX 写出来", "复现这张图", "重画" all trigger it.

The skill follows a tight visual feedback loop — **draft → compile → render to PNG → look at the result next to the original → refine** — and iterates until the rendered output visually matches the input. Most documents converge in 2–4 iterations.

## What it handles

| Content type | Reference | Notes |
|---|---|---|
| Tree / hierarchy / taxonomy / mind map | [`references/tree.md`](references/tree.md) | Uses the `forest` package. Includes a worked "paper-taxonomy" example. |
| Table (merged cells, colored headers) | [`references/table.md`](references/table.md) | Uses `booktabs`, `multirow`, `multicolumn`, `colortbl`. |
| Math equation / derivation / matrix | [`references/math.md`](references/math.md) | `align`, `cases`, `bm`, `\mathcal`, `\mathbb`, etc. |
| Code block / quote / theorem / definition box | [`references/code-and-quote.md`](references/code-and-quote.md) | `listings`, `tcolorbox`. |
| Flowchart / architecture diagram | [`references/flowchart.md`](references/flowchart.md) | `tikz` nodes + arrows. |
| Page with several of the above | [`references/mixed-page.md`](references/mixed-page.md) | Combines any of the above. |

Each reference file contains a canonical example plus the few pitfalls that actually break things (e.g. why `[...]` inside a `forest` node label can silently render an empty box, and how to escape it).

## Installation

The skill ships as a single `.skill` zip. To install:

```bash
unzip image-to-latex.skill -d ~/.claude/skills/image-to-latex
```

(Or just copy the contents of this directory into `~/.claude/skills/image-to-latex/`.)

Claude Code picks it up automatically — no restart required.

## Repository layout

```
image-to-latex/
├── SKILL.md                # the manifest Claude reads to decide when to trigger
├── README.md               # this file
├── references/             # per-content-type guides (loaded as needed)
│   ├── tree.md
│   ├── table.md
│   ├── math.md
│   ├── code-and-quote.md
│   ├── flowchart.md
│   └── mixed-page.md
├── scripts/
│   ├── compile_and_preview.sh   # pdflatex + sips, prints PNG path
│   └── grade.py                 # assertion-based grader
└── assets/                 # starting templates (copy one, then edit)
    ├── tree-template.tex
    ├── table-template.tex
    ├── math-template.tex
    ├── flowchart-template.tex
    ├── code-template.tex
    └── mixed-page-template.tex
```

## Usage (from a user perspective)

1. **Open Claude Code and attach an image** (a screenshot from a paper, a photo of a slide, a table from a PDF, etc.).
2. **Ask for the LaTeX.** For example: *"Recreate this tree diagram in LaTeX."* / *"Give me the LaTeX source for this table."* / *"用 LaTeX 复现这张图。"*
3. **Claude reads the matching reference, drafts a `.tex`, compiles, and renders a PNG.** You'll see both files.
4. **Look at the PNG.** If anything is off (a missing label, a wrong color, a branch connecting to the wrong parent), tell Claude what's wrong. The skill iterates.
5. **You're done** when a side-by-side glance at the PNG and the original image looks right.

## Usage (from a developer / skill-author perspective)

The skill is self-contained and has no Python or Node dependencies. It expects a LaTeX toolchain:

- **`pdflatex`** (TeX Live) — for compilation. The compile script also auto-detects `xelatex` when the source uses `ctex` / `fontspec` / Chinese characters.
- **`sips`** — macOS's built-in image tool, used to convert the rendered PDF to PNG. On Linux, swap it for `pdftoppm` or `gs`.

After the skill is installed, the agent runs:

```bash
./scripts/compile_and_preview.sh path/to/output.tex
```

…and gets a matching `path/to/output.png` back.

## What "done" looks like

- All textual content from the original appears in the rendered PNG
- Structural relationships are correct (branches, cell spans, alignment points)
- A human glancing at the two side-by-side would recognize them as the same figure
- The output compiles cleanly with no errors (warnings are fine)

The skill deliberately stops at "visually close" — pixel-perfect reproduction is not the goal.

## Limitations

- The skill does **not** OCR arbitrary images from scratch — it expects the user to have given Claude a high-fidelity image, and Claude uses its multimodal capability to read it. If a region is blank, low-contrast, or illegible, the skill leaves space rather than inventing content.
- It does **not** extract text from PDFs (use a dedicated PDF parser for that).
- It does **not** compile pre-existing `.tex` files written outside the skill's workflow.

## License

MIT.
