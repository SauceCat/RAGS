# RAGAS: Aspect Critique

- **Dimension:** Generated Answer
- **Reference:** https://docs.ragas.io/en/stable/concepts/metrics/critique.html
- **Type:** LLM-as-a-judge
- **Comment:** Defining objective criteria is challenging.

It enables a flexible, criteria-based evaluation of generated answers. Users define an assessment criterion by formulating a yes/no question. Then an LLM will act as a judge to analyzes the answer based solely on this specified criterion. 

Example Criteria:
- Does the response contain misinformation?
- Is the text grammatically flawless?
- Could the content potentially harm individuals or society?

## Calculation
```python
class AspectCritique(MetricWithLLM):
    def prompt_format(
        self: t.Self,
        question: str,
        answer: str,
        context: t.Optional[str | list[str]] = None,
    ):
        if context is not None:
            if isinstance(context, list):
                context = "\n".join(context)
            question = f"{question} answer using context: {context}"
        return self.critic_prompt.format(
            input=question, submission=answer, criteria=self.definition
        )

    async def _ascore(
        self: t.Self, row: t.Dict, callbacks: Callbacks, is_async: bool
    ) -> float:
        assert self.llm is not None, "set LLM before use"

        q, c, a = row["question"], row["contexts"], row["answer"]

        p_value = self.prompt_format(q, a, c)
        result = await self.llm.generate(
            p_value, callbacks=callbacks, is_async=is_async
        )
```

instruction:

```
Given a input and submission. Evaluate the submission only using the given criteria. Use only 'Yes' (1) and 'No' (0) as verdict.
```

example:
```python
{
    "input": "Who was the director of Los Alamos Laboratory?",
    "submission": "Einstein was the director of Los Alamos Laboratory.",
    "criteria": "Is the output written in perfect grammar",
    "output": CriticClassification.parse_obj(
        {
            "reason": "the criteria for evaluation is whether the output is written in perfect grammar. In this case, the output is grammatically correct.",
            "verdict": 1,
        }
    ).dict(),
}
```