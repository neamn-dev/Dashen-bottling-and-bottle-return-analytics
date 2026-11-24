# DS1 B2 – Dashen Bottling and Bottle-Return Analytics
**Course:** Math3051 – Numerical Analysis | **Group:** DS-1 | **Academic Term:** 2026

---

## 1. Project Overview & Repository Structure

This repository contains the complete numerical modeling, statistical process control, and algorithmic analysis for **Project B2: Dashen Bottling and Bottle-Return Analytics**. 

The investigation couples an automated high-speed packaging line producing fill-volume data with a closed-loop circular bottle return network moving reusable glass containers through collection, warehousing, washing/inspection, and packaging.

```text
numerics/
├── notebooks/
│   └── DS1_B2_Project.ipynb          # Fully executed, verified Jupyter Notebook (64 cells)
│
├── outputs/
│   ├── figures/                      # High-resolution (300 DPI) publication plots
│   │   ├── summary_dashboard.png     # Full 3-panel analytics dashboard
│   │   ├── fig1_acquisition_order.png
│   │   ├── fig2_bootstrap_distribution.png
│   │   ├── fig3_monte_carlo_resolution.png
│   │   ├── fig4_throughput_curve_dual_roots.png
│   │   ├── fig5_chapter2_convergence_semilog.png
│   │   ├── fig6_parameter_sensitivity_sweep.png
│   │   └── fig7_chapter3_jacobi_convergence.png
│   │
│   └── tables/                       # Tidy CSV outputs from numerical solvers & simulations
│       ├── results.csv               # 16 primary project metrics
│       ├── chapter1_summary_report.csv
│       ├── chapter1_uncertainty_summary.csv
│       ├── all_iteration_history.csv # Unified tidy history across all 4 solvers
│       ├── chapter2_iteration_history.csv
│       ├── chapter2_sensitivity.csv  # 21-point parameter sweep
│       ├── chapter3_jacobi_history.csv
│       ├── chapter3_sensitivity.csv  # +5% RHS perturbation analysis
│       └── decision_summary.csv      # Executive answers to decision questions
│
├── report/
│   └── report.pdf                    # Publication-quality technical report
│
├── presentation/
│   └── slides.pdf                    # Landscape presentation slide deck (11 slides)
│
└── README.md                         # Project documentation and reproduction guide
```

---

## 2. Industrial Scenario & System Pipeline

```text
+---------------+      +------------------+      +-------------------+      +---------------+      +----------+
| Retail Return | ---> | Return Warehouse | ---> |  Wash & Inspect   | ---> | Bottling Line | ---> |  Market  |
+---------------+      +------------------+      +-------------------+      +---------------+      +----------+
        ^                                                                                               |
        +---------------------------- Reusable Bottles Return -----------------------------------------+
```

1. **Fill-Volume Quality:** Evaluating fill-level variations against the nominal $330.0\text{ mL}$ specification, isolating measurement resolution from process variability, and diagnosing a suspicious reading ($323.8\text{ mL}$).
2. **Conveyor Operating Point:** Finding the low-speed conveyor setting where useful packaging throughput $P(v) = 48 v e^{-0.035 v}$ achieves target demand ($450\text{ bottles/min}$) using bracket-based, secant, and fixed-point solvers.
3. **Circular Bottle-Return Flow:** Solving steady-state container flows across four network nodes using strictly diagonally dominant Jacobi iteration, identifying the system bottleneck, and evaluating network sensitivity.

---

## 3. Chapter-by-Chapter Technical Summary

### Chapter 1: Fill-Volume Quality & Uncertainty Propagation
* **Data (10 observations):** `[329.6, 330.4, 329.8, 330.1, 329.9, 330.3, 329.7, 330.0, 323.8, 330.2] mL`
* **Baseline Metrics:** Nominal $= 330.0\text{ mL}$, Device Resolution $= 0.1\text{ mL}$ (rounding bound $\pm 0.05\text{ mL}$).
* **All 10 Observations:** Mean $= 329.38\text{ mL}$, Median $= 329.95\text{ mL}$, Sample $s = 1.9775\text{ mL}$.
* **Excluding 323.8 mL:** Mean $= 330.00\text{ mL}$ (exact target!), Median $= 330.00\text{ mL}$, Sample $s = 0.2739\text{ mL}$.
* **The 323.8 mL Anomaly:** The deviation $|323.8 - 330.0| = 6.2\text{ mL}$ constitutes a **$22.63\sigma \approx 23\sigma$ departure** from the clean process variation ($p < 10^{-100}$). It represents an assignable special cause that destroys process capability ($C_{pk} < 1$).
* **Evidence Required to Distinguish Sensor Fault vs. Under-fill:**
  1. *Check-Weigher Mass Log:* Confirming whether physical liquid mass was missing ($\sim 6.2\text{ g}$ low) or nominal ($330\text{ g}$).
  2. *PLC Valve Telemetry:* Dosing duration and manifold pressure logs for filling head #9.
  3. *Optical Meniscus Camera Diagnostics:* Checking for foam detection, tilted neck, or splash distortion.
  4. *Consecutive Container History:* Checking whether following bottles from head #9 showed identical defects.
