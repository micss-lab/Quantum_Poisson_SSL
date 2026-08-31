# Overleaf overlay

Upload these two files into the existing Overleaf project (or import this GitHub repo):

- `Main.tex` (this folder)
- `Main.bib` (this folder)

Then add, next to `Main.tex`, if you have them:

- `fig3_depth_scaling_diagnostics.png`
- `fig4_width_scaling_diagnostics.png`

The manuscript compiles without the PNGs. Placeholder boxes appear until the figures are uploaded.

## What this overlay already fixes

1. Duplicate `\usepackage{natbib}` removed. Keep only `\usepackage[numbers]{natbib}`.
2. Code-availability paragraph now points to `https://github.com/micss-lab/IPQSSL`.
3. Low-label table no longer prints `\mathbf{??}` for missing standard deviations. Cells show the published means only. The caption no longer claims standard deviations that are not in the source.
4. Missing diagnostic figures no longer abort compilation.

## Still for the authors (do not invent)

- Confirm equal-contribution footnotes with all co-authors (`TODO` near `\author`).
- Replace the illustrative sparsity `s = 15` with the measured maximum symmetrized row degree.
- After `python src/run_published_table.py` finishes, fill cell-wise standard deviations from the saved `.npz` and restore `mean ± std` in Table `tab:low-label-results`.
- Upload the two PNG figures when they are ready.
