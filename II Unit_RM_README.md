# II Unit_RM — Image Enhancement (Histograms, Spatial Filtering, Edge & Blob Detection)

Summary of `II Unit_RM.pdf` (138 pages; Digital Image Processing — Image Enhancement by R. Malmathanraj, NIT Tiruchirappalli, with LoG/DoG slides adapted from Robert Collins / O. Camps, PSU).

---

## 1. Image Histograms & Intensity Transformations

- **Histogram & Normalized Histogram (PMF)**: For an $M \times N$ image with intensity levels $r_k \in [0, L-1]$,
  $$h(r_k) = n_k, \qquad p(r_k) = \frac{n_k}{MN}$$
  where $n_k$ is the number of pixels with gray level $r_k$.
- **Visual Characteristics**:
  - **Dark image**: histogram concentrated toward low intensity values.
  - **Bright image**: histogram concentrated toward high intensity values.
  - **Low-contrast image**: narrow histogram span in the gray-level range.
  - **High-contrast image**: broad histogram spread across the dynamic range.
- **Limitation**: Histograms capture global gray-level statistics (mean brightness and contrast), **not** spatial structure or object content (different images, such as a sorted-pixel version, can have identical histograms).
- **Intensity Transformation $s = T(r)$** ($0 \le r \le L-1$):
  1. $T(r)$ is **monotonically increasing** on $[0, L-1]$ (preserves intensity ordering; **strictly** monotonically increasing guarantees a one-to-one inverse $r = T^{-1}(s)$).
  2. $0 \le T(r) \le L-1$ for $0 \le r \le L-1$ (preserves dynamic range).
- **Continuous PDF Transformation Law**:
  $$p_r(r)\,dr = p_s(s)\,ds \implies p_s(s) = \frac{p_r(r)}{\left|\frac{ds}{dr}\right|}$$

---

## 2. Histogram Equalization (HE)

- **Continuous Formulation (CDF Mapping)**:
  $$s = T(r) = (L-1)\int_0^r p_r(w)\,dw$$
  Since $\frac{ds}{dr} = (L-1)p_r(r)$, the output PDF is uniform:
  $$p_s(s) = \frac{1}{L-1}, \quad 0 \le s \le L-1$$
  - *Examples*: $p(r) = \frac{\pi}{2}\cos(\frac{\pi}{2}r) \Rightarrow T(r) = \sin(\frac{\pi}{2}r)$ gives $p_s(s)=1$; $p(r) = 2(1-r) \Rightarrow T(r) = 2r - r^2$.
- **Discrete Formulation**:
  $$s_k = T(r_k) = (L-1)\sum_{j=0}^k p_r(r_j) = \frac{L-1}{MN}\sum_{j=0}^k n_j$$
  Each $s_k$ is rounded to the nearest integer gray level in $[0, L-1]$, with the final cumulative sum reaching $L-1$.
  - Because of discretization/rounding and merging of bins, a discrete equalized histogram is rarely perfectly flat, though it spans a wider dynamic range.
- **Worked 3-Bit Example ($64 \times 64 = 4096$ pixels, $L=8$)**:
  - Input counts $n_k = [790, 1023, 850, 656, 329, 245, 122, 81]$
  - Unrounded $s_k = [1.33, 3.08, 4.55, 5.67, 6.23, 6.65, 6.86, 7]$ $\to$ Rounded $[1, 3, 5, 6, 6, 7, 7, 7]$
  - Output PMF $p_s$ occupies levels $\{1: 0.19,\; 3: 0.25,\; 5: 0.21,\; 6: 0.24,\; 7: 0.1\}$.
- **Limitations**: Global HE is not always desirable—on narrow histograms (e.g., coins or Phobos against a dark background), over-stretching can cause a washed-out appearance, false contours/regions, graininess, and patchiness.

---

## 3. Histogram Specification (Matching) & Local HE

- **Histogram Specification (Matching)**: Maps input $r$ (PDF $p_r$) to output $z$ with a prescribed PDF $p_z$:
  1. Equalize the input: $s = T(r) = (L-1)\int_0^r p_r(w)\,dw$.
  2. Compute the equalization transform of the target PDF: $G(z) = (L-1)\int_0^z p_z(w)\,dw$.
  3. Map via the inverse: $z = G^{-1}(s) = G^{-1}(T(r))$.
  - **Discrete Matching**: Compute rounded $s_k = T(r_k)$ and $G(z_q)$; map each $s_k$ to the $z_q$ whose $G(z_q)$ is closest to $s_k$. In the 3-bit example with target $p_z = [0, 0, 0, 0.15, 0.20, 0.30, 0.20, 0.15]$ ($G(z_q) = [0, 0, 0, 1, 2, 5, 6, 7]$), $s_k = [1, 3, 5, 6, 6, 7, 7, 7]$ maps to $[3, 4, 5, 6, 6, 7, 7, 7]$. Useful when global HE washes out dark backgrounds (e.g., Phobos).
- **Local Histogram Equalization**:
  - Applies histogram transformation within a sliding or tiled neighborhood around each pixel (supports spatially adaptive enhancement, adaptive thresholding, detection, and tracking).
  - Neighborhood size ($15\times 15$, $30\times 30$, $75\times 75$, $150\times 150$) trades local detail enhancement against global consistency.
  - **Block-wise drawback**: Non-overlapping tiles produce blocky artifacts and inconsistent contrast across tile boundaries unless post-processed.

---

## 4. Spatial Filters for Image Enhancement

- **Image Averaging (Temporal / Multi-Realization Denoising)**:
  For $M$ noisy observations $g_i(x,y) = f(x,y) + n_i(x,y)$ with uncorrelated zero-mean white noise of variance $\sigma_n^2$:
  $$\bar{g}(x,y) = \frac{1}{M}\sum_{i=0}^{M-1} g_i(x,y), \qquad E\{\bar{g}(x,y)\} = f(x,y), \qquad \sigma_{\bar{g}}^2 = \frac{1}{M}\sigma_n^2$$
