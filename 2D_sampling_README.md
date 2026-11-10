# 2D Fourier Transform & 2D Sampling — Summary Notes

---

## 1. 1D Fourier Review & 2D Fourier Transform (FT)

* **Fourier Concept**: Decomposes a signal into a weighted sum of sinusoidal / complex exponential basis functions (a change of basis: $M f(x) = F(\omega)$).
* **1D FT Pair**:
  $$F(u) = \int_{-\infty}^{\infty} f(x)e^{-j2\pi ux}\,dx \qquad \Longleftrightarrow \qquad f(x) = \int_{-\infty}^{\infty} F(u)e^{j2\pi ux}\,du$$
  * *Example*: A box of width $X$ transforms to $F(u) = X\,\text{sinc}(\pi Xu)$.
* **2D Continuous FT Pair** ($u, v$ are spatial frequencies):
  $$F(u,v) = \int_{-\infty}^{\infty}\int_{-\infty}^{\infty} f(x,y)e^{-j2\pi(ux+vy)}\,dx\,dy \qquad f(x,y) = \int_{-\infty}^{\infty}\int_{-\infty}^{\infty} F(u,v)e^{j2\pi(ux+vy)}\,du\,dv$$
* **Spectrum Representation**: $F(u,v) = F_R(u,v) + jF_I(u,v)$
  * **Magnitude Spectrum**: $|F(u,v)|$
  * **Phase Spectrum**: $\phi(u,v) = \arctan\!\left(\frac{F_I(u,v)}{F_R(u,v)}\right)$
* **2D Sinusoidal Plane Waves**: $e^{j2\pi(ux+vy)} = \cos 2\pi(ux+vy) + j\sin 2\pi(ux+vy)$ represents parallel wave crests normal to $\mathbf{u} = (u,v)^T$ with spatial frequency $\sqrt{u^2+v^2}$ and wavelength $\lambda = 1/\sqrt{u^2+v^2}$.

---

## 2. Important 2D Fourier Transform Pairs & Properties

| Spatial Function $f(x,y)$ | Fourier Transform $F(u,v)$ | Key Note |
| :--- | :--- | :--- |
| Rectangle (width $X \times Y$) | $XY\,\text{sinc}(\pi Xu)\,\text{sinc}(\pi Yv)$ | Separable in $x$ and $y$ |
| 2D Gaussian $\frac{1}{2\pi\sigma^2}e^{-(x^2+y^2)/(2\sigma^2)}$ | $e^{-2\pi^2(u^2+v^2)\sigma^2}$ | FT of Gaussian is Gaussian (inverse scale) |
| Circular disk (radius $a$) | $\frac{a\,J_1(\pi a\rho)}{\rho},\;\rho=\sqrt{u^2+v^2}$ | $J_1$: Bessel function (2D sinc-like) |
| Unit impulse $\delta(x,y)$ | $1$ | Constant across all frequencies |
| $\frac{1}{2}[\delta(x,y-a)+\delta(x,y+a)]$ | $\cos(2\pi av)$ | Two impulses $\leftrightarrow$ cosine grating |

### Key 2D FT Properties
1. **Linearity**: $\alpha f(x,y) + \beta g(x,y) \Leftrightarrow \alpha F(u,v) + \beta G(u,v)$.
2. **Scaling (Similarity)**: $f(ax, by) \Leftrightarrow \frac{1}{|ab|}F\!\left(\frac{u}{a}, \frac{v}{b}\right)$.
3. **Shift**: $f(x-a, y-b) \Leftrightarrow e^{-j2\pi(au+bv)}F(u,v)$ (only phase changes; magnitude unchanged).
4. **Affine / Rotation**: $\mathbf{x} \to A\mathbf{x} \Rightarrow \mathbf{u} \to A^{-T}\mathbf{u}$. For pure rotation ($A=R$), the FT rotates by the same angle.
5. **Parseval's Theorem (Energy Conservation)**:
   $$\iint_{-\infty}^{\infty}|x(u,v)|^2\,du\,dv = \frac{1}{4\pi^2}\iint_{-\infty}^{\infty}|X(\omega_1,\omega_2)|^2\,d\omega_1\,d\omega_2$$
6. **1D DFT Pair** ($N$ samples): $F_n = \sum_{k=0}^{N-1} f_k e^{-j2\pi nk/N}$, $f_k = \frac{1}{N}\sum_{n=0}^{N-1} F_n e^{j2\pi nk/N}$ ($n=0$ is DC).

---

## 3. LSI Eigenfunctions, Filtering vs. Convolution & Convolution Theorem

* **Eigenfunctions of 2D LSI Systems**: Complex exponentials $f(u,v) = e^{j(\omega_1 u + \omega_2 v)}$ pass through an LSI system unchanged in frequency, scaled by complex gain $H(\omega_1,\omega_2) = \mathcal{F}\{h(u,v)\}$ (the **frequency response**).
* **Filtering (Correlation) vs. Convolution**:
  * **Filtering** (`filter2(h,f)` in MATLAB): $g[m,n] = \sum_{k,l} h[k,l]\,f[m+k, n+l]$ (no flip).
  * **Convolution** (`conv2(h,f)` in MATLAB): $g[m,n] = \sum_{k,l} h[k,l]\,f[m-k, n-l]$ (flips kernel in both $x$ and $y$ / $180^\circ$ rotation).
  * *(Both are identical when $h$ is symmetric.)*
