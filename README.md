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
    $$ \mathcal{P}\hat{x}\mathcal{P} = -\hat{x}, \quad \mathcal{P}\hat{p}\mathcal{P} = -\hat{p} $$
* **Time-Reversal ($\mathcal{T}$):** Reverses momentum and applies complex conjugation.
    $$ \mathcal{T}\hat{x}\mathcal{T} = \hat{x}, \quad \mathcal{T}\hat{p}\mathcal{T} = -\hat{p}, \quad \mathcal{T}i\mathcal{T} = -i $$

The canonical example of a non-Hermitian yet PT-symmetric Hamiltonian is:
$$ \mathcal{H} = p^2 + ix^3 $$

---

## 🎲 The Two-Level Toy Model

To visualize the exact moment PT-symmetry "breaks" (the **Exceptional Point**), we use a standard $2 \times 2$ matrix representing two coupled modes: one amplifying (gain) and one dissipating (loss).

$$ \mathcal{H} = \begin{pmatrix} \varepsilon_0 + i\gamma & v \\ v & \varepsilon_0 - i\gamma \end{pmatrix} $$

Where:
* $\varepsilon_0$ is the unperturbed background energy.
* $v$ is the coupling strength.
* $\gamma$ is the balanced gain/loss parameter.

The eigenvalues define the system's phase:
$$ E_{\pm} = \varepsilon_0 \pm \sqrt{v^2 - \gamma^2} $$

* **Unbroken Phase ($\gamma < v$):** Purely real energies.
* **Exceptional Point ($\gamma = v$):** Eigenvalues and eigenvectors coalesce into a singularity.
* **Broken Phase ($\gamma > v$):** Complex conjugate energies emerge (one amplifying, one decaying).

---

## 💻 Visualization Code

Below is the Python implementation used to calculate the eigenspectrum and plot the bifurcation. 

<details>
<summary><b>Click to reveal `plot_pt_spectrum.py`</b></summary>
```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.linalg import eigvals

def generate_pt_symmetry_plot():
    print("Generating PT-Symmetry spectrum... (This may take ~10-15 seconds)")
    
    # ---------------------------------------------------------
    # 1. Physics Parameters & Grid Setup
    # ---------------------------------------------------------
    N_min, N_max = 1.05, 4.5
    N_steps = 200
    N_values = np.linspace(N_min, N_max, N_steps)
    
    L = 20.0       # Extent of the grid (needs to be wide for N -> 1)
    M = 1200       # Number of grid points
    t = np.linspace(-L, L, M)
    d = 1.0        # Smoothness parameter for the complex hyperbola
    
    plot_N = []
    plot_E = []

    # ---------------------------------------------------------
    # 2. Main Loop over N
    # ---------------------------------------------------------
    for N in N_values:
        # A. Define the dynamic complex contour x(t)
        # This hyperbola gracefully asymptotes into the correct Stokes wedges for any N
        x = t * np.sin(np.pi/N) - 1j * np.sqrt(t**2 + d**2) * np.cos(np.pi/N)
        
        # B. Compute non-uniform finite difference spacings
        h = np.diff(x)         # Spacing h_j = x_{j+1} - x_j
        hj = h[:-1]            # h_j
        hjp1 = h[1:]           # h_{j+1}
        Sj = hj + hjp1         # h_j + h_{j+1}
        
        # C. Construct Tridiagonal Kinetic Energy Matrix T = - d^2/dx^2
        diag_T = 2.0 / (hj * hjp1)
        upper_T = -2.0 / (Sj[:-1] * hjp1[:-1])
        lower_T = -2.0 / (Sj[1:] * hj[1:])
        
        T = np.diag(diag_T) + np.diag(upper_T, 1) + np.diag(lower_T, -1)
        
        # D. Construct Potential Matrix V(x) = -(ix)^N
        # We evaluate on the interior points to match matrix size
        x_int = x[1:-1]
        ix = 1j * x_int
        
        # To compute (ix)^N without branch cut discontinuities on the complex plane,
        # we calculate the magnitude and smoothly unwrap the phase.
        r = np.abs(ix)
        phi = np.unwrap(np.angle(ix))
        V = - (r**N) * np.exp(1j * N * phi)
        
        # E. Diagonalize the full Hamiltonian
        H = T + np.diag(V)
        evals = eigvals(H)
        
        # F. Filter for the Real Spectrum
        # At exceptional points, real eigenvalues merge and acquire large imaginary parts.
        for e in evals:
            if abs(e.imag) < 1e-2 and 0 < e.real <= 19:
                plot_N.append(N)
                plot_E.append(e.real)

    # ---------------------------------------------------------
    # 3. Plotting & Aesthetics (Recreating the original image)
    # ---------------------------------------------------------
    fig, ax = plt.subplots(figsize=(10, 8), facecolor='#40e0d0')
    ax.set_facecolor('#40e0d0') # Teal background
    
    # Plot the calculated eigenvalues
    ax.plot(plot_N, plot_E, 'k.', markersize=2)
    
    # Reference lines for N=1 (boundary of broken phase) and N=2 (Hermitian point)
    ax.axvline(1, color='yellow', linestyle='--', lw=2.5)
    ax.axvline(2, color='yellow', linestyle='-', lw=2.5)

    # Limits matching the provided image
    ax.set_xlim(0.8, 4.5)
    ax.set_ylim(0, 18)
    
    # Axis styling
    ax.set_xlabel('N', color='white', fontsize=26, weight='bold', loc='right')
    ax.set_ylabel('Energy-Spectrum', color='white', fontsize=26, weight='bold')
    
    # Tick styling
    ax.tick_params(axis='both', which='major', labelsize=14, direction='in', length=8)
    ax.tick_params(axis='both', which='minor', direction='in', length=4)
    ax.minorticks_on()
    
    # Overlay the exact text from your image
    ax.text(3.3, 13.5, 'PT-Symmetry\nin\nQuantum Th.', 
            color='white', fontsize=40, weight='bold', 
            ha='center', va='center', family='sans-serif')
            
    plt.tight_layout()
    plt.show()

if __name__ == "__main__":
    generate_pt_symmetry_plot()
```
<summary><b>Click to reveal `plot_pt_symmetry.py`</b></summary>

