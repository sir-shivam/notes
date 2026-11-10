# Lectures 7 & 8 — Image Quantization, Lloyd-Max Quantizer & 2-D DSFT (Summary Notes)

---

## 1. Image Quantization Process & Uniform Quantizer

* **Quantization**: Maps a continuous variable $x \in [t_1, t_{L+1}]$ into a discrete value $x_q$ from a finite set of reconstruction levels $\{r_1, \dots, r_L\}$ (a staircase function).
* **Quantization Rule**: Using transition (decision) levels $\{t_1, \dots, t_{L+1}\}$ (where $t_1 = \min(x)$, $t_{L+1} = \max(x)$):
  $$\text{If } x \in [t_k, t_{k+1}), \quad \text{then } x_q = r_k$$
* **Key Properties**:
  1. **Irreversible**: Original $x$ cannot be uniquely recovered from $x_q$.
  2. **Incurs Distortion**: Design balances simplicity vs. minimum distortion.
  3. **Zero-memory operation**: Output depends only on the current input sample.
* **Uniform Quantizer** (for sensor range $[0, A]$ quantized to $L$ levels, e.g. $L = 256$):
  $$t_k = \frac{A(k-1)}{L} \quad (k=1,\dots,L+1), \qquad r_k = t_k + \frac{A}{2L} \quad (k=1,\dots,L)$$
  * Constant **quantization interval**: $q = t_k - t_{k-1} = r_k - r_{k-1} = \frac{A}{L}$.
* **Uniform Quantization Error & SNR**:
  * Quantization error $e = x - x_q$ is uniformly distributed on $(-q/2, q/2)$, giving MSE:
    $$\epsilon = \frac{1}{q}\int_{-q/2}^{q/2} \alpha^2\,d\alpha = \frac{q^2}{12}$$
  * For a uniformly distributed image of range $A$ ($\sigma_x^2 = A^2/12$) quantized with $B$ bits ($L = 2^B$, $q = A/2^B$):
    $$\frac{\sigma_x^2}{\epsilon} = 2^{2B} \implies \text{SNR}_{\text{dB}} = 10\log_{10}(2^{2B}) = 6B\text{ dB} \quad (\text{i.e., } 6\text{ dB/bit})$$

---

## 2. Minimum Mean Squared (Lloyd-Max) Quantizer

* **Goal**: For a random variable $x \in [0, A]$ with continuous PDF $p_X(x)$, find optimal $t_k$ and $r_k$ ($k=1,\dots,L$) that minimize Mean Square Error (MSE):
  $$\epsilon = E[(x - x_q)^2] = \sum_{i=1}^{L}\int_{t_i}^{t_{i+1}} (x - r_i)^2 p_X(x)\,dx$$
* **Lloyd-Max Optimality Conditions** (setting $\frac{\partial\epsilon}{\partial t_k} = 0$ and $\frac{\partial\epsilon}{\partial r_k} = 0$):
  1. **Optimal Transition Levels** (halfway between reconstruction levels):
     $$t_k = \frac{r_k + r_{k-1}}{2}$$
  2. **Optimal Reconstruction Levels** (center of mass / centroid of the PDF in $[t_k, t_{k+1})$):
     $$r_k = \frac{\int_{t_k}^{t_{k+1}} x\,p_X(x)\,dx}{\int_{t_k}^{t_{k+1}} p_X(x)\,dx} = E[X \mid X \in \Omega_k]$$
* **Solution Methods**:
  * Solved simultaneously via iterative numerical methods (e.g., Newton's method).
  * **High-rate approximation (large $L$)**: Modeling $p_X(x)$ as piecewise constant decouples $t_{k+1}$ and $r_k = \frac{t_k+t_{k+1}}{2}$, giving approximate MSE:
    $$\epsilon \approx \frac{1}{12L^2}\left\{\int_{t_1}^{t_{L+1}} [p_X(x)]^{1/3}\,dx\right\}^3$$
* **Properties of Lloyd-Max Quantizers**:
  1. **Unbiased in the mean**: $E[X_q] = E[X]$.
  2. **Orthogonality of quantization error & output**: $E[(X - X_q)X_q] = 0$, i.e., $E[XX_q] = E[X_q^2]$.
* **Common Image PDFs & Uniform Case**:
  * Common models: **Gaussian** $p_X(x) = \frac{1}{\sqrt{2\pi\sigma_x^2}}e^{-(x-\mu)^2/(2\sigma_x^2)}$ and **Laplacian** $p_X(x) = \frac{\alpha}{2}e^{-\alpha|x-\mu|}$.
  * For a **uniform PDF**, Lloyd-Max reduces to the **uniform (linear) quantizer** ($t_k - t_{k-1} = q$).

---

## 3. Image Transforms & 2-D Discrete Space Fourier Transform (DSFT)

* **Applications of 2-D Unitary Transforms in DIP**:
  1. **Data compression (transform coding)**: Discard or coarsely quantize small transform coefficients.
  2. **Feature extraction**: E.g., high-frequency FT components capture image edges.
  3. **Dimensionality reduction**: Retain principal components, discard small ones.
  4. **Image filtering**: Wiener filtering and standard 2-D frequency filters.
* **2-D DSFT Definition**:
  For a sampled image $x(m\Delta u, n\Delta v) = x(m,n)$ with discrete frequencies $\Omega_1 = \omega_1\Delta u$, $\Omega_2 = \omega_2\Delta v$:
  $$X(\Omega_1, \Omega_2) = \sum_{m=-\infty}^{\infty}\sum_{n=-\infty}^{\infty} x(m,n)\,e^{-j(\Omega_1 m + \Omega_2 n)}$$
  * **Periodicity**: Because of spatial sampling, the 2-D DSFT is **doubly periodic** with period $2\pi$:
    $$X(\Omega_1, \Omega_2) = X(\Omega_1 + 2k\pi, \Omega_2 + 2l\pi)$$
  * **Inverse 2-D DSFT**:
    $$x(m,n) = \frac{1}{4\pi^2}\int_{-\pi}^{\pi}\int_{-\pi}^{\pi} X(\Omega_1, \Omega_2)\,e^{j(\Omega_1 m + \Omega_2 n)}\,d\Omega_1\,d\Omega_2$$
* **Key DSFT Examples**:
  1. **2-D Rectangular Impulse Response**: $h(m,n) = 1$ for $|m| < M, |n| < N$ ($0$ otherwise):
     $$H(\Omega_1,\Omega_2) = \frac{\sin(\Omega_1(M - 1/2))}{\sin(\Omega_1/2)} \times \frac{\sin(\Omega_2(N - 1/2))}{\sin(\Omega_2/2)} \quad (\text{aliased 2-D sinc})$$
  2. **Exponential × Finite Window**: $x(m,n) = a^m u(m) v(n)$ where $v(n)=1$ for $0 \le n \le N-1$:
     $$X(\Omega_1,\Omega_2) = \left(\frac{1}{1 - a e^{-j\Omega_1}}\right) \cdot e^{-j\Omega_2\left(\frac{N-1}{2}\right)}\,\frac{\sin(N\Omega_2/2)}{\sin(\Omega_2/2)}$$
