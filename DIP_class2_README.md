# DIP Class 2 — Signals, LSI Systems, PSF, MTF, SNR & CNR (Summary Notes)

---

## 1. Special Signals (1D & 2D)

### A. Rect and Sinc Functions
* **1D Rectangular Function** (unit area):
  $$\text{rect}(x) = \begin{cases} 1, & |x| < 1/2 \\ 0, & |x| > 1/2 \end{cases}$$
* **1D Sinc Function** (unit area, $\text{sinc}(0) = 1$):
  $$\text{sinc}(x) = \frac{\sin(\pi x)}{\pi x}$$
* **2D Extensions**:
  $$\text{rect}(x, y) = \begin{cases} 1, & |x| < 1/2 \text{ and } |y| < 1/2 \\ 0, & \text{otherwise} \end{cases} \qquad \text{sinc}(x, y) = \frac{\sin(\pi x)\sin(\pi y)}{\pi^2 x y}$$

### B. Impulse (Delta) Function
* **1D Dirac Delta $\delta(x)$**: Generalized function with zero width and unit area.
  * $\delta(x) = 0$ for $x \neq 0$, and $\int_{-\infty}^{\infty} \delta(x)\,dx = 1$.
  * **Sifting Property**: $\int_{-\infty}^{\infty} f(x)\delta(x - t)\,dx = f(t)$.
  * **Limit Forms**: $\delta(t) = \lim_{a\to\infty} a\,\text{rect}(at) = \lim_{a\to\infty} a\,\text{sinc}(at) = \lim_{a\to\infty} a e^{-\pi a^2 t^2}$.
* **2D Delta Functions** (separable):
  * **Continuous (Dirac)**: $\delta(u,v) = \delta(u)\delta(v)$ ($\infty$ at $(0,0)$, $0$ elsewhere; double integral $= 1$).
  * **Discrete (Kronecker)**: $\delta(m,n) = \delta(m)\delta(n) = \begin{cases} 1, & (m,n) = (0,0) \\ 0, & \text{otherwise} \end{cases}$
  * **Discrete Sifting**: $x(m,n) = \sum_{m'=-\infty}^{\infty}\sum_{n'=-\infty}^{\infty} x(m',n')\,\delta(m-m', n-n')$.

### C. Exponential, Sinusoidal & Periodic Signals
* **Euler's Formula**: $e^{j\theta} = \cos\theta + j\sin\theta$.
* **2D Complex Plane Wave**:
  $$e(x,y) = e^{j2\pi(u_0 x + v_0 y)} = \underbrace{\cos[2\pi(u_0 x + v_0 y)]}_{\text{real & even}} + j\underbrace{\sin[2\pi(u_0 x + v_0 y)]}_{\text{imaginary & odd}}$$
  where $u_0, v_0$ are fundamental spatial frequencies (units: $1/\text{distance}$) controlling oscillation rates along $x$ and $y$.
* **Continuous vs. Digital Image**:
  * Continuous image: $x(u,v)$; Sampled image: $x(m,n)$ with $m \in [0, M-1], n \in [0, N-1]$.
  * Quantized digital image uses $B$ bits/pixel (typically $B = 8$).
* **Doubly Periodic Signal**: Satisfies $x(m, n+N) = x(m,n)$ and $x(m+M, n) = x(m,n)$; has $MN$ distinct samples per fundamental period.

---

## 2. Linear Shift-Invariant (LSI) Systems & Convolution

A system maps input $f(x,y)$ (or $x(m,n)$) to output $g(x,y)$ (or $y(m,n) = T[x(m,n)]$).

* **Linearity** (Additivity + Homogeneity):
  $$T[a\,x_1(m,n) + b\,x_2(m,n)] = a\,y_1(m,n) + b\,y_2(m,n)$$
* **Shift / Spatial Invariance**: Shifting the input shifts the output identically (response does not depend on location):
  $$T[x(m-k, n-l)] = y(m-k, n-l) \quad \forall k, l$$
  *(Note: Linearity and shift-invariance are independent properties.)*
* **Convolution (LSI Output)**:
  * **Continuous 2D**: $g(x,y) = f(x,y) ** h(x,y) = \int_{-\infty}^{\infty}\int_{-\infty}^{\infty} f(\xi,\eta)\,h(\xi-x, \eta-y)\,d\xi\,d\eta$
  * **Discrete 2D**: $y(m,n) = \sum_k \sum_l x(k,l)\,h(m-k, n-l) = x(m,n) ** h(m,n)$
  * *Graphical steps*: Rotate PSF $h$ by $180^\circ$ about origin, shift by $(m,n)$, multiply overlapping elements with $x$, and sum.
