# Low-Label Graph Classification: Poisson and Quantum-Informed Benchmarks

This repository provides a reproducible Python runner for comparing four graph-based semi-supervised learning methods under limited label supervision:

1. **Quadratic-Regularized Poisson Learning**
2. **Original Poisson Learning**
3. **Improved Poisson Learning (IPL, $\mu=0$)**
4. **Quantum-Informed Poisson Semi-Supervised Learning (IPQSSL)**

The program evaluates all methods on matched stratified splits, reports classification accuracy on **unlabeled samples only**, saves seed-level results for statistical analysis, and prints publication-ready LaTeX table rows.

> [!IMPORTANT]
> The default configuration implements the intended experimental protocol, but exact reproduction of previously published values requires identical hyperparameters, loaders, preprocessing, graph construction, and random-seed conventions.

## Experimental scope

The default experiment uses 20 stratified random splits and four label budgets:

$$
\ell \in \{1,2,5,10\}\quad\text{labeled samples per class}.
$$

All four methods use the same graph, labeled subset, source term, and train/test partition for each seed and supervision budget. This paired design permits direct method comparison and supports paired statistical tests.

### Supported datasets

| Dataset | Source | Graph construction |
|---|---|---|
| Iris | scikit-learn | Symmetric Gaussian-weighted $k$-NN |
| Wine | scikit-learn | Symmetric Gaussian-weighted $k$-NN |
| Breast Cancer | scikit-learn | Symmetric Gaussian-weighted $k$-NN |
| Heart Disease | OpenML (`heart-statlog`) | Symmetric Gaussian-weighted $k$-NN |
| German Credit | OpenML (`credit-g`) | Symmetric Gaussian-weighted $k$-NN |
| Optdigits | Local UCI files or OpenML fallback | Symmetric Gaussian-weighted $k$-NN |
| Cora | Canonical Planetoid distribution | Intrinsic citation graph |

Disconnected graphs are restricted to their largest connected component before evaluation.

## Methodological pipeline


flowchart TD
    A[Load and preprocess data] --> B{Dataset type}
    B -->|Tabular| C[Build symmetric Gaussian k-NN graph]
    B -->|Cora| D[Use intrinsic citation graph]
    C --> E[Keep largest connected component]
    D --> E
    E --> F[Generate matched stratified label splits]
    F --> G[Construct mean-zero Poisson source]
    G --> H[Run four learning methods]
    H --> I[Evaluate unlabeled-node accuracy]
    I --> J[Aggregate mean and sample SD]
    J --> K[Save NPZ and print LaTeX rows]


## Core formulations

Let $W$ be the weighted adjacency matrix, $D$ the degree matrix, $L=D-W$ the unnormalized graph Laplacian, and $P=D^{-1}W$ the random-walk matrix.

For labeled set $\mathcal L$, the mean-zero class source is

$$
B_i =
\begin{cases}
e_{y_i}-\bar y, & i\in\mathcal L,\\
0, & i\notin\mathcal L,
\end{cases}
$$

where $e_{y_i}$ is the one-hot class vector and $\bar y$ is the empirical class distribution over labeled samples.

### Poisson variants

The direct Poisson solvers use

$$
(L+\mu I)U=B.
$$

- `mu > 0` produces Quadratic-Regularized Poisson Learning.
- `mu = 0` produces Original Poisson Learning.

The linear systems are solved class by class with diagonally preconditioned conjugate gradients.

### IPL propagation

IPL uses the fixed-point iteration

$$
U^{(m+1)}=PU^{(m)}+D^{-1}B.
$$

### IPQSSL propagation

IPQSSL introduces a circuit-derived diagonal quantity $q_\theta$ and the nodewise damping vector

$$
d_\theta=\frac{1}{1+\gamma q_\theta}.
$$

The propagation becomes

$$
U^{(m+1)}=\operatorname{diag}(d_\theta)
\left(PU^{(m)}+D^{-1}B\right).
$$

Circuit parameters are optimized with simultaneous perturbation stochastic approximation (SPSA) using labeled cross-entropy. The circuit uses data-encoding $R_Y$ rotations, trainable $R_Y$ layers, and a ring of controlled-$Z$ gates.

## Repository layout

The experiment script is expected to be stored as:


.
├── README.md
├── run_published_table.py
└── data/
    ├── optdigits.tra        # optional local UCI Optdigits data
    ├── optdigits.tes        # optional local UCI Optdigits data
    └── ind.cora.*           # downloaded automatically when required


## Requirements

- Python 3.9 or later
- NumPy
- SciPy
- scikit-learn
- pandas, required by the German Credit loader
- PennyLane, optional

Install the standard dependencies:


python -m pip install numpy scipy scikit-learn pandas


For the optional PennyLane backend:


python -m pip install pennylane


## Running the experiments

Run all seven datasets with the default 20 seeds:


python run_published_table.py


Run selected datasets:


python run_published_table.py --datasets iris wine cora


Run a short diagnostic experiment:


python run_published_table.py --datasets optdigits --seeds 2


Use the PennyLane statevector backend:


python run_published_table.py --backend pennylane --seeds 2


