deviation across seeds;
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

