# BLEU (Generated Answer <-> Ground Truth Answer)
Reference: https://huggingface.co/spaces/evaluate-metric/bleu

BLEU (Bilingual Evaluation Understudy) is a metric used to evaluate the quality of text which has been machine-translated from one language to another. It compares the machine's output to one or more reference translations. The BLEU score ranges from 0 to 1, with 1 being a perfect match.

## Limitations
BLEU's primary drawback for evaluating generated answers is its focus on token-level accuracy, which is less important than overall semantic correctness. Variations in wording are acceptable as long as the intended meaning is preserved.

## Calculation

Here's a step-by-step explanation with a concrete example:

### Step 1: Tokenize the Sentences
First, break down the sentences into individual words (tokens).
- Reference Translation: "The cat is on the mat."
- Machine Translation: "The cat is on mat."

### Step 2: Count n-grams
An n-gram is a contiguous sequence of n items from a given sample of text. BLEU typically uses unigrams (1-gram), bigrams (2-grams), trigrams (3-grams), and sometimes 4-grams.

**Unigrams:**
- Reference: "The", "cat", "is", "on", "the", "mat"
- Machine: "The", "cat", "is", "on", "mat"

**Bigrams:**
- Reference: "The cat", "cat is", "is on", "on the", "the mat"
- Machine: "The cat", "cat is", "is on", "on mat"

### Step 3: Count Matches
Count the number of n-gram matches between the machine translation and the reference translation.

**Unigram Matches:**
- "The" (1 match)
- "cat" (1 match)
- "is" (1 match)
- "on" (1 match)
- "mat" (1 match)

**Bigram Matches:**
- "The cat" (1 match)
- "cat is" (1 match)
- "is on" (1 match)
- "on mat" (0 matches)

**Step 4: Calculate Precision**
Precision is calculated for each n-gram level by dividing the number of matches by the total number of n-grams in the machine translation.

**Unigram Precision:**
- Matches: 5
- Total unigrams in machine translation: 5
- Precision: 5/5 = 1.0

**Bigram Precision:**
- Matches: 3
- Total bigrams in machine translation: 4
- Precision: 3/4 = 0.75

### Step 5: Calculate the Geometric Mean of Precisions
BLEU score uses the geometric mean of the precisions for different n-gram levels. For simplicity, let's consider up to bigrams.

```
Geometric Mean = sqrt(Unigram Precision * Bigram Precision) = sqrt(1.0 * 0.75) = sqrt(0.75) ≈ 0.866
```

### Step 6: Apply a Brevity Penalty
The brevity penalty (BP) is applied to penalize translations that are too short. It is calculated as follows:
- BP = 1 if the length of the machine translation is greater than or equal to the reference translation. 
- BP = exp(1 - reference length / machine length) if the machine translation is shorter.

In this case:
- Reference length: 6 words
- Machine length: 5 words
- BP = exp(1 - 6/5) = exp(1 - 1.2) = exp(-0.2) ≈ 0.818

### Step 7: Calculate the Final BLEU Score
Finally, multiply the geometric mean by the brevity penalty.

```
BLEU Score = Geometric Mean * BP = 0.866 * 0.818 ≈ 0.709
```

### Reproduce using nltk
```python
import nltk
hypothesis = ['The', 'cat', 'is', 'on', 'mat']
reference = ['The', 'cat', 'is', 'on', 'the', 'mat']
BLEUscore = nltk.translate.bleu_score.sentence_bleu([reference], hypothesis, weights=(0.5, 0.5))
print(BLEUscore)
# 0.7090416310250969
```