# RAGAS: Context Relevance (Retrieved Context <-> Question)
Reference: https://docs.ragas.io/en/stable/concepts/metrics/context_relevancy.html

$$
\text{Context Relevancy} =
\frac{|S|}{\text{Total number of sentences in the retrieved context}}
$$

where $|S|$ is the number of sentences extracted from the context that are relevant to the question.

## Comments

Extract sentences from the original context before LLM evaluation should be a more reliable approach. This method ensures:
- Preservation of the original text.
- Prevention of unintended alterations by the LLM, even if instructed not to change sentences. Following the instruction "While extracting candidate sentences you're not allowed to make any changes to sentences from given context." might be not necessary hard.

It's better to shift from sentence-level to chunk-level evaluation. Sentence-level judgement heavily relies on chunking strategies, large chunks containing a single-sentence answer may receive unfairly low scores. 

## Calculation
class: `ContextRelevancy`

### Step 1
Use LLM to extract sentences from the context that are relevant to the question.

```python
class ContextRelevancy(MetricWithLLM):
    async def _ascore(self, row: t.Dict, callbacks: Callbacks, is_async: bool) -> float:
        assert self.llm is not None, "LLM is not initialized"

        if self.show_deprecation_warning:
            logger.warning(
                "The 'context_relevancy' metric is going to be deprecated soon! Please use the 'context_precision' metric instead. It is a drop-in replacement just a simple search and replace should work."  # noqa
            )

        question, contexts = row["question"], row["contexts"]
        result = await self.llm.generate(
            self.context_relevancy_prompt.format(
                question=question, context="\n".join(contexts)
            ),
            callbacks=callbacks,
        )
        return self._compute_score(result.generations[0][0].text, row)
```

instruction:

```
Please extract relevant sentences from the provided context that is absolutely required answer the following question. If no relevant sentences are found, or if you believe the question cannot be answered from the given context, return the phrase "Insufficient Information".  While extracting candidate sentences you're not allowed to make any changes to sentences from given context.
```

### Step 2
Calculate the relevancy score by extracting sentences from both the response and the context.

```python
seg = pysbd.Segmenter(language="en", clean=False)
def sent_tokenize(text: str) -> List[str]:
    """
    tokenizer text into sentences
    """
    sentences = seg.segment(text)
    assert isinstance(sentences, list)
    return sentences


class ContextRelevancy(MetricWithLLM):
    def _compute_score(self, response: str, row: t.Dict) -> float:
        context = "\n".join(row["contexts"])
        context_sents = sent_tokenize(context)
        indices = (
            sent_tokenize(response.strip())
            if response.lower() != "insufficient information."
            else []
        )
        if len(context_sents) == 0:
            return 0
        else:
            return min(len(indices) / len(context_sents), 1)
```