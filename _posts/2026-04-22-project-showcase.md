---
title: Project Showcase and Technical Summaries
date: 2026-04-22 16:20:00 +0300
description: Real projects in ML, backend, and automation with measurable outcomes.
categories: [Portfolio, Projects]
tags: [projects, machine-learning, backend, automation, etl]
---

This page is organized for quick screening with a consistent format:

- Duration
- Tech
- Impact
- Links

## Project Cards

### WBC Analyzer: Robust OOD Generalization in Peripheral Blood Smears

- Duration: Jan 2026 - Apr 2026
- Tech: DenseNet121, WBCAttention, MedSwish, MEF Preprocessing, Flask REST API, Grad-CAM, GPT-4o, Gemini 2.5 Flash
- Impact:
	- Achieved **98.53% in-distribution accuracy** and **89.05% OOD accuracy** on unseen hardware data (**+32.09 pp** over unadapted baseline, retraining-free).
	- Designed 5-step Medical Enhanced Filter (MEF) pipeline for cross-device staining and exposure normalization.
	- Integrated autonomous multi-modal LLM agent (GPT-4o / Gemini fallback) for Grad-CAM-based shortcut detection.
	- Published academic preprint ([DOI: 10.13140/RG.2.2.34201.79208](https://doi.org/10.13140/RG.2.2.34201.79208)).
- Links:
	- [GitHub](https://github.com/frissonitte/wbc-analyzer-final)
	- [Live Demo](https://emirhanyildirim.me/wbc-analyzer/)

### Popcorn Wagon: Hybrid Movie Recommendation System

- Duration: Jan 2025 - May 2025
- Tech: TruncatedSVD, TMDB API, MovieLens, Pandas, Dask, Spotify Annoy
- Impact:
	- Built a hybrid recommendation strategy combining collaborative and content-based methods.
	- Processed large datasets with a scalable ETL pipeline.
	- Reduced recommendation latency to millisecond level using approximate nearest neighbors.
- Links:
	- [GitHub](https://github.com/frissonitte/popcorn-wagon)
	- Demo: Available on request

### Scalable Kinematic Action Recognition — Industry 5.0 (ISE446)

- Duration: Sep 2025 - Jun 2026
- Tech: Python, Dask, Scikit-Learn, River (ARF + ADWIN), Parquet, LightGBM, RandomForest
- Impact:
	- Processed 10 GB proprietary motion-capture CSV into Parquet (3× compression) via out-of-core Dask pipeline.
	- Extracted 97,612 sliding-window feature vectors (528 features) from 9.7M sensor rows.
	- RandomForest binary classifier: Macro F1 ≈ **0.9995**, 1.3s train, 8.7 MB RAM.
	- Adaptive Random Forest streaming: **81 windows/sec**, drift detected in ~58 windows (~290 ms) via ADWIN.
	- Competition notebook placed **1st on private leaderboard** (0.94169 accuracy) using LightGBM + RF ensemble.
- Links:
	- GitHub: Available on request (dataset proprietary)

### Listing Pilot Mobile Automation Suite for C2C Marketplaces

- Duration: Dec 2025 - Mar 2026
- Tech: Python, Appium, Selenium, UiAutomator2
- Impact:
	- Built a platform-agnostic, config-driven Appium automation suite utilizing advanced UI interaction algorithms (spatial pairing, overlap detection) and real-time Telegram alerting.
	- Successfully automated the management of ~1,000 active listings across major C2C marketplaces, eliminating hours of manual, repetitive workload every week.
- Links:
	- [GitHub](https://github.com/frissonitte/listing-pilot)
	- Demo: Private

### Portal Cleaner Ultimate: Desktop RPA Suite

- Duration: Jun 2025 - Jul 2025
- Tech: Python, Selenium, Tkinter, Multithreading, HTML/JS test harness
- Impact:
	- Reduced daily manual workload by **90%+** (4-6 hours to under 30 minutes).
	- Built a local mock ERP environment to run offline end-to-end reliability tests.
	- Implemented dynamic retry and centralized error logging for fault-tolerant execution.
	- Maintained responsive UI under heavy background ingestion and scraping tasks.
- Links:
	- [GitHub](https://github.com/frissonitte/portal-cleaner-ultimate)
	- Demo: Internal

## CV Download

- PDF CV: Please contact me if you want the latest signed PDF version.

## Professional Experience Highlights

### Software Engineer Intern - Esbi Bilişim ve Telekomünikasyon (Jul 2026 - Aug 2026)

- Software engineering internship focused on backend and systems work.

### Operations Manager and Python Developer - zidorun (Jun 2025 - Present)

- Founded and operated a niche collectibles micro-venture across multiple C2C marketplaces.
- Engineered Python backend workflows and sync pipelines for inventory and pricing.
- Processed **700+ transactions** with **99% success rate**.
- Reduced repetitive manual operations by **90%+**.

### Intern Engineer - Platech Coating (Jun 2025 - Jul 2025)

- Developed Python-based desktop automation suite using Selenium and Tkinter.
- Engineered multithreaded GUI architecture and fault-tolerant retry workflows for unstable ERP portal interactions.
- Built report generation, order frequency analysis, and HTML-based product image extraction scripts.

> Detailed case-study posts with architecture diagrams and API snippets will be published next.
{: .prompt-info }
