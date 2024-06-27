# BERTScore (Generated Answer <-> Ground Truth Answer)
Reference: https://huggingface.co/spaces/evaluate-metric/bertscore

BERTScore leverages the pre-trained contextual embeddings from BERT (Bidirectional Encoder Representations from Transformers) and matches words in candidate and reference sentences by cosine similarity. It has been shown to correlate with human judgment on sentence-level and system-level evaluation. Moreover, BERTScore computes precision, recall, and F1 measure, which can be useful for evaluating different language generation tasks.

BERTScore provides a nuanced evaluation by capturing semantic similarities using contextual embeddings, making it more robust compared to traditional word overlap metrics like BLEU or ROUGE.

## Calculation
Core concepts:
- **Embeddings**: BERTScore uses embeddings from BERT to represent words in a high-dimensional space. These embeddings capture the context in which words appear, making the comparisons more semantically meaningful.
- **Precision, Recall, and F1 Score**: Similar to other evaluation metrics, BERTScore computes precision, recall, and an F1 score based on the similarity of embeddings.

Example:
- **Reference Sentence**: "The cat sat on the mat."
- **Candidate Sentence**: "A cat is sitting on a mat."

### Step 1: Tokenization and Embedding
The reference and candidate sentences are tokenized, and each token is mapped to its corresponding BERT embedding.
- Reference Tokens: [The, cat, sat, on, the, mat]
- Candidate Tokens: [A, cat, is, sitting, on, a, mat]
- Each token is converted into its BERT embedding.

### Step 2: Similarity Calculation
The cosine similarity is calculated between each token in the candidate sentence and each token in the reference sentence. Here's a simplified similarity matrix (not actual values):

|          | The | cat | sat | on  | the | mat |
|----------|-----|-----|-----|-----|-----|-----|
| A        | 0.1 | 0.2 | 0.1 | 0.3 | 0.1 | 0.1 |
| cat      | 0.1 | 1.0 | 0.2 | 0.4 | 0.1 | 0.2 |
| is       | 0.2 | 0.3 | 0.2 | 0.2 | 0.2 | 0.1 |
| sitting  | 0.3 | 0.4 | 0.9 | 0.5 | 0.3 | 0.4 |
| on       | 0.3 | 0.4 | 0.3 | 1.0 | 0.3 | 0.3 |
| a        | 0.2 | 0.3 | 0.2 | 0.2 | 0.2 | 0.1 |
| mat      | 0.2 | 0.3 | 0.2 | 0.2 | 0.2 | 1.0 |

### Step 3: Matching and Score Calculation
For each token in the candidate sentence, the highest similarity score with any token in the reference sentence is chosen. This forms the basis for calculating precision and recall. The precision and recall scores are averaged across all tokens to produce an overall precision and recall score for the sentence. The F1 score is the harmonic mean of these precision and recall scores.

**Precision**: For each token in the candidate, find the maximum similarity with any token in the reference.
- A -> max(0.1, 0.2, 0.1, 0.3, 0.1, 0.1) = 0.3 -> on
- cat -> max(0.1, 1.0, 0.2, 0.4, 0.1, 0.2) = 1.0 -> cat
- is -> max(0.2, 0.3, 0.2, 0.2, 0.2, 0.1) = 0.3 -> cat
- sitting -> max(0.3, 0.4, 0.9, 0.5, 0.3, 0.4) = 0.9 -> sat
- on -> max(0.3, 0.4, 0.3, 1.0, 0.3, 0.3) = 1.0 -> on
- a -> max(0.2, 0.3, 0.2, 0.2, 0.2, 0.1) = 0.3 -> cat
- mat -> max(0.2, 0.3, 0.2, 0.2, 0.2, 1.0) = 1.0 -> mat
- **Precision** = (0.3 + 1.0 + 0.3 + 0.9 + 1.0 + 0.3 + 1.0) / 7 ≈ 0.686

**Recall**: For each token in the reference, find the maximum similarity with any token in the candidate.
- The -> max(0.1, 0.1, 0.2, 0.3, 0.3, 0.2, 0.2) = 0.3 -> sitting/on
- cat -> max(0.2, 1.0, 0.3, 0.4, 0.4, 0.3, 0.3) = 1.0 -> cat
- sat -> max(0.1, 0.2, 0.2, 0.9, 0.3, 0.2, 0.2) = 0.9 -> sitting
- on -> max(0.3, 0.4, 0.2, 0.5, 1.0, 0.2, 0.2) = 1.0 -> on
- the -> max(0.1, 0.1, 0.2, 0.3, 0.3, 0.2, 0.2) = 0.3 -> sitting/on
- mat -> max(0.1, 0.2, 0.1, 0.4, 0.3, 0.1, 1.0) = 1.0 -> mat
- **Recall** = (0.3 + 1.0 + 0.9 + 1.0 + 0.3 + 1.0) / 6 ≈ 0.75

**F1 Score**:
F1 = 2 * (Precision * Recall) / (Precision + Recall)
    ≈ 2 * (0.686 * 0.75) / (0.686 + 0.75)
    ≈ 0.716
