# RAGAS: Context Recall (Retrieved Context <-> Ground Truth Answer)
Reference: https://docs.ragas.io/en/stable/concepts/metrics/context_recall.html

Evaluates if each sentence in the ground truth answer can be traced back to the retrieved context. Ideal scenario: Every sentence in the ground truth answer is attributable to the retrieved context.

## Comments
The current methodology, which involves breaking the answer into statements and judging each statement within a single prompt, may be overly complex. A more effective approach could be to first extract the statements from the answer and then evaluate each statement individually.

## Calculation
class: `ContextRecall`

### Step 1
Break the ground truth answer into individual statements.

Statements:
- Statement 1: "France is in Western Europe."
- Statement 2: "Its capital is Paris."

For each of the ground truth statements, verify if it can be attributed to the retrieved context using LLM-as-a-judge.

Statements:
- Statement 1: Yes
- Statement 2: No


```python
class ContextRecall(MetricWithLLM):
    def _create_context_recall_prompt(self, row: t.Dict) -> PromptValue:
        qstn, ctx, gt = row["question"], row["contexts"], row["ground_truth"]
        ctx = "\n".join(ctx) if isinstance(ctx, list) else ctx

        return self.context_recall_prompt.format(question=qstn, context=ctx, answer=gt)
```

Here, we use prompt with a few examples:

instruction:

```
Given a context, and an answer, analyze each sentence in the answer and classify if the sentence can be attributed to the given context or not. Use only "Yes" (1) or "No" (0) as a binary classification. Output json with reason.
```

sample example:
```python
{
    "question": """What can you tell me about albert Albert Einstein?""",
    "context": """Albert Einstein (14 March 1879 - 18 April 1955) was a German-born theoretical physicist, widely held to be one of the greatest and most influential scientists of all time. Best known for developing the theory of relativity, he also made important contributions to quantum mechanics, and was thus a central figure in the revolutionary reshaping of the scientific understanding of nature that modern physics accomplished in the first decades of the twentieth century. His mass-energy equivalence formula E = mc2, which arises from relativity theory, has been called 'the world's most famous equation'. He received the 1921 Nobel Prize in Physics 'for his services to theoretical physics, and especially for his discovery of the law of the photoelectric effect', a pivotal step in the development of quantum theory. His work is also known for its influence on the philosophy of science. In a 1999 poll of 130 leading physicists worldwide by the British journal Physics World, Einstein was ranked the greatest physicist of all time. His intellectual achievements and originality have made Einstein synonymous with genius.""",
    "answer": """Albert Einstein born in 14 March 1879 was  German-born theoretical physicist, widely held to be one of the greatest and most influential scientists of all time. He received the 1921 Nobel Prize in Physics for his services to theoretical physics. He published 4 papers in 1905.  Einstein moved to Switzerland in 1895""",
    "classification": ContextRecallClassificationAnswers.parse_obj(
        [
            {
                "statement": "Albert Einstein, born on 14 March 1879, was a German-born theoretical physicist, widely held to be one of the greatest and most influential scientists of all time.",
                "reason": "The date of birth of Einstein is mentioned clearly in the context.",
                "attributed": 1,
            },
            {
                "statement": "He received the 1921 Nobel Prize in Physics for his services to theoretical physics.",
                "reason": "The exact sentence is present in the given context.",
                "attributed": 1,
            },
            {
                "statement": "He published 4 papers in 1905.",
                "reason": "There is no mention about papers he wrote in the given context.",
                "attributed": 0,
            },
            {
                "statement": "Einstein moved to Switzerland in 1895.",
                "reason": "There is no supporting evidence for this in the given context.",
                "attributed": 0,
            },
        ]
    ).dicts(),
}
```

### Step 2
Use the formula depicted above to calculate context recall.

```python
class ContextRecall(MetricWithLLM):
    def _compute_score(self, response: t.Any) -> float:
        response = [1 if item.attributed else 0 for item in response.__root__]
        denom = len(response)
        numerator = sum(response)
        score = numerator / denom if denom > 0 else np.nan

        if np.isnan(score):
            logger.warning("The LLM did not return a valid classification.")

        return score
```