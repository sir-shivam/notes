# Fourier Analysis & Transforms (`transforms.pdf`) — Summary Notes

---

## 1. Fourier Series (Periodic Signals)

* **Origin & Purpose**: Discovered by Joseph Fourier (heat equation); fundamental in image/signal filtering, optics, and PDEs.
* **Definition**: Any periodic function $f(t)$ with period $T$ can be represented as a weighted sum of harmonically related complex exponentials (sines and cosines):
  $$f(t) = \sum_{n=-\infty}^{\infty} c_n\,e^{j2\pi n t / T}, \qquad c_n = \frac{1}{T}\int_{-T/2}^{T/2} f(t)\,e^{-j2\pi n t / T}\,dt$$
  where $n = 0, \pm 1, \pm 2, \dots$ is the frequency index and $e^{j\theta} = \cos\theta + j\sin\theta$.
* **Why Complex Exponentials?**: Sine waves alone cannot represent cosine or phase-shifted waves ($\sin(\omega t + \phi) = \cos\omega t\sin\phi + \sin\omega t\cos\phi$); complex exponentials compactly combine both sine and cosine.
* **Orthogonality (Inner Product)**: For periodic complex functions, $\langle f, g\rangle = \int_{-T/2}^{T/2} f^*(t)g(t)\,dt$:
  $$\int_{-T/2}^{T/2} e^{j2\pi n t / T} e^{-j2\pi m t / T}\,dt = \begin{cases} T, & n = m \\ 0, & n \neq m \end{cases}$$
  Hence complex sinusoids form an **orthogonal basis**.

---

## 2. Continuous 1D Fourier Transform (FT)

* **Extension to Non-Periodic Signals**: For absolutely/square-integrable signals with continuous frequency $\mu$:
  $$F(\mu) = \mathbf{F}(f(t))(\mu) = \int_{-\infty}^{\infty} f(t)\,e^{-j2\pi\mu t}\,dt \qquad \text{(Forward FT)}$$
  $$f(t) = \mathbf{F}^{-1}(F(\mu))(t) = \int_{-\infty}^{\infty} F(\mu)\,e^{j2\pi\mu t}\,d\mu \qquad \text{(Inverse FT)}$$
* **Fourier Pair**: A signal $f(t)$ (lowercase) and its spectrum $F(\mu)$ (uppercase) form a unique lossless pair. Real-valued signals generally yield complex-valued coefficients.

### Important 1D FT Examples
1. **Rectangular Pulse $\leftrightarrow$ Sinc**:
   For $f(t) = A$ on $[-W/2, W/2]$ ($0$ elsewhere):
   $$F(\mu) = AW\,\frac{\sin(\pi\mu W)}{\pi\mu W} = AW\,\text{sinc}(\mu W)$$
   where $\text{sinc}(0)=1$ and $\text{sinc}(m)=0$ for non-zero integers $m$. Conversely, $\mathbf{F}(\text{sinc}) = \text{rect}$.
2. **Dirac Delta vs. Kronecker Delta**:
   * **Dirac Delta** (continuous $t$): $\delta(t) = \infty$ at $t=0$, $0$ elsewhere, $\int_{-\infty}^{\infty}\delta(t)\,dt = 1$.
     * Sifting property: $\int_{-\infty}^{\infty}\delta(t-t_0)f(t)\,dt = f(t_0)$.
     * FT: $\mathbf{F}(\delta(t))(\mu) = 1$, and $\mathbf{F}(\delta(t-t_0))(\mu) = e^{-j2\pi\mu t_0}$.
   * **Kronecker Delta** (discrete $t$): $\delta(t)=1$ at $t=0$, $0$ elsewhere; sifting: $\sum_{t=-\infty}^{\infty} f(t)\delta(t-t_0) = f(t_0)$.
3. **Cosine & Sine Waves**:
   $$\mathbf{F}(\cos 2\pi a t)(\mu) = \frac{1}{2}[\delta(\mu - a) + \delta(\mu + a)], \qquad \mathbf{F}(\sin 2\pi a t)(\mu) = \frac{1}{2j}[\delta(\mu - a) - \delta(\mu + a)]$$
