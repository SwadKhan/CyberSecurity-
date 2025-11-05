Here you go—complete, numbered answers based on your uploaded dataset.

(SGS501-hw1_2025-1.pdf)

# A) Summary of the data (2019, 23 cities)

- n = 23, mean = **179.52** L/(person·day), SD = **60.70**, median = **150**, min–max = **123–341**.  
- By income: **H (n=9)** mean = **216.44**, SD = 78.15; **L (n=14)** mean = **155.79**, SD = 30.45.  
- By location: **Sea (S, n=11)** mean = **198.18**, SD = 70.53; **Land (L, n=12)** mean = **162.42**, SD = 46.69.  
- Visuals: distribution histogram and boxplots (H vs L; S vs L) are shown above.

Normality checks (Shapiro–Wilk): overall **p=0.0011** (non-normal); H **p=0.385**, L **p=0.0377**; S **p=0.0678**, Land **p=0.0100**. With small samples, the t-tests below are generally robust, but this is noted.

# B) Test \( \mu = 165 \) L/(person·day) (two-sided)

1) Hypotheses:  
H₀: μ = 165; H₁: μ ≠ 165.

2) Small sample? Yes (n=23). Assumptions: independent observations; approximate normality (or use robustness of t-test).

3) Distribution & df: one-sample **t** with **df = 22**; two-tailed (we test “different from”).

4) Confidence intervals for the mean:  
- **90% CI:** [**157.79**, **201.26**]  
- **95% CI:** [**153.27**, **205.77**]  
- **99% CI:** [**143.84**, **215.20**]

5) Test result: \( t = 1.147 \), **p = 0.264** → **Fail to reject H₀**. The sample mean does not differ significantly from 165.

# C) Test \( \mu > 165 \) L/(person·day) (one-sided)

- \( t = 1.147 \), **p(one-sided) = 0.132** → **Fail to reject H₀**. Insufficient evidence the mean is above 165.

# D) Sea vs Land (two independent means, 95% confidence)

1) Hypotheses:  
H₀: μ_S − μ_L = 0; H₁: μ_S − μ_L ≠ 0.

2) Assumptions: independent samples; approximate normality in each group; equal variances if pooling. Shapiro: S **p=0.0678**, Land **p=0.0100**. Variances: Levene **p=0.115** → treat as **equal variances (pooled)**.

3) Method: **Pooled two-sample t-test** (equal variances).

4) Result: \( \bar x_S = 198.18 \), \( \bar x_L = 162.42 \) → difference **35.77**.  
\( t = 1.446 \), **df = 21**, **p = 0.163**; **95% CI for (S−L): [−15.67, 87.20]** → **No significant difference** between Sea and Land cities at 5%.

# E) High- vs Low-income cities (one-sided, 95% confidence; H > L)

- Hypotheses: H₀: μ_H ≤ μ_L; H₁: μ_H > μ_L.  
- Normality: H **p=0.385**, L **p=0.0377**. Variances: Levene **p=0.0007** → **unequal variances**.  
- Method: **Welch’s t-test** (unequal variances).  
- Means: \( \bar x_H = 216.44 \) (n=9), \( \bar x_L = 155.79 \) (n=14) → difference **60.66**.  
- Test: \( t = 2.223 \), **df ≈ 9.58**, **p(one-sided) = 0.0258** → **Reject H₀**.  
- 95% one-sided lower bound for (H−L): **≥ 10.98** L/(person·day).  
  (Two-sided 95% CI: [−0.51, 121.83], reported for completeness.)

---

If you want, I can export a clean Word/PDF report with the tables, plots, and these write-ups formatted for submission.