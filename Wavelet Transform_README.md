# Wavelet Transform — Summary Notes

---

## 1. Motivation: Why Wavelets? (FT vs. STFT vs. WT)

* **Fourier Transform (FT) Limitation**:
  * FT decomposes a signal into global sine/cosine waves (infinite duration, non-local).
  * It gives **frequency content**, but **no time localization** (cannot tell *when* a frequency occurs). Thus, stationary and non-stationary signals with the same frequencies produce nearly identical spectra, and FT struggles with sharp spikes and discontinuities.
* **Short-Time Fourier Transform (STFT) & Its Trade-off**:
  * Multiplies the signal by a fixed-width window $w(t)$ (where the segment is assumed stationary) and takes its FT to produce a **Time-Frequency Representation (TFR)**.
  * **Fixed-window dilemma**:
    * *Narrow window* $\Rightarrow$ good time resolution, poor frequency resolution.
    * *Wide window* $\Rightarrow$ good frequency resolution, poor time resolution.
* **Wavelet Transform (WT) Solution**:
  * Uses **variable-size windows** (Multiresolution Analysis) to capture both time and frequency simultaneously.

---

## 2. What is a Wavelet? Properties & Examples

* **Definition**: A wavelet ("small wave") is a compactly supported (finite-duration), oscillatory function $\psi(t)$.
* **Two Fundamental Properties**:
  1. **Zero Mean** (oscillatory): $\displaystyle \int_{-\infty}^{\infty} \psi(t)\,dt = 0$
  2. **Finite Energy** (square-integrable / localized): $\displaystyle \int_{-\infty}^{\infty} |\psi(t)|^2\,dt < \infty$
* **Common Wavelet Families**: Haar, Daubechies (e.g., 4-tap), Gaussian, Morlet, Mexican Hat, Coiflet.
* **Applications**: Image/signal processing, sub-band coding, MRI, speech, radar, fractals, astronomy, and PDEs.

---

## 3. Continuous Wavelet Transform (CWT) & Multiresolution Analysis (MRA)

* **CWT Definition**:
  $$\text{CWT}_x^\psi(\tau, s) = \Psi_x^\psi(\tau, s) = \frac{1}{\sqrt{|s|}} \int x(t)\,\psi^*\!\left(\frac{t - \tau}{s}\right) dt$$
  where $\psi(t)$ is the **mother wavelet** (prototype), $\tau$ is **translation** (time shift), and $s$ is **scale**.
* **Scale vs. Frequency**:
  * **High scale ($s$ large)** $\Rightarrow$ dilated (stretched) wavelet $\Rightarrow$ captures **low frequencies** (global trends) with **good frequency resolution, poor time resolution**.
  * **Low scale ($s$ small)** $\Rightarrow$ compressed wavelet $\Rightarrow$ captures **high frequencies** (short spikes/details) with **good time resolution, poor frequency resolution**.
* **CWT vs. STFT**:
  1. CWT does not take the FT of the windowed signal (no negative frequencies).
  2. Window width adapts automatically for each scale/spectral component.
* **CWT Computation Steps**: Start at $s=1$, shift $\tau$ across the signal multiplying & integrating with $\psi((t-\tau)/s)$ and scaling by $1/\sqrt{|s|}$ to fill one row of the time-scale plane; repeat for increasing values of $s$.
* **Time-Frequency Plane (MRA Tiling)**: Boxes at high frequencies are narrow in time and tall in frequency; boxes at low frequencies are wide in time and short in frequency.

---

## 4. Scaling Function, Wavelet Spaces & Two-Scale Relations

* **Subspace Decomposition**: Approximation space $V_j$ splits into coarser approximation $V_{j-1}$ and detail space $W_{j-1}$:
  $$V_j = V_{j-1} \oplus W_{j-1} = W_{j-1} \oplus W_{j-2} \oplus \dots \oplus W_0 \oplus V_0$$