4. **Physical Intuition (e.g., Audio/Music)**: Integrating $f(t)e^{-j2\pi\mu t}$ yields a large value only when $\mu$ matches a frequency present in $f(t)$ (e.g., fundamental frequency + overtones of musical notes).

---

## 3. Properties & Theorems of the Fourier Transform

| Property / Theorem | Mathematical Statement | Key Meaning / Use |
| :--- | :--- | :--- |
| **Linearity** | $\mathbf{F}(af + g)(\mu) = a\mathbf{F}(f)(\mu) + \mathbf{F}(g)(\mu)$ | Complex scalar $a$; follows from integral linearity |
| **Time Shift** | $\mathbf{F}(f(t - t_0))(\mu) = e^{-j2\pi\mu t_0}F(\mu)$ | Time shift adds linear phase; magnitude unchanged |
| **Frequency Shift** | $\mathbf{F}^{-1}(F(\mu - \mu_0))(t) = e^{j2\pi\mu_0 t}f(t)$ | Frequency shift modulates signal in time |
| **Convolution** | $\mathbf{F}[(h * f)](\mu) = F(\mu)\,H(\mu)$ | Convolution in time = multiplication in frequency |
| **Multiplication (Dual)** | $\mathbf{F}[f \cdot h](\mu) = (F * H)(\mu)$ | Point-wise product = frequency convolution |
| **Parseval's Theorem** | $\int_{-\infty}^{\infty}|f(t)|^2 dt = \int_{-\infty}^{\infty}|F(\mu)|^2 d\mu$ | Energy conservation (important in compression) |
| **Parseval (General)** | $\int_{-\infty}^{\infty}f(t)g^*(t)\,dt = \int_{-\infty}^{\infty}F(\mu)G^*(\mu)\,d\mu$ | Inner product preserved across domains |
| **Differentiation** | $\mathbf{F}(f^{(n)}(t))(\mu) = (j2\pi\mu)^n F(\mu)$ | Turns PDE derivatives into algebraic products |

*(Recall continuous convolution: $(h * f)(t) = \int_{-\infty}^{\infty} h(s)f(t-s)\,ds$.)*

---

## 4. Time-Limited vs. Band-Limited Signals

* **Time-limited (or Space-limited in 2D)**: Signal is zero outside a finite duration/extent (all digital images in a computer are space-limited).
* **Band-limited**: $F(\mu) = 0$ for all $|\mu| > B$ (finite frequency bandwidth $B$).
* **Duality / Impossibility Theorems**:
  1. **A non-zero time-limited signal cannot be band-limited**: $g(t) = g(t)\cdot\text{rect}(t; W/2) \Rightarrow G(\mu) = G(\mu) * W\text{sinc}(W\mu)$. Since $\text{sinc}$ extends to $\pm\infty$, $G(\mu)$ cannot have finite support.
  2. **A non-zero band-limited signal cannot be time-limited**: $H(\mu) = H(\mu)\cdot\text{rect}(\mu; B/2) \Rightarrow h(t) = h(t) * B\text{sinc}(Bt)$, which never vanishes identically outside a finite interval.
* **Differentiability**: Every band-limited signal is **infinitely differentiable** (the converse is false, e.g., a Gaussian is infinitely differentiable but not band-limited).

---

## 5. 2D Fourier Transform & Separability

* **2D FT Pair** (spatial $(x,y)$, frequency $(\mu,\nu)$):
  $$F(\mu,\nu) = \iint_{-\infty}^{\infty} f(x,y)\,e^{-j2\pi(\mu x + \nu y)}\,dx\,dy, \qquad f(x,y) = \iint_{-\infty}^{\infty} F(\mu,\nu)\,e^{j2\pi(\mu x + \nu y)}\,d\mu\,d\nu$$
* All 1D theorems (shift, convolution, Parseval's, etc.) hold in 2D and higher dimensions.
* **Separability Property**: A 2D FT (and inverse 2D FT) is computed as **two successive 1D FTs**:
  1. For each fixed $x$, take the 1D FT across $y$: $F(x,\nu) = \int_{-\infty}^{\infty} f(x,y)e^{-j2\pi\nu y}\,dy$.
  2. Take the 1D FT across $x$: $F(\mu,\nu) = \int_{-\infty}^{\infty} F(x,\nu)e^{-j2\pi\mu x}\,dx$.
