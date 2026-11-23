# Distance geometry problem solver

Distance geometry problem (DGP) is the problem of recovering coordinates of points from pairwise distances. This project contains a three-phase DGP solver (theory: Yajima, 2002) and an augmented Lagrangian variant that reduces peak memory usage to roughly 1/10.

Note. `dgp_master_thesis_code.ipynb` is the coding part of a master's thesis project. All code cells are committed with their outputs, so the notebook can be read without running anything.

距离几何问题（从稀疏的成对距离恢复点的坐标）：这个项目包含一个三阶段的 SDP 求解器，以及一个使用增广拉格朗日方法把峰值内存降低约一个数量级的变体。`dgp_master_thesis_code.ipynb` 是硕士论文项目的代码部分，所有单元都保留了输出，因此 notebook 可以直接阅读。

## Results

All results can be found in the outputs of the notebook. Distances are the number of pairwise distances we know. In the phase 1/2/3 columns, the data reads error/time. Error is the L in §1.3, and the lower, the better the estimate to the true conformation. Time is measured in seconds. Random rows average over 5 instances.

### Three-phase solver

| Problem | Points | Distances | Phase 1 | Phase 2 | Phase 3 |
|---|---|---|---|---|---|
| Lattice | 64 | 360 | 106.48 / 0.50 | 44.73 / 0.038 | 4.7e-07 / 0.94 |
| Lattice | 64 | 468 | 205.64 / 0.33 | 86.21 / 0.047 | 2.8e-07 / 1.14 |
| Lattice | 125 | 780 | 359.24 / 9.84 | 118.94 / 0.143 | 1.1e-06 / 3.37 |
| Lattice | 125 | 1036 | 442.14 / 6.70 | 91.19 / 0.216 | 2.0e-07 / 4.59 |
| Lattice | 216 | 1440 | 640.27 / 272.26 | 261.91 / 0.371 | 1.0e-09 / 10.28 |
| Lattice | 216 | 1940 | 1125.75 / 244.42 | 409.87 / 0.550 | 5.3e-05 / 14.57 |
| Random | 100 | 997 | 1.1e-05 / 3.00 | 2.5e-06 / 0.21 | 5.1e-07 / 2.56 |
| Random | 100 | 2015 | 3.2e-06 / 2.53 | 2.1e-06 / 0.60 | 6.5e-07 / 10.18 |
| Random | 100 | 3059 | 3.8e-06 / 2.22 | 3.0e-06 / 1.32 | 1.2e-06 / 18.82 |
| Random with noise | 100 | 981 | 4.78 / 4.38 | 1.85 / 0.21 | 0.713 / 2.14 |
| Random with noise | 100 | 1960 | 2.19 / 4.78 | 1.18 / 0.58 | 0.853 / 6.07 |
| Random with noise | 100 | 2984 | 1.44 / 5.16 | 1.05 / 1.25 | 0.906 / 12.55 |
| Protein 1A1P | 63 | 187 | 1078.80 / 0.41 | 808.37 / 0.014 | 7.89 / 0.77 |

In the lattice case, an exact fit exists and the Phase 3 column is solver tolerance. With 5% noise, the distances are mutually inconsistent, so the final error (0.7–0.9) is likely the noise floor.

Protein 1A1P has 205 points and 1736 distances before reduction, 63 and 187 after. Its distances come from real atomic coordinates, so an exact rank-3 fit does exist and the gap belongs to the heuristic, not the model.

Note. Phase 1 scales badly in both time and memory since it solves a large SDP.

### Starting from a random rank-3 matrix

| Start | Points | Distances | Phase 2 | Phase 3 |
|---|---|---|---|---|
| SDP solution | 216 | 1440 | 261.91 / 0.37 | 1.0e-09 / 10.28 |
| Random rank-3 | 343 | 8673 | 2088.67 / 0.83 | 846.59 / 35.19 |

The second row uses a randomly generated rank-3 matrix to replace phase 1, which fails on the large case with 343 points. It is averaged over 10 instances. Phase 3 ends at 846.59, the same order as the phase 1 error above (106 to 1126). So phase 3 only refines a good start.

### Augmented Lagrangian vs phase 1

The augmented Lagrangian method is a first-order method that replaces phase 1.

| Method | Points | Distances | Peak memory |
|---|---|---|---|
| Phase 1 (SDP) | 64 | 360 | 38.47 MB |
| Augmented Lagrangian | 64 | 360 | 2.18 MB |
| Phase 1 (SDP) | 125 | 780 | 433.67 MB |
| Augmented Lagrangian | 125 | 780 | 48.01 MB |

The peak memory is RSS growth over a warmed baseline, and drifts between runs because MOSEK keeps a memory pool. The numbers are lower bounds, so read them as an order of magnitude.

## Quick start

Python 3.11 is required; the pinned versions have no wheels for newer Python versions. MOSEK is a commercial solver, so a license file is needed and is available from https://www.mosek.com/.

`requirements.txt` downgrades numpy to 1.23.5. Section 4.3 downloads 1A1P from RCSB PDB and needs network access.

**快速开始**

需要 Python 3.11，因为锁定的版本在新版解释器上没有预编译包。MOSEK 是商业求解器，需要一个 license 文件，可从官网获取，见 https://www.mosek.com/。

`requirements.txt` 会把 numpy 降到 1.23.5。第 4.3 节会从 RCSB PDB 下载 1A1P，需要联网。

## Notes

- All cells have their outputs, so the notebook can be read without running it.
- Phase 1 ran out of memory on the case (343, 8673).
- The accuracy of the augmented Lagrangian could be improved by changing the stopping criterion.

**说明**

- 所有单元都保留了输出，因此 notebook 无需运行即可阅读。
- Phase 1 在 (343, 8673) 这个实例上内存不足。
- 增广拉格朗日的精度可以通过调整停止判据来提升。

## Attribution

The augmented Lagrangian code is adapted from [ChristophorusX/Burer-Monteiro](https://github.com/ChristophorusX/Burer-Monteiro/blob/master/burer_monteiro.py) (Apache-2.0). Functions, variables, constraints and convergence criterion are rewritten for distance constraints instead of unit row norms.

## References

1. Yajima, Y. (2002). Positive semidefinite relaxations for distance geometry problems. *Japan Journal of Industrial and Applied Mathematics*, 19, 87–112.
2. Burer, S., & Monteiro, R. D. C. (2003). A nonlinear programming algorithm for solving semidefinite programs via low-rank factorization. *Mathematical Programming*, 95(2), 329–357.

## License

Apache License 2.0, see [LICENSE](LICENSE).
