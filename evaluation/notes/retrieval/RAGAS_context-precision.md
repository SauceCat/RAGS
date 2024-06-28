# RAGAS: Context Precision

- **Dimension:** Retrieved Context <-> Ground Truth Answer, Question
- **Reference:** https://docs.ragas.io/en/stable/concepts/metrics/context_precision.html
- **Type:** LLM-as-a-judge, IR Metrics

Context Precision assesses if relevant items from ground truth are ranked higher in the retrieved contexts.

## Calculation
### Step 1: Retrieved context relevancy
For each chunk in retrieved context, check if it is relevant to arrive at the ground truth for the given question.

```python
class ContextPrecision(MetricWithLLM):
    def _context_precision_prompt(self, row: t.Dict) -> t.List[PromptValue]:
        # get the prompt per context chunk
        question, contexts, answer = self._get_row_attributes(row)
        return [
            self.context_precision_prompt.format(
                question=question, context=c, answer=answer
            )
            for c in contexts
        ]
```

instruction:

```
Given question, answer and context verify if the context was useful in arriving at the given answer. Give verdict as "1" if useful and "0" if not with json output.
```

sample example:
```python
{
    "question": """who won 2020 icc world cup?""",
    "context": """The 2022 ICC Men's T20 World Cup, held from October 16 to November 13, 2022, in Australia, was the eighth edition of the tournament. Originally scheduled for 2020, it was postponed due to the COVID-19 pandemic. England emerged victorious, defeating Pakistan by five wickets in the final to clinch their second ICC Men's T20 World Cup title.""",
    "answer": """England""",
    "verification": ContextPrecisionVerification(
        reason="the context was useful in clarifying the situation regarding the 2020 ICC World Cup and indicating that England was the winner of the tournament that was intended to be held in 2020 but actually took place in 2022.",
        verdict=1,
    ).dict(),
}
```

### Step 2: Precision@k
Calculate the mean of `precision@k` to arrive at the final context precision score.

```python
class ContextPrecision(MetricWithLLM):
    def _calculate_average_precision(
        self, verifications: t.List[ContextPrecisionVerification]
    ) -> float:
        score = np.nan

        verdict_list = [1 if ver.verdict else 0 for ver in verifications]
        denominator = sum(verdict_list) + 1e-10
        numerator = sum(
            [
                (sum(verdict_list[: i + 1]) / (i + 1)) * verdict_list[i]
                for i in range(len(verdict_list))
            ]
        )
        score = numerator / denominator
        if np.isnan(score):
            logger.warning(
                "Invalid response format. Expected a list of dictionaries with keys 'verdict'"
            )
        return score
```