- **Low-Pass / Local Averaging Masks**:
  - Box filters ($3\times 3$ with $1/9$, $5\times 5$ with $1/25$, $7\times 7$ with $1/49$) and center-weighted ($5\times 5$ Gaussian-like) averaging masks.
  - The frequency response of a rectangular averaging filter is a **sinc** function.
  - Reduces Gaussian noise, at the expense of blurring edges and fine details.
- **Median Filtering**:
  - Non-linear order-statistic filter replacing the center pixel with the neighborhood median.
  - Excels at removing **impulse (salt-and-pepper) noise** while preserving **step edges** far better than linear averaging (though thin lines/ridges narrower than half the window width can be attenuated).
- **High-Pass Spatial Filter**:
  - Zero-sum mask, e.g.:
    $$\frac{1}{9}\begin{bmatrix}-1 & -1 & -1 \\ -1 & 8 & -1 \\ -1 & -1 & -1\end{bmatrix}$$
  - Produces zero output in constant regions and highlights edges, but removes background contrast and amplifies noise.
- **High-Boost Filtering**:
  $$\text{High-boost} = A\cdot\text{Original} - \text{Low-pass} = (A - 1)\cdot\text{Original} + \text{High-pass}, \qquad A > 1$$
  - Reduces to standard high-pass when $A = 1$; typical values $A \approx 1.15, 1.2$.
  - For a $3\times 3$ mask, the center weight becomes $9A - 1$ (with $-1$ neighbors, scaled by $1/9$). Preserves low-frequency background while sharpening edges (commonly used in printing/publishing).

---

## 5. Edge Detection & Spatial Derivatives

- **Edge Types & Physical Causes**: Step edges, roof edges, and line (ridge) edges caused by discontinuities in surface orientation, depth, surface color/reflectance, or illumination.
- **First-Order Gradient**:
  $$\nabla f = \begin{bmatrix}\frac{\partial f}{\partial x} \\[4pt] \frac{\partial f}{\partial y}\end{bmatrix}, \qquad \|\nabla f\| = \sqrt{\left(\frac{\partial f}{\partial x}\right)^2 + \left(\frac{\partial f}{\partial y}\right)^2} \approx \left|\frac{\partial f}{\partial x}\right| + \left|\frac{\partial f}{\partial y}\right|, \qquad \theta = \tan^{-1}\!\left(\frac{\partial f / \partial y}{\partial f / \partial x}\right)$$
  - **Roberts ($2\times 2$)**: Axis/diagonal cross-differences $|r_5 - r_6| + |r_5 - r_8|$ and $|r_6 - r_8| + |r_5 - r_9|$.
  - **Prewitt ($3\times 3$)**: $\begin{bmatrix}-1&0&1\\-1&0&1\\-1&0&1\end{bmatrix}$ and $\begin{bmatrix}-1&-1&-1\\0&0&0\\1&1&1\end{bmatrix}$.
  - **Sobel ($3\times 3$)**: Center-weighted by 2 for smoothing perpendicular to the derivative: $\begin{bmatrix}-1&0&1\\-2&0&2\\-1&0&1\end{bmatrix}$ and $\begin{bmatrix}-1&-2&-1\\0&0&0\\1&2&1\end{bmatrix}$.
- **Second-Order Derivative & Laplacian**:
  - 1D central difference via Taylor series: $[1, -2, 1]$. Peaks of the 1st derivative correspond to **zero-crossings** of the 2nd derivative (sharper localization than gradient peaks, though requires checking 1st-derivative magnitude to suppress flat-region false zero-crossings).
  - 2D Laplacian:
    $$\nabla^2 f = \frac{\partial^2 f}{\partial x^2} + \frac{\partial^2 f}{\partial y^2}, \qquad \text{4-neighbor mask: } \begin{bmatrix}0 & 1 & 0 \\ 1 & -4 & 1 \\ 0 & 1 & 0\end{bmatrix}$$
  - Scalar (isotropic, loses edge direction) and highly sensitive to noise $\implies$ smooth before differentiating.

---

## 6. LoG, DoG, Blob Detection & Laplacian Pyramid

- **Laplacian of Gaussian (LoG)**:
  By associativity of linear filtering, $\nabla^2(f * G) = (\nabla^2 G) * f$:
  $$g''(x) = \left(\frac{x^2}{\sigma^4} - \frac{1}{\sigma^2}\right)e^{-x^2/(2\sigma^2)}$$
  In 2D, LoG has an inverted "Mexican Hat" profile and acts as a **band-pass filter**. Raw LoG zero-crossings form **closed contours**.
- **Blob Detection (Lindeberg)**:
  - Filtering acts like template matching with the filter kernel; extrema of the LoG response across space and scale $\sigma$ detect blobs (local maxima correspond to dark blobs on a light background; local minima correspond to light blobs on a dark background, with scale $\sigma$ tied to blob radius).
- **Difference of Gaussians (DoG)**:
  - Approximates LoG via two separable Gaussian low-pass filters:
    $$\nabla^2 G_\sigma \approx G_{\sigma_1} - G_{\sigma_2}, \qquad \sigma_1 = \frac{\sigma}{\sqrt{2}},\quad \sigma_2 = \sqrt{2}\sigma$$
  - Explains LoG's band-pass nature and enables fast separable/cascaded computation.
- **Image Coding (Laplacian Pyramid, Burt & Adelson 1983)**:
  - Coarse Gaussian pyramid levels predict finer levels; the Laplacian pyramid stores only the prediction error (difference images), which requires fewer bits for compression than the original image.
