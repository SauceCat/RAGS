# TruLens: Context Relevance (Retrieved Context <-> Question)
To verify the quality of our retrieval, we want to make sure that each chunk of context is relevant to the question. This is critical because this context will be used by the LLM to form an answer, so any irrelevant information in the context could be weaved into a hallucination.

Use LLM-as-a-judge to evaluate the relevance of the context to the question:
- Assigning a score 0-10 based on the given relevance criteria.
- Providing the criteria for the evaluation.
- Providing the reasons for scoring based on the listed criteria step by step.

## Code and Prompts

```python
def context_relevance_with_cot_reasons(
    self,
    question: str,
    context: str,
    temperature: float = 0.0
) -> Tuple[float, Dict]:
    """
    Uses chat completion model. A function that completes a
    template to check the relevance of the context to the question.
    Also uses chain of thought methodology and emits the reasons.

    Args:
        question (str): A question being asked.
        context (str): Context related to the question.

    Returns:
        float: A value between 0 and 1. 0 being "not relevant" and 1 being "relevant".
    """
    system_prompt = prompts.CONTEXT_RELEVANCE_SYSTEM
    user_prompt = str.format(
        prompts.CONTEXT_RELEVANCE_USER, question=question, context=context
    )
    user_prompt = user_prompt.replace(
        "RELEVANCE:", prompts.COT_REASONS_TEMPLATE
    )

    return self.generate_score_and_reasons(
        system_prompt=system_prompt,
        user_prompt=user_prompt,
        temperature=temperature
    )
```

`prompts.CONTEXT_RELEVANCE_SYSTEM`:

```
You are a RELEVANCE grader; providing the relevance of the given CONTEXT to the given QUESTION.
Respond only as a number from 0 to 10 where 0 is the least relevant and 10 is the most relevant. 
A few additional scoring guidelines:
- Long CONTEXTS should score equally well as short CONTEXTS.
- RELEVANCE score should increase as the CONTEXTS provides more RELEVANT context to the QUESTION.
- RELEVANCE score should increase as the CONTEXTS provides RELEVANT context to more parts of the QUESTION.
- CONTEXT that is RELEVANT to some of the QUESTION should score of 2, 3 or 4. Higher score indicates more RELEVANCE.
- CONTEXT that is RELEVANT to most of the QUESTION should get a score of 5, 6, 7 or 8. Higher score indicates more RELEVANCE.
- CONTEXT that is RELEVANT to the entire QUESTION should get a score of 9 or 10. Higher score indicates more RELEVANCE.
- CONTEXT must be relevant and helpful for answering the entire QUESTION to get a score of 10.
- Never elaborate.
```

`prompts.CONTEXT_RELEVANCE_USER`:

```
QUESTION: {question}
CONTEXT: {context}
RELEVANCE: 
```

`prompts.COT_REASONS_TEMPLATE`:

```
Please answer using the entire template below.

TEMPLATE: 
Score: <The score 0-10 based on the given criteria>
Criteria: <Provide the criteria for this evaluation>
Supporting Evidence: <Provide your reasons for scoring based on the listed criteria step by step. Tie it back to the evaluation being completed.>
```

## Example

Question: 

```python
"When was the University of Washington founded?"
```

Context: list of chunks

```python
[
    [
        '\n'
        'The University of Washington, founded in 1861 in Seattle, is a public '
        'research university\n'
        'with over 45,000 students across three campuses in Seattle, Tacoma, and '
        'Bothell.\n'
        'As the flagship institution of the six public universities in Washington '
        'state,\n'
        'UW encompasses over 500 buildings and 20 million square feet of space,\n'
        'including one of the largest library systems in the world.\n'
    ]
]
```

Result: 0.9 (normalized score)

Reason:

- Criteria: The context provides the exact founding year of the University of Washington and additional relevant information about the university.

- Supporting Evidence: The context states that the University of Washington was founded in 1861 in Seattle, which directly answers the question about the university's founding year. Additionally, it provides details about the university's campuses, student population, and infrastructure, demonstrating a comprehensive overview of the institution. This information is highly relevant to understanding the background and scope of the University of Washington, making the context very helpful in answering the question.
