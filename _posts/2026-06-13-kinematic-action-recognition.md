---
title: "Scalable Kinematic Action Recognition for Industry 5.0"
date: 2026-06-13 12:00:00 +0300
description: End-to-end big data pipeline for classifying factory operator actions from 10 GB, 200 Hz motion-capture sensor streams — covering out-of-core engineering, feature extraction, supervised/unsupervised learning, and real-time streaming with concept drift detection.
categories: [Portfolio, Projects]
tags: [big-data, dask, machine-learning, streaming, time-series, industry-5.0, random-forest, concept-drift, competition]
---

## Overview

An end-to-end pipeline for classifying factory operator actions from high-frequency motion-capture sensor data — ~10 GB, 132 channels sampled at 200 Hz. The project covers every stage of a production-grade data science workflow: out-of-core ingestion, time-series feature engineering, unsupervised and supervised modeling, and real-time streaming with adaptive drift detection.

A parallel competition notebook placed **1st on the private leaderboard** (0.94169 accuracy) in the ISE446 Scalable Kinematic Action Recognition competition.

---

## Results at a Glance

| Phase | Method | Key Result |
|-------|--------|-----------|
| 1 | Dask out-of-core ingestion | 10 GB CSV → 6.86 GB Parquet (3× compression) |
| 2 | Scalable EDA | README said binary; found 15 distinct classes |
| 3 | Rolling-window feature extraction | 9.7M rows → 97,612 windows × 528 features |
| 4A | MiniBatchKMeans + PCA + Hungarian | Macro F1 ≈ 0.14 (curse of dimensionality) |
| 4B | RandomForest (binary, class 0 vs 1) | Macro F1 ≈ 0.9995 · 1.3s train · 8.7 MB RAM |
| 5 | ARF + ADWIN streaming | 81 win/sec · drift in ~58 windows (~290 ms) · 59 MB peak RAM |

---

## Phase 1: Data Architecture — Out-of-Core Ingestion

The raw dataset was a ~10 GB CSV file of continuous motion-capture sensor logs. Loading it naively into pandas would immediately exhaust RAM, so I built a Dask-based out-of-core pipeline.

### The 3 Challenges

**Hidden repeated headers and EOF characters.** The sensors occasionally restarted mid-recording, writing column headers again in the middle of the file. Invisible `\x1a` EOF characters were also embedded throughout. Standard float parsing crashed immediately.

**RAM ballooning and dead workers.** The fix — reading everything as `object` (string) before casting — caused each 64 MB block to expand far beyond the 3 GB per-worker memory limit, triggering `KilledWorker` errors.

**Malformed rows.** Some sensor logs were concatenated, producing rows with 225 columns instead of 134.

### Solution

```python
client = Client(n_workers=8, threads_per_worker=2, memory_limit='3GB')
df = dd.read_csv(..., blocksize='64MB', dtype='object', on_bad_lines='skip')
df_cleaned = df.map_partitions(clean_and_cast_partition)
df_cleaned.to_parquet('data/main_data_parquet', ...)
```

- Read as plain text in 64 MB partitions (down from 128 MB) to prevent ballooning
- `clean_and_cast_partition` uses `pd.to_numeric(errors='coerce')` — silently converts garbage to NaN, then `dropna()` removes those rows
- Clean numeric values packed into `float32` / `int8` for memory efficiency
- Written immediately to Parquet — subsequent reads are 10–20× faster than CSV

**Outcome:** 10 GB CSV → 6.86 GB Parquet with 3× effective compression.

---

## Phase 2: Scalable EDA — The "Binary" Trap

The dataset README described a "binary classification (0 or 1)" task. EDA revealed something completely different.

### The Discovery

```python
label_counts = sampled_df['LABEL'].value_counts().sort_index()
anomalies = sampled_df[~sampled_df['LABEL'].isin([0, 1])]
```

The `~isin([0, 1])` tripwire exposed **15 distinct classes** (0, 1, 3–15), with ~435,000 rows in the non-binary classes. They were roughly uniformly distributed (~36K instances each in the 5% sample) — these were sustained, distinct operator actions, not noise.

This reframed the entire project: the "binary" label was a rubric simplification, not ground truth. The primary modeling task was natively multiclass, justifying the unsupervised approach in Phase 4A.

**Stationarity check:** Rather than running a heavy ADF test on 10M rows, I used rolling mean and standard deviation plots to visually confirm weak stationarity across sensor channels — sufficient for the rolling-window assumption in Phase 3.

---

## Phase 3: Time-Series Feature Engineering

Raw sensor streams at 200 Hz cannot be fed directly to standard ML algorithms. The challenge was converting continuous time-series into a fixed-width feature matrix without losing temporal context.

### The Window Logic

At 200 Hz, each row is 5ms. A typical industrial action (tightening a screw, picking up a tool) lasts ~1 second. So:

- **Window size:** 200 rows = 1 second of sensor data
- **Step size:** 100 rows = 50% overlap for dense coverage

### Feature Extraction

For each window and each of the 132 sensor channels, I extracted 4 statistical features: mean, std, min, max.

```python
for start in range(0, n - WINDOW_SIZE + 1, STEP_SIZE):
    window = df.iloc[start : start + WINDOW_SIZE]
    label = int(window['LABEL'].mode()[0])  # majority vote
    feats[f'{col}_mean'] = np.mean(vals)
    feats[f'{col}_std']  = np.std(vals)
    # ... min, max
```

A high standard deviation on `R_Wrist_Rx`, for example, tells the model the operator's wrist was rotating rapidly. The RF model later performing feature importance ranking over these 528 features is effectively doing sequence-aware feature selection — without requiring an LSTM.

