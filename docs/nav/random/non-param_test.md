## 🌸 Q1
I want to do t-test for comparison of two sets of data. The first dataset are the reconstructed images from raw data including subject 1 to 4, with method1. The second dataset are reconstructed with the same raw data including subject 1 to 4, but with method 2. I want to check if the difference is significant or not, and I cannot promise the distribution is normal Gaussian. What should I do?
---
Since you cannot assume a normal Gaussian distribution for your datasets, it's better to use a **non-parametric test** to check for significant differences between the two methods.

### Steps:
1. **Choose a Non-Parametric Test**: 
   - Use the **Wilcoxon Signed-Rank Test** if the data are paired (e.g., the same subjects reconstructed with two different methods).
   - If the data are independent (e.g., different subjects for the two methods), use the **Mann-Whitney U Test**.

2. **Check Assumptions**:
   - **Wilcoxon Signed-Rank Test**: Assumes the data are paired and the differences are symmetrically distributed.
   - **Mann-Whitney U Test**: Does not assume symmetry but requires independent samples.

3. **Perform the Test**:
   - In Python, you can use `scipy.stats` to perform these tests.
     - **Wilcoxon Test**:
       ```python
       from scipy.stats import wilcoxon
       stat, p = wilcoxon(data_method1, data_method2)
       print(f"Statistic={stat}, p-value={p}")
       ```
     - **Mann-Whitney U Test**:
       ```python
       from scipy.stats import mannwhitneyu
       stat, p = mannwhitneyu(data_method1, data_method2)
       print(f"Statistic={stat}, p-value={p}")
       ```

4. **Interpret the Results**:
   - If the **p-value < 0.05**, you can reject the null hypothesis and conclude that the difference between the methods is statistically significant.
   - If **p-value ≥ 0.05**, there is no significant difference.

5. **For Robustness**:
   - If the datasets are small, consider performing a **permutation test** as a supplementary analysis to confirm the results.

Let me know if you need help coding this or further clarification!

## 🌸 Q2
Like I compare the union set of sub1_method1, sub2_method1,sub3_method1 and sub4_method1 with the other union set of sub1_method2, sub2_method2,sub3_method2 and sub4_method2.
---
If you're comparing the **union** of data from all subjects for Method 1 against the **union** for Method 2, the data are no longer paired (since they represent combined distributions). Here's how you can proceed:

### 1. **Combine the Data**:
   - Create two datasets:
     - `data_method1`: Concatenate all data for subjects 1 to 4 under Method 1.
     - `data_method2`: Concatenate all data for subjects 1 to 4 under Method 2.

### 2. **Non-Parametric Test**:
   Since the union sets are independent, use the **Mann-Whitney U Test**, which does not assume normality or paired data.

   ```python
   from scipy.stats import mannwhitneyu

   # Assuming data_method1 and data_method2 are your combined datasets
   stat, p = mannwhitneyu(data_method1, data_method2)
   print(f"Statistic={stat}, p-value={p}")

   if p < 0.05:
       print("Significant difference between Method 1 and Method 2")
   else:
       print("No significant difference between Method 1 and Method 2")
   ```

### 3. **Permutation Test (Optional)**:
   To increase robustness (especially for small or complex data), you can also run a **permutation test**. This involves randomly shuffling the labels (`method1` and `method2`) many times and comparing the observed test statistic to the distribution of shuffled statistics.

   ```python
   from scipy.stats import permutation_test

   def statistic(x, y):
       return abs(x.mean() - y.mean())

   result = permutation_test((data_method1, data_method2), statistic, permutation_type='independent', n_resamples=10000)
   print(f"P-value: {result.pvalue}")

   if result.pvalue < 0.05:
       print("Significant difference based on permutation test")
   else:
       print("No significant difference based on permutation test")
   ```

### 4. **Key Considerations**:
   - Be cautious when interpreting the results if the sample sizes for the union sets are very different (e.g., if some subjects contribute significantly more data than others).
   - Ensure your `union sets` are balanced to avoid bias, especially if one method generates substantially more or fewer values.

This approach allows you to determine whether the reconstructed images' quality metrics differ significantly between the two methods when considering all subjects collectively. Let me know if you need further assistance!

## 🌸 Q3
They are not paired? really? but if I compare sub1_method1 with sub1_method2, it should be paired?
---
You're absolutely correct—if you're comparing **subject-wise results** (e.g., `sub1_method1` with `sub1_method2`, `sub2_method1` with `sub2_method2`, etc.), then the data are **paired**. Pairing arises because the same subject's data are reconstructed using two methods, which introduces a natural correspondence.

