# Haystack: Diversity

- **Dimension:** Retrieved Context
- **Reference:** [Enhancing RAG Pipelines in Haystack: Introducing DiversityRanker and LostInTheMiddleRanker](https://towardsdatascience.com/enhancing-rag-pipelines-in-haystack-45f14e2bc9f5)
- **Type:** Semantic Similarity
- **Comment:**
  - Diversity alone does not guarantee quality or relevance. Therefore, combining diversity with quality filtering is crucial. The primary focus should be on achieving highly relevant and diverse context.
  - This approach is particularly important in scenarios where the question requires multiple context chunks, either due to small chunk sizes or the inherent complexity of the query necessitating references to multiple contexts.

Diversity focus on assessing the impact of the DiversityRanker on the RAG pipeline. The `DiversityRanker` is a novel component designed to enhance the diversity of the paragraphs selected for the context window in the RAG pipeline. It operates on the principle that a diverse set of documents can increase the LLM's ability to generate answers with more breadth and depth.

The evaluation process was simplified to calculate the mean pairwise cosine distance between context documents inserted into the LLM's context.