# Image_enhancement_DIP_2025_Final — Spatial & Frequency-Domain Image Enhancement

Summary of `Image_enhancement_DIP_2025_Final.pptx` (232 slides; R. Malmathanraj, NIT Tiruchirappalli, incorporating slides from Dr. Hasan Demirel and Gonzalez & Woods).

---

## 1. Foundations & Image Quality Metrics

- **Enhancement vs. Restoration**:
  - **Image enhancement**: subjective, heuristic/ad-hoc cosmetic improvement of brightness, contrast, or feature visibility for human observers or downstream analysis; does not increase inherent information content.
  - **Image restoration**: objective, mathematically formulated recovery requiring degradation modeling.
- **Two Domains**: **Spatial domain** (direct pixel manipulation, $g(x,y) = T[f(x,y)]$) and **Frequency domain** (modifying the 2D Fourier transform).
- **Contrast & Brightness Metrics**:
  - Contrast ratio: $C = \frac{f_{\text{object}} - f_{\text{background}}}{f_{\text{object}} + f_{\text{background}}}$; Weber contrast: $C_{\text{Weber}} = \frac{f_{\text{object}} - f_{\text{background}}}{f_{\text{background}}}$; Michelson contrast: $C_{\text{Michelson}} = \frac{f_{\max} - f_{\min}}{f_{\max} + f_{\min}}$; log ratio: $C = \log_{10}\!\left(\frac{f_{\text{object}}}{f_{\text{background}}}\right)$.
  - Contrast manipulation: $g(x,y) = a \cdot f(x,y)$; brightness manipulation: $g(x,y) = f(x,y) \pm b$.
  - **Dynamic range**: $f_{\max} - f_{\min}$ (or $20\log(f_{\max} - f_{\min})\text{ dB}$).
  - **MSE, SNR, PSNR**: $\text{MSE} = \frac{1}{MN}\sum\sum [f(x,y) - \hat{f}(x,y)]^2$; PSNR for 8-bit images (slide 20 notation): $\text{PSNR} = 20\log_{10}\frac{255^2 MN}{\sum_{i=0}^{M-1}\sum_{j=0}^{N-1}[f(x,y)-\hat{f}(x,y)]^2}\text{ dB}$ (high SNR/PSNR does not always guarantee subjective clarity).

---

## 2. Point Processing & Intensity Transformations ($s = T(r)$)

- **Image Negative**: $s = L - 1 - r$ for $r \in [0, L-1]$.
- **Log Transformation**: $s = c\log(1 + r)$; expands dark intensities and compresses wide dynamic ranges (e.g., displaying Fourier magnitude spectra spanning $0$ to $1.5\times 10^6$).
- **Power-Law (Gamma) Transformation**: $s = c\,r^\gamma$ ($c, \gamma > 0$).
  - $\gamma < 1$ expands dark regions (e.g., MRI spine); $\gamma > 1$ expands bright regions / darkens washed-out aerial images.
  - **Gamma correction**: CRT displays have power-law voltage-to-intensity exponents $\approx 1.8\text{–}2.5$, corrected prior to display via $s = r^{1/2.5} = r^{0.4}$.
  - MATLAB: `im1 = mat2gray(double(im).^0.3);`.
- **Piecewise Linear Transformations**:
  - **Contrast stretching**: piecewise mapping via control points $(r_1, s_1), (r_2, s_2)$; $(r_1,s_1)=(r_{\min},0), (r_2,s_2)=(r_{\max},L-1)$ stretches full range; $r_1=r_2, s_1=0, s_2=L-1$ reduces to binary thresholding. MATLAB: `imadjust(im, [0.2 0.5], [0 1])`, `im2bw(im, 0.42)`.
  - **Intensity-level slicing**: highlights a target band $[A, B]$ either while preserving background or zeroing out the background.
  - **Bit-plane slicing**: decomposes 8-bit pixels into 8 binary planes (MSBs carry structural appearance; LSBs carry fine/noisy variation).
