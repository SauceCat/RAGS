# ARES: Context Relevance (Retrieved Context <-> Question)
Reference: [ARES: An Automated Evaluation Framework for Retrieval-Augmented Generation Systems](https://arxiv.org/abs/2311.09476)

Fine-tune `DeBERTa-v3-Large` with a binary classifier head for Context Relevance: Is the passage returned relevant for answering the given query? Use a human preference validation set to assess model improvement after each epoch.

## Training Data
Positive Samples: The questions are generated for each passage using `FLAN-T5 XXL`, providing natural positive samples where the passage-question relationship is known.

Negative Samples:
- Weak negatives: Randomly selected in-domain passages unrelated to the synthetic query.
- Strong negatives: In-domain passages from the same document as the gold passage. When multiple passages per document are unavailable, use BM25 to retrieve the top 10 similar passages and sample from these.

For generating synthetic questions, the following prompt is used for `FLAN-T5 XXL`:

```
Example #1
Document: <few-shot example here>
Query: <few-shot example here>
Example #2
Document: <few-shot example here>
Query: <few-shot example here>
Example #3
Document: <few-shot example here>
Query: <few-shot example here>
Example #4
Document: <in-domain passage>
Query:
```

Low-quality queries are filtered by testing if a given query can retrieve its original passage as the top result using its retriever.