Use a relative Frobenius convergence criterion:


python run_published_table.py --relative-tol


Specify data and output locations:


python run_published_table.py \
  --data-dir ./data \
  --out published_table_results.npz


### Command-line arguments

| Argument | Default | Description |
|---|---:|---|
| `--datasets` | All datasets | One or more datasets to evaluate |
| `--data-dir` | `./data` | Dataset cache and local-data directory |
| `--seeds` | `20` | Number of matched random splits |
| `--relative-tol` | Disabled | Use relative rather than absolute convergence tolerance |
| `--backend` | `numpy` | Circuit backend: `numpy` or `pennylane` |
| `--out` | `published_table_results.npz` | Output archive path |

## Default configuration

The primary settings are stored in the script-level `CONFIG` dictionary:

| Parameter | Default | Meaning |
|---|---:|---|
| `budgets` | `[1, 2, 5, 10]` | Labels per class |
| `n_seeds` | `20` | Number of stratified splits |
| `knn_k` | `10` | Number of nearest neighbors |
| `mu` | `0.1` | Quadratic regularization strength |
| `gamma` | `1.0` | IPQSSL damping strength |
| `n_qubits` | `4` | Circuit-register width |
| `n_layers` | `2` | Number of variational layers |
| `spsa_iters` | `30` | SPSA optimization iterations |
| `eps` | `1e-6` | Propagation convergence tolerance |
| `max_iter` | `500` | Maximum propagation iterations |
| `cg_tol` | `1e-8` | Conjugate-gradient tolerance |
| `cg_maxiter` | `2000` | Maximum conjugate-gradient iterations |

The parameters marked as experimental flags in the source must be matched to the final manuscript protocol before claiming exact numerical reproduction.

## Outputs

The program creates an `.npz` archive containing, for every selected dataset:

- `<dataset>_acc`: seed-level accuracies with shape `(methods, seeds, budgets)`;
- `<dataset>_mean`: mean accuracy across seeds;
- `<dataset>_std`: sample standard deviation across seeds;
- `methods`: ordered method names;
- `budgets`: label budgets.

Load the archive as follows:


import numpy as np

results = np.load("published_table_results.npz", allow_pickle=True)

print(results["methods"])
print(results["budgets"])
print(results["iris_mean"])
print(results["iris_std"])


The console also prints ready-to-paste LaTeX rows of the form


\multirow{4}{*}{Iris}
 & Quadratic-Reg.\ Poisson & $0.xxx \pm 0.xxx$ & ... \\
 & Original Poisson        & $0.xxx \pm 0.xxx$ & ... \\
 & IPL ($\mu=0$)           & $0.xxx \pm 0.xxx$ & ... \\
 & \textsc{IPQSSL}         & $\mathbf{0.xxx \pm 0.xxx}$ & ... \\


Means and sample standard deviations are rounded to three decimal places. The highest displayed mean for each dataset and label budget is bolded; ties at the displayed precision are bolded jointly.

## Statistical analysis

The saved seed-level arrays should be used for paired significance testing because every method is evaluated on identical splits. Do not perform statistical tests on rounded table values.

For multiple pairwise comparisons, apply a family-wise correction such as Holm's procedure to the raw paired test $p$-values.

## Reproducibility notes

- Random generators use deterministic seeds `1000 + seed_index`.
- Tabular features are standardized before graph construction.
- The Gaussian bandwidth is the median pairwise distance computed from a deterministic subsample of at most 2,000 observations.
- Circuit inputs are reduced to `n_qubits` dimensions using PCA or Truncated SVD and min-max scaled to $[0,\pi]$.
- NumPy is the default exact-statevector backend.
- Selecting PennyLane triggers a numerical equivalence check against the NumPy backend.
- Cora files are downloaded automatically from the canonical Planetoid repository.
- Optdigits preferentially uses local `optdigits.tra` and `optdigits.tes` files, with OpenML as fallback.

## Important limitations

1. **This is not an exact reproduction guarantee.** Published results cannot be claimed unless all final experimental settings and data-loading conventions are identical.
2. **Heart Disease and German Credit preprocessing may differ across sources.** Replace the OpenML loaders if the manuscript used another preprocessing pipeline.
3. **Pendigits is not included.** If it remains a headline result, it must be regenerated under the same finalized method set or added through a consistent loader.
4. **Absolute stopping tolerances are scale-dependent.** Use `--relative-tol` when a relative convergence rule is required, and report the selected rule consistently.
5. **Single-seed runs are diagnostic only.** The script reports zero standard deviation for one seed to avoid `NaN`; such runs are not suitable for inferential claims.

## Citation

If this code supports a publication, add the final bibliographic record and software citation here:


@article{ipqssl,
  title   = {Quantum-Informed Poisson Learning for Graph-Based Semi-Supervised Classification},
  author  = {Author names},
  journal = {Journal name},
  year    = {Year},
  doi     = {DOI}
}


## License

Add the repository license before public release. A research-code repository should state explicitly whether reuse is governed by MIT, BSD-3-Clause, Apache-2.0, GPL, or another license.