- **Image Averaging**: Averaging $K$ aligned realizations $g_i(x,y) = f(x,y) + \eta_i(x,y)$ with uncorrelated zero-mean noise yields $E\{\bar{g}(x,y)\} = f(x,y)$ and variance $\sigma_{\bar{g}}^2(x,y) = \frac{1}{K}\sigma_\eta^2(x,y)$ (standard deviation scales as $1/\sqrt{K}$). Alignment/registration is required to avoid blur.

---

## 3. Histogram Modification, Equalization & Matching (Gray & Color)

- **Histogram Stretch, Shrink & Slide**:
  - **Stretch**: $\text{Stretch}(I(r,c)) = \left[\frac{I(r,c)-I_{\min}}{I_{\max}-I_{\min}}\right][\text{MAX}-\text{MIN}] + \text{MIN}$ (optional 1% tail clipping prevents outliers from compressing contrast).
  - **Shrink**: $\text{Shrink}(I(r,c)) = \left[\frac{\text{Shrink}_{\max}-\text{Shrink}_{\min}}{I_{\max}-I_{\min}}\right][I(r,c)-I_{\min}] + \text{Shrink}_{\min}$ (used as a step inside unsharp masking).
  - **Slide**: $\text{Slide}(I(r,c)) = I(r,c) + \text{OFFSET}$ (clipped to valid bounds; shifts brightness).
- **Histogram Equalization (HE)**:
  - Remaps intensities using the normalized cumulative distribution (CDF) scaled by $L-1$ (4 steps: running sum $\to$ normalize by total pixels $\to$ multiply by $L-1$ & round $\to$ build LUT). Produces the flattest discrete distribution achievable, though rarely perfectly uniform.
  - **Color Image HE**:
    - Equalizing R, G, B bands independently causes hue/color shifts.
    - Better color preservation: (1) equalize one dominant band or **luminance** ($L = 0.299R + 0.587G + 0.114B$, vs. value $V = (R+G+B)/3$) and scale RGB proportionally or apply the luminance LUT across bands, or (2) transform to HSL, modify lightness, and invert.
- **Histogram Specification (Matching)**:
  - Remaps discolored image $I_d$ to match a reference/desired histogram or continuous function (e.g., $\exp$, $\log$) via **percentile/CDF matching** ($z = G^{-1}(T(r))$), efficiently implemented via a 256-entry LUT (`K = uint8(LUT(I+1))`).

---

## 4. Homomorphic Filtering

- Models an image as multiplicative **illumination** (slowly varying, low-frequency) and **reflectance** (rapidly varying at object boundaries, high-frequency):
  $$f(x,y) = i(x,y)\cdot r(x,y) \xrightarrow{\ln} g(x,y) = \ln i(x,y) + \ln r(x,y) \xrightarrow{\mathcal{F}} G(u,v) = I(u,v) + R(u,v)$$
- Applying a frequency-domain filter that attenuates low frequencies and amplifies high frequencies simultaneously normalizes uneven illumination and boosts local contrast/detail.

---

## 5. Spatial Filtering: Smoothing, Denoising & Sharpening

- **Correlation vs. Convolution**:
  - Correlation: $g(x,y) = \sum_{s=-a}^a \sum_{t=-b}^b w(s,t)f(x+s, y+t)$ (no mask rotation).
  - Convolution: rotates the mask $180^\circ$ before sliding ($f(x-s, y-t)$).
- **Smoothing & Edge-Preserving Filters**:
  - **Averaging (Box) & Weighted Average / Gaussian filters**: $3\times 3$, $5\times 5$, etc. reduce random Gaussian noise but blur edges.
  - **Directional smoothing**: averages along candidate angles $\theta$ and picks $\theta^*$ minimizing $|y(m,n) - v(m,n:\theta^*)|$ to avoid blurring across edges.
  - **Conservative smoothing**: clamps center pixel to the $[\min, \max]$ range of its 8-neighborhood.
  - **Non-linear / Restoration Mean Filters**:
    - *Arithmetic mean*, *Geometric mean* ($\left[\prod g\right]^{1/N^2}$, better detail retention on Gaussian noise), *Harmonic mean* (removes salt noise, fails on pepper), *Contraharmonic mean* (order $R$: $R<0$ removes salt, $R>0$ removes pepper), and *$Y_p$ Mean* ($\left[\sum d(r,c)^P / N^2\right]^{1/P}$, $P<0$ removes salt, $P>0$ removes pepper).
  - **Median filter**: removes salt-and-pepper noise while preserving edges ($3\times 3$ may miss dense impulses; larger $5\times 5$ windows remove more noise at the cost of a "painted"/blurred effect).
