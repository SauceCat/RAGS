# Haystack: Diversity

- **Dimension:** Retrieved Context
- **Reference:** [Enhancing RAG Pipelines in Haystack: Introducing DiversityRanker and LostInTheMiddleRanker](https://towardsdatascience.com/enhancing-rag-pipelines-in-haystack-45f14e2bc9f5)
- **Type:** Semantic Similarity
- **Comment:**
  - Diversity alone does not guarantee quality or relevance. Therefore, combining diversity with quality filtering is crucial. The primary focus should be on achieving highly relevant and diverse context.
  - This approach is particularly important in scenarios where the question requires multiple context chunks, either due to small chunk sizes or the inherent complexity of the query necessitating references to multiple contexts.

The evaluation process was simplified to focus on assessing the `DiversityRanker`'s impact. This was achieved by calculating the mean pairwise cosine distance between context documents inserted into the LLM's context.