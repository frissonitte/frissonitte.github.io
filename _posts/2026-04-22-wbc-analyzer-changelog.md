---
title: AI-Powered Pathology Assistant - WBC Analyzer Changelog
date: 2026-04-22 17:30:00 +0300
description: Versioned update notes for the WBC Analyzer project covering model improvements, domain adaptation, and deployment changes.
categories: [Portfolio, Projects]
tags: [wbc-analyzer, pathology-ai, machine-learning, domain-shift, tta, xai, flask-api, changelog]
image:
  path: /assets/img/wbc_banner.png
  alt: Project banner
---

### CHANGELOG – Final Report Updates

## Part – 1

### Newly Added Features and Components

#### Inference Time Domain Adaptation Pipeline — Part 3.5

- A three-stage adaptation pipeline has been added that can be implemented without the need to retrain the model. - Steps: Reinhard color normalization; Binary routing (restriction from 5 classes to Lymphocyte/Neutrophil space); Mild Test-Time Augmentation (TTA).
- A net improvement of +31.19 points was achieved in TestB.

#### Shortcut Learning Resistant Training Architecture — Part 3.6

- A new training strategy was developed with the `train_shortcut_resistant.py` script. - 20% Foreground Cropping and 15% Background Noise Injection were applied to the images. - An XAIFocusMonitor callback class was added that performs autonomous focus measurement via Grad-CAM at the end of the epoch.

#### Background Segmentation

- OpenCV-based background masking (Otsu Thresholding + Morphological Operations) integrated into the shortcut-resistant training architecture.

---

### Ablation Analysis and Comparative Experiments

#### Summary Results

- Baseline (TestB): 56.96%
- Binary routing: 73.90% (+16.94 pp)
- Reinhard: 86.46% (+12.56 pp)
- TTA (final): 88.15% (+1.69 pp)

#### Experiment Comparison Table

| Configuration | TestB Accuracy | Change |
| -------------- | -------------: | --------: |
| Baseline | 56.96% | — |
| Binary routing | 73.90% | +16.94 pp |
| Reinhard | 86.46% | +12.56 pp |
| TTA (final) | 88.15% | +1.69 pp |

---

### Changed and Updated Content

#### Test Set Accuracy

- TestA accuracy: 93.4% → 98.53% (+5.13 points)
- TestB accuracy (overall): 89% → 88.15%
- TestB accuracy (baseline → adaptation): 56.96% → 88.15%

#### Classification Report Updates

- Neutrophil Precision (TestA): 0.987 → 0.9958 (+0.009)
- Weighted avg F1 (TestA): 0.956 → 0.9853 (+0.029)

| Metric | Before | After | Change |
| ----------------- | ----: | -----: | ------: |
| Basophil Precision | 0.937 | 1.0000 |  +0.063 |
| Monocyte F1 | 0.866 | 0.9376 | +0.0716 |
| Eosinophil F1 | 0.849 | 0.9541 | +0.1051 |
| Lymphocyte F1 | 0.964 | 0.9865 | +0.0225 |

#### Confidence Score and Response Time

- Neutrophil test confidence score: 99.8% → 97.7%
- Response time: ~200 ms (unchanged)

#### Preprocessing Flowchart updated

---

### Domain Shift Analysis and Constraints

- It was shown that the largest source of domain shift is staining/color variation and that this is corrected with Reinhard normalization. - It was noted that the Raabin-WBC dataset only includes Giemsa staining and that full-size WSI images are not yet supported.

- ---

### Discussion Section Expansions

The discussion section in the final report was divided into 7 subheadings:

- Literature Comparison
- Class Imbalance
- Shortcut Learning and XAI
- Domain Shift and TTA
- Clinical MLOps
- Original Contributions
- Limitations

---

### Reference Updates

- References: 27 → 43

- Added fields: Domain shift, TTA, Shortcut learning (Panboonyuen 2026, Geirhos 2020, Bassi 2024), Clinical MLOps (Spadacini 2026, Ali 2026), LLM-based XAI control (Mermigkis 2026), Stain-aware domain alignment (Li 2026)

---

### Abstract Updates

Key points added to the abstract:

- Domain shift management

- TestB baseline 56.96% → after adaptation 88.15%
- Shortcut-Resistant Training
- +31.19 points improvement

---

### New Sections Added to the Final Report

- § 3.5 — Domain Shift Management and Inference Time Improvements
- § 3.6 — Shortcut-Resistant Training Architecture
- § 4.1.2 — Inter-Experimental Comparative Analysis and Shortcut Learning Finding
- § 4.1.3 — Inference-Time Domain Adaptation and Ablation Analysis
- § 5.3–5.7 — Discussion subsections (XAI, domain shift, MLOps, contributions, limitations)

## Part – 2

### Model and Provider Configuration

- The primary model was configured as **openai/gpt-4o** via GitHub Models (with GITHUB_TOKEN).
- The secondary (fallback) provider was retained as **Gemini 2.5-flash**.
- The backend now returns the actual provider used for each response.
- The model badge (**agentReportModelBadge**) in the frontend has been made dynamic and shows the actual provider.

### Token and Environment Variable Management

- The old **MODEL_TOKEN** has been completely removed.
- A normalized, secure read mechanism has been added for **GITHUB_TOKEN** and the optional **GEMINI_API_KEY** with the `get_env_token` function.

### Multimodal and Fallback Logic

- Multimodal calls are wrapped with `try/except`. - Automatic **text-only fallback** is applied in case of visual support rejection or error.
- Response parsing process strengthened (`robust parsing`, `extract_completion_text`).

### Model Parameters and Error Handling

- GitHub/OpenAI model parameters updated (e.g., `max_tokens` compliance ensured).
- Parameter mismatches leading to 400 series errors fixed.

### Response Reliability and Backup Mechanism

- Response parsing mechanism made more robust.
- A deterministic, rule-based final fallback layer (**build_rule_based_report**) capable of producing a result in every case has been added.

### Prompt and Style Improvements

- Global **SYSTEM_INSTRUCTION** updated. - Class-weighted **CLASS_MORPHOLOGY_CONTEXT** added.
- A dynamic **build_agent_prompt** system has been implemented to reduce repetitive and template-like reports.

### API Call Ordering

- Call order has been reorganized:
1. GitHub (GPT-4o)
2. Gemini fallback in case of failure
- The backend returns the provider used for each response and passes it to the frontend.

### Frontend Updates

- The model badge on `index.html` has been made dynamic. - The model information shown to the user is now updated according to the actual provider from the backend.

### Documentation Updates

- README.md and README.tr.md have been updated. - Required environment variables (GITHUB_TOKEN, GEMINI_API_KEY) and model ordering have been documented.

### Modified Files

- `app.py`

- `index.html`

- `README.md`

- `README.tr.md`

### Test / Validation Status

- Static and syntax checks were performed; no errors were found. - End-to-end validation with actual API keys is pending in the user environment.