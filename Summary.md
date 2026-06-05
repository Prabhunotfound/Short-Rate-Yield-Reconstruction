# About This Project

## What It Does

This project builds a pipeline to reconstruct the **full US zero-coupon yield curve** (spanning 9 maturities from 3 months to 30 years) using **only the 3-month rate as input**. The core idea is that if the short rate drives long-run interest rate dynamics, a well-calibrated term structure model should be able to extrapolate the entire curve from a single observable.

The model is evaluated on the **2022–2024 hiking cycle**, one of the most aggressive rate environments in decades, making it a genuinely difficult out-of-sample test.

---

## How It Works

### Base CIR
The foundation is the **Cox-Ingersoll-Ross (1985)** model, which describes the short rate as a mean-reverting diffusion:

$$dr_t = \kappa(\theta - r_t)\,dt + \sigma\sqrt{r_t}\,dW_t$$

The square-root diffusion ensures rates stay non-negative, and the model has **closed-form bond pricing**: the entire yield curve on any day is determined analytically by three parameters and today's short rate. This makes it both theoretically grounded and computationally tractable.

### Calibration Strategy
Rather than fitting parameters to the short-rate time series alone (which collapses to $\kappa \approx 0$ over mixed-regime data), calibration minimises **cross-sectional MSE across all 9 maturities simultaneously**. This is done in three stages: an OLS warm start for a stable initialisation, Differential Evolution for global search, and L-BFGS-B for high-precision refinement.

### CIR++
The base model is systematically biased when calibrated over one rate regime and tested on another. CIR++ adds a **deterministic spread** $\phi(\tau, r_t)$ to correct this:

- **Adaptive Ridge (Layer 1):** A linear function of $r_t$ fit per maturity via ridge regression. The ridge penalty $\alpha$ is selected per maturity using exact Leave-One-Out CV via the hat matrix, with no refitting required, computationally free, and extrapolating correctly into high-rate territory unlike flat bucket-spread approaches.
- **Regime Blend (Layer 2):** Training data is split at the median short rate into LOW and HIGH regimes. A sigmoid-weighted blend applies the appropriate bias correction at test time, smoothly handling the nonlinear regime-dependence of yield curve dynamics.

### Jump-Diffusion Extension
The 2022–2024 cycle saw sudden large repricing events driven by central bank decisions, moves that a continuous Brownian diffusion structurally cannot replicate. Following Duffie, Pan & Singleton (2000), a **compound Poisson jump term** is added:

$$dr_t = \kappa(\theta - r_t)\,dt + \sigma\sqrt{r_t}\,dW_t + J\,dN_t$$

Affine term structure theory gives a closed-form jump premium for each maturity, peaking at the 2–5Y belly and dissipating at long maturities, consistent with the empirically observed bear-flattening during 2022–2023.

---

## Notable Design Choices

**Feller-aware hyperparameter tuning**
The Feller condition ($2\kappa\theta \geq \sigma^2$) guarantees the short rate never touches zero. A naive calibration over the volatile 2022–2024 data tends to produce a large $\sigma$ that violates this. The hyperparameter tuning phase explicitly tracks the Feller value $2\kappa\theta - \sigma^2$ for every tested configuration and includes a soft Feller penalty in the calibration loss. The result is a configuration that sits near the Feller boundary, **balancing theoretical compliance with predictive performance**, rather than blindly sacrificing one for the other.

**Exact LOO-CV for ridge penalty selection**
Instead of k-fold cross-validation (which wastes data and requires multiple refits), the ridge penalty for each maturity's spread is chosen via the hat matrix formula for exact LOO-CV. This is $O(n)$ per candidate $\alpha$, costs a single matrix inversion, and wastes no training observations.

**Rate-scaled jump premium**
The jump premium is scaled by $(r_t / \bar{r})^p$ where $p$ is jointly optimised. This allows the premium to grow with the rate level, reflecting that high-rate environments tend to see larger absolute shocks, rather than applying a flat constant correction regardless of where rates are.

**Positivity floor as a practical safety net**
A `1e-5` floor is enforced in preprocessing regardless of Feller compliance. This ensures the pipeline never evaluates the CIR square-root diffusion or chi-squared transition density at zero or negative values (both theoretically undefined) across any rate environment.

---

## Results

The best model (**CIR++ JD HT-Best**) achieves the strongest out-of-sample performance. The short end fits well; the hardest maturities are 1Y–2Y, dominated by forward guidance pricing during 2022–2024 that a single-factor model has no mechanism to capture. The jump-diffusion extension is correctly specified but contributes minimal empirical lift, as the CIR++ spread layers already absorb the regime bias, leaving too little residual signal across only 27 detected jump events.

The fundamental ceiling is the **single-factor structure**: all maturities move in lockstep through one latent state $r_t$. Capturing independent slope and curvature dynamics requires at least two factors.