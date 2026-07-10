
# Smoke, Flame & Cigarette Detection for Indoor Environments

Multi-class object detection system for identifying **fire, smoke, and cigarettes** in images and video captured in indoor (and outdoor) environments, built with **YOLOX**.

> Computer Vision and Deep Learning course project — A.A. 2022/2023, Università Politecnica delle Marche
> Authors: Nicola Mochi, Umberto Maraglino

[![Python](https://img.shields.io/badge/Python-3.x-blue)]()
[![PyTorch](https://img.shields.io/badge/PyTorch-YOLOX-orange)]()
[![License](https://img.shields.io/badge/License-MIT-green)]()

---

## Demo



https://github.com/user-attachments/assets/e9112f3a-ec9d-4e9c-8d74-a01bc80988fd



https://github.com/user-attachments/assets/8382c547-803e-47b5-9733-6127fc0d729f





*(please enable full screen for better visibility)

---

## Overview

The goal of this project is to detect the presence of **fire, smoke, or cigarettes** in images and video streams. At the time of the project, no existing work trained a **single network** to jointly detect all three classes — prior literature addressed either fire+smoke detection or cigarette detection separately, but never all three together in one model.

This project trains and compares:
1. **Three independent single-class networks** (fire, smoke, cigarette)
2. **One unified multi-class network** trained on the merged, re-annotated dataset

and evaluates whether the unified model outperforms the specialized ones.

## Related Work

| Approach | Classes | Notes |
|---|---|---|
| Faster R-CNN Inception V2 / SSD MobileNet V2 (Rentao et al., 2019) | Fire | Promising results with only 480 training images; SSD MobileNet V2 showed low precision |
| YOLOv3-tiny + k-means anchor clustering (Pincott et al., 2022) | Fire | High precision, small model size; anchor boxes require clustering overhead |
| Detection Transformer — DETR (Yuming Li et al., 2022) | Fire & Cigarette | End-to-end detector; original DETR needs long training times and heavy compute, weaker on small/early-stage fires |

## Dataset

- **~80,000 images** total, split into three class folders: `fire`, `cigarette`, `smoke`
- Each folder further split into `train` / `valid` / `test`, with COCO-format (`coco.json`) annotations per subfolder
- **Challenges identified:**
  - Artifacts in the smoke subset (some smoke images composited via Photoshop)
  - Visual overlap between "smoke" and "cigarette" classes, requiring careful re-annotation when merging

## Architecture

**YOLOX** ([Zheng Ge et al., 2021](https://arxiv.org/abs/2107.08430)) was selected as the detection backbone:
- **Backbone:** DarkNet-53 for feature extraction
- **Decoupled head:** separates classification and localization to resolve the task conflict between the two

**Model variant:** `YOLOX-Nano`
- Lowest parameter count in the YOLOX family
- Strong mAP despite the small size
- Lower training time (relevant given the ~80k-image dataset) and lower CO₂ emissions from training compute

## Workflow

```
1. Train single-class networks (fire, smoke, cigarette)
2. Test & evaluate single-class networks
3. Annotate the merged dataset (unified classes)
4. Train the unified network
5. Test & evaluate the unified network
6. Compare results (single vs. unified)
```

## Training Setup

| Parameter | Value |
|---|---|
| Loss | total loss = IoU loss + L1 loss + confidence loss + class loss |
| Learning rate | 0.001 |
| Batch size | 128 |
| Optimizer | SGD, momentum = 0.9 |
| Epochs | 300 |

## Results

- All three single-class networks trained successfully with **no overfitting** — train/validation loss curves stayed close, and both converged near 0
- **Cigarette detection** showed the weakest performance among the three classes, mainly due to limited clear cigarette-smoke imagery in the dataset
- The **unified network outperformed the single-class networks** in several comparisons:
  - **Fire:** the unified model correctly avoided a false-positive (sky misclassified as fire) and additionally identified smoke correctly
  - **Cigarette:** correctly detected both the cigarette and the fire in the same image; smoke was still missed due to limited clean training examples
  - **Smoke:** the unified model showed **+25% confidence** on a previously weak detection, plus correctly identifying fire in the same frame

## CO₂ Emissions

Training-time CO₂ estimates were tracked and reported for a 10-epoch reference run; actual training ran for 300 epochs, so reported consumption figures should be **scaled ×30** for the real total footprint. This was one of the reasons `YOLOX-Nano` was preferred over larger YOLOX variants — comparable mAP at a fraction of the compute cost.

## Conclusions & Future Work

The project's objective — training three networks on three separate datasets, then re-annotating and training a single unified network — was achieved. The unified model matched or exceeded the single-class networks across all three categories, without overfitting.

**Possible improvements:**
- Expand the cigarette-smoke subset with more labeled images
- Increase the overall size of the cigarette dataset
- Experiment with larger, more performant YOLOX variants

**Potential applications:**
- Collaboration with law enforcement / public safety
- Home security systems
- Automated enforcement of no-smoking zones in public spaces

---





## Authors

- **Nicola Mochi** — [LinkedIn](https://linkedin.com/in/nicola-mochi-9a70331b9)
- **Umberto Maraglino**

## References

- Ge, Z. et al. (2021). *YOLOX: Exceeding YOLO Series in 2021.* [arXiv:2107.08430](https://arxiv.org/abs/2107.08430)
- Rentao, S. et al. (2019). Fire detection via Faster R-CNN / SSD MobileNet V2.
- Pincott, J. et al. (2022). Fire detection via YOLOv3-tiny with k-means anchor clustering.
- Li, Y. et al. (2022). Fire & cigarette detection via Detection Transformer (DETR).