- **Sharpening & Gradient Filters**:
  - **Digital Derivatives**: 1st order $f(x+1)-f(x)$ (nonzero along ramps); 2nd order $f(x+1)+f(x-1)-2f(x)$ (zero along constant ramps, nonzero at step/ramp endpoints, enhancing fine detail).
  - **2D Laplacian**: $\nabla^2 f = [f(x+1,y)+f(x-1,y)+f(x,y+1)+f(x,y-1)] - 4f(x,y)$ (4-neighbor isotropic at $90^\circ$; 8-neighbor diagonal extension with $-8$ center isotropic at $45^\circ$).
  - **Sobel Gradient Masks**: $3\times 3$ horizontal $\begin{bmatrix}-1&-2&-1\\0&0&0\\1&2&1\end{bmatrix}$ and vertical $\begin{bmatrix}-1&0&1\\-2&0&2\\-1&0&1\end{bmatrix}$ masks (weight 2 provides smoothing).
  - **High-Boost & Unsharp Masking**:
    - High-boost spatial mask: $\begin{bmatrix}0&0&0\\0&A&0\\0&0&0\end{bmatrix} + \frac{1}{9}\begin{bmatrix}-1&-1&-1\\-1&-1&-1\\-1&-1&-1\end{bmatrix}$.
    - Unsharp masking: blur $f \to \bar{f}$, $\text{mask} = f - \bar{f}$, output $g(x,y) = f(x,y) + k \cdot \text{mask}$.

---

## 6. Frequency-Domain Enhancement

- **2D DFT Properties**:
  - Multiplying $f(x,y)$ by $(-1)^{x+y}$ shifts the origin to $(M/2, N/2)$; $F(0,0) = \frac{1}{MN}\sum\sum f(x,y)$ is the average/DC component.
  - Image rotation rotates the spectrum identically; spatial lines/edges produce perpendicular lines in the spectrum.
- **6-Step Filtering Pipeline**: multiply by $(-1)^{x+y}$ $\to$ DFT $F(u,v)$ $\to$ multiply $G(u,v)=H(u,v)F(u,v)$ $\to$ inverse DFT $\to$ take real part $\to$ multiply by $(-1)^{x+y}$.
- **Lowpass (Smoothing) Filters** (where $D(u,v)$ is distance from center, $D_0$ is cutoff):
  1. **Ideal LPF (ILPF)**: $H=1$ for $D(u,v)\le D_0$, $0$ otherwise ($D_0$ often chosen by enclosed power percentage $\alpha$). Spatial inverse $h(x,y)$ has a central peak (blurring) and concentric circular lobes (severe **ringing artifacts**).
  2. **Butterworth LPF (BLPF)**: $H(u,v) = \dfrac{1}{1 + [D(u,v)/D_0]^{2n}}$. Order $n=1$ has **no ringing**; $n \ge 2$ introduces increasing ringing with higher $n$.
  3. **Gaussian LPF (GLPF)**: $H(u,v) = e^{-D^2(u,v)/(2D_0^2)}$. Inverse FT is also Gaussian $\implies$ **zero ringing** (preferred in medical imaging).
- **Highpass (Sharpening) Filters** ($H_{\text{hp}}(u,v) = 1 - H_{\text{lp}}(u,v)$):
  1. **Ideal HPF (IHPF)**: $0$ for $D \le D_0$, $1$ otherwise (ringing at low $D_0$).
  2. **Butterworth HPF (BHPF)**: $H(u,v) = \dfrac{1}{1 + [D_0 / D(u,v)]^{2n}}$ (smoother, minimal ringing).
  3. **Gaussian HPF (GHPF)**: $H(u,v) = 1 - e^{-D^2(u,v)/(2D_0^2)}$ (smooth sharpening with **no ringing**).
- **Notch Filter (DC Removal)**: setting $H(M/2, N/2) = 0$ and $1$ elsewhere removes the average DC component.
