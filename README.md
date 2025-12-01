# LLM Personas vs Human Personality Baselines (DS440 Final Project)

> 中文简介：本项目把大语言模型（LLM）当作“虚拟被试”，让不同人设的 LLM 去填写 IPIP-NEO 120 题人格问卷，并与真实人类数据做比较，观察人格画像的一致性和偏差。

This repository contains the code and analysis for our DS440 final project.  
We treat large language models (LLMs) as **simulated survey participants** on the IPIP-NEO-120 Big Five personality inventory and compare their personality profiles to **real human baseline data** from a public Kaggle dataset.

---

## 1. Project Overview

- **Goal**
  - Use standard psychometric tools (IPIP-NEO 120) to probe the *personality profiles* expressed by different LLM personas.
  - Compare persona-prompted LLMs (e.g., “US male college student”, “Chinese international student”) with **matched human groups**.

- **Key Questions**
  - Do persona prompts systematically shift the Big Five profile produced by an LLM?
  - How close are these profiles to average human profiles of similar demographic groups?
  - Are there systematic biases (e.g., the model is “more extraverted” or “less agreeable” than real people)?

- **High-Level Pipeline**

  1. Load and preprocess the **Kaggle IPIP-NEO 120** human dataset.
  2. Define several **LLM personas** and construct the 120 questionnaire items.
  3. Use OpenAI / Gemini APIs to let each persona answer all 120 items (with batching and error handling).
  4. Aggregate item scores into the **Big Five** domains (O, C, E, A, N).
  5. **Scale human scores** to the same 1–5 range for comparability.
  6. Compute domain-level differences and effect sizes between LLM personas and human groups.
  7. Visualize results with bar plots showing where LLMs are higher / lower than humans.

A vertical flowchart of the methodology is shown below (you can add the PNG to the repo):

```text
Research Question & Goal
        ↓
Setup & Imports
        ↓
Human Data & Baseline
        ↓
Questionnaire & Personas
        ↓
LLM Survey Engine
        ↓
Scoring & Scaling
        ↓
Visualization & Interpretation
