# ROUGE (Generated Answer <-> Ground Truth Answer)
Reference: https://huggingface.co/spaces/evaluate-metric/rouge

ROUGE (Recall-Oriented Understudy for Gisting Evaluation) is a set of metrics and a software package used for evaluating automatic summarization and machine translation software in natural language processing. The metrics compare an automatically produced summary or translation against a reference or a set of references (human-produced) summary or translation. 

The following five evaluation metrics are available.
- ROUGE-N: Overlap of n-grams between the system and reference summaries.
- ROUGE-L: Longest Common Subsequence (LCS) based statistics. Longest common subsequence problem takes into account sentence-level structure similarity naturally and identifies longest co-occurring in sequence n-grams automatically.
- ROUGE-W: Weighted LCS-based statistics that favors consecutive LCSes.
- ROUGE-S: Skip-bigram based co-occurrence statistics. Skip-bigram is any pair of words in their sentence order.
- ROUGE-SU: Skip-bigram plus unigram-based co-occurrence statistics.

## Limitations

## Calculation
Let's go through the steps to calculate ROUGE-L with a concrete example.

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

Precision: 
$$
\text{Precision} = \frac{\text{LCS Length}}{\text{Number of words in machine translation}} 
= \frac{5}{5} = 1.0
$$

Recall:
$$
\text{Recall} = \frac{\text{LCS Length}}{\text{Number of words in reference translation}}
= \frac{5}{6} \approx 0.833
$$

F1-Score:
$$
\text{F1-Score} = \frac{2 \times \text{Precision} \times \text{Recall}}{\text{Precision} + \text{Recall}}
= \frac{2 \times 1.0 \times 0.833}{1.0 + 0.833}
= \frac{1.666}{1.833} \approx 0.909
$$

### Summary
- LCS Length: 5
- Precision: 1.0
- Recall: 0.833
- F1-Score: 0.909

So, the ROUGE-L score for this example is approximately 0.909.

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

Step 1: Tokenize the Sentences
- Reference: "The", "quick", "brown", "fox", "jumps", "over", "the", "lazy", "dog"
- Machine: "The", "fast", "brown", "fox", "leaps", "over", "the", "lazy", "dog"

Step 2: Find the Longest Common Subsequence (LCS)
- LCS: "The brown fox over the lazy dog"

Step 3: Calculate LCS Length
- LCS Length: 7 words

Step 4: Calculate Precision, Recall, and F1-Score
- Precision: $\text{Precision} = \frac{7}{9} \approx 0.778$
- Recall: $\text{Recall} = \frac{7}{9} \approx 0.778$
- F1-Score: $\text{F1-Score} = \frac{2 \times 0.778 \times 0.778}{0.778 + 0.778}$

In this case, the ROUGE-L score is approximately 0.778.

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
