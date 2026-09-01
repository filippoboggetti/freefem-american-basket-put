# Pricing an American put on the maximum of two assets with the finite element method (FreeFem++)

Finite-element pricing of an **American put on the best of two assets** (payoff `max(K − max(S1, S2), 0)`)
under a two-dimensional Black–Scholes model, implemented in [FreeFem++](https://freefem.org/). The
early-exercise feature makes the pricing problem a **variational inequality** (linear complementarity problem);
it is solved at each backward time step with a **primal–dual active-set / semi-smooth Newton** iteration on a
P1 Lagrange mesh that is **adaptively refined** around the strike. A European put with the same payoff is priced
alongside as a benchmark, which gives the early-exercise premium.

Authors of the project: **Filippo Felice Boggetti, Andrea Cesari, Isacco Lotti** — group project, MSc in Finance,
Bocconi University, January 2024. The report is
[`report/Finite_Element_Methods_for_Option_Pricing.pdf`](report/Finite_Element_Methods_for_Option_Pricing.pdf)
(15 pp.: problem, variational formulation, weak form and discretisation, semi-smooth Newton method, results).

## Contents

| File | What it is |
|---|---|
| `american_basket_put.edp` | **Revised script (September 2026)** — same numerical method as the original, with the corrections listed below and built-in checks |
| `original/VIA-american_r2.edp` | The script as submitted with the project (release r2, 4 Jan 2024), kept verbatim |
| `report/Finite_Element_Methods_for_Option_Pricing.pdf` | The written report (January 2024) |
| `results/diagonal_cut_american_vs_european.txt` | Prices along the diagonal `S1 = S2 = s`, `s = 0 … 79`, from the verified run of the revised script |
| `results/diagonal_cut_original_script.txt` | The same cut from the original script, for comparison |
| `results/VERIFICATION.md` | Record of the verified runs (version, command, invariants, SHA-256) |

## Model and default parameters

Two lognormal assets with volatilities `σ1 = 0.35`, `σ2 = 0.30`, correlation `ρ = −0.3`, risk-free rate `r = 2 %`,
strike `K = 40`, maturity `T = 1` year, time step `dt = 0.01` (100 backward steps). Domain `[0, 80]²`, initial
uniform mesh `40 × 40`, Dirichlet condition `u = 0` on the far boundaries; the mesh is re-adapted to the solution
during the time loop (`adaptmesh`, tolerance halved down to `1e-4`). Financial and algorithm parameters are set at
the top of the script; the European benchmark's mesh-adaptation settings are in its own block.

`COMPAREMODE = true` (default) also prices the European put, runs the American-vs-European check and writes the
comparison outputs; set it to `false` for the American-only run with its own plots.

## What the revised script changes

The original project code priced correctly in substance but had three rough edges, all fixed in
`american_basket_put.edp` without touching the discretisation or the active-set algorithm:

1. **Time loop.** The original used an inclusive test (`while (iter*dt <= T)`), i.e. 101 steps and a maturity of
   1.01 years; the revised script takes exactly `nSteps = T/dt = 100` steps (both for the American and the
   European solve).
2. **Terminal condition.** The lagged solution `uhp` entering the first backward step now starts at the payoff,
   `u(T) = max(K − max(S1,S2), 0)`, instead of at 0.
3. **Active-set iteration.** The inner loop was capped at 7 iterations with no record of whether it had converged.
   The cap is now 30 and the script counts the time steps that reach it without the convergence test firing and
   prints the largest remaining change (2 of 100 steps, largest L2 change 0.014, in the verified run — these are
   the steps on which the mesh is re-adapted).

It also prints, at the end of the run, two **no-arbitrage checks** on the nodal values — American ≥ intrinsic
value, and (in `COMPAREMODE`) American ≥ European, the latter with the European solution interpolated on the
American mesh — each against a tolerance of 0.02 (0.05 % of the strike).

Effect on the prices: negligible at the scale of the discretisation error (e.g. `uh(35,41)` 1.6628 → 1.6621).

| `S1 = S2 = s` | American, original | American, revised | European, original | European, revised |
|---|---|---|---|---|
| 0 | 40.000 | 40.000 | 39.130 | 39.140 |
| 20 | 19.997 | 19.998 | 11.794 | 11.840 |
| 30 | 9.997 | 9.997 | 2.897 | 2.917 |
| 35 | 4.996 | 4.999 | 1.084 | 1.088 |
| 38 | 2.049 | 2.048 | 0.561 | 0.557 |
| 40 | 1.083 | 1.083 | 0.351 | 0.349 |
| 42 | 0.601 | 0.602 | 0.219 | 0.216 |
| 45 | 0.252 | 0.254 | 0.106 | 0.104 |
| 50 | 0.061 | 0.061 | 0.029 | 0.033 |

## How to run

FreeFem++ 4.x (the report's figures were made with 4.13; the verified runs used 4.5). From this directory:

```sh
FreeFem++ american_basket_put.edp        # interactive: graphics windows, Enter/Esc to step through
FreeFem++ -nw american_basket_put.edp    # headless
```

Console output ends with the summary block (solve time, steps, adaptations, steps at the iteration cap, the two
sample prices, the two checks). Files written to the working directory: `Compare AMERICAN-EUROPEAN.eps`,
`plotCOMPAREamerican-european.txt` (the diagonal cut), `mm-VIA-american.*` / `mm-VIA-european.*` (meshes with the
solution, gnuplot-readable) and `graphTime value of the option 3D.txt`; the American-only branch writes
`VIA-*american.eps`, `mm-VIA-american.*` and `plotVIA-american.gp`. All of these are git-ignored.

## Verified run (revised script, FreeFem++ 4.5, macOS, 2026-09-01)

```
    solve time (s)          : 10.7413
    time steps              : 100
    mesh adaptations        : 5  (final dof 5888)
    steps at iteration cap  : 2 of 100  (largest L2 change at exit 0.014248)
    uh(0,0)   = 40   (intrinsic value K = 40)
    uh(35,41) = 1.66208
    check  American - intrinsic >= 0 : min over nodes = -3.55271e-15   PASS (tol 0.02)
    uhe(0,0)  = 39.14   (K e^{-rT} = 39.2079)
    check  American - European >= 0  : min over nodes = -0.0137027 at (S1,S2) = (0,65.7814)   PASS (tol 0.02)
```

Along the diagonal the American price equals the intrinsic value `K − s` in the exercise region and stays above
the European price; the largest violation of American ≥ European over all nodes is 0.014, at `(S1, S2) ≈ (0, 66)`
where both prices are essentially zero — interpolation between two independently adapted meshes, not a solver
error. The original script gives the same picture up to its 101st time step and the 0-initialisation of `uhp`
(see `results/diagonal_cut_original_script.txt`).

## Remarks on the report

The report describes the European benchmark as obtained from a closed-form formula; in both scripts it is computed
with a second finite-element solve. The report is kept as submitted.

## License

© 2024–2026 Filippo Felice Boggetti, Andrea Cesari, Isacco Lotti. Code and report are shared for reference and
portfolio purposes; please contact the authors before reusing them.
