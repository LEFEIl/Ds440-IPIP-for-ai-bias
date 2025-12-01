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
```

## 2. Data
2.1 Human Dataset (Kaggle: IPIP-NEO 120)

Source: public Kaggle dataset of IPIP-NEO-120 scores.

Each row = one participant, with:

Demographics: sex, age, etc.

Big Five domain scores: Openness, Conscientiousness, Extraversion, Agreeableness, Neuroticism.

Preprocessing:

load_human_scores():

Reads the original CSV.

Maps numeric sex to labels (1 → "male", 2 → "female") → sex_label.

Bins age into groups such as 18–25, 26–35, … → age_bin.

Filters out rows with missing or obviously invalid scores.

## 3. Personas & Questionnaire
# 3.1 IPIP-NEO-120 Items

We build a DataFrame items_df representing the questionnaire:

item_id: 1–120

item_text: text of the item (here represented as “Item {i}” in the minimal version; can be replaced by full IPIP text if licensing permits).

domain: which of the Big Five the item belongs to, via item_to_domain().

reversed: whether the item is reverse-scored.

This structure allows us to aggregate item-level scores into each Big Five domain.

# 3.2 Persona Design

We define several personas in a list PERSONAS.
Each persona is a small dictionary, for example:

describe_group(df, group_cols):

Computes mean, standard deviation, and sample size for each domain within each group (e.g., by sex_label, age_bin).

These group summaries provide the human baselines we compare LLM personas against.
