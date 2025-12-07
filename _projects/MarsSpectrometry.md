---
layout: page
title: Mars Spectrometry
description: Machine Learning Pipeline for Detecting Biosignatures in Planetary Mass Spectrometry Data
img: assets/img/projects/NASA/NASA.png
importance: 4
category: educational
related_publications: false
---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/NASA/NASA.png" title="Mass Spectrometry Data" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Analysis of mass spectrometry data from the SAM instrument on the Curiosity Rover.
</div>

## Abstract

**Mars Spectrometry** is a data science project developed for the DrivenData competition "Mars Spectrometry: Detect Evidence of Past Life". The objective was to build a machine learning model capable of analyzing evolved gas analysis (EGA) mass spectrometry data to detect the presence of specific chemical compounds relevant to astrobiology. The project implements a robust pipeline for high-dimensional data processing, feature extraction, and classification, achieving high accuracy in identifying potential biosignatures amidst the noise of planetary soil samples.

## Methodology

The solution follows a standard data science workflow, optimized for the specific characteristics of spectral data.

### Data Preprocessing
The raw mass spectrometry data consists of time-series intensity values for various mass-to-charge (m/z) ratios.
*   **Standardization**: Applied Z-score normalization to handle varying signal intensities across different samples.
*   **Dimensionality Reduction**: Utilized **Principal Component Analysis (PCA)** to reduce the feature space while retaining 95% of the variance, effectively filtering out sensor noise and focusing on the principal chemical signatures.

### Model Selection
A variety of supervised learning algorithms were evaluated using **Grid Search Cross-Validation** to optimize hyperparameters.
*   **Support Vector Machines (SVM)**: Effective for high-dimensional spaces.
*   **Logistic Regression**: Used as a baseline for interpretability.
*   **Random Forests**: Employed to capture non-linear relationships in the spectral data.

## Implementation Details

The pipeline was implemented in **Python** using the **Scikit-learn** and **Pandas** libraries.

```python
# Snippet: Pipeline Construction
from sklearn.pipeline import Pipeline
from sklearn.decomposition import PCA
from sklearn.svm import SVC

pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('pca', PCA(n_components=0.95)),
    ('classifier', SVC(kernel='rbf', C=10, gamma='scale'))
])

pipeline.fit(X_train, y_train)
```

## Key Results

The final model demonstrated robust performance in distinguishing between samples containing biological analogs and inert control samples. The use of PCA proved critical in handling the "curse of dimensionality" inherent in mass spectrometry data, significantly improving the model's generalization capabilities.

## Resources

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
    <div class="repo p-2 text-center">
        <a href="https://github.com/DanielRossi1/MarsSpectrometry" class="btn btn-primary z-depth-1">
            <i class="fab fa-github"></i> View Code
        </a>
    </div>
</div>
