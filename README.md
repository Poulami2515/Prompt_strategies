# Prompt Strategies

Dataset source : : https://www.kaggle.com/datasets/omkarsabnis/yelp-reviewsdataset

# 🌟 **Experimental Evaluation of Yelp Star Prediction Methods**

*Baseline vs PRM vs Reflex — A Comparative Study*

This section dives into the behavior of three prompting strategies applied to a Yelp star-rating prediction task. Across 200 samples, each model attempted to produce a valid JSON response containing a predicted star rating (1–5) and a brief explanation.

Two metrics guided the evaluation:

1. **Accuracy** — percentage of cases where predicted_stars == gold_stars
2. **JSON Validity Rate** — percent of responses that were syntactically valid JSON

Your plots (performance_plot.svg & delta_plot.svg) visualized these results. Below is the deeper narrative they deserve.

---

# 📌 **1. Baseline Prompting (Standard Instruction Prompt)**

**Accuracy:** 47.00%
**JSON Validity:** 80.00%

### 🚀 *How it works*

This method is the bare-bones setup — a single prompt that directly asks the model to predict a star rating and explanation.

### 🧠 *Why it behaves this way*

* It gives a decent JSON validity rate (80%), showing the base model mostly obeys format instructions.
* Accuracy underperforms because the model:

  * Overfits to positive sentiment
  * Lacks error correction
  * Has no mechanism for candidate comparison or self-reflection

### 📉 *Where it falls apart*

Baseline models tend to hallucinate ratings or flatten subtle sentiment differences.
Thus, accuracy suffers even though formatting stays mostly intact.

---

# 📌 **2. PRM — *Process Reward Model* Sampling & Scoring**

**Accuracy:** 59.50%
**JSON Validity:** 100.00%

### 🔧 *How PRM works*

For each review:

1. **Generate N candidate responses** using stochastic sampling (temperature/top-p).
2. **Score each candidate** using your custom PRM reward:

   * +3 for valid JSON
   * −3 for invalid
   * Add difference in squared deviation from neutral star “3”
   * * jitter (1e-2)
   * Normalize & clamp
3. **Pick the highest-scoring candidate**

### 🌞 *Why PRM is a glow-up*

* JSON validity hits **100%** — your reward makes invalid JSON so costly that the model quickly learns to behave.
* Accuracy jumps significantly to **59.50%**, because:

  * Multiple candidates → diversity of reasoning
  * Scoring → selects the closest semantically aligned response
  * Neutrality bias correction reduces over-enthusiastic 5-star hallucinations

### 💡 *Interpretation*

PRM transforms the model from a single-shot guesser into a reflective sampler that optimizes behavior over candidates.
This is the only method that improved **both** correctness and reliability.

---

# 📌 **3. Reflex — Self-Verification / Self-Correction Prompting**

**Accuracy:** 48.50%
**JSON Validity:** 77.50%

### 🔍 *How Reflex works*

1. The model generates an initial answer.
2. A "review" prompt asks the model to critique or validate the previous output.
3. A final improvement step re-generates the JSON.

### 🤔 *Why results dipped*

* JSON validity dropped to **77.50%**, lowest in the set.
* Accuracy only marginally better than baseline.

Reflex fails here for two reasons:

1. **Free-tier models can’t reliably critique their own JSON formatting** — they mutate the structure.
2. **Error cascades**: if step 1 is weak, step 2 amplifies mistakes instead of fixing them.

### 🧠 *Takeaway*

Reflex shines with larger foundation models that excel at boundary enforcement and error checking.
On smaller or constrained free-tier models, it performs no better than baseline.

---

# 📈 **Visualization Summary**

## **Overall Performance (Accuracy & JSON Validity)**

Displayed in your `performance_plot.svg`, we clearly see:

* PRM towering in both axes
* Baseline and Reflex clustered closely, with Reflex slightly better accuracy but worse JSON stability

## **Delta Improvement vs Baseline**

Your `delta_plot.svg` further emphasizes:

* **PRM → +12.5% accuracy, +20% JSON validity**
* **Reflex → +1.5% accuracy, -2.5% JSON validity**

PRM is the only method that strictly dominates baseline in both metrics.

---

# 🧠 **Reliability & Consistency Analysis**

| Method       | Accuracy | JSON Validity | Reliability Summary                                      |
| ------------ | -------- | ------------- | -------------------------------------------------------- |
| **Baseline** | 47.0%    | 80.0%         | Reasonably consistent format, but inconsistent reasoning |
| **PRM**      | 59.5%    | 100.0%        | Highly reliable, stable JSON, improved predictions       |
| **Reflex**   | 48.5%    | 77.5%         | Unstable multi-step reasoning, frequent JSON corruption  |

### 💬 Qualitative Reliability Score

* **Baseline:** Moderate
* **PRM:** **High**
* **Reflex:** Low (due to structural failures in JSON)

PRM is objectively the most production-ready method on free-tier models.

---

# 🏁 Final Conclusions

### 🎯 **Best Method: PRM**

Because it meaningfully increases accuracy while perfecting JSON validity.

### 🛑 **Worst Method: Reflex**

Because self-criticism requires a more capable model—free-tier doesn’t cut it.

### ⚖️ **Baseline**

Sufficient for quick tests, not for reliable deployment.
