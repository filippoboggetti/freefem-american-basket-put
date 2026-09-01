# Verification record

- Date: 2026-09-01
- Software: FreeFem++ v4.5 (macOS build), run with `FreeFem++ -nw VIA-american_r2.edp` (`-ne -v 0` added to quiet the console)
- Termination: `Ok: Normal End`, 101 time steps × 2–3 active-set iterations each (310 inner iterations in total), solve time 15.5 s
- Parameters: defaults in the script (T=1, σ1=0.35, σ2=0.30, ρ=−0.3, r=0.02, K=40, dt=0.01, m=40, L=LL=80, COMPAREMODE=true)
- Printed values: `uh(0,0) = 40`, `uh(35,41) = 1.66276`
- Diagonal cut written to `plotCOMPAREamerican-european.txt`, kept here as `diagonal_cut_american_vs_european.txt`:

```
6f0401a39c69c9e709645fa5dc2202d53db2f98c4908122eacb93e03f7d2cc99  results/diagonal_cut_american_vs_european.txt
```

See the README section *Numerical accuracy and known limitations* for the tolerances observed in this run.
