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
- Prevention of unintended alterations by the LLM, even if instructed not to change sentences.

It's better to shift from sentence-level to chunk-level evaluation. Sentence-level judgement heavily relies on chunking strategies, large chunks containing a single-sentence answer may receive unfairly low scores. 

## Code and Prompts

```python
@dataclass
class ContextRelevancy(MetricWithLLM):
    """
    Extracts sentences from the context that are relevant to the question with
    self-consistency checks. The number of relevant sentences and is used as the score.

    Attributes
    ----------
    name : str
    """

    name: str = "context_relevancy"  # type: ignore
    evaluation_mode: EvaluationMode = EvaluationMode.qc  # type: ignore
    context_relevancy_prompt: Prompt = field(default_factory=lambda: CONTEXT_RELEVANCE)
    show_deprecation_warning: bool = False

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

    def adapt(self, language: str, cache_dir: str | None = None) -> None:
        assert self.llm is not None, "set LLM before use"

        logger.info(f"Adapting Context Relevancy to {language}")
        self.context_relevancy_prompt = self.context_relevancy_prompt.adapt(
            language, self.llm, cache_dir
        )

    def save(self, cache_dir: str | None = None) -> None:
        self.context_relevancy_prompt.save(cache_dir)
```

`CONTEXT_RELEVANCE`:

```python
CONTEXT_RELEVANCE = Prompt(
    name="context_relevancy",
    instruction="""Please extract relevant sentences from the provided context that is absolutely required answer the following question. If no relevant sentences are found, or if you believe the question cannot be answered from the given context, return the phrase "Insufficient Information".  While extracting candidate sentences you're not allowed to make any changes to sentences from given context.""",
    input_keys=["question", "context"],
    output_key="candidate sentences",
    output_type="json",
)
```