**The Dask meta schema** had to be explicitly pre-defined (`float32` typed) to prevent schema mismatch errors during `.compute()` — a common pitfall with custom `map_partitions` functions.

**Outcome:** 9.7M rows → **97,612 windows × 528 features**

---

## Phase 4A: Unsupervised Learning — Confronting the Curse of Dimensionality

Objective: cluster the 15 production actions using only the feature matrix, no labels.

### What Happened

Fed full 528-feature matrix into `MiniBatchKMeans(n_clusters=15)`. F1-Score: **~0.15**.

The cause was immediate and clear: **the curse of dimensionality**. K-Means relies on Euclidean distance. In 528 dimensions, the distance between any two points converges — the algorithm has no geometric signal to work with.

### The Fix Attempt — PCA

```python
pca = PCA(n_components=50, random_state=42)
X_pca = pca.fit_transform(X_scaled)
```

50 components retained **94.3% of explained variance**, reduced multicollinearity, and partially restored meaningful geometry. F1 improved marginally — still ~0.14.

### Why It Still Failed

K-Means cannot separate 15 fine-grained kinematic action classes even with PCA. The difference between "tighten" and "loosen" lives in subtle rotational variation — not in cluster centroids. Distance-based separation without label supervision is fundamentally insufficient here.

**The Hungarian Algorithm** (`scipy.optimize.linear_sum_assignment`) was used for fair evaluation: since K-Means assigns arbitrary cluster numbers, Hungarian matching finds the optimal 1-to-1 mapping between cluster IDs and ground-truth labels before computing F1.

**Conclusion:** Unsupervised distance-based models are ill-suited for fine-grained kinematic classification. This result directly motivated the supervised approach in Phase 4B.

---

## Phase 4B: Supervised Learning — Defeating Temporal Leakage

Task: binary classification (class 0 vs 1) using `RandomForestClassifier`.

### The Silent Enemy

Initial run with `train_test_split(stratify=y)`: F1-Score = **1.0000**. Suspicious.

Root cause: **temporal data leakage**. The 50% overlapping windows from Phase 3 meant adjacent windows shared 50% of the same raw sensor rows. A random shuffle placed a window's "overlapping twin" in the training set while the original went to test — the model memorized, not generalized.

### The Fix

`TimeSeriesSplit` was degenerate here: Class 0 and Class 1 were concentrated in completely separate temporal segments. A time-ordered split would result in the model never seeing one class during training.

Solution — **systematic split**:

```python
test_mask = np.zeros(len(X), dtype=bool)
test_mask[::5] = True  # every 5th window → test set
```

Every 5th window to test, ensuring adjacent overlapping windows are strictly separated while maintaining class balance.

### Results

```
Macro F1-Score: 0.9995
Training time:  1.3s
Peak RAM:       8.7 MB
Inference:      27.36 ms for 2,143 samples (~0.012 ms/window)
```

### What the Model Actually Learned

Top 20 features by importance were almost exclusively `_Tz` (Z-axis / vertical height) metrics: `Head_Tz_max`, `Lowerback_Tz_min`, `L_Femur_Tz_max`.

Physical interpretation: Class 0 (Normal Assembly) vs Class 1 (Exceptional Intervention) primarily differ in **vertical posture**. The operator is upright during Class 0 and bent down during Class 1. The RF learned: if the head and lower back drop below a specific height threshold, the window is Class 1.

---

## Phase 5: Real-Time Streaming with Concept Drift Detection

Production ML systems degrade over time — sensor decalibration, operator fatigue, process changes. This phase built a streaming architecture that learns incrementally and detects when ground reality shifts.

### Architecture — Prequential Evaluation

Using the `river` library, every window is processed as:

1. **Predict** — model outputs class based on current knowledge
2. **Evaluate** — log whether prediction was correct
3. **Learn** — model updates on the revealed true label

This test-then-train loop eliminates data leakage mathematically and provides unbiased real-time accuracy estimates.

### Drift Injection

```python
# At the 50% mark (window 48,806):
# 30% of labels randomly flipped to simulate production-line fault
```

### The ADWIN Response

```python
model = preprocessing.StandardScaler() | forest.ARFClassifier(n_models=10, seed=42)
drift_detector = drift.ADWIN(delta=0.002)
```

- **Pre-drift baseline accuracy:** 0.996
- **Drift injected at:** window 48,806
- **Drift detected at:** window **48,864** — 58 windows (~290 ms) after injection
- **Post-adaptation accuracy:** ~0.846 (mathematically expected with 30% noise)

The ARF adapted its background trees to the noisy data without catastrophic failure. At 81 windows/sec throughput, the system comfortably handles 200 Hz real-time sensors.

### System Profile

| Metric | Value |
|--------|-------|
| Total stream time | 1199.2s (~20 min) |
| Throughput | 81 windows/sec |
| Peak RAM | **59.4 MB** |

59 MB peak RAM makes this viable for **edge deployment directly on the factory floor**.

---

## Key Takeaways

- **Out-of-core is non-negotiable at 10 GB scale.** The combination of 64 MB blocks, `dtype='object'` initial read, and immediate Parquet write was the only stable path through three distinct failure modes.
- **EDA before modeling.** The "binary" framing in the rubric was a trap. Discovering 15 classes restructured the entire project strategy.
- **Random splits on sliding windows = leakage.** 50% window overlap makes standard `train_test_split` invalid. Systematic every-Nth splitting is the correct approach when `TimeSeriesSplit` is degenerate.
- **Distance-based clustering fails for fine-grained kinematic classes.** PCA helps but cannot overcome fundamental semantic overlap in 15 action classes. Supervised tree-based models are the right tool.
- **Streaming drift detection is fast.** ADWIN detected a 30% label corruption in under 300ms. At this latency, a real production system could trigger human review before a bad batch ships.
