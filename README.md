# Density Adaptive Real-Time Crowd People Counting Network (DA-CSRNet)

> Density-adaptive crowd counting for CCTV-like scenes: CSRNet + Self-Attention + Count Calibration (α)

<img width="514" height="286" alt="image" src="https://github.com/user-attachments/assets/fbe0506a-8a5d-468e-b067-632039ed3ead" />

---

## Overview
This project implements a **density-adaptive real-time crowd counting** approach that improves robustness when the **crowd density distribution** in test scenes differs from training scenes.

While **CSRNet** achieves strong counting performance with a simple architecture (without multi-scale branches), it can suffer performance degradation under **highly congested** or **density-shifted** conditions.  
To address this, we propose a system that:
1) augments CSRNet with **Self-Attention** to better capture global context and occlusion patterns, and  
2) introduces an additional **count calibration network** that predicts a multiplicative factor **α(X; φ)** to adjust the final count.

---

## Key Idea (Intuition)
Crowd counting models often output a **density map** and derive the final count by summing it.
However, when density changes drastically (e.g., sparse ↔ highly congested), the same model may systematically **over/under-estimate**.

We treat this as a **density-adaptation problem** and learn an additional lightweight network that looks at image features and outputs **α(X)** to rescale the count:
- baseline count: `C`
- calibrated count: `α(X) · C`

---

## Method Summary

### 1) CSRNet Backbone (Density Map Prediction)
Given an input image `X_i`, CSRNet predicts a density map:
- predicted density map: `Z(X_i; θ)`
- ground-truth density map: `Z_i^{GT}`

Training objective for CSRNet is the density-map regression loss:

$$
L_1(\theta) = \frac{1}{2N}\sum_{i=1}^{N} \left\| Z(X_i;\theta) - Z_i^{GT} \right\|_{2}^{2}
$$

---

### 2) Self-Attention Module (Context Modeling)
We add **Self-Attention** so that the model can attend to:
- surrounding patterns,
- global layout,
- occlusion relationships in dense scenes.

This helps detect heads/people patterns more reliably when targets are heavily overlapped.

---

### 3) Count Calibration Network (Density-Adaptive Scaling)
We introduce a CNN-based calibration network that outputs:

- scaling factor: `α(X_i; φ)`
- calibrated count: `α(X_i; φ) · C_i`

The calibration network is trained using:

$$
L_2(\varphi) = \frac{1}{N}\sum_{i=1}^{N} \left| \alpha(X_i;\varphi)\cdot C_i - C_i^{GT} \right|
$$

Where the CSRNet-derived count `C_i` is computed by summing the predicted density map:

$$
C_i = \sum_{l=1}^{L}\sum_{w=1}^{W} Z_{l,w}
$$

So the final predicted crowd count becomes:

- **Final Count**: `α(X_i; φ) · C_i`

---

## Evaluation

### Dataset
- **Shanghai dataset** (as described in the paper)
- Total people instances: **330,165**
- Total labeled images: **1,198**

### Metric: MAE
Baseline CSRNet MAE:

$$
MAE = \frac{1}{N_{test}}\sum_{i=1}^{N} \left| C_i - C_i^{GT} \right|
$$

Proposed model MAE (using calibrated count):

$$
MAE = \frac{1}{N_{test}}\sum_{i=1}^{N} \left| \alpha(X_i;\varphi)C_i - C_i^{GT} \right|
$$

---

## Reported Result (from the paper)
On **ShanghaiTech Part A** test set:
- CSRNet: **MAE 68.42**
- Proposed (DA-CSRNet): **MAE 67.98**
- Improvement: **−0.44 MAE**

> Note: This README reports the numbers as stated in the paper; reproduction may vary depending on implementation details, preprocessing, and training setup.

---

## Roadmap
- [ ] Add training pipeline (CSRNet + Self-Attention)
- [ ] Add calibration network training (α)
- [ ] Add evaluation scripts (MAE, density map visualization)
- [ ] Provide pretrained weights & reproducibility notes

---

## Citation
If you use this work, please cite the original paper (fill with your final bib entry):

```bibtex
@inproceedings{kim2023densityadaptive,
  title     = {Density Adaptive Real-Time Crowd People Counting Network},
  author    = {Kim, Jun-Hee and Kim, Chae-Ho and Kim, Dae-Seok and Lee, Suk-Ho},
  booktitle = {Proceedings of the Korea Multimedia Society (Autumn Conference)},
  year      = {2023}
}
