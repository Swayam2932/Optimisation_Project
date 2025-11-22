# Ethanol Production Optimization — Overview & How to Use

This repository contains a Jupyter notebook (`optimization_problem.ipynb`) that demonstrates an optimization model to maximize profit for ethanol production by tuning four decision variables:
- `C_p`: initial glucose concentration (g/L)
- `X_0`: initial biomass (g/L)
- `k_E`: temperature controller gain
- `k_pH`: pH controller gain

This README explains, at a surface level, how the code is structured, how the model was created, the requirements to run it, simple step-by-step usage, and short definitions of the main terms used.

---

## 1) Surface-level code structure
The project is implemented as a single notebook `optimization_problem.ipynb`. The notebook is organized into sequential sections (cells):

1. Imports — `numpy`, `scipy.optimize`, `matplotlib`, and small helper imports.
2. Problem parameters — physical, biological, and economic parameters (e.g., `K_s`, `C_inhib`, `Price_EtOH`, cost coefficients). These are set in a parameter cell so you can quickly change scenarios.
3. Objective function — a function that computes negative profit (so minimizers maximize profit). It combines a revenue model and a cost model.
4. Bounds & initial guess — bounds for each decision variable and an initial guess for the optimizer.
5. Optimization method — the notebook uses a penalty approach combined with BFGS by default.
6. Result extraction & diagnostics — clamps values to bounds (if needed), computes revenue/cost breakdowns, shows optimization status, and prints diagnostics.
7. Visualizations — sensitivity plots for each variable and comparison bar/stack charts.
8. Documentation cells — detailed markdown explanation of model design and interpretation.

All code is self-contained inside the notebook; you can run cells sequentially to reproduce results.

---

## 2) How the model is created (high-level)
The model computes Profit = Revenue − Cost. Key modeling choices:

A. Revenue (multiplicative structure)
- Monod-like saturation: `C_p / (K_s + C_p)` — models how production increases with substrate but saturates.
- Biomass saturation: `1 - exp(-alpha_R * X_0)` — diminishing returns from adding starter biomass.
- Inhibition term: `1 - (C_p**2)/(C_inhib**2 + C_p**2)` — models high-substrate inhibition.
- Small revenue interaction: `1 + gamma_1 * C_p * X_0 + gamma_2 * k_E * k_pH` — rewards synergistic configurations.

These multiplicative factors produce a natural internal sweet spot (a peak in revenue at intermediate substrate and biomass levels).

B. Cost (additive structure)
- Material costs: `a_1*C_p + a_2*C_p**2 + b_1*X_0 + b_2*X_0**2 + c_1*C_p*X_0` (linear + quadratic + cross-term).
- Operational costs: `d_1*k_E**2 + d_2*k_pH**2 + d_3*k_E*k_pH + e_1*C_p*k_E + e_2*X_0*k_pH` (captures controller energy/effort and interactions).

Note: an earlier joint multiplicative interaction *cost* term was removed (you requested it removed). The revenue interaction remains unless you request its removal as well.

C. Objective returned to optimizer
- The implemented `objective_function` returns negative profit (i.e., `-profit`) because `scipy.optimize.minimize` minimizes by default.

---

## 3) Why these choices are realistic
- Biological production saturates with substrate and biomass — Monod and exponential saturation models capture common biochemical kinetics.
- Very high substrate can inhibit microbes — the inhibition rational function provides a smooth reduction in effective revenue at high `C_p`.
- Costs often grow non-linearly as you push variables (energy, wear, and handling complexity) — quadratic terms reflect increasing marginal costs.
- Cross-terms capture coupling: e.g., higher biomass changes effective material handling or increases control cost.

These forms are commonly used in engineering models: they balance realism and differentiability needed for gradient-based optimization.

---

## 4) Requirements
- Python 3.8+ (Python 3.10 recommended)
- Jupyter or JupyterLab (to run the notebook interactively)
- Required Python packages:

```powershell
pip install numpy scipy matplotlib ipython
```

(If you use an environment manager such as `conda`, create an environment and install the same packages.)

---

## 5) How to run 
1. Open a terminal in the `code` folder (where `optimization_problem.ipynb` lives).
2. (Optional) Create & activate a virtual environment.
3. Install requirements (see commands above).
4. Launch Jupyter and open `optimization_problem.ipynb`.
5. Run the notebook top-to-bottom. Recommended sequence:
   - Run the imports cell (top) to ensure dependencies load.
   - Run the parameter cells if you want to change values.
   - Run the objective/optimization cell (it will attempt to solve and print progress).
   - Run the result extraction and visualization cells to see breakdowns and plots.

Notes about optimization: the notebook uses a penalty method together with BFGS in the default flow. If you want strict enforcement of bounds (recommended), change the optimizer call to `method='L-BFGS-B'` and pass `bounds=bounds` to `minimize()`.

---

## 6) Simple definitions
- Monod term: a function that models how the rate of production increases with substrate up to a saturation point.
- Biomass saturation: additional cells (biomass) increase production, but each additional unit produces less extra output (diminishing returns).
- Inhibition: when substrate is too high it can reduce yield (microbes stressed or poisoned by high concentrations).
- Quadratic cost: a cost that grows as the square of a variable — used to model increasing marginal cost (e.g., energy, wear).
- Penalty method: a way to enforce constraints by adding a large penalty to the objective when a solution violates bounds. It converts a constrained problem into an unconstrained one.
- BFGS: gradient-based optimization algorithms. BFGS is unconstrained; L-BFGS-B supports box constraints.
- Negative profit objective: because `scipy.optimize.minimize` minimizes, the notebook hands it `-profit` so minimizing negative profit equals maximizing real profit.

---

## 7) Where to look in the notebook
- Parameter definitions: the parameter cell near the top (search for `Define problem parameters`).
- Objective function: cell with `def objective_function(...)`.
- Optimization call: the cell that calls `minimize(...)` and prints optimization output.
- Diagnostics: the diagnostics cell (penalty checks) is near the end and prints per-variable violations.
- Visualizations: the cell with a `matplotlib` subplot that produces the 4 sensitivity plots and the bar/stacked comparisons.

---