```python
import numpy as np
import matplotlib.pyplot as plt

def calculate_eigenvalues(v, gamma_array, e0=0):
    """Calculates eigenvalues for the 2x2 PT-symmetric Hamiltonian."""
    E_plus_real, E_plus_imag = np.zeros_like(gamma_array), np.zeros_like(gamma_array)
    E_minus_real, E_minus_imag = np.zeros_like(gamma_array), np.zeros_like(gamma_array)
    
    for i, gamma in enumerate(gamma_array):
        discriminant = v**2 - gamma**2
        
        if discriminant >= 0: # Unbroken Phase
            E_plus_real[i] = e0 + np.sqrt(discriminant)
            E_minus_real[i] = e0 - np.sqrt(discriminant)
        else:                 # Broken Phase
            E_plus_real[i] = e0
            E_minus_real[i] = e0
            E_plus_imag[i] = np.sqrt(-discriminant)
            E_minus_imag[i] = -np.sqrt(-discriminant)
            
    return E_plus_real, E_minus_real, E_plus_imag, E_minus_imag

def plot_bifurcation():
    # Model Parameters
    v_coupling = 1.0
    gamma = np.linspace(0, 2.5, 500)
    
    # Compute
    Re_plus, Re_minus, Im_plus, Im_minus = calculate_eigenvalues(v_coupling, gamma)
    
    # Styling & Plotting
    plt.style.use('seaborn-v0_8-darkgrid')
    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 6))
    fig.suptitle('Eigenvalue Bifurcation at the Exceptional Point', fontsize=18, fontweight='bold', y=0.95)
    
    # Real Spectrum
    ax1.plot(gamma, Re_plus, color='#2ca02c', linewidth=2.5, label='$Re(E_+)$')
    ax1.plot(gamma, Re_minus, color='#d62728', linestyle='--', linewidth=2.5, label='$Re(E_-)$')
    ax1.axvline(x=v_coupling, color='#7f7f7f', linestyle=':', linewidth=2, label='Exceptional Point')
    ax1.set_xlabel('Gain/Loss Parameter ($\gamma$)', fontsize=14)
    ax1.set_ylabel('Real Energy', fontsize=14)
    ax1.legend(fontsize=12, loc='upper right')
    
    # Imaginary Spectrum
    ax2.plot(gamma, Im_plus, color='#2ca02c', linewidth=2.5, label='$Im(E_+)$')
    ax2.plot(gamma, Im_minus, color='#d62728', linestyle='--', linewidth=2.5, label='$Im(E_-)$')
    ax2.axvline(x=v_coupling, color='#7f7f7f', linestyle=':', linewidth=2, label='Exceptional Point')
    ax2.set_xlabel('Gain/Loss Parameter ($\gamma$)', fontsize=14)
    ax2.set_ylabel('Imaginary Energy', fontsize=14)
    ax2.legend(fontsize=12, loc='upper left')
    
    plt.tight_layout(rect=[0, 0, 1, 0.93])
    plt.savefig('assets/pt_symmetry_plot.png', dpi=300, bbox_inches='tight')
    plt.show()

if __name__ == "__main__":
    plot_bifurcation()
