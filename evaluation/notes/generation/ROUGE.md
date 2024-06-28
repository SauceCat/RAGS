# ROUGE

- **Dimension:** Generated Answer <-> GroundTruth Answer
- **Reference:** https://huggingface.co/spaces/evaluate-metric/rouge
- **Type:** Token-wise Accuracy

ROUGE (Recall-Oriented Understudy for Gisting Evaluation) is a set of metrics and a software package used for evaluating automatic summarization and machine translation software in natural language processing. The metrics compare an automatically produced summary or translation against a reference or a set of references (human-produced) summary or translation. 

The following five evaluation metrics are available.
- ROUGE-N: Overlap of n-grams between the system and reference summaries.
- ROUGE-L: Longest Common Subsequence (LCS) based statistics. Longest common subsequence problem takes into account sentence-level structure similarity naturally and identifies longest co-occurring in sequence n-grams automatically.
- ROUGE-W: Weighted LCS-based statistics that favors consecutive LCSes.
- ROUGE-S: Skip-bigram based co-occurrence statistics. Skip-bigram is any pair of words in their sentence order.
- ROUGE-SU: Skip-bigram plus unigram-based co-occurrence statistics.

## Calculation
Let's go through the steps to calculate **ROUGE-L** with a concrete example.

### Step 1: Tokenize the Sentences
First, break down the sentences into individual words (tokens).
- Reference Translation: "The cat is on the mat."
- Machine Translation: "The cat is on mat."

### Step 2: Find the Longest Common Subsequence (LCS)
Identify the longest sequence of words that appear in both the reference and machine translations in the same order.

LCS: "The cat is on mat" and "The cat is on the mat" share the LCS: "The cat is on mat"

### Step 3: Calculate LCS Length
Count the number of words in the LCS.
- LCS: "The cat is on mat"
- Length: 5 words

### Step 4: Calculate Precision, Recall, and F1-Score
Using the LCS length, calculate precision, recall, and F1-score.

- Precision: `LCS Length / Number of words in machine translation = 5 / 5 = 1.0`
- Recall: `LCS Length / Number of words in reference translation = 5 / 6 = 0.833`
- F1-Score: `2 * Precision * Recall / (Precision + Recall) = 2 * 1.0 * 0.833 / (1.0 + 0.833) = 0.909`

So, the ROUGE-L score for this example is approximately 0.909. Reproduce using `rouge_score`:

```python
from rouge_score import rouge_scorer

scorer = rouge_scorer.RougeScorer(['rougeL'], use_stemmer=True)
scores = scorer.score(
    target="The cat is on the mat.", 
    prediction="The cat is on mat."
)
print(scores)
# {'rougeL': Score(precision=1.0, recall=0.8333333333333334, fmeasure=0.9090909090909091)}
```

### Another example
Let's consider another example to illustrate the calculation further.
- Reference Translation: "The quick brown fox jumps over the lazy dog."
- Machine Translation: "The fast brown fox leaps over the lazy dog."

**Step 1: Tokenize the Sentences**
- Reference: "The", "quick", "brown", "fox", "jumps", "over", "the", "lazy", "dog"
- Machine: "The", "fast", "brown", "fox", "leaps", "over", "the", "lazy", "dog"

**Step 2: Find the Longest Common Subsequence (LCS)**
- LCS: "The brown fox over the lazy dog"

**Step 3: Calculate LCS Length**
- LCS Length: 7 words

**Step 4: Calculate Precision, Recall, and F1-Score**
- Precision: `7 / 9 = 0.778`
- Recall: `7 / 9 = 0.778`
- F1-Score: `(2 * 0.778 * 0.778) / (0.778 + 0.778) = 0.778`

In this case, the ROUGE-L score is approximately 0.778. Reproduce using `rouge_score`:

```python
from rouge_score import rouge_scorer

scorer = rouge_scorer.RougeScorer(['rougeL'], use_stemmer=True)
scores = scorer.score(
    target="The quick brown fox jumps over the lazy dog.", 
    prediction="The fast brown fox leaps over the lazy dog."
)
print(scores)
# {'rougeL': Score(precision=0.7777777777777778, recall=0.7777777777777778, fmeasure=0.7777777777777778)}
```

## Limitations
Check the **Limitations and Bias** session in the reference.