# Pricing an American basket put with the finite element method (FreeFem++)

Finite-element pricing of an **American put on the best of two assets** (payoff
`max(K − max(S1, S2), 0)`) under a two-dimensional Black–Scholes model, implemented in
[FreeFem++](https://freefem.org/). The early-exercise feature makes the pricing problem a
**variational inequality** (linear complementarity problem); it is solved at each time step with a
**primal–dual active-set / semi-smooth Newton** iteration on a P1 Lagrange mesh that is
**adaptively refined** around the strike. A European basket put with the same parameters is
priced alongside as a benchmark, which gives the early-exercise premium.

Authors: **Filippo Felice Boggetti, Andrea Cesari, Isacco Lotti** — group project, MSc in
Finance, Bocconi University, January 2024. The accompanying report is
[`report/Finite_Element_Methods_for_Option_Pricing.pdf`](report/Finite_Element_Methods_for_Option_Pricing.pdf)
(15 pp.: problem, variational formulation, weak form and discretisation, semi-smooth Newton
method, results).

## Contents

| File | What it is |
|---|---|
| `VIA-american_r2.edp` | The FreeFem++ script (release r2, 4 Jan 2024): American basket put via the active-set iteration, optional European benchmark, plots and text/gnuplot exports |
| `report/Finite_Element_Methods_for_Option_Pricing.pdf` | The written report |
| `results/diagonal_cut_american_vs_european.txt` | Prices along the diagonal `S1 = S2 = s`, `s = 0 … 79`, from the verified run below (American vs European) |
| `results/run_log_freefem_4.5_macos.txt` | Console log of that run (solver chatter removed) |

## Model and default parameters

Two lognormal assets with volatilities `σ1 = 0.35`, `σ2 = 0.30`, correlation `ρ = −0.3`,
risk-free rate `r = 2 %`, strike `K = 40`, maturity `T = 1` year, time step `dt = 0.01`
(100 steps). Domain `[0, 80]²`, initial uniform mesh `40 × 40`, Dirichlet condition `u = 0` on
the far boundaries; the mesh is re-adapted to the solution during the time loop
(`adaptmesh`, tolerance halved down to `1e-4`). All parameters are set at the top of the script.

`COMPAREMODE = true` (default) also prices the European basket put and produces the comparison
plots and the diagonal cut; set it to `false` for the American-only run with its own plots.

## How to run

You need FreeFem++ (any 4.x; the report's figures were produced with 4.13, the verified run
below used 4.5). From this directory:

```sh
# interactive: opens the FreeFem++ graphics windows, press Enter/Esc to step through the plots
FreeFem++ VIA-american_r2.edp

# headless (no graphics; e.g. on a server or in CI)
FreeFem++ -nw VIA-american_r2.edp
```

The script prints the American price at two points (`uh(0,0)` and `uh(35,41)`), the solve time,
and writes into the working directory: `Compare AMERICAN-EUROPEAN.eps`,
`plotCOMPAREamerican-european.txt` (the diagonal cut), `mm-VIA-american.*` /
`mm-VIA-european.*` (meshes with the solution, gnuplot-readable) and
`graphTime value of the option 3D.txt`. These outputs are git-ignored.

Note (from the script header): mesh and plot windows are tuned for the default financial
parameters — if you change them substantially you may need to resize the domain/mesh and
re-adjust the plot bounding boxes.

## Verified run

Re-run on 2026-09-01 with FreeFem++ v4.5 on macOS (`FreeFem++ -nw VIA-american_r2.edp`):
normal termination, 101 time steps, ~15 s of solve time on a laptop.

| Quantity | Value |
|---|---|
| American put at `S1 = S2 = 0` (`uh(0,0)`) | 40.000 (= `K`, the intrinsic value) |
| American put at `(S1, S2) = (35, 41)` (`uh(35,41)`) | 1.663 |
| European put at `S1 = S2 = 0` | 39.13 (≈ `K e^{−rT}` = 39.21 up to discretisation) |
| Diagonal `s = 30 / 35 / 40 / 45` — American | 9.997 / 4.996 / 1.083 / 0.252 |
| Diagonal `s = 30 / 35 / 40 / 45` — European | 2.897 / 1.084 / 0.351 / 0.106 |

Along the whole diagonal the American price is ≥ the European price and, deep in the money, equals
the intrinsic value `K − s` (immediate exercise is optimal), which is the behaviour discussed in
Section 5.2 of the report. The full cut is in `results/diagonal_cut_american_vs_european.txt`.

## License

© 2024 Filippo Felice Boggetti, Andrea Cesari, Isacco Lotti. Code and report are shared for
reference and portfolio purposes; please contact the authors before reusing them.
