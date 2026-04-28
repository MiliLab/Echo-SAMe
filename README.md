<div align="center">

# SAMe: A Semantic Anatomy Mapping Engine for Robotic Ultrasound

[![Status](https://img.shields.io/badge/Status-Under_Review-orange)]()
[![arXiv](https://img.shields.io/badge/arXiv-TBD-b31b1b.svg)]()
[![License](https://img.shields.io/badge/License-MIT-lightgrey.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.8%2B-blue)](https://www.python.org/)

**A plug-in anatomical prior layer that grounds clinical intent and a single external body image into patient-specific anatomical hypotheses and control-facing 6-DoF probe initialization cues.**

[Introduction](#-introduction) | [Key Features](#-key-features) | [Method Overview](#-method-overview) | [Clinical Semantics Grounding](#-clinical-semantics-grounding) | [Anatomical Prior Learning](#-anatomical-prior-learning) | [Main Results](#-main-results) | [Getting Started](#-getting-started) | [Release Status](#-release-status) | [Citation](#-citation)

</div>

---

## 📖 Introduction

Robotic ultrasound has made substantial progress in local image-driven control, contact regulation, and view optimization. However, current systems still lack an explicit anatomical prior layer for deciding **what to scan, where to start, and how to place the probe before local refinement begins**.

**SAMe** bridges this gap as a **target-to-anatomy-to-action** framework for anatomy-aware scan initialization. It provides a plug-in prior layer that grounds clinical intent into structured anatomical targets, instantiates patient-specific anatomy from a single external body image, and converts the result into robot-readable, contact-aware 6-DoF initialization cues — without preoperative CT/MRI or additional registration sweeps.

<div align="center">
  <img src="assets/same-overview.png" alt="SAMe overview" width="100%">
  <p><em>From manual expertise and surface-bound robotic pipelines to the SAMe prior-layer paradigm.</em></p>
</div>

---

## 🚀 Key Features

* **🧠 Structured Anatomy Mapping:** Decomposed into **3** coupled modules (CSG → ARI → ATI) spanning complaint grounding to probe initialization.
* **📷 Single RGB Input:** One monocular external body image — no preoperative CT/MRI and no additional registration sweep at scan time.
* **⚡ Lightweight Inference:** Full-organ prior inference in **0.76 s** on CPU; liver-only in **0.08 s**.
* **🤖 Real-Robot Validation:** **97.3%** organ-hit rate for liver and **81.7%** for kidney initialization on physical platform.

### At a Glance

| Item | Description |
| :--- | :--- |
| **Input** | Clinical complaint + single monocular RGB body image |
| **Core Output** | Grounded organ/ROI, patient-specific anatomical hypothesis, control-facing 6-DoF initialization cues |
| **Design Goal** | Anatomy-aware scan initialization before local image-based refinement |
| **No Extra Acquisition** | No CT/MRI; no registration sweep |
| **Inference Cost** | **0.76 s** (full organ set, CPU) / **0.08 s** (liver only, CPU) |
| **Real-Robot Hit Rate** | **97.3%** liver / **81.7%** kidney |

---

## 🧠 Method Overview

SAMe is organized into three coupled components that form a **target-to-anatomy-to-action** pipeline:

<div align="center">
  <img src="assets/same-pipeline.png" alt="SAMe pipeline" width="100%">
  <p><em>Overview of the SAMe pipeline from semantic grounding to anatomical instantiation and robot-facing initialization.</em></p>
</div>

| Module | Core Question | Input | Output | Key Capability |
| :--- | :--- | :--- | :--- | :--- |
| **CSG** | What should be scanned? | Complaint text | Organ- and anatomy-level targets | Clinical semantics grounding via retrieval-augmented prior |
| **ARI** | Where is the target anatomy? | Single RGB body image | Patient-specific anatomical hypothesis | Skeleton-conditioned organ instantiation with uncertainty |
| **ATI** | How should scanning begin? | Anatomical hypothesis | Candidate contacts, target rays, 6-DoF probe states | Contact-aware initialization for downstream control |

Importantly, SAMe is **not** a full autonomous scanning system. It is an explicit anatomical prior layer designed to improve initialization and remain compatible with downstream control and online posterior updating.

---

## 🔍 Clinical Semantics Grounding (CSG)

CSG grounds under-specified clinical complaints into explicit organ- and anatomy-level targets through a semantic prior backed by structured clinical retrieval.

The grounding process is supported by a structured semantic prior (SAMe-DB) constructed from curated clinical text sources, organized as a hierarchical knowledge base for retrieval-augmented grounding.

<div align="center">
  <img src="assets/same-semantic-db.png" alt="SAMe semantic prior database and RAG results" width="100%">
  <p><em>Semantic prior database and retrieval-augmented grounding performance.</em></p>
</div>

Retrieval support consistently improved organ- and location-level grounding, especially at the anatomy level where outputs become directly useful for downstream instantiation and initialization.

---

## 🏗️ Anatomical Prior Learning

<div align="center">
  <img src="assets/same-organ-prior.png" alt="SAMe organ-layer modeling pipeline" width="100%">
  <p><em>Offline organ-layer modeling and rig-anchored prior learning used to support patient-specific anatomical instantiation.</em></p>
</div>

SAMe builds a canonical organ-layer representation from CT-derived skin, skeletal, and organ meshes. These assets are registered into a rig-consistent body frame, unposed into a shared anatomical space, and decomposed into compact organ-wise descriptors.

This keeps the anatomical state:

* **Explicit** — not hidden inside dense latent fields,
* **Lightweight** — downstream inference predicts low-dimensional, control-relevant quantities,
* **Robot-compatible by construction** — output representation is already organized around geometry needed for initialization.

---

## 🏆 Main Results

### Comparison with Existing Robotic Ultrasound Systems

| System / Approach | Anatomical Prior | Input Modality | Online Initialization | Complaint-Driven |
| :--- | :---: | :--- | :---: | :---: |
| Surface-heuristic baselines | ❌ | RGB / none | ⚠️ Heuristic | ❌ |
| CT/MRI-registered methods | ✅ Volumetric | Preoperative CT/MRI | ✅ | ❌ |
| Image-based controllers | ❌ | Ultrasound B-mode | ✅ | ❌ |
| **SAMe (Ours)** | **✅ Skeleton-conditioned** | **Single RGB** | **✅** | **✅** |

### Quantitative Results

<div align="center">
  <img src="assets/2.png" alt="Real-robot setup" width="88%">
  <p><em>Representative real-robot setup and anatomy-aware liver target placement used in the study.</em></p>
</div>

| Component | Evaluation Setting | Key Result |
| :--- | :--- | :--- |
| **CSG** | 1,000 held-out symptom descriptions | Location-level F1: **0.357** (macro) / **0.356** (micro) with SAMe-DB |
| **ARI** | 35 held-out cases, 11 organs, 385 pairs | Mean centroid error: **22.55 mm**; support IoU: **0.391** |
| **ARI Efficiency** | CPU inference | Full-organ: **0.76 s**; liver-only: **0.08 s** |
| **ATI** | Real-robot ultrasound | Organ-hit rate: **97.3%** (liver) / **81.7%** (kidney) |

Additional findings include:
* Centroid-based SAMe initialization outperformed surface-heuristic baselines for both liver and kidney.
* The explicit low-dimensional representation improved point localization, organ support estimation, and downstream initialization usability.
* The formulation preserved robustness across substantial body-habitus variation while remaining lightweight enough for deployment.

---

## 🚀 Getting Started

### Installation

```bash
git clone https://github.com/your-org/Echo-SAMe.git
cd Echo-SAMe
```


---

## 📅 Release Status

**2026.04**
* Repository structure and overview materials published.
* Manuscript under review.

### Currently Available
* Project overview and visual materials
* Method summary aligned with the manuscript

Updates will be added here as the release package is finalized.

---

## 🖊️ Citation

If you find **SAMe** useful for your research, please cite:

```bibtex
@misc{same2026,
  title={SAMe: A Semantic Anatomy Mapping Engine for Robotic Ultrasound},
  author={Jing Zhang and Duojie Chen and Wentao Jiang and Zihan Lou and Jianxin Liu and Xinwu Cui and Qinghong Zhao and Bo Du and Christoph F. Dietrich and Dacheng Tao},
  year={2026},
  note={Under review}
}
```

---

<div align="center">
<p>Maintained by the Echo-SAMe Team</p>
</div>
