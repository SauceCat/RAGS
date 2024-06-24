# TruLens: Groundedness
LLMs can generate answers that may stray from provided context, potentially leading to plausible but inaccurate information. To assess the accuracy of these responses, we can:

1. Break down the answer into individual claims.
2. Evaluate each claim against the retrieved context.

We use LLM-as-a-judge to assess response groundedness by:
1. Split the response into sentences using the Punkt Sentence Tokenizer.
2. For each sentence, prompt the LLM to:
   - Rate the overlap between the source and the sentence on a 0-10 scale.
   - Explain the rating, citing specific supporting evidence from the source.

## Comments
Similar to the comments [here](../../retrieval/relevance/TruLens_context-relevance.md#comments)

Also, evaluating information overlap is unnecessarily complex. Instead, we should focus on whether each claim in the response is supported by the context. It's important to note that not every sentence contains a claim requiring verification. Our process should:

1. Identify and extract the actual claims from the response.
2. Evaluate the groundedness of these specific claims against the provided context.


## Code and Prompts

```python
def groundedness_measure_with_cot_reasons(self, source: str, statement: str) -> Tuple[float, dict]:
    """A measure to track if the source material supports each sentence in
    the statement using an LLM provider.

    The LLM will process the entire statement at once, using chain of
    thought methodology to emit the reasons. 

    Args:
        source: The source that should support the statement.
        statement: The statement to check groundedness.

    Returns:
        Tuple[float, dict]: A tuple containing a value between 0.0 (not grounded) and 1.0 (grounded) and a dictionary containing the reasons for the evaluation.
    """
    nltk.download('punkt', quiet=True)
    groundedness_scores = {}
    reasons_str = ""

    # Punkt Sentence Tokenizer: This tokenizer divides a text into a list of sentences
    hypotheses = sent_tokenize(statement)
    system_prompt = prompts.LLM_GROUNDEDNESS_SYSTEM

    def evaluate_hypothesis(index, hypothesis):
        user_prompt = prompts.LLM_GROUNDEDNESS_USER.format(
            premise=f"{source}", hypothesis=f"{hypothesis}"
        )
        score, reason = self.generate_score_and_reasons(system_prompt, user_prompt)
        return index, score, reason

    results = []

    with ThreadPoolExecutor() as executor:
        futures = [executor.submit(evaluate_hypothesis, i, hypothesis) for i, hypothesis in enumerate(hypotheses)]
        
        for future in as_completed(futures):
            results.append(future.result())

    results.sort(key=lambda x: x[0])  # Sort results by index

    for i, score, reason in results:
        groundedness_scores[f"statement_{i}"] = score
        reasons_str += f"STATEMENT {i}:\n{reason['reason']}\n"

    # Calculate the average groundedness score from the scores dictionary
    average_groundedness_score = float(np.mean(list(groundedness_scores.values())))
    
    return average_groundedness_score, {"reasons": reasons_str}
```

`prompts.LLM_GROUNDEDNESS_SYSTEM`:

```
You are a INFORMATION OVERLAP classifier; providing the overlap of information between the source and statement.
Respond only as a number from 0 to 10 where 0 is no information overlap and 10 is all information is overlapping.
Never elaborate.
```

`prompts.LLM_GROUNDEDNESS_USER`:

```
SOURCE: {premise}
Hypothesis: {hypothesis}

Please answer with the template below for all statement sentences:

Criteria: <Statement Sentence>, 
Supporting Evidence: <Identify and describe the location in the source where the information matches the statement. Provide a detailed, human-readable summary indicating the path or key details. if nothing matches, say NOTHING FOUND>
Score: <Output a number between 0-10 where 0 is no information overlap and 10 is all information is overlapping>
```

## Example

Source (Context): list of chunks 

```python
[[['\n'
   'The University of Washington, founded in 1861 in Seattle, is a public '
   'research university\n'
   'with over 45,000 students across three campuses in Seattle, Tacoma, and '
   'Bothell.\n'
   'As the flagship institution of the six public universities in Washington '
   'state,\n'
   'UW encompasses over 500 buildings and 20 million square feet of space,\n'
   'including one of the largest library systems in the world.\n']]]
```

Statement (Response):

```python
"The University of Washington was founded in 1861."
```

Result: 1

Reasons:
- STATEMENT 0:
    - Criteria: The University of Washington was founded in 1861.
    - Supporting Evidence: The source states, "The University of Washington, founded in 1861 in Seattle..."
    - Score: 10