* **Uncertainty Evaluation:**
  - *Device Resolution Uncertainty (Uniform Rectangular Model):* $u = 0.05/\sqrt{3} \approx 0.02887\text{ mL}$. Mean uncertainty: $u_{\bar{V}} = 0.00913\text{ mL}$.
  - *Monte Carlo Propagation (10,000 draws):* Empirical standard uncertainty $= 0.00916\text{ mL}$ (exact agreement with first-order analytical).
  - *Bootstrap Sampling Uncertainty (10,000 resamples):* Standard error $\text{SE}_{\bar{V}} = 0.6003\text{ mL}$ ($95\%\text{ CI: } [328.18, 330.13]\text{ mL}$).
  - *Significant Digits Report:* Full sample: $329.4 \pm 0.6\text{ mL}$; Clean sample: $330.00 \pm 0.09\text{ mL}$.

---

### Chapter 2: Low-Speed Conveyor Operating Point
* **Model Equation:** $P(v) = 48 v e^{-0.035 v} = 450 \implies f(v) = 48 v e^{-0.035 v} - 450 = 0$.
* **Dual Roots & Peak:**
  - Low-Speed Operating Root: $v_1^* \approx 16.9930$ (Operating setpoint)
  - High-Speed Root: $v_2^* \approx 44.4954$ (Unstable operating point)
  - Peak Throughput: $v_{\max} = 1/0.035 \approx 28.57$, $P_{\max} \approx 504.52\text{ bottles/min}$.
* **Numerical Solvers Comparison ($|f(v_k)| < 10^{-6}$):**
  - *False Position on $[15, 20]$:* 8 iterations, $v^* = 16.9930033$, residual $= 8.97 \times 10^{-7}$, observed order $p = 1.000$.
  - *Secant from $v_0 = 15, v_1 = 20$:* 6 iterations, $v^* = 16.9930032$, residual $= 1.07 \times 10^{-8}$, observed order $p = 1.405$ (theoretical: 1.618).
  - *Fixed-Point Iteration ($v_0 = 17.5, \lambda = 0.08$):* 9 iterations, $v^* = 16.9930033$, residual $= 1.52 \times 10^{-7}$, observed order $p = 1.000$.