* **Scaling Function $\phi(t)$ Properties**:
  1. Integrates to 1: $\int_{-\infty}^{\infty} \phi(t)\,dt = 1$
  2. Unit energy: $\int_{-\infty}^{\infty} |\phi(t)|^2\,dt = 1$
  3. Orthogonal integer translates: $\langle \phi(t), \phi(t-n)\rangle = \delta(n)$ and $\langle \phi(2^{-k}t), \phi(2^{-k}t - n)\rangle = 2^k\delta(n)$.
* **Wavelet Function $\psi(t)$ Orthogonality**:
  $$\int_{-\infty}^{\infty}\psi(t)\,dt = 0, \quad \int_{-\infty}^{\infty}|\psi(t)|^2\,dt = 1, \quad \langle\psi(t),\psi(t-n)\rangle = \delta(n), \quad \langle\psi(t),\phi(t-n)\rangle = 0$$
* **Two-Scale (Dilation) Equations**:
  $$\phi(t) = \sum_{n=-\infty}^{\infty} c(n)\,\phi(2t - n) \quad \left(\sum c(n) = 2\right), \qquad \psi(t) = \sum_{n=-\infty}^{\infty} d(n)\,\phi(2t - n) \quad \left(\sum d(n) = 0\right)$$
* **Approximation $a(k,n)$ & Detail $b(k,n)$ Coefficients**:
  $$f_k(t) = \sum_{n=-\infty}^{\infty} a(k,n)\,\phi(2^{-k}t - n) + \sum_{n=-\infty}^{\infty} b(k,n)\,\psi(2^{-k}t - n)$$
  $$a(k,n) = \sum_{m=-\infty}^{\infty} a(k-1,m)\,\frac{c(m-2n)}{2}, \qquad b(k,n) = \sum_{m=-\infty}^{\infty} a(k-1,m)\,\frac{d(m-2n)}{2}$$

---

## 5. Discrete Wavelet Transform (DWT) — Sub-band Coding

* **Filter Bank Concept**: Signal $x[n]$ passes through a half-band **low-pass filter $h[n]$** (scaling/approximation) and **high-pass filter $g[n]$** (wavelet/detail), then is **sub-sampled by 2** ($\downarrow 2$, keeping half the samples per Nyquist's rule):
  $$y_{\text{high}}[k] = \sum_n x[n]\,g[2k - n], \qquad y_{\text{low}}[k] = \sum_n x[n]\,h[2k - n]$$
* **Multi-Level Iteration**: Repeat filtering and $\downarrow 2$ on $y_{\text{low}}$ ($0\text{–}\pi/2 \to 0\text{–}\pi/4 \to 0\text{–}\pi/8 \dots$). At every level, sample count (time resolution) halves and frequency bandwidth halves (frequency resolution doubles).

---

## 6. 2D Image Decomposition & Reconstruction using DWT

* **One-Level 2D Decomposition (Filter Bank)**:
  1. Filter **rows** with LPF ($h$) and HPF ($g$), then downsample each by $2$ ($\downarrow 2$).
  2. Filter **columns** of both branches with LPF ($h$) and HPF ($g$), then downsample by $2$ ($\downarrow 2$).
  3. Produces **4 critically sub-sampled sub-bands** (octave-band decomposition):
     * **LL1**: Coarse **approximation** image (low-low).
     * **LH1, HL1, HH1**: **Detail** images (finest-scale horizontal, vertical, diagonal edges).
* **Multi-Level 2D DWT**: Only **LL1** is decomposed further into **LL2, HL2, LH2, HH2**, etc.
* **Image Reconstruction (Inverse DWT)**:
  * Upsample each sub-band ($\text{LL}_m, \text{LH}_m, \text{HL}_m, \text{HH}_m$) by $2$ ($\uparrow 2$), convolve with synthesis LPF ($\tilde{h}$) and HPF ($\tilde{g}$) along columns, sum pairs, upsample by $2$, convolve along rows with $\tilde{h}, \tilde{g}$, and sum to recover level $(m-1)$.
