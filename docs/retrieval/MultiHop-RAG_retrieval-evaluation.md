# MultiHop-RAG: Retrieval Evaluation (Retrieved Context <-> Ground Truth Context)
Reference: https://www.evidentlyai.com/ranking-metrics/evaluating-recommender-systems

The quality of the retrieval set $R_q$ significantly impacts the final generation quality in RAG systems. To assess this, we compare the retrieved set against the ground truth evidence associated with each query. Assuming we retrieve the top K chunks (i.e., $|R_q| = K$), we employ standard retrieval evaluation metrics.

## Metrics
### Mean Average Precision at K (MAP@K)
It evaluates the average Precision at all relevant ranks within the list of top K retrieval. 

- **Precision@K**: This is the proportion of relevant chunks in the top-K retrieved chunks.
- **Average Precision@K (AP@K)**: This is the average of precision values calculated at the ranks where relevant chunks appear, considering only up to K items.
- **Mean Average Precision@K (MAP@K)**: This is the mean of the AP@K values over all queries.

### Mean Reciprocal Rank at K (MRR@K)
It calculates the average of the reciprocal ranks of the first relevant chunk for each query.

### Hit Rate at K (Hit@K)
It calculates the share of queries for which at least one relevant chunk is present in the K. 

## Example
**Query 1**:
- True relevant chunks: [A, B, C]
- Retrieved chunks: [A, D, B, E, F]

**Query 2**:
- True relevant chunks: [D, E]
- Retrieved chunks: [D, E, F, G, H]

**Query 3**:
- True relevant chunks: [G, H, I]
- Retrieved chunks: [A, B, C, D, E]

Let's calculate the metrics Mean Average Precision at K (MAP@K), Mean Reciprocal Rank at K (MRR@K), and Hit Rate at K (Hit@K) for the given example.

### Mean Average Precision at K (MAP@K)

**Query 1**:
- Precision@1: 1/1 (A is relevant)
- Precision@2: 1/2 (only A is relevant out of top 2)
- Precision@3: 2/3 (A and B are relevant)
- Precision@4: 2/4 (A and B are relevant)
- Precision@5: 2/5 (A and B are relevant)
- Relevant positions: 1 (A), 3 (B)
- AP@5 = (1/1 + 2/3) / 2 = 0.8333

**Query 2**:
- Precision@1: 1/1 (D is relevant)
- Precision@2: 2/2 (D and E are relevant)
- Precision@3: 2/3 (D and E are relevant)
- Precision@4: 2/4 (D and E are relevant)
- Precision@5: 2/5 (D and E are relevant)
- Relevant positions: 1 (D), 2 (E)
- AP@5 = (1/1 + 2/2) / 2 = 1.0

**Query 3**:
- Precision@1: 0/1 (A is not relevant)
- Precision@2: 0/2 (A and B are not relevant)
- Precision@3: 0/3 (A, B, and C are not relevant)
- Precision@4: 0/4 (A, B, C, and D are not relevant)
- Precision@5: 0/5 (A, B, C, D, and E are not relevant)
- No relevant items in the top 5.
- AP@5 = 0.0

**Average**:
MAP@5 = (AP@5 for Query 1 + AP@5 for Query 2 + AP@5 for Query 3) / 3
      = (0.8333 + 1.0 + 0.0) / 3
      = 0.6111

### Mean Reciprocal Rank at K (MRR@K)
**Query 1**: 
- First relevant chunk (A) is at rank 1. 
- Reciprocal Rank = 1/1 = 1.0

**Query 2**: 
- First relevant chunk (D) is at rank 1.
- Reciprocal Rank = 1/1 = 1.0

**Query 3**: 
- There are no relevant chunks in the top 5 retrieved chunks.
- Reciprocal Rank = 0

**Average**: MRR@5 = (1.0 + 1.0 + 0) / 3 = 0.6667

### Hit Rate at K (Hit@K)
**Query 1**: 
- Contains relevant chunks [A, B] in the top 5.
- Hit = 1

**Query 2**: 
- Contains relevant chunks [D, E] in the top 5.
- Hit = 1

**Query 3**: 
- Does not contain relevant chunks [G, H, I] in the top 5.
- Hit = 0

**Average**: Hit@5 = (1 + 1 + 0) / 3 = 2/3 = 0.6667
