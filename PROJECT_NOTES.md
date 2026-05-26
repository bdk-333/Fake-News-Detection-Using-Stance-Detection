# Fake News Stance Detection - Project Notes

## Project Overview

**Objective:** Classification model to predict whether a headline's stance is supported by the article body.

**Task Type:** Multi-class classification (4 classes)

**Dataset Size:** ~75,000 rows (estimated based on class distribution)

**Text Features:** `headline`, `body`, `stance`

---

## EDA Strategy

### Dataset Characteristics

- **Total samples:** ~75,000 rows (approximately)
- **Class Distribution (Imbalanced):**
  - `agree`: ~5,500 (7.3%)
  - `disagree`: ~1,500 (2%)
  - `discuss`: ~13,000 (17.3%)
  - `unrelated`: ~55,000 (73.3%)

**Important Note:** Unrelated samples were intentionally created by the dataset author through random headline-body matching to make the task more challenging. This is an artificial imbalance by design.

---

## Stance Categories Explained

| Stance | Definition | Example |
|--------|-----------|---------|
| **agree** | Body directly confirms/supports what headline claims | Headline: "Apple bought Nvidia" → Body: "Apple bought Nvidia for $1T, deal to be completed entirely by the end of Nov 2029" |
| **disagree** | Body contradicts or opposes headline's claim | Body says opposite of what headline claims |
| **discuss** | Overall theme is same but body doesn't take explicit stance | Headline & body discuss same topic (e.g., Apple & Nvidia) but body is speculative/exploratory (e.g., "wouldn't it be beneficial if Apple bought Nvidia?") |
| **unrelated** | Topics in headline and body are completely different | Randomly matched headline-body pairs with no thematic connection |

---

## Comprehensive EDA Roadmap

### 1. **Class Imbalance Analysis** (CRITICAL)

**Why it matters:**
- Simple baseline achieves ~55% accuracy by always predicting "unrelated"
- Extreme imbalance requires special handling in modeling
- Different evaluation metrics needed

**Analysis tasks:**
- [ ] Stacked bar chart: absolute counts vs. percentages
- [ ] Log-scale bar chart (to visualize minority classes)
- [ ] Pie chart showing proportions
- [ ] Document imbalance ratio (unrelated : related = 55:45)

**Expected findings:**
- Unrelated class dominates due to artificial creation
- Agree is rarest legitimate class
- Need for stratified cross-validation and class weighting

---

### 2. **Text Length & Structure Analysis**

**For both headline and body:**

**Metrics to compute:**
- [ ] Character count distribution
- [ ] Word count distribution
- [ ] Sentence count distribution
- [ ] Average/median values by stance
- [ ] Outliers and extreme values

**Visualizations:**
- [ ] Box plots of text length by stance (separate for headline/body)
- [ ] Violin plots to show distribution shape
- [ ] Scatter plots: headline length vs. body length, colored by stance

**Expected patterns:**
- `agree`: Likely longer bodies with repetition of key terms
- `discuss`: Moderate-to-long bodies with exploratory language
- `disagree`: Longer bodies to present counterargument
- `unrelated`: Random length distribution

---

### 3. **Vocabulary & Lexical Overlap Analysis** (KEY FEATURE)

**Overlap metrics to calculate:**
- [ ] Unique words in headline
- [ ] Unique words in body
- [ ] Words appearing in BOTH headline and body (intersection)
- [ ] Overlap ratio: `(words in both) / (total unique words)` per sample
- [ ] Overlap ratio: `(words in both) / (headline words)` (headline coverage)
- [ ] Overlap ratio: `(words in both) / (body words)` (body coverage)

**N-gram analysis:**
- [ ] Most common bigrams (2-word phrases) by stance
- [ ] Most common trigrams (3-word phrases) by stance
- [ ] Unique n-grams per stance (what distinguishes each class?)

**Vocabulary diversity:**
- [ ] Type-token ratio (unique words / total words) per sample
- [ ] Diversity comparison by stance

**Expected patterns:**
- `agree`: HIGH overlap (key terms repeated in both)
- `disagree`: MODERATE overlap (same topic, different conclusion)
- `discuss`: LOW-MODERATE overlap (related topics but different angles)
- `unrelated`: MINIMAL overlap (random connection)

**Why this matters:** Overlap metrics are strong predictive features for this task

---

### 4. **Sentiment & Entity Analysis**

**Sentiment analysis:**
- [ ] Sentiment polarity of headlines by stance
- [ ] Sentiment polarity of bodies by stance
- [ ] Sentiment agreement: does body sentiment match headline sentiment?
- [ ] Sentiment intensity/magnitude by stance

**Named Entity Recognition (NER):**
- [ ] Entity types: PERSON, ORG, LOCATION, etc.
- [ ] Entity overlap: entities appearing in both headline and body
- [ ] Entity count distribution by stance
- [ ] Do "agree" samples have entity overlap?

**Expected patterns:**
- `agree`/`disagree`: High entity overlap (discussing same entities)
- `discuss`: Moderate entity overlap (same domain, different focus)
- `unrelated`: Minimal-to-no entity overlap

---

### 5. **Linguistic Patterns & Language Indicators**

**Patterns to detect:**
- [ ] Presence of question marks (frequency by stance)
- [ ] Modal verbs (would, could, might, can, should) → suggests uncertainty/discussion
- [ ] Certainty markers (confirms, proves, revealed, denied) → suggests agreement/disagreement
- [ ] Speculative language (reportedly, allegedly, believed to) → suggests discussion
- [ ] Negation words (not, no, never, deny, reject) → suggests disagreement
- [ ] Exclamation marks (intensity)
- [ ] Quotes (how often are headlines vs. bodies quoted?)

