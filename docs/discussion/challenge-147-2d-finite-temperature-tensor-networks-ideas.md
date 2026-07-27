# Ideas Session - 2026-07-27 17:02

## User request

The user asked to read QuantumBFS/quantum.harness issue #147, surface 3-5 distinct research angles, name the riskiest assumption in each, recommend one starting point, and answer in Chinese.

## Source evidence

Issue #147 is `[challenge]: 2D finite-temperature tensor networks`, released by Wei Li (Institute of Theoretical Physics, Chinese Academy of Sciences). It is open and carries the `accepted` and `challenge` labels; it does not carry the `autoresearch` label. The issue page shows no linked pull request.

The target is the transverse-field Ising model on a 10x10 square lattice with open boundaries:

`H = -J sum_<i,j> sigma_i^z sigma_j^z - h sum_i sigma_i^x`, with `J = 1`, `h/J` near the 2D quantum critical value 3.044, and examples `h/J = 2.5, 3.0, 3.5`.

Mandatory outputs cover `beta J` from 0.1 to 1.0: free-energy density `f`, internal-energy density `u`, specific heat `C`, convergence in PEPO bond dimension or METTS sample count, QMC validation on the same finite lattice, source code, technical documentation, and a one-command reproduction script. PEPO should show at least three bond dimensions; METTS should demonstrate below 1% statistical error in `u` and below 3% in `C` at `beta J = 0.8`. Uniform susceptibility and a tanTRG cost comparison are bonuses.

Relevant model and method-card constraints:

- The Hamiltonian uses Pauli matrices, not spin operators; confusing the two changes bond terms by a factor of four.
- The model is sign-problem free, so SSE QMC is an appropriate finite-size oracle.
- The global spin-flip Z2 symmetry can reduce tensor cost.
- Near criticality, both PEPS bond dimension and contraction environment dimension require independent convergence checks.
- Specific heat is especially fragile because numerical differentiation amplifies contraction and truncation noise; a direct fluctuation estimator is costlier but cleaner.

## Angles explored

### 1. Deterministic Trotter-PEPO with variational compression

Initialize the infinite-temperature operator or a purification, apply a second-order Trotter decomposition in imaginary time, and compress after each layer using a full-environment or variational update. Use a boundary-MPS or finite-PEPS contraction for the 10x10 open patch. Track normalization factors for `f`, use direct insertions for `u`, and compare fluctuation and differentiated estimators for `C`.

Most dangerous assumption: bond dimensions around `D = 4, 6, 8` plus feasible environment dimensions are sufficient to control critical correlations through `beta J = 1`; otherwise both memory and contraction cost may become prohibitive.

### 2. Globally optimized vectorized thermal PEPS

Represent the square root of the density matrix as a vectorized PEPS and optimize a thermal residual or free-energy objective globally, using stochastic reconfiguration or automatic differentiation and continuation in beta. This follows the direction of Zhang et al., Phys. Rev. B 111, 075146 (2025), cited by the issue.

Most dangerous assumption: the global optimizer and its metric remain well conditioned near the critical fan on a finite open lattice, so optimizer error does not dominate the requested thermodynamic derivatives.

### 3. Two-dimensional PEPS-METTS

Generate minimally entangled typical thermal states by imaginary-time evolving product states into PEPS, measure observables, then collapse in alternating bases. Run independent chains in parallel and reconstruct free energy by thermodynamic integration from the high-temperature limit.

Most dangerous assumption: autocorrelation and sample variance remain low enough near `h/J approximately 3` and `beta J = 0.8` to reach the issue's 1% (`u`) and 3% (`C`) thresholds before per-sample PEPS evolution becomes too expensive. Free energy is also indirect and inherits quadrature error.

### 4. Hybrid low-D PEPO plus METTS correction

Use a cheap low-bond-dimension PEPO as a deterministic baseline or control variate, then estimate its residual bias with PEPS-METTS samples. Pair Z2-related samples and share contraction environments where possible. The intended payoff is lower variance than plain METTS and less bias than the low-D PEPO.

Most dangerous assumption: the stochastic correction is strongly correlated with the PEPO baseline; if the correlation is weak, the hybrid simply pays the cost of both methods without improving error per wall-clock time.

## Recommendation

Start with angle 1, but begin by building the validation spine: SSE QMC on the exact 10x10 geometry and tiny-lattice exact checks for gates, normalization, and insertion tensors. Then run one narrow PEPO pilot at `h/J = 3.0`, `beta J = 0.1, 0.5`, and small `D` before expanding the grid. This route maps most directly onto every mandatory deliverable, produces `f` naturally, gives deterministic convergence plots, and has the cleanest early stop test. Angle 2 is the best fallback or follow-on if local Trotter compression, rather than contraction cost, is the observed bottleneck.
