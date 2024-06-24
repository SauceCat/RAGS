# TruLens: Answer Relevance
The response needs to helpfully answer the original question. We can verify this by evaluating the relevance of the final response to the user input.

We use LLM-as-a-judge to evaluate the relevance of the answer to the query by:
- Assigning a score 0-10 based on the given relevance criteria.
- Providing the criteria for the evaluation.
- Providing the reasons for scoring based on the listed criteria step by step.

## Comments
Same as the comments [here](../../retrieval/relevance/TruLens_context-relevance.md#Comments)

## Code and Prompts

```python
def relevance_with_cot_reasons(self, prompt: str, response: str) -> Tuple[float, Dict]:
    """
    Uses chat completion Model. A function that completes a template to
    check the relevance of the response to a prompt. Also uses chain of
    thought methodology and emits the reasons.

    Args:
        prompt (str): A text prompt to an agent. 
        response (str): The agent's response to the prompt.

    Returns:
        float: A value between 0 and 1. 0 being "not relevant" and 1 being "relevant".
    """
    system_prompt = prompts.ANSWER_RELEVANCE_SYSTEM

    user_prompt = str.format(
        prompts.ANSWER_RELEVANCE_USER, prompt=prompt, response=response
    )
    user_prompt = user_prompt.replace(
        "RELEVANCE:", prompts.COT_REASONS_TEMPLATE
    )
    return self.generate_score_and_reasons(system_prompt, user_prompt)
```

`prompts.ANSWER_RELEVANCE_SYSTEM`:

```
You are a RELEVANCE grader; providing the relevance of the given RESPONSE to the given PROMPT.
Respond only as a number from 0 to 10 where 0 is the least relevant and 10 is the most relevant. 
A few additional scoring guidelines:
- Long RESPONSES should score equally well as short RESPONSES.
- Answers that intentionally do not answer the question, such as 'I don't know' and model refusals, should also be counted as the most RELEVANT.
- RESPONSE must be relevant to the entire PROMPT to get a score of 10.
- RELEVANCE score should increase as the RESPONSE provides RELEVANT context to more parts of the PROMPT.
- RESPONSE that is RELEVANT to none of the PROMPT should get a score of 0.
- RESPONSE that is RELEVANT to some of the PROMPT should get as score of 2, 3, or 4. Higher score indicates more RELEVANCE.
- RESPONSE that is RELEVANT to most of the PROMPT should get a score between a 5, 6, 7 or 8. Higher score indicates more RELEVANCE.
- RESPONSE that is RELEVANT to the entire PROMPT should get a score of 9 or 10.
- RESPONSE that is RELEVANT and answers the entire PROMPT completely should get a score of 10.
- RESPONSE that confidently FALSE should get a score of 0.
- RESPONSE that is only seemingly RELEVANT should get a score of 0.
- Never elaborate.
```

`prompts.ANSWER_RELEVANCE_USER`:

```
PROMPT: {prompt}
RESPONSE: {response}
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

Response:

```python
"The University of Washington was founded in 1861."
```

Result: 1

Reason:

- Criteria: The response directly answers the question in the prompt.

- Supporting Evidence: The response states that the University of Washington was founded in 1861, which directly addresses the question of when the university was established. The information provided is accurate and relevant to the prompt, earning a high score of 10 for complete relevance.