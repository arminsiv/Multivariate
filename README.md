# Data Analysis Portfolio: Genomics & Clinical Outcomes

This repository contains a dual-part data analysis project exploring two distinct datasets: **ATAC-seq (Chromatin Accessibility)** and **Heart Failure Clinical Records**. The project demonstrates the application of various dimensionality reduction, regression, and classification techniques to extract meaningful biological and clinical insights.

---

## 🧬 Part 1: ATAC-seq Dataset Analysis
**Overview:** This dataset contains ~241,000 rows and 28 columns of chromatin peak data from treated and untreated samples. The goal is to explore chromatin accessibility patterns and predict treatment effects.

### Data Preprocessing
* **Cleaning:** Removed non-biological metadata (`PeakID`, `start`, `end`, `width`, `seqnames`, `strand`) to prevent noise from confusing the models.
* **Transformation:** Converted `Fold` and `FDR` to numeric types and dropped remaining NaNs, resulting in a clean matrix of 24 numeric biological features.

### Method 1: Principal Component Analysis (PCA)
PCA was utilized to reduce dimensionality and isolate the main drivers of variation.
* **Variance Explained:** 2 components captured **83.8%** of the total variance, indicating a very strong signal-to-noise ratio.
* **Biological Interpretation:**
  * **PC1 (78.3%):** Driven heavily by concentration variables (`Conc`, `Conc_NonTreatment`, `Conc_Treatment`).
  * **PC2 (5.3%):** Driven by differential features (`Fold` and `FDR`). 
* **Takeaway:** PCA provided a highly interpretable summary of the data, outperforming Factor Analysis for this specific dataset by neatly separating concentration metrics from treatment-induced fold changes.

### Method 2: Multiple Linear Regression (MLR)
Using the extracted Principal Components, an MLR model was trained to predict `Fold` change.
* **Performance:**
  * **Test R²:** 0.916
  * **CV Mean MSE:** 0.014
* **Insights:** PC2 was the strongest predictor of Fold change (coefficient: 0.3620). The train/test scores were nearly identical (0.917 vs 0.916), indicating a highly stable model with no significant overfitting. Minor heteroscedasticity in the residuals suggests a potential future need for target transformation.

### Method 3: Non-Metric Multidimensional Scaling (MDS)
To visualize group differences without overloading memory, a 2,000-row sample was analyzed. 
* **Setup:** A synthetic `Group` column (Treatment/Control) was introduced to test clustering visualization.
* **Results:** Achieved an acceptable stress level of **0.0643**. Density contour overlays revealed clear structural spread and underlying separation between the randomly assigned groups, establishing a robust pipeline for when actual group labels are introduced.

---

## 🫀 Part 2: Heart Failure Clinical Records Analysis
**Overview:** A dataset of 299 patient records with 13 clinical features. The objective is to predict patient survival (`DEATH_EVENT`) and uncover the underlying medical factors driving these outcomes.

### Method 1: Factor Analysis (FA)
FA was employed to discover latent structures and interpret clinical relationships.
* **Configuration:** Extracted 5 orthogonal factors.
* **Results:**
  * **Total Variance Explained:** 31.3%
  * **Strong Communalities:** `DEATH_EVENT` (0.70), `sex` (0.63), `time` (0.55).
  * **Weak Communalities:** `platelets` (0.04), `diabetes` (0.06).
* **Takeaway:** While the extracted factors (e.g., Prognosis, Cardiac/Renal function) are highly interpretable and clinically meaningful, the low overall variance explained suggests these factors shouldn't be used alone for predictive modeling.

### Method 2: Partial Least Squares Regression (PLSR)
PLSR was used to predict `DEATH_EVENT` while managing the multicollinearity of the 12 clinical features.
* **Results:** A conservative 3-component model yielded an R² of 0.997 and an AUC of 1.0 on the test set.
* **⚠️ Critical Evaluation:** These "perfect" scores are highly unnatural for complex biological/medical data. This serves as a massive red flag for **data leakage**. Future work requires auditing the pipeline to ensure the test set was strictly isolated during scaling and component selection, and verifying that `DEATH_EVENT` did not accidentally bleed into the predictor variables.

### Method 3: Linear Discriminant Analysis (LDA)
LDA was deployed to explicitly classify survival vs. death and explain the separation between the two groups.
* **Performance:** Overall accuracy of **83.3%**.
  * **Survivors:** 85% precision, 92% recall.
  * **Deaths:** 79% precision, 66% recall. (The model is appropriately cautious regarding false positives for death).
* **Clinical Drivers (Top LDA Coefficients):**
  | Feature | Coefficient | Clinical Implication |
  | :--- | :--- | :--- |
  | `time` | -1.717 | Longer monitoring times correlate strongly with survival. |
  | `ejection_fraction` | -0.925 | Better heart pumping capability reduces mortality risk. |
  | `serum_creatinine` | +0.832 | Worse kidney function (high creatinine) increases mortality risk. |
  | `creatinine_phosphokinase` | +0.374 | Muscle/heart damage markers increase mortality risk. |
* **Takeaway:** The LDA model is highly successful. Not only is it statistically sound, but its feature weights align perfectly with established medical literature (e.g., the interplay of heart and kidney health). This provides a highly interpretable scoring system that could theoretically aid clinicians in stratifying patient risk.

---

## 📌 Project Conclusion
This portfolio highlights a flexible approach to data analysis. For the high-dimensional genomic data, unsupervised learning (PCA) and dimensionality reduction yielded the clearest insights. For the clinical data, supervised classification (LDA) provided actionable, medically sound predictions. The project also demonstrates vital analytical rigor—specifically, the ability to look past "perfect" metrics (PLSR) to identify potential methodological flaws like data leakage.
