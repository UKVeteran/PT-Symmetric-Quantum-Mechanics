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
<summary><b>Click to reveal <code></b></summary>

```python
import numpy as np
import matplotlib.pyplot as plt

def calculate_eigenvalues(v, gamma_array, e0=0):
    """
    Calculates the real and imaginary parts of the eigenvalues
    for the 2x2 PT-symmetric Hamiltonian.
    """
    E_plus_real = np.zeros_like(gamma_array)
    E_plus_imag = np.zeros_like(gamma_array)
    E_minus_real = np.zeros_like(gamma_array)
    E_minus_imag = np.zeros_like(gamma_array)
    
    for i, gamma in enumerate(gamma_array):
        # Discriminant of the eigenvalues
        discriminant = v**2 - gamma**2
        
        if discriminant >= 0:
            # Unbroken PT symmetry: Real eigenvalues
            E_plus_real[i] = e0 + np.sqrt(discriminant)
            E_minus_real[i] = e0 - np.sqrt(discriminant)
        else:
            # Broken PT symmetry: Complex conjugate pairs
            E_plus_real[i] = e0
            E_minus_real[i] = e0
            E_plus_imag[i] = np.sqrt(-discriminant)
            E_minus_imag[i] = -np.sqrt(-discriminant)
            
    return E_plus_real, E_minus_real, E_plus_imag, E_minus_imag

def plot_bifurcation():
    # Parameters
    v_coupling = 1.0
    gamma = np.linspace(0, 2.5, 500)
    
    # Get eigenvalues
    Re_plus, Re_minus, Im_plus, Im_minus = calculate_eigenvalues(v_coupling, gamma)
    
    # Create the plot
    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 5))
    fig.suptitle('Eigenvalue Spectrum of a 2x2 $\mathcal{PT}$-Symmetric System', fontsize=16)
    
    # --- Real Part Plot ---
    ax1.plot(gamma, Re_plus, 'b-', linewidth=2, label='$Re(E_+)$')
    ax1.plot(gamma, Re_minus, 'r--', linewidth=2, label='$Re(E_-)$')
    ax1.axvline(x=v_coupling, color='k', linestyle=':', label='Exceptional Point ($\gamma = v$)')
    ax1.set_xlabel('Gain/Loss Parameter ($\gamma$)', fontsize=12)
    ax1.set_ylabel('Real Part of Energy', fontsize=12)
    ax1.set_title('Real Spectrum', fontsize=14)
    ax1.grid(True, alpha=0.3)
    ax1.legend()
    
    # --- Imaginary Part Plot ---
    ax2.plot(gamma, Im_plus, 'b-', linewidth=2, label='$Im(E_+)$')
    ax2.plot(gamma, Im_minus, 'r--', linewidth=2, label='$Im(E_-)$')
    ax2.axvline(x=v_coupling, color='k', linestyle=':', label='Exceptional Point ($\gamma = v$)')
    ax2.set_xlabel('Gain/Loss Parameter ($\gamma$)', fontsize=12)
    ax2.set_ylabel('Imaginary Part of Energy', fontsize=12)
    ax2.set_title('Imaginary Spectrum', fontsize=14)
    ax2.grid(True, alpha=0.3)
    ax2.legend()
    
    plt.tight_layout()
    plt.savefig('pt_symmetry_plot.png', dpi=300)
    plt.show()

if __name__ == "__main__":
    plot_bifurcation()
