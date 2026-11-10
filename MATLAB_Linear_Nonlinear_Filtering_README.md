# MATLAB_Linear_Nonlinear_Filtering — Linear & Nonlinear Spatial Filtering in MATLAB

Summary of `MATLAB_Linear_Nonlinear_Filtering.pptx` (15 slides; based on Gonzalez, Woods & Eddins, *Digital Image Processing Using MATLAB*, Chapter 3, Sections 3.4 & 3.5).

---

## 1. Spatial Filtering Foundations

- **Neighborhood Processing Loop**: At each pixel $(x, y)$:
  1. Define center point $(x, y)$.
  2. Perform an operation on the pixels in the surrounding $m \times n$ neighborhood.
  3. Assign the result as the filter response at $(x, y)$.
  4. Slide the window to the next pixel and repeat.
- **Linear vs. Nonlinear**:
  - **Linear spatial filtering**: computes a sum of products between mask weights $w(s, t)$ and image intensities $f(x+s, y+t)$:
    $$g(x, y) = \sum_{s}\sum_{t} w(s, t)\, f(x+s, y+t)$$
    Odd mask dimensions ($3\times 3$, $5\times 5$, ...) provide a unique center pixel.
  - **Nonlinear spatial filtering**: any other neighborhood operation (e.g., `max`, `min`, `median`, geometric mean) without fixed linear weights.

---

## 2. Linear Spatial Filtering in MATLAB (`imfilter`, `fspecial`)

- **Correlation vs. Convolution**:
  - **Correlation** (MATLAB `imfilter` default): slides mask $w$ over $f$ without rotating $w$; order matters ($f \otimes w \ne w \otimes f$).
  - **Convolution**: rotates $w$ by $180^\circ$ before sliding; commutative and satisfies the sifting property.
  - Symmetric masks give identical results under both; standard `fspecial` masks are pre-rotated where needed so `imfilter`'s default `'corr'` produces the expected convolution output.
- **`imfilter` Syntax & Options**:
  ```matlab
  g = imfilter(f, w, filtering_mode, boundary_options, size_options)
  ```
  - `filtering_mode`: `'corr'` (default) or `'conv'`.
  - `boundary_options`: constant `P` (default `0`, which darkens borders near bright regions), `'replicate'`, `'symmetric'` (avoids dark border halos), or `'circular'`.
  - `size_options`: `'same'` (default) or `'full'`.
  - **Data-type pitfall**: `imfilter` computes in `double` and casts back to the class of `f`; `uint8` images clip out-of-range/negative values unless the mask is normalized or `f` is converted with `im2double`.
- **Standard Masks via `fspecial('type', parameters)`**:
  - `'average'` (box blur, $r \times c$), `'disk'` (circular blur of radius $r$), `'gaussian'` (size & $\sigma$), `'laplacian'` ($3 \times 3$ parameterized by $\alpha$), `'log'` (Laplacian of Gaussian), `'motion'` (linear motion blur), `'prewitt'` / `'sobel'` (gradient edge masks), `'unsharp'` ($3 \times 3$ unsharp-mask sharpener).
- **Worked Laplacian Sharpening Example**:
  ```matlab
  w = fspecial('laplacian', 0);   % [0 1 0; 1 -4 1; 0 1 0]
  f = im2double(f);               % preserve negative Laplacian responses
  g1 = imfilter(f, w, 'replicate');
  g2 = f - g1;                    % f - \nabla^2 f sharpens edges
  ```

---

## 3. Nonlinear & General Neighborhood Filtering (`nlfilter`, `colfilt`)

- **`nlfilter`**: evaluates a user function directly in 2-D; straightforward but slow.
- **`colfilt`**: reshapes every $m \times n$ sliding neighborhood into a column of matrix $A$ ($mn$ rows) and applies a vectorized function once across all columns—significantly faster:
  ```matlab
  g = colfilt(f, [m n], 'sliding', @fun, parameters)
  ```
  - **Explicit padding required**: unlike `imfilter`, `colfilt` does not pad automatically; call `padarray(f, [m n], 'replicate')` beforehand.
- **Worked Geometric Mean Example** ($\left(\prod_{(s,t)} f(s,t)\right)^{1/(mn)}$):
  ```matlab
  function v = gmean(A)
    mn = size(A, 1);
    v = prod(A, 1) .^ (1/mn);
  end
  f = padarray(f, [m n], 'replicate');
  g = colfilt(f, [m n], 'sliding', @gmean);
  ```

---

## 4. Order-Statistic & Median Filters (`ordfilt2`, `medfilt2`)

- **General Order-Statistic Filter (`ordfilt2`)**: sorts pixels selected by a binary `domain` mask and selects the `order`-th ranked pixel:
  ```matlab
  g = ordfilt2(f, order, domain)
  ```
  - **Min filter** (0th percentile): `ordfilt2(f, 1, ones(m, n))`
  - **Median filter** (50th percentile): `ordfilt2(f, median(1:m*n), ones(m, n))`
  - **Max filter** (100th percentile): `ordfilt2(f, m*n, ones(m, n))`
- **Dedicated Median Filter (`medfilt2`)**:
  ```matlab
  g = medfilt2(f, [m n], padopt)   % padopt: 'zeros' (default), 'symmetric', 'indexed'
  ```
  - Ideal for **salt-and-pepper (impulse) noise**: extreme black/white outliers fall at the ends of the sorted neighborhood and are discarded without blurring edges.
  - Using `'symmetric'` padding avoids dark border artifacts caused by default `'zeros'` padding.

---

## 5. Summary Comparison

| Category | Core Mechanism | MATLAB Functions | Typical Use Cases |
| :--- | :--- | :--- | :--- |
| **Linear** | Sum of products with fixed mask | `imfilter`, `fspecial` | Smoothing/blurring, Laplacian sharpening, Sobel/Prewitt gradients |
| **Nonlinear** | Arbitrary / rank function on neighborhood | `nlfilter`, `colfilt`, `ordfilt2`, `medfilt2` | Salt-and-pepper denoising, min/max filtering, geometric mean |
