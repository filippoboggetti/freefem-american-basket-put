# Verification record

## Revised script (american_basket_put.edp)

- Date: 2026-09-01 · FreeFem++ v4.5 (macOS) · command: `FreeFem++ -nw -ne -v 0 american_basket_put.edp`
- Termination: `Ok: Normal End`; summary block:

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

- Diagonal cut kept as `diagonal_cut_american_vs_european.txt`:

```
62dba8f18a35ab77b8589449482baa6e22af75a9cb05ad179ad7fd902655dc2b  diagonal_cut_american_vs_european.txt
```

## Original script (original/VIA-american_r2.edp)

- Same date/version/command; `Ok: Normal End`, 101 time steps, `uh(0,0) = 40`, `uh(35,41) = 1.66276`, solve time 15.5 s.
- Diagonal cut kept as `diagonal_cut_original_script.txt`:

```
6f0401a39c69c9e709645fa5dc2202d53db2f98c4908122eacb93e03f7d2cc99  diagonal_cut_original_script.txt
```
