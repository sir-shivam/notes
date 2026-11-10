# Fourier Analysis & Transforms (`Transforms.pptx`) — Summary Notes

Summary of `Transforms.pptx` (45 slides; matches `transforms.pdf` slide-for-slide).

---

## 1. Fourier Series (Periodic Signals)

- **Origin & Applications**: Introduced by Joseph Fourier (motivated by the heat equation); foundational in signal/image filtering, optics, probability theory, and PDEs.
- **Definition**: Any periodic signal $f(t)$ with period $T$ is expressed as a linear combination of harmonically related complex exponentials (sines and cosines):
  $$f(t) = \sum_{n=-\infty}^{\infty} c_n\,e^{j2\pi n t / T}, \qquad c_n = \frac{1}{T}\int_{-T/2}^{T/2} f(t)\,e^{-j2\pi n t / T}\,dt$$
- **Why Complex Exponentials?**: Sine waves alone cannot represent cosines or arbitrarily phase-shifted sinusoids; complex exponentials ($e^{j\theta} = \cos\theta + j\sin\theta$) encode both sine and cosine components.
- **Orthogonality**: Under the periodic inner product $\langle f, g\rangle = \int_{-T/2}^{T/2} f^*(t)g(t)\,dt$:
  $$\int_{-T/2}^{T/2} e^{j2\pi n t / T} e^{-j2\pi m t / T}\,dt = \begin{cases} T, & n = m \\ 0, & n \neq m \end{cases}$$
  Forming an **orthogonal basis** for periodic functions.

---

## 2. Continuous 1D Fourier Transform (FT)

- **Definition (Non-Periodic, Absolutely/Square-Integrable Signals)**:
  $$F(\mu) = \mathbf{F}(f(t))(\mu) = \int_{-\infty}^{\infty} f(t)\,e^{-j2\pi\mu t}\,dt, \qquad f(t) = \mathbf{F}^{-1}(F(\mu))(t) = \int_{-\infty}^{\infty} F(\mu)\,e^{j2\pi\mu t}\,d\mu$$
  A signal $f(t)$ and its spectrum $F(\mu)$ form a unique, lossless **Fourier pair** (real signals generally yield complex coefficients and continuous real-valued frequencies $\mu$).
