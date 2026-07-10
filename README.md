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
from scipy.sparse import diags
from scipy.linalg import eigvals

def generate_pt_spectrum():
    # 1. Physics Parameters
    # N (or epsilon) range corresponds to the target plot
    N_values = np.linspace(1.0, 5.0, 100) 
    M = 400 # Grid size
    
    # 2. Contour Definition
    # PT-symmetric systems require a contour in the complex plane
    # defined by x = R * exp(-i * phi)
    phi = np.pi / 4.0
    R = np.linspace(-10, 10, M)
    x = R * np.exp(-1j * phi)
    dx = x[1] - x[0]
    
    plot_N, plot_E = [], []

    print("Calculating PT-Symmetric Spectrum...")

    # 3. Solver Loop
    for N in N_values:
        # Kinetic Energy: Second derivative matrix (Central Difference)
        main_diag = -2.0 * np.ones(M) / dx**2
        off_diag = np.ones(M-1) / dx**2
        T = diags([off_diag, main_diag, off_diag], [-1, 0, 1], format='csr')
        
        # Potential: V(x) = x^2 * (ix)^(N-2)
        # This is the standard form for the Bender-Boettcher potential
        V = (x**2) * (1j * x)**(N - 2)
        
        # Hamiltonian
        H = T.toarray() + np.diag(V)
        
        # Solve
        evals = eigvals(H)
        
        # Filter for Real Spectrum
        # In PT-Symmetric regions, eigenvalues are real. 
        # We look for eigenvalues with near-zero imaginary parts.
        real_evals = evals[np.abs(evals.imag) < 0.1].real
        
        # Sort and filter range to match plot
        real_evals = real_evals[(real_evals > 0) & (real_evals < 20)]
        real_evals = np.sort(real_evals)
        
        for e in real_evals:
            plot_N.append(N)
            plot_E.append(e)

    # 4. Plotting
    fig, ax = plt.subplots(figsize=(10, 8), facecolor='white')
    ax.scatter(plot_N, plot_E, s=2, color='black')
    
    # Stylings
    ax.axvline(1, color='gray', linestyle='--', lw=1)
    ax.axvline(2, color='gray', linestyle='--', lw=1)
    
    ax.set_xlabel('N', fontsize=18)
    ax.set_ylabel('Re[E]', fontsize=18)
    ax.set_title('PT-Symmetry Energy Spectrum', fontsize=16)
    
    plt.tight_layout()
    plt.show()

if __name__ == "__main__":
    generate_pt_spectrum()
