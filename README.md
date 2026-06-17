# Option Pricing - Heston Model

Numerical methods for pricing European and American options under the **Heston stochastic-volatility model**, implemented as a set of self-contained Jupyter notebooks. The project covers transform (FFT) pricing, model calibration, the implied-volatility surface, PDE finite-difference solvers (ADI), and Monte Carlo.

## The model

Under the risk-neutral measure the asset price $S_t$ and its instantaneous variance $v_t$ follow

$$
\begin{aligned}
dS_t &= (r - q)\,S_t\,dt + \sqrt{v_t}\,S_t\,dW_t^{S},\\
dv_t &= \kappa(\theta - v_t)\,dt + \sigma\sqrt{v_t}\,dW_t^{v},\qquad dW_t^{S}\,dW_t^{v} = \rho\,dt,
\end{aligned}
$$

| Symbol | Meaning |
|---|---|
| $v_0$ | initial variance |
| $\theta$ | long-run variance |
| $\kappa$ | mean-reversion speed |
| $\sigma$ | volatility of variance ("vol of vol") |
| $\rho$ | spot–variance correlation (typically negative) |
| $r,\ q$ | risk-free rate, dividend yield |

The negative correlation $\rho$ introduces a **mixed $S\!-\!v$ derivative** in the pricing PDE, which is what makes the ADI schemes (and their stability) nontrivial.

## Notebooks

| Notebook | Methods | Output |
|---|---|---|
| [`heston_model_fft_calibration_iv.ipynb`](heston_model_fft_calibration_iv.ipynb) | Carr–Madan **FFT** pricing (stable characteristic function), **calibration** (L-BFGS-B / Nelder–Mead on price RMSE), **implied-volatility surface** (Newton–Raphson / bisection BS inversion) | European call curves/surface, fitted $(v_0,\theta,\kappa,\sigma,\rho)$, IV surface |
| [`heston_model_adi.ipynb`](heston_model_adi.ipynb) | **ADI** finite differences: Hundsdorfer–Verwer (`scipy.linalg.solve_banded`) and Douglas (sparse `spsolve`); degenerate $v=0$ boundary; early-exercise projection | American put price + value surface |
| [`heston_model_monte_carlo.ipynb`](heston_model_monte_carlo.ipynb) | **Monte Carlo**: log-Euler / full-truncation Euler path simulation + **Longstaff–Schwartz** least-squares regression | American put price |

### Pricing methods at a glance

| Method | Best for | Trade-off |
|---|---|---|
| FFT (Carr–Madan) | fast European prices across many strikes | European only |
| ADI (HV / Douglas) | American / early-exercise via the PDE | grid discretization error; HV is 2nd-order, Douglas 1st-order in time |
| Monte Carlo (LSM) | high-dimensional / path-dependent payoffs, American | statistical (sampling) error |

## Requirements

- Python 3.10+
- `numpy`, `scipy`, `pandas`, `matplotlib`, `tqdm`

```bash
pip install numpy scipy pandas matplotlib tqdm
```

## Usage

Open any notebook in Jupyter / VS Code and run the cells top to bottom. Each notebook starts with a markdown overview, then defines its model/grid parameters, the pricing functions, and a runnable example with plots (price curves, option-value surfaces, and the implied-volatility surface).

## Validation

The transform, PDE and Monte Carlo prices are cross-checked against one another for consistency (e.g. the ADI/Monte Carlo American price sits above the corresponding European value from the FFT/characteristic-function pricer, reflecting the early-exercise premium).