**Expected patterns:**
- `discuss`: More modal verbs, fewer certainty markers
- `agree`/`disagree`: More certainty markers
- `disagree`: More negation words than agree

---

### 6. **Data Quality Check**

**Issues to investigate:**
- [ ] Missing values: headline, body, stance columns
- [ ] Duplicate rows (same headline-body pair)
- [ ] Encoding issues or corrupted characters
- [ ] Extremely short texts (< 5 words) - are they valid?
- [ ] Empty or whitespace-only texts
- [ ] Case sensitivity issues (proper nouns)
- [ ] URL patterns in text (common in news data)

**Action items based on findings:**
- Document any data cleaning needed
- Decide on threshold for minimum text length
- Handle duplicates appropriately

---

### 7. **Intentional vs. Real-World Differences**

**Analysis:**
- [ ] Compare statistics for unrelated vs. related samples
- [ ] Are unrelated samples "too easy" to detect? (very different patterns)
- [ ] Statistical tests: are feature distributions significantly different?
- [ ] What makes unrelated samples distinguishable in EDA?

**Why it matters:**
- If unrelated samples are too distinct from real-world unrelated content, model might overfit
- If they're indistinguishable from related samples, the task is very hard

---

## Key Visualizations to Create

### Essential Plots:

1. **Class Distribution**
   - Bar chart with absolute counts
   - Pie chart with percentages
   - Log-scale version to see minority classes

2. **Text Length Distributions**
   - Violin plots: headline length by stance
   - Violin plots: body length by stance
   - Combined scatter plot with size/color encoding

3. **Lexical Overlap Analysis**
   - Histogram: overlap ratio distribution by stance
   - Box plot: overlap ratio by stance
   - Scatter plot: headline length vs. overlap ratio (colored by stance)

4. **N-gram Frequency**
   - Bar charts: top 20 bigrams per stance
   - Word clouds: one per stance class

5. **Sentiment Analysis**
   - Histogram: sentiment polarity by stance
   - Scatter plot: headline sentiment vs. body sentiment (colored by stance)

6. **Entity Overlap**
   - Bar chart: average entity count by stance
   - Bar chart: average entity overlap percentage by stance

7. **Correlation Heatmap**
   - Correlate engineered features with stance class

---

## Feature Engineering Implications

### Hand-crafted Features to Consider:

1. **Overlap Features:**
   - Lexical overlap ratio (intersection / union)
   - Headline coverage: (overlap) / (headline_words)
   - Body coverage: (overlap) / (body_words)
   - Unique words ratio

2. **Length Features:**
   - Headline length, body length, length ratio
   - Sentence count differences

3. **Sentiment Features:**
   - Headline sentiment, body sentiment
   - Sentiment difference/agreement
   - Sentiment polarity magnitude

4. **Entity Features:**
   - Entity overlap count
   - Entity overlap percentage
   - Entity count ratio

5. **Linguistic Features:**
   - Modal verb count
   - Certainty marker count
   - Negation count
   - Question mark presence

6. **Text Statistics:**
   - Type-token ratio (vocabulary diversity)
   - Average word length
   - Punctuation frequency

---

## Modeling Implications & Strategy

### Key Insights from EDA Will Inform:

1. **Class Imbalance Handling:**
   - Use **class weights** in loss function (penalize minority class errors)
   - Consider **SMOTE** or other resampling for training data
   - Use **stratified k-fold** cross-validation
   - **Threshold tuning** for class probability predictions

2. **Evaluation Metrics** (NOT just accuracy):
   - **Macro F1-score** (average F1 across all classes)
   - **Weighted F1-score** (accounts for imbalance)
   - Per-class precision/recall to see if model struggles with minorities
   - **Confusion matrix** to identify which classes are confused
   - **ROC-AUC** (for binary one-vs-rest if needed)

3. **Baseline Model:**
   - Create **rule-based classifier** using overlap metrics
   - Compare neural models against this baseline
   - Example rule: if overlap > 0.7 → agree, if overlap < 0.2 → unrelated, etc.

4. **Model Selection:**
   - **Ensemble methods** (Random Forest, XGBoost) tend to handle imbalance well
   - **Neural networks** with class weights and/or focal loss
   - **SVM** with class weights
   - Compare multiple architectures

5. **Feature Importance:**
   - Track which hand-crafted features are most predictive
   - Consider combining them with deep learning embeddings

6. **Potential Issues to Prepare For:**
   - Easy discrimination of "unrelated" might make other classes harder
   - "Discuss" might be confused with other classes (fuzzy boundary)
   - Real-world data might differ from artificially created unrelated samples

---

## EDA Execution Checklist

- [ ] Load and inspect data (shape, dtypes, missing values)
- [ ] Compute class distribution
- [ ] Create text length statistics
- [ ] Calculate lexical overlap metrics
- [ ] Perform sentiment analysis
- [ ] Extract named entities
- [ ] Identify linguistic patterns
- [ ] Create visualizations
- [ ] Document findings and patterns
- [ ] Engineer candidate features
- [ ] Create summary statistics table
- [ ] Identify data quality issues
- [ ] Plan preprocessing steps

---

## Notes & Observations

*(To be updated as EDA progresses)*

- Dataset is heavily imbalanced by design
- Unrelated class created artificially → might be "too easy" to detect
- High lexical overlap likely strong signal for "agree"
- "Discuss" might be the hardest class due to fuzzy definition
- Consider domain-specific language patterns in news articles
