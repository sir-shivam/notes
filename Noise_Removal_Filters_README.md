# Noise_Removal_Filters — Spatial & Frequency-Domain Image Restoration in MATLAB

Summary of `Noise_Removal_Filters.pptx` (11 slides; MATLAB Image Processing Toolbox restoration workflows for random, impulse, and periodic noise).

---

## 1. Overview of Noise-Removal Families

- **Spatial-Domain Filters**: Operate on a local $m \times n$ neighborhood $S_{xy}$; suited to random (Gaussian, uniform) and impulse (salt-and-pepper) noise.
- **Frequency-Domain Filters**: Operate on the 2-D Fourier spectrum (`fft2`, `fftshift`, `ifft2`, `meshgrid`); suited to structured periodic/sinusoidal interference (e.g., scan lines) that appears as localized bright spikes in the spectrum.

---

## 2. Spatial Mean Restoration Filters

| Filter | Formula over $(s,t) \in S_{xy}$ | Target Noise & Behavior | MATLAB Implementation |
| :--- | :--- | :--- | :--- |
| **Arithmetic Mean** | $\hat{f}(x,y) = \dfrac{1}{mn}\sum g(s,t)$ | Gaussian / uniform noise; box smoothing blurs edges proportionally to window size | `w = fspecial('average', [m n]); g = imfilter(fn, w, 'replicate');` |
| **Geometric Mean** | $\hat{f}(x,y) = \left[\prod g(s,t)\right]^{1/mn}$ | Gaussian noise; comparable smoothing to arithmetic mean with better detail preservation (add `eps` to avoid $\log 0$) | `avgLog = imfilter(log(fd + eps), ones(m,n)/(m*n), 'replicate'); g = exp(avgLog);` |
| **Contraharmonic Mean** | $\hat{f}(x,y) = \dfrac{\sum g(s,t)^{Q+1}}{\sum g(s,t)^Q}$ | Single-polarity impulse noise: $Q > 0$ (e.g., $1.5$) removes **pepper**; $Q < 0$ (e.g., $-1.5$) removes **salt**; $Q=0 \to$ arithmetic mean; $Q=-1 \to$ harmonic mean. Fails if both salt and pepper coexist | `num = imfilter(fd.^(Q+1), ones(m,n), 'replicate'); den = imfilter(fd.^Q, ones(m,n), 'replicate'); g = num ./ den;` |

---

## 3. Spatial Order-Statistic Filters

| Filter | Formula over $(s,t) \in S_{xy}$ | Target Noise & Behavior | MATLAB Implementation |
| :--- | :--- | :--- | :--- |
| **Median** | $\hat{f}(x,y) = \operatorname{median}\{g(s,t)\}$ | **Salt-and-pepper noise** (both polarities); discards extreme outliers while preserving edges; can be iterated for denser noise | `g = medfilt2(fn, [3 3], 'symmetric');` |
| **Max** | $\hat{f}(x,y) = \max\{g(s,t)\}$ | **Pepper-only** (dark) noise; 100th percentile rank filter (tends to brighten image) | `gMax = ordfilt2(fn, m*n, ones(m, n));` |
| **Min** | $\hat{f}(x,y) = \min\{g(s,t)\}$ | **Salt-only** (bright) noise; 0th percentile rank filter (tends to darken image) | `gMin = ordfilt2(fn, 1, ones(m, n));` |
| **Midpoint** | $\hat{f}(x,y) = \dfrac{1}{2}\big(\max\{g(s,t)\} + \min\{g(s,t)\}\big)$ | Symmetric random noise (**Gaussian / uniform**); unsuitable for impulse noise since one impulse dominates max/min | `g = (double(gMax) + double(gMin)) / 2;` |

---

## 4. Frequency-Domain Periodic Noise Removal

Periodic interference (e.g., sinusoidal or scan-line artifacts) concentrates into sharp spikes at symmetric frequencies $\pm(u_0, v_0)$ in the centered Fourier spectrum.

- **Workflow**: Compute padded centered DFT (`fftshift(fft2(double(f), 2*M, 2*N))`) $\to$ inspect log-magnitude spectrum to find spike coordinates or radial band $\to$ multiply by filter $H(u, v)$ $\to$ invert (`real(ifft2(fftshift(...)))`) and crop to $M \times N$.
- **Butterworth Bandreject Filter** (suppresses radial frequency band of width $W$ centered at radius $D_0$, order $n$):
  $$H_{\text{br}}(u, v) = \frac{1}{1 + \left[\dfrac{D(u,v)\,W}{D(u,v)^2 - D_0^2}\right]^{2n}}, \qquad D(u,v) = \sqrt{u^2 + v^2}$$
- **Butterworth Notch Reject Filter** (surgically rejects conjugate spike pair at $\pm(u_0, v_0)$ with cutoff $D_{0n}$):
  $$H_{\text{notch}}(u, v) = \frac{1}{1 + (D_{0n}/D_k)^{2n}} \cdot \frac{1}{1 + (D_{0n}/D_{-k})^{2n}}$$
  where $D_k = \sqrt{(u - u_0)^2 + (v - v_0)^2}$ and $D_{-k} = \sqrt{(u + u_0)^2 + (v + v_0)^2}$.
