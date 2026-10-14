> **Disclaimer — This repository builds directly on the framework introduced by Oscar Chapron, Ronan Fablet, and Yann Stéphan in _Optimal sensor placement for the reconstruction of ocean states using differentiable Gumbel-Softmax sampling operator_ (arXiv:2604.22511), with code released in `CIA-Oceanix/GumbleSoftmaxOptimalPlacement`. The Gumbel-Softmax sensor-placement framework, Optimal Interpolation setup, isotropic covariance baseline, and experimental setting originate from their work and are not claimed here as original contributions. This repository explores covariance extensions on top of that baseline.**

# Non-stationary Ocean Sensor Placement

This repository studies covariance extensions for differentiable sensor placement in Sea Surface Height (SSH) reconstruction.

In the baseline setting, we start from a true SSH state $\mathbf{u}$. A learned Gumbel-Softmax mask selects a sparse set of observations:

```math
\mathbf{y} = H_{\ell}\mathbf{u}
```

Optimal Interpolation then reconstructs the full state from these observations:

```math
\hat{\mathbf{u}}
=
K_{*o}
\left(
K_{oo}+\sigma_n^2 I
\right)^{-1}
\mathbf{y}
```

where $K_{oo}$ is the covariance matrix between observed locations and $K_{*o}$ is the covariance between all grid locations and the observations.

The reconstruction therefore depends directly on the assumed spatial covariance. The original baseline uses the isotropic covariance

```math
K_{ij}
=
C(\mathbf{r}_i,\mathbf{r}_j)
=
\sigma_f^2
\exp\left(
-\frac{
\left\|
\mathbf{r}_i-\mathbf{r}_j
\right\|_2^2
}{
2L^2
}
\right)
```

with a single learned length scale $L$.

We study two extensions of this covariance model:

- **A2 — global anisotropic covariance**
- **A3 — learned non-stationary covariance geometry**

---

## Covariance extensions

<p align="center">
  <img src="./assets/covariance_geometry.jpg" width="500">
</p>

### Isotropic baseline

The baseline uses a single spatial length scale:

```math
C(\mathbf{r}_i,\mathbf{r}_j)
=
\sigma_f^2
\exp\left(
-\frac{
\left\|
\mathbf{r}_i-\mathbf{r}_j
\right\|_2^2
}{
2L^2
}
\right)
```

Equal-covariance contours are circles: correlation decays identically in every direction and the same geometry is used everywhere.

### A2 — Global anisotropy

A2 introduces two directional length scales and a rotation:

```math
\phi_{\mathrm{A2}}(\mathbf{r})
=
\begin{bmatrix}
(\cos\theta\,x+\sin\theta\,y)/L_1 \\
(-\sin\theta\,x+\cos\theta\,y)/L_2
\end{bmatrix}
```

The covariance becomes:

```math
C(\mathbf{r}_i,\mathbf{r}_j)
=
\sigma_f^2
\exp\left(
-\frac{1}{2}
\left\|
\phi_{\mathrm{A2}}(\mathbf{r}_i)
-
\phi_{\mathrm{A2}}(\mathbf{r}_j)
\right\|_2^2
\right)
```

Equivalently, A2 replaces the isotropic Euclidean distance with a learned **Mahalanobis distance**, allowing different correlation scales along different spatial directions. A3 further generalizes this idea by making the effective distance geometry spatially non-stationary.

This replaces circular covariance contours with a single learned ellipse over the whole domain.

### A3 — Non-stationary geometry

A3 allows the covariance geometry to vary spatially by augmenting the A2 coordinates with a learned deformation:

```math
\phi_{\mathrm{A3}}(\mathbf{r})
=
\left[
\phi_{\mathrm{A2}}(\mathbf{r}),0,0
\right]
+
g_{\eta}(\mathbf{r}_{\mathrm{norm}})
```

with

```math
g_{\eta}:\mathbb{R}^2\rightarrow\mathbb{R}^4,
\qquad
2\rightarrow64\rightarrow64\rightarrow4
```

The covariance becomes:

```math
C(\mathbf{r}_i,\mathbf{r}_j)
=
\sigma_f^2
\exp\left(
-\frac{1}{2}
\left\|
\phi_{\mathrm{A3}}(\mathbf{r}_i)
-
\phi_{\mathrm{A3}}(\mathbf{r}_j)
\right\|_2^2
\right)
```

Unlike A2, the effective covariance geometry can vary with position.

The final layer of $g_{\eta}$ is initialized to zero, so A3 starts from the A2 solution before learning the non-stationary deformation.

---

## Results

Experiments use 25 SSH states sampled throughout the NATL60 year, with the same 0.1% observation-budget setting and reconstruction protocol used for the baseline comparison.

| Method | RMSE (m) ↓ | EV (%) ↑ | Sensors |
|---|---:|---:|---:|
| **A3** | **0.0696** | **95.68** | 68.3 |
| A2 | 0.1099 | 89.57 | 68.4 |
| Isotropic baseline | 0.1123 | 89.11 | 68.2 |
| Random | 0.1841 | 70.76 | 78.4 |

A3 reduces RMSE by approximately **38% relative to the isotropic baseline** under this experimental setting.

A2 provides only a small improvement over isotropic covariance, whereas the learned spatial deformation in A3 produces a substantially larger gain.

---

## Experimental setting

For each selected day:

1. Sensor locations are represented by learnable logits.
2. Hard observation masks are sampled using the straight-through Gumbel-Softmax estimator.
3. Sparse observations are reconstructed with Optimal Interpolation.
4. Sensor placement and covariance parameters are optimized jointly.
5. Final performance is evaluated using fresh hard observation masks.

The experiment is an **idealized same-state reconstruction experiment**: optimization and evaluation use the same underlying SSH state, following the baseline experimental setting.

The purpose of this experiment is to isolate the impact of covariance geometry on reconstruction and sensor placement.

---

## Data

Experiments use Sea Surface Height fields from the NATL60 Gulf Stream dataset.

The preprocessing follows the spatial domain used by the baseline experiment, with a final reconstruction grid of 280 × 280.

---

## Acknowledgment

This project would not exist without the original sensor-placement framework developed by Oscar Chapron, Ronan Fablet, and Yann Stéphan.

Please refer to their paper:

**Optimal sensor placement for the reconstruction of ocean states using differentiable Gumbel-Softmax sampling operator**  
Oscar Chapron, Ronan Fablet, Yann Stéphan  
arXiv:2604.22511

and the accompanying implementation:

`CIA-Oceanix/GumbleSoftmaxOptimalPlacement`

for the original method, baseline experiments, and implementation.
