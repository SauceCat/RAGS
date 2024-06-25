# RAGAS: Answer semantic similarity (Generated Answer <-> Answer)
Reference: https://docs.ragas.io/en/stable/concepts/metrics/semantic_similarity.html

The concept of Answer Semantic Similarity pertains to the assessment of the semantic resemblance between the generated answer and the ground truth. 

## Calculation
class: `AnswerSimilarity`

1. Vectorize the ground truth answer using the specified embedding model.
2. Vectorize the generated answer using the same embedding model.
3. Compute the cosine similarity between the two vectors.

```python
class AnswerSimilarity(MetricWithLLM, MetricWithEmbeddings):
    async def _ascore(
        self: t.Self, row: t.Dict, callbacks: Callbacks, is_async: bool
    ) -> float:
        assert self.embeddings is not None, "embeddings must be set"

        ground_truth = t.cast(str, row["ground_truth"])
        answer = t.cast(str, row["answer"])

        if self.is_cross_encoder and isinstance(self.embeddings, HuggingfaceEmbeddings):
            raise NotImplementedError(
                "async score [ascore()] not implemented for HuggingFace embeddings"
            )
        else:
            embedding_1 = np.array(await self.embeddings.embed_text(ground_truth))
            embedding_2 = np.array(await self.embeddings.embed_text(answer))
            # Normalization factors of the above embeddings
            norms_1 = np.linalg.norm(embedding_1, keepdims=True)
            norms_2 = np.linalg.norm(embedding_2, keepdims=True)
            embedding_1_normalized = embedding_1 / norms_1
            embedding_2_normalized = embedding_2 / norms_2
            similarity = embedding_1_normalized @ embedding_2_normalized.T
            score = similarity.flatten()

        assert isinstance(score, np.ndarray), "Expects ndarray"
        if self.threshold:
            score = score >= self.threshold

        return score.tolist()[0]
```
