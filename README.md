# LLM Personas vs Human Personality Baselines (DS440 Final Project)

> 中文简介：本项目把大语言模型（LLM）当作“虚拟被试”，让不同人设的 LLM 去填写 IPIP-NEO 120 题人格问卷，并与真实人类数据做比较，观察人格画像的一致性和偏差。

This repository contains the code and analysis for our DS440 final project.  
We treat large language models (LLMs) as **simulated survey participants** on the IPIP-NEO-120 Big Five personality inventory and compare their personality profiles to **real human baseline data** from a public Kaggle dataset.

DATA link: https://www.kaggle.com/datasets/edersoncorbari/pip-neo-big-five-personality-120-item-version
---

## 1. Project Overview

- **Goal**
  - Use standard psychometric tools (IPIP-NEO 120) to probe the *personality profiles* expressed by different LLM personas.
  - Compare persona-prompted LLMs ( “US male college student”, “Chinese international student”) with **matched human groups**.

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
### 3.1 IPIP-NEO-120 Items

We build a DataFrame items_df representing the questionnaire:

item_id: 1–120

item_text: text of the item (here represented as “Item {i}” in the minimal version; can be replaced by full IPIP text if licensing permits).

domain: which of the Big Five the item belongs to, via item_to_domain().

reversed: whether the item is reverse-scored.

This structure allows us to aggregate item-level scores into each Big Five domain.

### 3.2 Persona Design

We define several personas in a list PERSONAS.
Each persona is a small dictionary, for example:

describe_group(df, group_cols):

Computes mean, standard deviation, and sample size for each domain within each group (e.g., by sex_label, age_bin).

These group summaries provide the human baselines we compare LLM personas against.
```text
PERSONAS = [
    {
        "name": "US_male(18-25)",
        "prompt_snippet": "You are a 21-year-old male college student in the United States.",
        "human_filter": {"sex_label": "male", "age_bin": "18-25"}
    },
    {
        "name": "US_female(18-25)",
        "prompt_snippet": "You are a 21-year-old female college student in the United States.",
        "human_filter": {"sex_label": "female", "age_bin": "18-25"}
    },
    {
        "name": "Chinese_international_student(18-25)",
        "prompt_snippet": "You are a 21-year-old Chinese international student studying in the United States.",
        "human_filter": {"sex_label": "male", "age_bin": "18-25"}
    }
]
```
## 4. LLM Survey Engine
### 4.1 Prompt Template

We treat the model exactly like a human participant:

Global instruction string IPIP_INSTRUCTION explains:

The 1–5 Likert scale (1 = strongly disagree, 5 = strongly agree).

The requirement to answer with a single integer per item.

build_item_prompt(persona_snippet, item_text) combines:

Persona description (who you are).

Questionnaire instructions.

The item text.

### 4.2 Calling the APIs

Main experiments use OpenAI (gpt-4o-mini).

We also test Gemini (gemini-2.5-flash) for cross-model comparison.

API keys are passed via environment variables or via the Kaggle UserSecretsClient when running on Kaggle.

## 5. Scoring, Scaling, and Comparison
### 5.1 Aggregating LLM Scores

df_llm_items: item-level responses from all personas.

df_llm_domains:

Group by ["persona", "domain"] and compute the mean score.

Pivot to a wide table with one row per persona and one column per domain.

Each persona ends up with five average scores: O, C, E, A, N on the 1–5 scale.

### 5.2 Scaling Human Data

Human domain scores in Kaggle are on a different numerical range.
We rescale human scores to the 1–5 range using:

$$
\text{scaled} = 1 + 4 \times \frac{\text{score}}{\max(\text{score})}
$$


## 6. Visualization
We visualize domain-level biases as bar plots:

For each persona:

X-axis: domains [Openness, Conscientiousness, Extraversion, Agreeableness, Neuroticism].

Y-axis: mean_diff (LLM – human).

Horizontal line at 0 = “no difference” baseline.

We can produce:

GPT-4o-mini vs Human plots.

Gemini 2.5 vs Human plots.

Positive bars indicate the persona-prompted LLM is higher than humans on that domain; negative bars indicate it is lower.

## 7. Getting Started
### 7.1 Environment
All the necessary imports are listed separately in the code notebook; you only need to change the file paths.

### 7.2 Running the Notebook

Download the IPIP-NEO 120 Kaggle dataset and save it under data/.

Set your API keys as environment variables (or configure Kaggle secrets).

Open “final_code_for_DS440.ipynb” in Jupyter / VS Code / Kaggle.

Run all cells from top to bottom:

Data loading & preprocessing.

Persona definition.

LLM survey calls.

Scoring & comparisons.

Plotting.

outputting.

## 8. Acknowledgements
Course: DS440 — (Penn State).

Data: IPIP-NEO-120 dataset from Kaggle.

Models: OpenAI GPT-4o-mini and Google Gemini 2.5 Flash.

Thanks to our professor and TA for guidance and feedback throughout the project.

