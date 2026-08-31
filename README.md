# IPQSSL

Reference implementation for **Improved Poisson Quantum Semi-Supervised Learning** (IPQSSL): quantum-informed regularized Poisson learning for graph-based semi-supervised classification.

Paper: *Quantum-Informed Regularized Poisson Learning for Graph-Based Semi-Supervised Classification*

Code: [https://github.com/furkanpolaat/IPQSSL](https://github.com/furkanpolaat/IPQSSL)

Lab affiliation: [MICSS Lab](https://github.com/micss-lab) (University of Antwerp)

## What this repo contains

| Path | Role |
| --- | --- |
| `src/run_published_table.py` | Table runner for the low-label experiment |
| `requirements.txt` | Python dependencies |

The runner evaluates four methods on Iris, Wine, Breast Cancer, Heart Disease, German Credit, Optdigits, and Cora:

- Quadratic-regularized Poisson
- Original Poisson
- IPL (`mu = 0`)
- IPQSSL

Protocol: 20 stratified seeds, label budgets `l in {1, 2, 5, 10}` per class, accuracy on unlabeled nodes only. All methods share the same graph, source term, and splits.

## Setup

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

PennyLane is optional. Install it only if you want the exact-statevector cross-check:

```bash
pip install pennylane
```

## Reproduce the published table

Small sanity check (Iris and Wine, 2 seeds):

```bash
python src/run_published_table.py --datasets iris wine --seeds 2 --out iris_wine_smoke.npz
```

Full suite (20 seeds, all seven datasets):

```bash
python src/run_published_table.py --out published_table_results.npz
```

Selected datasets or the PennyLane backend:

```bash
python src/run_published_table.py --datasets optdigits cora
python src/run_published_table.py --backend pennylane --seeds 2
```

The script prints LaTeX rows for the low-label results table and writes per-seed accuracies to an `.npz` file for Holm-corrected paired tests.

### Data

- Iris, Wine, and Breast Cancer load from scikit-learn.
- Heart Disease (`heart-statlog`) and German Credit (`credit-g`) load from OpenML.
- Optdigits prefers local `data/optdigits.tra` and `data/optdigits.tes` (UCI). If those files are absent, it falls back to OpenML. Do not use `sklearn.datasets.load_digits()`: that is only the 1797-sample test subset.
- Cora downloads the Planetoid distribution into `--data-dir` (default `./data`).

Cora uses its intrinsic citation graph (symmetrized, self-loops removed, largest connected component). The tabular datasets use a symmetric 10-NN Gaussian graph with median-pairwise-distance bandwidth.

## Implementation note

`run_published_table.py` implements the Section 4 table protocol with the defaults in `CONFIG`. It will not reproduce published numbers exactly unless those constants, loaders, graph construction, and the seed protocol match the runners that generated the manuscript table. Use the published Iris and Wine Original-Poisson rows as a consistency check before trusting a full sweep.

Pendigits is not in this suite. If you need that dataset, add a dedicated loader.

## Citation

```bibtex
@software{ipqssl2026,
  title  = {IPQSSL: Quantum-Informed Regularized Poisson Learning},
  author = {Gholipour, Hamed and Bozorgnia, Farid and Mohammadigheymasi, Hamzeh
            and Proen{\c{c}}a, Hugo and Neves, Jo{\~a}o and Challenger, Moharram},
  year   = {2026},
  url    = {https://github.com/furkanpolaat/IPQSSL}
}
```

## License

Apache License 2.0. See `LICENSE`.
