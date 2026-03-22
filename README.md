# Monte Carlo Simulation of Neutron Transport & Shielding — PHYS20762 Computational Physics

Monte Carlo simulation of neutron transport through single and multi-slab shielding geometries. Transmission, absorption, and reflection probabilities are computed for three materials across thermal, epithermal, and fast neutron energy regimes. The project covers random number quality validation, OOP material modelling, exponential path sampling, isotropic scattering, the Woodcock delta-tracking method, and attenuation curve fitting.

---

## Physics Background

Neutron shielding is governed by the competition between absorption and scattering interactions, characterised by material-specific microscopic cross-sections. The mean free path between interactions is:

```
λ = 1 / (n · σ)
```

where n is the number density and σ the relevant cross-section. Step lengths between interactions are sampled from an exponential distribution. At each interaction, a neutron is absorbed with probability P_abs = σ_a / σ_total, or scatters isotropically otherwise.

Three representative materials are studied:
- **Water** — hydrogen-rich moderator, high scattering cross-section, effective for thermal neutrons
- **Lead** — high atomic mass, effective for fast neutron attenuation via inelastic scattering
- **Graphite** — low absorption, useful as a neutron moderator

---

## Simulation Components

### 1. Random Number Quality Validation
Before running simulations, the quality of NumPy's random number generator is assessed via:
- 1D histogram uniformity check
- 2D scatter plot of 30,000 (x, y) pairs in the unit square
- 3D scatter plot of 20,000 (x, y, z) triplets in the unit cube

A custom Linear Congruential Generator (`randssp`) is implemented to demonstrate the planar artefacts and spectral correlations that arise from low-quality PRNGs, and to justify the use of NumPy's Mersenne Twister instead.

### 2. Isotropic Sampling Validation
Two sphere-sampling methods are compared:
- **Incorrect**: uniform sampling of polar angle θ ∈ [0, π] — produces clustering at the poles
- **Correct**: uniform sampling of cos(θ) ∈ [−1, 1] and φ ∈ [0, 2π) — produces true angular isotropy

The distinction is visualised via 3D scatter plots and is critical for unbiased neutron scattering direction sampling.

### 3. Shield Class (OOP)
A `shield` class encapsulates all nuclear and material data for a shielding medium:

```python
shield(Absorption, Scattering, Density, Molar_mass, name)
```

Computed attributes include number density, microscopic cross-sections (barns → cm²), mean free paths for absorption, scattering, and total interactions, and absorption probability per interaction.

### 4. Attenuation Length Validation
Exponential path lengths sampled from the absorption MFP are histogrammed over 20 independent trials of 10,000 samples each. A linear fit to log(frequency) vs distance recovers the attenuation length λ, validating the sampling procedure against the known theoretical value.

### 5. Single-Slab Monte Carlo Simulation
For each material, 10,000 neutrons are simulated through a slab of fixed thickness (default 10 cm) over 10 repeated trials. At each step:
- Step length drawn from Exponential(λ_total)
- Absorbed with probability P_abs; otherwise scatters isotropically
- Neutron fate recorded as transmitted, absorbed, or reflected

Mean and standard deviation of each outcome are computed across trials.

### 6. Shielding vs. Slab Thickness
Single-slab simulations are run across 20 thickness values from 1 to 20 cm. Transmission, absorption, and reflection are plotted as a function of thickness. An exponential attenuation model is fitted to the transmission data to extract the effective attenuation length for each material:

```
T(L) ∝ exp(−L / λ)
```

### 7. Two-Slab Simulation — Woodcock Delta-Tracking
Neutron transport through two adjacent slabs of different materials is implemented using the **Woodcock delta-tracking method**. Rather than tracking material boundaries explicitly, a fictitious cross-section equal to the maximum total interaction rate across both slabs is used uniformly. Real interactions are accepted with probability σ_real / σ_max; otherwise the step is treated as a virtual (null) interaction.

Three material pairs are simulated at 5 cm per slab:
- Water + Lead
- Lead + Graphite
- Graphite + Water

### 8. Energy-Dependent Shielding
Each material is instantiated separately for three neutron energy regimes using appropriate cross-section values:

| Energy Regime | Energy Range |
|---|---|
| Thermal | ~0.025 eV |
| Epithermal | 0.1 eV – 100 keV |
| Fast | > 0.1 MeV |

Single-slab simulations are run per material per energy type, and results are displayed as pie charts showing transmission, absorption, and reflection fractions.

### 9. Mixed Energy Spectrum
A population of 9,000 neutrons split equally across the three energy types is simulated through each material, approximating a realistic mixed neutron field.

---

## Key Results

- Water was the most effective shielding material for thermal neutrons due to its high scattering cross-section and strong moderation
- Lead performed best against fast neutrons
- The Woodcock method successfully handled heterogeneous two-slab geometries without explicit boundary tracking
- Attenuation lengths recovered from Monte Carlo transmission data were in good agreement with values computed from the material cross-sections

---

## Requirements

```
numpy
matplotlib
scipy
tqdm
```

Install with:
```bash
pip install numpy matplotlib scipy tqdm
```

---

## Usage

Open `11306505.ipynb` in Jupyter and run cells sequentially. No external data files are required — all material cross-section data is defined within the notebook.