- **Key Examples**:
  - **Rect $\leftrightarrow$ Sinc**: $\text{rect}(t; W/2)$ of amplitude $A$ transforms to $AW\,\text{sinc}(\mu W) = AW\,\frac{\sin(\pi\mu W)}{\pi\mu W}$; inversely, $\mathbf{F}(\text{sinc}) = \text{rect}$.
  - **Dirac Delta vs. Kronecker Delta**:
    - *Dirac delta* $\delta(t)$ (continuous $t$): $\int_{-\infty}^{\infty}\delta(t)\,dt = 1$, sifting $\int_{-\infty}^{\infty}\delta(t-t_0)f(t)\,dt = f(t_0)$, $\mathbf{F}(\delta(t)) = 1$, $\mathbf{F}(\delta(t-t_0))(\mu) = e^{-j2\pi\mu t_0}$.
    - *Kronecker delta* (discrete $t$): $\delta[0]=1$, $0$ elsewhere; discrete sifting $\sum_{t=-\infty}^{\infty} f(t)\delta(t-t_0) = f(t_0)$.
  - **Gaussian-Windowed Cosine Illustration**: $f(t) = \cos(6\pi t)e^{-\pi t^2}$ (3 Hz pulse) yields a real-valued FT with large integrand average at $\mu = 3$ and near-zero oscillatory integral at $\mu = 5$.
  - **Cosine & Sine Waves**:
    $$\mathbf{F}(\cos 2\pi a t)(\mu) = \frac{1}{2}[\delta(\mu - a) + \delta(\mu + a)], \qquad \mathbf{F}(\sin 2\pi a t)(\mu) = \frac{1}{2j}[\delta(\mu - a) - \delta(\mu + a)]$$
  - **Audio / Music**: FT identifies fundamental note frequencies (e.g., C\# at 138.59 Hz for male vocals, 246.94 Hz scale for female vocals) and overtones.

---

## 3. Properties & Theorems

| Property / Theorem | Mathematical Statement | Key Implication |
| :--- | :--- | :--- |
| **Linearity** | $\mathbf{F}(af + g)(\mu) = a\mathbf{F}(f)(\mu) + \mathbf{F}(g)(\mu)$ | Follows from integral linearity |
| **Time Shift** | $\mathbf{F}(f(t - t_0))(\mu) = e^{-j2\pi\mu t_0}F(\mu)$ | Shift adds linear phase; magnitude unchanged |
| **Frequency Shift** | $\mathbf{F}^{-1}(F(\mu - \mu_0))(t) = e^{j2\pi\mu_0 t}f(t)$ | Frequency shift modulates time signal |
| **Convolution** | $\mathbf{F}[(h * f)](\mu) = F(\mu)\,H(\mu)$ | Enables frequency-domain filtering |
| **Multiplication (Dual)** | $\mathbf{F}[f \cdot h](\mu) = (F * H)(\mu)$ | Point-wise product = spectral convolution |
| **Parseval's Theorem** | $\int_{-\infty}^{\infty}|f(t)|^2 dt = \int_{-\infty}^{\infty}|F(\mu)|^2 d\mu$ | Energy conservation (compression basis) |
| **Parseval (General)** | $\int_{-\infty}^{\infty}f(t)g^*(t)\,dt = \int_{-\infty}^{\infty}F(\mu)G^*(\mu)\,d\mu$ | Inner product invariance |
| **Differentiation** | $\mathbf{F}(f^{(n)}(t))(\mu) = (j2\pi\mu)^n F(\mu)$ | Converts PDE derivatives into products |

---

## 4. Time-Limited vs. Band-Limited Signals

- **Time-limited** (or **space-limited** in 2D): $f(t)=0$ outside a finite interval (all digital images in memory are space-limited).
- **Band-limited**: $F(\mu) = 0$ for $|\mu| > B$.
- **Impossibility Theorems**:
  1. **A non-zero time-limited signal cannot be band-limited**: $g(t) = g(t)\cdot\text{rect}(t; W/2) \implies G(\mu) = G(\mu) * W\,\text{sinc}(W\mu)$, which cannot have finite spectral support since $\text{sinc}$ never vanishes identically.
  2. **A non-zero band-limited signal cannot be time-limited**: $H(\mu) = H(\mu)\cdot\text{rect}(\mu; B/2) \implies h(t) = h(t) * B\,\text{sinc}(Bt)$, which cannot have finite time support.
  - Note: For band-limited $h(t)$, both $h = h * \delta$ and $h(t) = h(t) * B\,\text{sinc}(Bt)$ hold.
- Every band-limited signal is **infinitely differentiable** (converse false; e.g., a Gaussian is $C^\infty$ but not band-limited).

---

## 5. 2D Fourier Transform & Separability

- **2D FT Pair** (spatial $(x, y)$, frequency $(\mu, \nu)$):
  $$F(\mu,\nu) = \iint_{-\infty}^{\infty} f(x,y)\,e^{-j2\pi(\mu x + \nu y)}\,dx\,dy, \qquad f(x,y) = \iint_{-\infty}^{\infty} F(\mu,\nu)\,e^{j2\pi(\mu x + \nu y)}\,d\mu\,d\nu$$
- All 1D theorems (shift, convolution, Parseval's, etc.) generalize to 2D and higher dimensions.
- **Separability**: Both the 2D forward and inverse FTs decompose into **two 1D transforms**: transform along $y$ for each $x$ to obtain $F(x, \nu)$, then transform along $x$ to obtain $F(\mu, \nu)$.