* **Fixed-Point Stability Proof:**
  - $g(v) = v - \lambda f(v) \implies g'(v) = 1 - \lambda f'(v)$.
  - $f'(v^*) = 48 e^{-0.035 v^*}(1 - 0.035 v^*) = +10.7315 > 0$.
  - Convergence condition: $|1 - \lambda(10.7315)| < 1 \iff 0 < \lambda < \frac{2}{10.7315} \approx 0.186367$.
  - Given $\lambda = 0.08 \implies |g'(v^*)| = 0.1415 \ll 1$ (contracts error by $\approx 86\%$ per iteration).
  - Numerically demonstrated that $\lambda = 0.1957$ ($|g'| = 1.10 > 1$) diverges locally away from the root.
* **Why Low-Speed Root Matters (5 Engineering Reasons):**
  1. *Energy consumption:* Motor power scales superlinearly ($P \propto v^2$ to $v^3$); running at $v \approx 44.5$ wastes massive power.
  2. *Mechanical wear:* Excessive belt friction, roller bearing fatigue, and drive heat at high speeds.
  3. *Bottle stability:* Glass bottles rattle and topple at high transfer velocities.
  4. *Yield vs rejection:* The term $e^{-0.035v}$ models defect/jam rates; at $v=44.5$, gross throughput is huge but catastrophic rejection rates discard most bottles to net 450.
  5. *Controller stability:* Near $v_1^*$, $P'(v) > 0$ (open-loop stable). Near $v_2^*$, $P'(v) < 0$ (unstable: speeding up reduces throughput, destabilizing PID loops).
* **Sensitivity Sweep ($\pm 10\%$ in 21 steps):**
  - Solved across $c \in [43.2, 52.8]$ with Secant solver: **21/21 points converged** with residuals $< 10^{-6}$.
  - $v^*$ shifts smoothly from $24.91$ down to $13.83$.

---

### Chapter 3: Circular Bottle-Return Network Analytics
* **Linear Flow Balance System:**
  $$\begin{aligned}
  -x_2 + 10x_3 - 2x_4 &= 58 \\
  10x_1 - x_2 - x_4 &= 72 \\
  -x_1 - 2x_3 + 10x_4 &= 69 \\
  -2x_1 + 10x_2 - x_3 &= 65
  \end{aligned}$$
* **Diagonally Dominant Rearrangement:**
  - Row Permutation: $P = [2, 4, 1, 3]$ (1-based original equation order).
  - Dominance: Row 1 ($10 > 2$), Row 2 ($10 > 3$), Row 3 ($10 > 3$), Row 4 ($10 > 3$).
  - Jacobi spectral radius: $\rho(T_J) = 0.2758 \ll 1$, guaranteeing fast linear contraction.
* **Why Updates Use Only Previous Iterate:**
  - Matrix splitting $A = D - L - U \implies D \mathbf{x}^{(k+1)} = (L + U)\mathbf{x}^{(k)} + \mathbf{b}$.
  - Component updates are completely decoupled within sweep $k+1$, enabling simultaneous parallel execution across multi-core CPUs and SIMD registers without memory contention.
* **Convergence & Solution ($x^{(0)} = \mathbf{0}, \text{tol} = 10^{-5}$):**
  - Converged in 12 iterations (step $\|\mathbf{x}^{(12)} - \mathbf{x}^{(11)}\|_\infty = 5.19 \times 10^{-6}$).
  - Normalized flows: $x_1 = 9.070845, x_2 = 9.176571, x_3 = 8.624034, x_4 = 9.531890$.
  - Original system residual: $\|A\mathbf{x} - \mathbf{b}\|_\infty = 1.4899 \times 10^{-5}$.
  - Difference from `numpy.linalg.solve`: $2.0399 \times 10^{-6}$.
* **Bottleneck Identification:**
  - **Bottling ($x_4 = 9.5319$)** carries the highest steady load, followed by Return Warehouse ($x_2 = 9.1766$). Bottling is the primary system bottleneck.
* **Sensitivity Analysis (+5% on $b_2 = 72 \to 75.6$):**
  - Shift governed by $\Delta \mathbf{x} = A^{-1} \Delta \mathbf{b}$.
  - Column 2 of $A^{-1}$: $[0.1032, 0.0211, 0.0043, 0.0112]^T$.
  - Equation 2 directly isolates $x_1$ with diagonal weight 10: $\Delta x_1 \approx 0.10 \times 3.60 = 0.360$.
  - **Relative Shift:** $x_1$ Retail Collection shifts by **$+4.097\%$**, which is $5\times$ higher than any other unknown.

---

## 4. Executive Decision Questions & Synthesis

| # | Decision Question | Analytical Finding | Operational Directive |
|---|---|---|---|
| **Q1** | **Would 323.8 mL change your process conclusion?** | **YES.** It inflates standard deviation by $7\times$ ($0.27 \to 1.98\text{ mL}$) and collapses $C_{pk} < 1$. It is a $22.6\sigma$ assignable special-cause event. | **Do not delete silently.** Quarantine the observation and inspect check-weigher mass logs to prevent unnecessary line stoppage. |
| **Q2** | **Most attractive root-finder for repeated controller?** | **FIXED-POINT ITERATION** ($g(v) = v - 0.08f(v)$). Operates as a discrete integral controller with guaranteed contraction ($|g'| = 0.1415 \ll 1$). | Zero bracketing logic, zero risk of division by zero, 1 mult/sub per step. Warm-starts in 1–2 iterations during active operation. |
| **Q3** | **Stage with highest modelled load?** | **BOTTLING LINE ($x_4 = 9.5319$).** Carries maximum steady flow in the network. | Primary system bottleneck. Capacity expansions and buffer sizing must prioritize Bottling. |

---

## 5. Reproduction & Verification Instructions

### Environment Requirements
- Python 3.10+ (tested on Python 3.12)
- Required packages: `numpy`, `pandas`, `matplotlib`, `scipy`, `reportlab`, `nbformat`, `nbconvert`

```bash
pip install numpy pandas matplotlib scipy reportlab nbformat nbconvert
```

### Reproducing All Outputs
1. **Run the Complete Notebook:**
   Open `notebooks/DS1_B2_Project.ipynb` in VS Code or JupyterLab and select **Kernel > Restart and Run All Cells**.
   All 64 cells will execute cleanly in under 15 seconds.

2. **Verify Output Exports:**
   Running the notebook automatically generates and refreshes:
   - `outputs/tables/results.csv`
   - `outputs/figures/summary_dashboard.png`
   - All supporting iteration histories and sensitivity sweeps.

3. **Re-generating PDF Documents:**
   To recompile the technical report and presentation slides from source:
   ```bash
   python scratch/generate_report_pdf.py
   python scratch/generate_slides_pdf.py
   ```

---

## 6. Project Compliance Sign-Off

- [x] **Restart & Run All executed without errors:** 0 errors across 64 notebook cells.
- [x] **All three Chapter 2 methods meet $|f(x_k)| < 10^{-6}$:** False Position ($8.97 \times 10^{-7}$), Secant ($1.07 \times 10^{-8}$), Fixed Point ($1.52 \times 10^{-7}$).
- [x] **Chapter 2 sensitivity sweep:** 21/21 points converged unconditionally.
- [x] **Chapter 3 Jacobi convergence:** $\|A\mathbf{x} - \mathbf{b}\|_\infty = 1.49 \times 10^{-5}$ on original unpermuted system.
- [x] **All figures labelled with axes, units, and titles:** 7 publication figures + dashboard.
- [x] **Results export and dashboard pipeline:** `results.csv` drives the summary dashboard.
- [x] **Decision questions thoroughly answered:** Detailed engineering synthesis provided.
- [x] **Submission directory structure formatted:** Exact match to requested tree layout.
