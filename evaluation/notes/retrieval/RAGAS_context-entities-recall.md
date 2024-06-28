# RAGAS: Context Entity Recall

- **Dimension:** Retrieved Context <-> GroundTruth Answer
- **Reference:** https://docs.ragas.io/en/stable/concepts/metrics/context_entities_recall.html
- **Type:** LLM-as-a-tool, Entity Recall

Context Entity Recall measures the proportion of ground truth entities present in the retrieved context. Particularly useful for fact-based applications (e.g., tourism help desks, historical Q&A).

## Calculation
### Step 1: Extract entities
Using LLM to find entities present in the ground truths and in the context.

```python
class ContextEntityRecall(MetricWithLLM):
    async def get_entities(
        self,
        text: str,
        callbacks: Callbacks,
        is_async: bool,
    ) -> t.Optional[ContextEntitiesResponse]:
        assert self.llm is not None, "LLM is not initialized"
        p_value = self.context_entity_recall_prompt.format(
            text=text,
        )
        result = await self.llm.generate(
            prompt=p_value,
            callbacks=callbacks,
            is_async=is_async,
        )

        result_text = result.generations[0][0].text
        answer = await _output_parser.aparse(
            result_text, p_value, self.llm, self.max_retries
        )
        if answer is None:
            return ContextEntitiesResponse(entities=[])

        return answer

    async def _ascore(
        self,
        row: Dict,
        callbacks: Callbacks,
        is_async: bool,
    ) -> float:
        ground_truth, contexts = row["ground_truth"], row["contexts"]
        ground_truth = await self.get_entities(
            ground_truth, callbacks=callbacks, is_async=is_async
        )
        contexts = await self.get_entities(
            "\n".join(contexts), callbacks=callbacks, is_async=is_async
        )
        if ground_truth is None or contexts is None:
            return np.nan
        return self._compute_score(ground_truth.entities, contexts.entities)
```

instruction:

```
Given a text, extract unique entities without repetition. Ensure you consider different forms or mentions of the same entity as a single entity.
```

sample example:
```python
{
    "text": """The Eiffel Tower, located in Paris, France, is one of the most iconic landmarks globally.
    Millions of visitors are attracted to it each year for its breathtaking views of the city.
    Completed in 1889, it was constructed in time for the 1889 World's Fair.""",
    "output": ContextEntitiesResponse.parse_obj(
        {
            "entities": [
                "Eiffel Tower",
                "Paris",
                "France",
                "1889",
                "World's Fair",
            ],
        }
    ).dict(),
}
```

### Step 2: Calculate context entity recall

Context Entity recall = `| CN ∩ GN | / | GN |`

```python
class ContextEntityRecall(MetricWithLLM):
    def _compute_score(
        self, ground_truth_entities: t.Sequence[str], context_entities: t.Sequence[str]
    ) -> float:
        num_entities_in_both = len(
            set(context_entities).intersection(set(ground_truth_entities))
        )
        return num_entities_in_both / (len(ground_truth_entities) + 1e-8)
```