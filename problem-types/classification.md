## Table of Contents
- [1. Open-Set Recognition](#1-open-set-recognition--ood-detection-for-document-identification)


## 1 Open-Set Recognition / OOD Detection for Document Identification

https://arxiv.org/html/2401.06521v1

Open-set recognition is a classification setting where the model must:
- classify inputs from **known classes**, and
- reject inputs from **unknown classes**.

For document ID, if a new template/vendor/type appears, the model should return `unknown` or send it to human review — not force it into a known class.

A normal softmax classifier is **closed-set**. It always outputs probabilities over known classes and picks one. It cannot say “I don’t know.” It is often confidently wrong.

**Why simple softmax threshold is weak**  
You can reject when `max softmax < threshold`, but softmax is overconfident and thresholds are hard to tune. Better scores: entropy, energy score, Mahalanobis distance, kNN distance, class prototype similarity. Calibrate with temperature scaling. Still not reliable alone.

**Adding `other` / `unknown` class**  
Collect negatives: blank pages, receipts, other templates, handwritten notes, noise, other languages. Train an extra class. Helpful, but not enough: unknown space is huge, and the model may still force new documents into known or `other`.

**Open-set / metric learning approach**  
Train an embedding model with ArcFace, CosFace, triplet, contrastive, or prototypical loss. At inference:
- compute embedding,
- compare to known class prototypes/centroids,
- if distance is too large or similarity too low → `unknown`.

For documents, combine with OCR keywords, layout embedding, logo/header detection, and template matching. Layout distance is especially useful.

**Evaluation**  
Simulate unknowns by leave-one-class-out: train on N-1 classes, treat held-out class as unknown. Measure unknown detection rate, false positive rate on knowns, AUROC, open-set F1. Tune threshold by business cost: rejecting a known doc vs misclassifying an unknown doc.

**Practical recipe**
1. Collect known classes + large `other` set.
2. Train classifier or embedding model.
3. Hold out classes to simulate unknowns.
4. Compute confidence/OOD/distance scores.
5. If confident and close to known class → predict class.
6. Else → `unknown` / human review.
7. Log unknowns and retrain periodically.

Key idea: a standard classifier cannot handle unseen classes by itself. You need a reject option, negative/unknown data, and confidence/distance scoring. For document ID, use visual layout + OCR + embedding-distance thresholds, then route uncertain cases to human review.