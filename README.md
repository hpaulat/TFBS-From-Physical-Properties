# Prediction of Transcription Factor Binding Sites Using DNA Physical Properties

This repository implements an interpretable machine-learning pipeline for predicting transcription factor binding site (TFBS) occupancy using DNA physical shape features rather than sequence motifs or large deep-learning models.

The core idea is simple: local DNA structure encodes biologically meaningful signals for TF binding, and lightweight models can exploit these signals effectively without becoming black boxes.

This work was developed as part of a COMP561 project at McGill University and is described in detail in the accompanying paper: Prediction of TF Binding Based on DNA Physical Properties paper.

### What This Repository Does

At a high level, the pipeline:

- Maps TF binding sites to regulatory regions on human Chromosome 1 using BEDTools
- Extracts fixed-length genomic windows (101 bp) around TF binding events
- Computes DNA shape features (MGW, ProT, Roll, Opening, Buckle) and GC content
- Builds balanced, fixed-length training examples
- Trains interpretable classifiers (Logistic Regression and a small MLP)
- Evaluates performance under strong class imbalance using balanced accuracy and ROC-AUC
- Assesses robustness via bootstrapping and threshold tuning
- The result is a practical, efficient TFBS screening approach that achieves competitive performance while remaining kbiologically interpretable.

### How Do I Run It?

Prerequisites:

- Python ≥ 3.9
- BEDTools
- NumPy, Pandas, scikit-learn, Matplotlib
- DNA shape tracks generated via DNAshapeR

High-Level Workflow:

```
BED files (TFBS + regulatory regions)
↓
BEDTools intersect (Chromosome 1)
↓
Sliding-window extraction (101 bp)
- build prefix-sum arrays for each DNA shape track
↓
DNA shape feature aggregation
- single dataframe
↓
Feature scaling + class balancing
↓
Model training + evaluation
- scikit-learn pipelines
```

### How Did I Build It?

Data Design Choices:

- Fixed window size (101 bp) to eliminate length confounds
- Strict feature coverage intersection to ensure consistency across shape tracks
- Class-imbalance handling via inverse-frequency weighting
- Chromosome-level restriction (Chr1) to control genomic heterogeneity

Each window is summarized by:

- Minor Groove Width (MGW)
- Propeller Twist (ProT)
- Roll
- Opening
- Buckle
- GC Content

To keep computation tractable at scale, prefix-sum arrays are used to compute window means in constant time.

Logistic Regression Specifications

- L2 regularization
- LBFGS solver
- Threshold tuning to maximize balanced accuracy

Multilayer Perceptron Specifications:

- ReLU activations
- Hidden layers: (64, 64)
- Early stopping + L2 regularization

Both models are trained using stratified cross-validation and evaluated using balanced accuracy and ROC AUC, which are robust to class imbalance.

### Results:

Logistic Regression:

- Balanced Accuracy ≈ 0.74
- ROC-AUC ≈ 0.83

MLP:

- Balanced Accuracy ≈ 0.75

* ROC-AUC ≈ 0.834'

Performance parity between linear and non-linear models suggests that current limitations stem from feature aggregation, not model capacity.

### Lessons Learned

Interpretability still matters: simple models can be competitive when features are well-designed. GC content dominates global models, but this likely masks TF-family-specific structural preferences. Aggregating all TFs into a single class causes signal cancellation. Flanking-region structure likely matters more than window-averaged features. Balanced accuracy is essential when working with genomic class imbalance.

### Future Directions

1. Cluster TFs into structural families instead of modeling them jointly
2. Use position-resolved shape features rather than window means
3. Extend beyond Chromosome 1
4. Compare against sequence-only baselines directly

### Data & References

Most datasets used are publicly available via UCSC Genome Browser (hg19) and DNAshapeR. Full methodological details and citations are provided in the paper