* **Convolution Theorem**:
  $$f(x,y) * h(x,y) \;\Longleftrightarrow\; F(u,v)\,H(u,v)$$
  * **Space convolution = Frequency multiplication** (linear filtering can be done either via spatial convolution or by taking FTs, multiplying, and taking the inverse FT).

---

## 4. 2D Sampling, Comb Function & Nyquist Theorem

* **Digitization Pipeline**: Continuous image $x(u,v) \xrightarrow{\text{Sampler}} x_s(u,v) \xrightarrow{\text{Quantizer}} x(m,n)$ (pixels in 2D, voxels in 3D).
* **2D Comb & Sampling Function**:
  $$\text{comb}(x,y) = \sum_{m=-\infty}^{\infty}\sum_{n=-\infty}^{\infty}\delta(x-m, y-n)$$
  $$\delta_s(x,y;\Delta x,\Delta y) = \sum_{m=-\infty}^{\infty}\sum_{n=-\infty}^{\infty}\delta(x-m\Delta x, y-n\Delta y) = \frac{1}{\Delta x\Delta y}\text{comb}\!\left(\frac{x}{\Delta x},\frac{y}{\Delta y}\right)$$
* **Sampled Image**: $f_s(x,y) = f(x,y)\cdot \delta_s(x,y;\Delta x,\Delta y) = \sum\sum f(m\Delta x, n\Delta y)\,\delta(x-m\Delta x, y-n\Delta y)$.
* **Spectrum of Sampled Image (Replication)**:
  Sampling in space causes **periodic replication ("islands")** of $F(u,v)$ in frequency every $\frac{1}{\Delta x}$ in $u$ and $\frac{1}{\Delta y}$ in $v$ (or $\omega_{us} = \frac{2\pi}{\Delta u}, \omega_{vs} = \frac{2\pi}{\Delta v}$ in rad/unit):
  $$F_s(u,v) = \frac{1}{\Delta x\Delta y}\sum_{m=-\infty}^{\infty}\sum_{n=-\infty}^{\infty} F\!\left(u - \frac{n}{\Delta x},\, v - \frac{m}{\Delta y}\right)$$
* **Bandlimited Image**: $X(\omega_1,\omega_2) = 0$ for $|\omega_1| > \omega_{10},\; |\omega_2| > \omega_{20}$ (bandlimits $\omega_{10}, \omega_{20}$).
* **Nyquist Sampling Theorem**: To avoid spectral overlap and recover $f(x,y)$ exactly, sampling frequencies must exceed twice the highest frequency component:
  $$\omega_{us} \ge 2\omega_{10}, \quad \omega_{vs} \ge 2\omega_{20} \qquad \text{or Nyquist interval:} \quad \Delta x \le \frac{1}{2u_c}, \quad \Delta y \le \frac{1}{2v_c}$$

---

## 5. Exact Reconstruction, Undersampling & Aliasing

* **Exact Reconstruction** (when Nyquist holds):
  Multiply $X_s(\omega_1,\omega_2)$ by an ideal 2D Low-Pass Filter (LPF) $H(\omega_1,\omega_2) = \Delta u\Delta v$ inside passband $R$ (e.g., $[-\omega_{us}/2, \omega_{us}/2] \times [-\omega_{vs}/2, \omega_{vs}/2]$) and $0$ outside.
  * In the spatial domain, this corresponds to **2D sinc interpolation**:
    $$\tilde{x}(u,v) = \sum_{m=-\infty}^{\infty}\sum_{n=-\infty}^{\infty} x(m\Delta u, n\Delta v)\,\text{sinc}(\omega_{us}(u - m\Delta u))\,\text{sinc}(\omega_{vs}(v - n\Delta v))$$
* **Undersampling & Aliasing**:
  * If $\omega_{us} < 2\omega_{10}$ or $\omega_{vs} < 2\omega_{20}$, spectral replicas **overlap**.
  * **Fold-over frequencies** above $\omega_{us}/2$ fold back and appear as lower frequencies in the reconstructed image:
    $$\text{Original freq } \frac{\omega_{us}}{2} + \Delta\omega \;\longrightarrow\; \text{Aliased to } \frac{\omega_{us}}{2} - \Delta\omega$$
  * *Visual effects*: Edge smearing, false low-frequency patterns, and loss of fine detail.
  * *Example*: $x(u,v) = 2\cos(3u+4v)$ ($\omega_{10}=3, \omega_{20}=4$) sampled at $\Delta u = \Delta v = 0.4\pi$ ($\omega_{us}=\omega_{vs}=5 < \text{Nyquist } 6, 8$) aliases to $\tilde{x}(u,v) = 2\cos(2u+v)$.
* **Anti-Aliasing Rule**: Aliasing **cannot be removed by post-filtering**. It must be prevented by **pre-filtering** the continuous image with an anti-aliasing LPF before sampling (crucial when wideband noise with long spectral tails is present).