* **Properties of LSI Convolution**: Linearity, Shift Invariance, **Associativity** ($h_2 * [h_1 * f] = [h_2 * h_1] * f$), **Commutativity** ($h_1 * h_2 = h_2 * h_1$), **Distributivity** ($h_1 * f + h_2 * f = [h_1 + h_2] * f$).
* **Why LSI Models Matter**: Simplify system analysis, approximate many real non-LSI systems, and allow fast Fourier-transform computation.

---

## 3. Point Spread Function (PSF)

* **Definition**: The response of an imaging system to an ideal point source (impulse).
  $$h(m,n; k,l) = T[\delta(m-k, n-l)]$$
  For optical/intensity systems, PSF is real and non-negative, measuring the **blurring** added by optics.
* **General Linear Degradation Model** (Blur + Additive Noise $\eta$):
  $$g(x,y) = \int_{-\infty}^{\infty}\int_{-\infty}^{\infty} h(x,y;\alpha,\beta)\,f(\alpha,\beta)\,d\alpha\,d\beta + \eta(x,y)$$
* **Key Characteristics**:
  * **Narrower PSF** $\Rightarrow$ **higher spatial resolution** (finer detail).
  * Can be **isotropic** (radially symmetric) or **non-isotropic** (e.g., ultrasound, X-ray CT, MRI depending on acquisition).
  * PSF often degrades away from the center of the Field of View (FOV); a zone of roughly constant PSF is called an **isoplanatic region**.
* **Interplay with Sampling**:
  * PSF acts as a **spatial low-pass filter**.
  * **Undersampling** (rate too low for PSF width) $\Rightarrow$ loss of high-frequency details and **aliasing**.
  * **Oversampling** (above Nyquist rate) $\Rightarrow$ faithful PSF capture, reduced noise, better separation of close objects.
* **PSF Estimation Techniques**:
  1. **Measurement-based**: Point sources (sub-resolution microbeads in microscopy, stars in astronomy), random noise targets, or speckle illumination (PEPSI + Wiener deconvolution).
  2. **Model-based / Analytical**: Theoretical calculation from NA, wavelength, aberrations; or parametric fitting (Gaussian, Airy disk).
  3. **Blind Deconvolution**: Estimates both PSF and original image from blurred data alone (no calibration needed, but ill-posed).
  4. **Super-resolution / Interpolation** and advanced computational methods.
* **Significance**: Models image formation, quantifies resolution/sharpness, aids aberration correction, enables **deconvolution** (image restoration), and guides optical system design.

---

## 4. Optical & Modulation Transfer Functions (OTF & MTF)

* **Optical Transfer Function (OTF)**: Normalized frequency response of a spatially invariant system:
  $$\text{OTF} = \frac{H(\xi_1, \xi_2)}{H(0,0)}$$
* **Modulation Transfer Function (MTF)**: Magnitude of the OTF (expresses spatial resolution in the frequency domain):
  $$\text{MTF} = |\text{OTF}| = \frac{|H(\xi_1, \xi_2)|}{|H(0,0)|}$$
* **Relation to PSF**: $\text{MTF} \leftrightarrow \mathcal{F}(\text{PSF})$ and $\text{PSF} \leftrightarrow \mathcal{F}^{-1}(\text{MTF})$.
* **MTF Behavior**: Acts like a low-pass filter that rolls off at higher spatial frequencies (units: e.g., $\text{mm}^{-1}$), plotting normalized contrast vs. spatial frequency to show limiting spatial resolution.
* **Computing MTF from a PSF Kernel**:
  1. Divide kernel by sum of its elements (normalizes MTF to $0\text{–}1$).
  2. Zero-pad by centering the kernel in an array matching the image size.
  3. Compute `fftshift(abs(fft2(...)))`.

---

## 5. Signal-to-Noise Ratio (SNR) & Contrast-to-Noise Ratio (CNR)

* **Signal-to-Noise Ratio (SNR)**: Ratio of signal power to noise power:
  $$\text{SNR} = \frac{P_{\text{signal}}}{P_{\text{noise}}}, \qquad \text{SNR (dB)} = 10\log_{10}\!\left(\frac{P_{\text{signal}}}{P_{\text{noise}}}\right) = 20\log_{10}\!\left(\frac{A_{\text{signal}}}{A_{\text{noise}}}\right)$$
  where $P$ is average power and $A$ is RMS amplitude.
  * **In Image Processing**: Often defined as $\frac{\text{Mean pixel value}}{\text{Standard deviation of pixel values}}$.
  * **In MRI** (uniform noise): $\text{SNR} = \frac{\text{Mean intensity in ROI}}{\text{SD of background outside image}}$. Higher SNR = cleaner image.
* **Contrast-to-Noise Ratio (CNR)**:
  * Raw contrast alone cannot distinguish true tissue contrast from noise fluctuations.
  * CNR measures an imaging system's ability to produce **clinically useful contrast** relative to noise (human object discrimination increases with contrast and decreases linearly with noise).
