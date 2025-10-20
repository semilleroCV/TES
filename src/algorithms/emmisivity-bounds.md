---
toc: false
---

<link rel="stylesheet" href="/algorithms/algorithm.css">

<div>
<a href="/" class="alg-back" aria-label="Back to home">←</a>
</div>

# EB
**Emissivity Bounds Method**

**Authors**: Gillespie et al.  
**Date**: 1990  

---

## Concept

The **Emissivity Bounds (EB)** method estimates temperature and spectral emissivity by constraining emissivity values between two physical limits — typically between **ε_min = 0.95** and **ε_max = 1.00** for natural surfaces — and iteratively finding the temperature that minimizes the spread of derived emissivities.  

It assumes that:
- The observed radiance \( L_i \) at wavelength \( \lambda_i \) follows Planck’s law:
  \[
  L_i = \varepsilon_i B_\lambda(T)
  \]
  where \( B_\lambda(T) \) is the blackbody radiance and \( \varepsilon_i \) is the spectral emissivity.
- For the correct temperature \(T\), all computed emissivities \( \varepsilon_i \) must lie within the physical bounds.

---

## Step 1 — Compute Apparent Emissivity for a Trial Temperature

For each spectral band \(i\), given measured radiance \(L_i\), compute the *apparent emissivity* at a trial temperature \(T\):

\[
\varepsilon_i(T) = \frac{L_i}{B_\lambda(T)}.
\]

If the assumed \(T\) is too low, the emissivities will exceed 1.  
If \(T\) is too high, emissivities will drop below the physical lower bound.

---

## Step 2 — Define Physical Bounds

We set limits on possible emissivities:

\[
\varepsilon_{\min} \leq \varepsilon_i(T) \leq \varepsilon_{\max}.
\]

Typical values used for land-surface temperature estimation are:

\[
\varepsilon_{\min} = 0.95, \qquad \varepsilon_{\max} = 1.00.
\]

These represent the plausible range for natural materials in the thermal infrared region (8–14 µm).

---

## Step 3 — Identify the Temperature Consistent with Bounds

We now search for the temperature \(T\) such that **all** emissivities computed from measured radiances fall within the valid interval.

Graphically, this can be visualized as the intersection of the individual temperature ranges implied by each band:

\[
T_i^{(\min)} = B_\lambda^{-1}\!\left(\frac{L_i}{\varepsilon_{\max}}\right), \qquad
T_i^{(\max)} = B_\lambda^{-1}\!\left(\frac{L_i}{\varepsilon_{\min}}\right).
\]

Each band \(i\) defines an interval \([T_i^{(\min)}, T_i^{(\max)}]\).  
The intersection of all such intervals gives the allowable temperature region:

\[
T_{\text{EB}} \in \bigcap_i [T_i^{(\min)}, T_i^{(\max)}].
\]

If the intersection is non-empty, we take the midpoint as the estimated temperature \(T_{\text{EB}}\).

---

## Step 4 — Retrieve Emissivities at the Final Temperature

Once \(T_{\text{EB}}\) is found, emissivities are recomputed using:

\[
\varepsilon_i = \frac{L_i}{B_\lambda(T_{\text{EB}})}.
\]

This yields the final emissivity spectrum consistent with both the measured radiances and the physical emissivity limits.

---

## Step 5 — Example (Conceptual)

Suppose we have three spectral bands and the following measured radiances:

| Band | λ (µm) | \(L_i\) (W·m⁻²·sr⁻¹·µm⁻¹) |
|:----:|:-------:|:--------------------------:|
| 1 | 8.6 | 9.32 |
| 2 | 10.8 | 8.01 |
| 3 | 12.0 | 6.77 |

We assume \( \varepsilon_{\min} = 0.95 \), \( \varepsilon_{\max} = 1.00 \).  
Each band defines a temperature range consistent with those emissivity limits.  
The intersection of these intervals gives \(T_{\text{EB}} \approx 320~\text{K}\).  
Using that, we compute:

| Band | \( \varepsilon_i \) |
|:----:|:--------------------:|
| 1 | 0.987 |
| 2 | 0.974 |
| 3 | 0.962 |

---

## Step 6 — Algorithm Summary

1. Input measured radiances \(L_i\) and central wavelengths \(\lambda_i\).
2. Choose emissivity bounds \(\varepsilon_{\min}\), \(\varepsilon_{\max}\).
3. For each band \(i\), compute \(T_i^{(\min)}\) and \(T_i^{(\max)}\).
4. Find temperature range intersection \([T_{\min}^{\ast}, T_{\max}^{\ast}]\).
5. Set \(T_{\text{EB}} = (T_{\min}^{\ast} + T_{\max}^{\ast})/2.\)
6. Compute emissivities \(\varepsilon_i = L_i / B_\lambda(T_{\text{EB}})\).

---

## Step 7 — Remarks

- EB does not require an empirical regression or prior emissivity knowledge.
- It guarantees **physical realism** by bounding emissivities.
- The main limitation is its sensitivity to radiometric noise: small radiance errors can cause non-overlapping temperature intervals.
- EB often serves as an initialization or constraint in more advanced TES algorithms like NEM, β–MMD, or ADE.

---

## References

1. Gillespie, A. R., Rokugawa, S., Matsunaga, T., Cothern, J. S., Hook, S. J., & Kahle, A. B. (1998). *A temperature and emissivity separation algorithm for Advanced Spaceborne Thermal Emission and Reflection Radiometer (ASTER) images*. **IEEE Transactions on Geoscience and Remote Sensing**, 36(4), 1113–1126.  
2. Kealy, P. S., & Hook, S. J. (1993). *Separating temperature and emissivity in thermal infrared multispectral scanner data: Implications for recovering land surface temperatures*. **IEEE Transactions on Geoscience and Remote Sensing**, 31(6), 1155–1164.
