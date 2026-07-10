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

## 💻 Visualization Code

<details>
<summary><b>Click to reveal <code>plot_pt_spectrum.py</code> (Complex Spectrum)</b></summary>

```python
def compute_spectrum_point(N):
    # Parameters
    L, M = 20.0, 500
    t = np.linspace(-L, L, M)
    d = 1.0
    
    # 1. Complex contour
    x = t * np.sin(np.pi/N) - 1j * np.sqrt(t**2 + d**2) * np.cos(np.pi/N)
    
    # 2. Finite difference spacings
    # h has length M-1 (499)
    h = np.diff(x)
    
    # We want to solve on interior points (index 1 to M-2), 
    # so the matrix size is (M-2) x (M-2) = 498 x 498
    # h_j (distance to previous point) and h_next (distance to next point)
    hj = h[:-1]    # indices 0 to 497 (length 498)
    hjp1 = h[1:]   # indices 1 to 498 (length 498)
    sj = hj + hjp1 # length 498
    
    # 3. Construct Diagonals for (M-2) x (M-2) matrix
    main_diag = 2.0 / (hj * hjp1)
    # Off-diagonals must be length (M-2) - 1 = 497
    lower_diag = -2.0 / (sj[1:] * hj[1:])
    upper_diag = -2.0 / (sj[:-1] * hjp1[:-1])
    
    # Build sparse kinetic matrix
    T = diags([lower_diag, main_diag, upper_diag], [-1, 0, 1], shape=(M-2, M-2), format='csr')
    
    # 4. Potential Matrix V (must match interior points: indices 1 to M-2)
    x_int = x[1:-1]
    ix = 1j * x_int
    V = - (np.abs(ix)**N) * np.exp(1j * N * np.unwrap(np.angle(ix)))
    
    # 5. Solve
    H = T.toarray() + np.diag(V)
    evals = eigvals(H)
    
    # 6. Filter
    mask = (np.abs(evals.imag) < 0.1) & (evals.real > 0) & (evals.real <= 19)
    return N, evals[mask].real
