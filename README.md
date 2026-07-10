<div align="center">
  <h1>🌌 $\mathcal{PT}$-Symmetric Quantum Mechanics</h1>
  <p><b>A Comprehensive Guide to Non-Hermitian Physics, Parity-Time Symmetry, and Exceptional Points</b></p>
  
  <a href="https://github.com/yourusername/pt-symmetry/stargazers"><img src="https://img.shields.io/github/stars/yourusername/pt-symmetry?style=for-the-badge&color=yellow" alt="Stars"></a>
  <a href="https://github.com/yourusername/pt-symmetry/blob/main/LICENSE"><img src="https://img.shields.io/github/license/yourusername/pt-symmetry?style=for-the-badge&color=blue" alt="License"></a>
  <a href="https://www.python.org/"><img src="https://img.shields.io/badge/Python-3.8+-blue?style=for-the-badge&logo=python&logoColor=white" alt="Python"></a>
</div>

<br>

## 📖 Table of Contents
- [Introduction](#-introduction)
- [Mathematical Foundations](#-mathematical-foundations)
- [The $2 \times 2$ Toy Model](#-the-2-times-2-toy-model)
- [Python Visualization](#-python-visualization)
- [Gallery](#-gallery)
- [Usage & Installation](#-usage--installation)
- [References](#-references)

---

## 🔬 Introduction

For decades, a fundamental postulate of quantum mechanics has been that physical observables must be represented by **Hermitian operators** ($H = H^\dagger$). This guarantees real energy spectra and probability-conserving (unitary) time evolution. 

In 1998, Carl Bender and Stefan Boettcher challenged this axiom, demonstrating that **non-Hermitian Hamiltonians** can also possess purely real spectra, provided they commute with the joint $\mathcal{PT}$ (Parity-Time) operator. This repository explores the mathematical beauty of $\mathcal{PT}$-symmetry and provides classical plotting tools to visualize the phase transition between exact and broken $\mathcal{PT}$-symmetry, known as the **Exceptional Point**.

---

## 🧮 Mathematical Foundations

The $\mathcal{PT}$ operator is a product of the Parity operator $\mathcal{P}$ (spatial reflection) and the Time-reversal operator $\mathcal{T}$.

* **Parity ($\mathcal{P}$):** Reverses spatial coordinates and momentum.
    $$ \mathcal{P}\hat{x}\mathcal{P} = -\hat{x}, \quad \mathcal{P}\hat{p}\mathcal{P} = -\hat{p} $$
* **Time-Reversal ($\mathcal{T}$):** Reverses momentum and applies complex conjugation.
    $$ \mathcal{T}\hat{x}\mathcal{T} = \hat{x}, \quad \mathcal{T}\hat{p}\mathcal{T} = -\hat{p}, \quad \mathcal{T}i\mathcal{T} = -i $$

A Hamiltonian is $\mathcal{PT}$-symmetric if it commutes with $\mathcal{PT}$:
$$ [\mathcal{H}, \mathcal{PT}] = 0 $$

The most famous example is the generic non-Hermitian Hamiltonian:
$$ \mathcal{H} = p^2 + ix^3 $$

---

## 🎲 The $2 \times 2$ Toy Model

To visualize $\mathcal{PT}$-symmetry breaking, we use a classic two-level open quantum system representing two coupled optical cavities—one with gain (amplification) and one with loss (dissipation).

$$ \mathcal{H} = \begin{pmatrix} \varepsilon_0 + i\gamma & v \\ v & \varepsilon_0 - i\gamma \end{pmatrix} $$

Where:
* $\varepsilon_0$ is the unperturbed background energy.
* $v$ is the coupling strength between the two modes.
* $\gamma$ represents the balanced gain/loss rate.

Solving the characteristic equation $\det(\mathcal{H} - E I) = 0$ yields the eigenvalues:
$$ E_{\pm} = \varepsilon_0 \pm \sqrt{v^2 - \gamma^2} $$

### The Exceptional Point (EP)
* **Unbroken $\mathcal{PT}$-Symmetry ($\gamma < v$):** The term under the square root is positive. Eigenvalues are purely **real**.
* **Exceptional Point ($\gamma = v$):** The eigenvalues degenerate. The eigenvectors coalesce, forming a singularity.
* **Broken $\mathcal{PT}$-Symmetry ($\gamma > v$):** The term is negative. Eigenvalues form a **complex conjugate pair** (one amplifying mode, one decaying mode).

---

## 💻 Python Visualization

Below is the Python script used to solve the $2 \times 2$ model and plot the eigenvalue bifurcation across the Exceptional Point.

<details>
<summary><b>Click to expand the Python code (`plot_pt_symmetry.py`)</b></summary>

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
