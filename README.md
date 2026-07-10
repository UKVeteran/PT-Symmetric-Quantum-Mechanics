<div align="center">

<h1>🌌 PT-Symmetric Quantum Mechanics</h1>
<h3><b>Visualizing Non-Hermitian Physics & The Exceptional Point</b></h3>

<p>
  <a href="https://www.python.org/"><img src="https://img.shields.io/badge/Python-3.8+-blue?style=for-the-badge&logo=python&logoColor=white" alt="Python"></a>
</p>

<p><i>A mathematical and computational guide to systems with balanced gain and loss.</i></p>

</div>

<br>

> **The Elevator Pitch**
> In 1998, physics underwent a paradigm shift. Carl Bender and Stefan Boettcher proved that physical observables do not strictly require a Hermitian operator. Systems with mathematically balanced gain and loss—commuting with the joint Parity-Time ($\mathcal{PT}$) operator—can possess entirely real energy spectra. This repository explores that phenomenon computationally.

---

## 📖 Table of Contents
- [A Tale of Two Paradigms](#-a-tale-of-two-paradigms)
- [Mathematical Foundations](#-mathematical-foundations)
- [The Two-Level Toy Model](#-the-two-level-toy-model)
- [Visualization Code](#-visualization-code)
- [Gallery of Phase Transitions](#-gallery-of-phase-transitions)
- [Quick Start](#-quick-start)

---

## ⚖️ A Tale of Two Paradigms

Understanding PT-Symmetry is easiest when contrasted with the standard quantum mechanics taught in textbooks.

| Feature | Standard (Hermitian) QM | PT-Symmetric (Non-Hermitian) QM |
| :--- | :--- | :--- |
| **Hamiltonian Postulate** | $H = H^\dagger$ | $[\mathcal{H}, \mathcal{PT}] = 0$ |
| **Energy Spectrum** | Strictly Real | Real (Unbroken phase) or Complex (Broken phase) |
| **Time Evolution** | Unary (Probability is conserved) | Non-Unitary (Gain/Loss of probability) |
| **Key Phenomenon** | Standard Energy Levels | The Exceptional Point (Phase Transition) |
| **Physical Analogs** | Closed, isolated systems | Open systems, coupled optical wave-guides |

---

## 🧮 Mathematical Foundations

The core of this theory relies on the Parity ($\mathcal{P}$) and Time-Reversal ($\mathcal{T}$) operators.

* **Parity ($\mathcal{P}$):** Reverses spatial coordinates and momentum.
  $$\mathcal{P}\hat{x}\mathcal{P} = -\hat{x}, \quad \mathcal{P}\hat{p}\mathcal{P} = -\hat{p}$$
* **Time-Reversal ($\mathcal{T}$):** Reverses momentum and applies complex conjugation.
  $$\mathcal{T}\hat{x}\mathcal{T} = \hat{x}, \quad \mathcal{T}\hat{p}\mathcal{T} = -\hat{p}, \quad \mathcal{T}i\mathcal{T} = -i$$

The canonical example of a non-Hermitian yet PT-symmetric Hamiltonian is:
$$\mathcal{H} = p^2 + ix^3$$

---

## 🎲 The Two-Level Toy Model

To visualize the exact moment PT-symmetry "breaks" (the **Exceptional Point**), we use a standard $2 \times 2$ matrix representing two coupled modes: one amplifying (gain) and one dissipating (loss).

$$\mathcal{H} = \begin{pmatrix} \varepsilon_0 + i\gamma & v \\ v & \varepsilon_0 - i\gamma \end{pmatrix}$$

Where:
* $\varepsilon_0$ is the unperturbed background energy.
* $v$ is the coupling strength.
* $\gamma$ is the balanced gain/loss parameter.

The eigenvalues define the system's phase:
$$E_{\pm} = \varepsilon_0 \pm \sqrt{v^2 - \gamma^2}$$

* **Unbroken Phase ($\gamma < v$):** Purely real energies.
* **Exceptional Point ($\gamma = v$):** Eigenvalues and eigenvectors coalesce into a singularity.
* **Broken Phase ($\gamma > v$):** Complex conjugate energies emerge (one amplifying, one decaying).

---


## 💻 Visualization Code

<details>
<summary><b>Click to reveal <code>plot_pt_spectrum.py</code> (Complex Spectrum)</b></summary>

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.linalg import eigvals
from scipy.sparse import diags

def generate_spectrum_optimized():
    # 1. Setup Parameters
    # 120 steps is plenty for a smooth, high-quality resolution
    N_values = np.linspace(1.05, 4.5, 120)
    L, M = 20.0, 500
    t = np.linspace(-L, L, M)
    d = 1.0
    
    plot_N, plot_E = [], []

    print("Generating PT-Symmetry spectrum... (This will run in a stable, single-threaded process)")

    # 2. Main Loop
    for N in N_values:
        # Define complex contour
        x = t * np.sin(np.pi/N) - 1j * np.sqrt(t**2 + d**2) * np.cos(np.pi/N)
        
        # Finite difference spacing: h_i = x_{i+1} - x_i
        h = np.diff(x)  # Length M-1 (499)
        
        # We solve on interior points (1 to M-2), so our matrix is (M-2)x(M-2)
        # Using M=500, matrix is 498x498
        # Formula for second derivative on non-uniform grid:
        # T_ii = 2 / (h_{i-1} * h_i)
        # T_i,i-1 = -2 / (h_{i-1} * (h_{i-1} + h_i))
        # T_i,i+1 = -2 / (h_i * (h_{i-1} + h_i))
        
        h_prev = h[:-1]  # h_{i-1}
        h_curr = h[1:]   # h_i
        
        main_diag = 2.0 / (h_prev * h_curr)
        lower_diag = -2.0 / (h_prev * (h_prev + h_curr))
        upper_diag = -2.0 / (h_curr * (h_prev + h_curr))
        
        # The main_diag has length M-2 (498).
        # The off-diagonals must have length M-3 (497) to fit perfectly.
        # We slice them to match.
        T = diags([lower_diag[:-1], main_diag, upper_diag[1:]], [-1, 0, 1], 
                  shape=(len(main_diag), len(main_diag)), format='csr')
        
        # Potential Matrix V
        ix = 1j * x[1:-1]
        V = - (np.abs(ix)**N) * np.exp(1j * N * np.unwrap(np.angle(ix)))
        
        # Solve for Eigenvalues
        # Convert to dense only for the solver (required by scipy.linalg.eigvals)
        H = T.toarray() + np.diag(V)
        evals = eigvals(H)
        
        # Filter for Real Spectrum
        mask = (np.abs(evals.imag) < 0.1) & (evals.real > 0) & (evals.real <= 19)
        plot_N.extend([N] * np.sum(mask))
        plot_E.extend(evals[mask].real)

    # 3. Plotting
    fig, ax = plt.subplots(figsize=(10, 8), facecolor='#40e0d0')
    ax.set_facecolor('#40e0d0')
    
    # Efficient scatter plotting
    ax.scatter(plot_N, plot_E, s=1, color='black', marker='.')
    
    # Plot formatting
    ax.axvline(1, color='yellow', linestyle='--', lw=2.5)
    ax.axvline(2, color='yellow', linestyle='-', lw=2.5)
    
    ax.set_xlim(0.8, 4.5)
    ax.set_ylim(0, 18)
    
    ax.set_xlabel('N', color='white', fontsize=26, weight='bold', loc='right')
    ax.set_ylabel('Energy-Spectrum', color='white', fontsize=26, weight='bold')
    
    plt.tight_layout()
    plt.show()

if __name__ == "__main__":
    generate_spectrum_optimized()