However, when you **union** the datasets for all subjects (combining them into a single dataset for Method 1 and Method 2), you lose this pairing, and the data should be treated as **independent** unless you analyze them subject-by-subject.

### How to Proceed:
1. **Subject-Wise Paired Test (Preferred)**:
   If you want to account for the pairing, you should calculate a single summary metric (e.g., mean, median, standard deviation, etc.) for each subject's data under each method and then compare those metrics **subject-wise**. 

   - Example: Compute the mean SNR for `sub1_method1` and `sub1_method2`, then for `sub2`, etc., resulting in paired values for each subject.

   - Use the **Wilcoxon Signed-Rank Test** for these paired metrics:
     ```python
     from scipy.stats import wilcoxon

     # Example subject-wise metrics
     metrics_method1 = [sub1_method1_mean, sub2_method1_mean, sub3_method1_mean, sub4_method1_mean]
     metrics_method2 = [sub1_method2_mean, sub2_method2_mean, sub3_method2_mean, sub4_method2_mean]

     stat, p = wilcoxon(metrics_method1, metrics_method2)
     print(f"Statistic={stat}, p-value={p}")

     if p < 0.05:
         print("Significant difference between Method 1 and Method 2")
     else:
         print("No significant difference between Method 1 and Method 2")
     ```

2. **Union of All Data (Independent Test)**:
   If you choose to compare the union of all data for all subjects between Method 1 and Method 2, the test is no longer paired because the subject correspondence is lost. In this case, use the **Mann-Whitney U Test** as described earlier.

### Key Recommendation:
Use a **subject-wise paired test** whenever possible because it respects the pairing inherent in the experimental design. This approach is statistically more powerful and avoids conflating inter-subject variability with the effect of the reconstruction method. Only use the union-based approach if there's a compelling reason to analyze the data in aggregate form without considering subject correspondence.

Let me know if you'd like more clarification or help coding this!

## 🌸 Q4
If I use paired test, subject by subject, I get 4 sets of wilcocon test. How can I combine the 4 sets then?
---
When you perform **subject-wise paired Wilcoxon tests** for each subject individually, you end up with four separate p-values (one for each subject). To combine these results and assess the overall significance across all subjects, you can use **meta-analysis techniques** designed for combining p-values. Here are the common methods:



### 1. **Fisher's Combined Probability Test**:
   This method combines p-values into a single test statistic, which follows a chi-square distribution.

```python

import numpy as np
from scipy.stats import combine_pvalues

# Example p-values from subject-wise Wilcoxon tests
p_values = [p1, p2, p3, p4]

# Combine p-values using Fisher's method
stat, combined_p = combine_pvalues(p_values, method='fisher')
print(f"Fisher's combined statistic: {stat}, Combined p-value: {combined_p}")

if combined_p < 0.05:
print("Overall significant difference between methods")
else:
print("No overall significant difference between methods")
```



### 2. **Stouffer’s Z-Score Method**:
   This method weights p-values based on sample sizes (if sample sizes differ across subjects). 

```python
from scipy.stats import norm
# Example p-values and sample sizes for each subject
p_values = [p1, p2, p3, p4]
sample_sizes = [n1, n2, n3, n4]  # Number of samples per subject

# Calculate Z-scores from p-values
z_scores = [norm.ppf(1 - p) for p in p_values]

# Weighted Z-score
weights = np.sqrt(sample_sizes)
combined_z = np.sum(weights * z_scores) / np.sqrt(np.sum(weights**2))

# Convert back to p-value
combined_p = 1 - norm.cdf(combined_z)
print(f"Combined Z-score: {combined_z}, Combined p-value: {combined_p}")

if combined_p < 0.05:
print("Overall significant difference between methods")
else:
print("No overall significant difference between methods")


```

---

### 3. **Bonferroni Correction**:
   Adjust the threshold for significance to account for multiple comparisons. For 4 tests, divide the alpha level (e.g., 0.05) by 4.

```python

p_values = [p1, p2, p3, p4]
corrected_alpha = 0.05 / len(p_values)

significant_tests = [p < corrected_alpha for p in p_values]
print(f"Significant tests after Bonferroni correction: {significant_tests}")

if any(significant_tests):
print("At least one method difference is significant after correction")
else:
print("No significant differences after correction")
```

---

### Recommendation:
- Use **Fisher’s method** if you want a simple and commonly used approach to combine p-values.
- Use **Stouffer’s Z-score method** if you have varying sample sizes across subjects.
- Use **Bonferroni correction** if you want a conservative approach to control for multiple comparisons.

Let me know if you need further clarification or help implementing these methods!
