# RAGAS: Faithfulness (Generated Answer <-> Retrieved Context)
Reference: https://docs.ragas.io/en/stable/concepts/metrics/faithfulness.html

$$
\text{Faithfulness score} = {|\text{Number of claims in the generated answer that can be inferred from given context}| \over |\text{Total number of claims in the generated answer}|}
$$

## Comments

Extract sentences from the original context before LLM evaluation should be a more reliable approach. This method ensures:
- Preservation of the original text.
- Prevention of unintended alterations by the LLM, even if instructed not to change sentences.

It's better to shift from sentence-level to chunk-level evaluation. Sentence-level judgement heavily relies on chunking strategies, large chunks containing a single-sentence answer may receive unfairly low scores. 

## Calculation
class: `Faithfulness`

### Step 1
Break the generated answer into individual statements.

Statements:
- Statement 1: "Einstein was born in Germany."
- Statement 2: "Einstein was born on 20th March 1879."

```python
class Faithfulness(MetricWithLLM):
    def _create_statements_prompt(self, row: t.Dict) -> PromptValue:
        assert self.sentence_segmenter is not None, "sentence_segmenter is not set"

        text, question = row["answer"], row["question"]
        sentences = self.sentence_segmenter.segment(text)
        sentences = [
            sentence for sentence in sentences if sentence.strip().endswith(".")
        ]
        sentences = "\n".join([f"{i}:{x}" for i, x in enumerate(sentences)])
        prompt_value = self.statement_prompt.format(
            question=question, answer=text, sentences=sentences
        )
        return prompt_value
```

instruction:

```
Given a question, an answer, and sentences from the answer analyze the complexity of each sentence given under 'sentences' and break down each sentence into one or more fully understandable statements while also ensuring no pronouns are used in each statement. Format the outputs in JSON.
```

example:

```python
{
    "question": "Who was Albert Einstein and what is he best known for?",
    "answer": "He was a German-born theoretical physicist, widely acknowledged to be one of the greatest and most influential physicists of all time. He was best known for developing the theory of relativity, he also made important contributions to the development of the theory of quantum mechanics.",
    "sentences": """
    0:He was a German-born theoretical physicist, widely acknowledged to be one of the greatest and most influential physicists of all time. 
    1:He was best known for developing the theory of relativity, he also made important contributions to the development of the theory of quantum mechanics.
    """,
    "analysis": StatementsAnswers.parse_obj(
        [
            {
                "sentence_index": 0,
                "simpler_statements": [
                    "Albert Einstein was a German-born theoretical physicist.",
                    "Albert Einstein is recognized as one of the greatest and most influential physicists of all time.",
                ],
            },
            {
                "sentence_index": 1,
                "simpler_statements": [
                    "Albert Einstein was best known for developing the theory of relativity.",
                    "Albert Einstein also made important contributions to the development of the theory of quantum mechanics.",
                ],
            },
        ]
    ).dicts(),
}
```

### Step 2
For each of the generated statements, verify if it can be inferred from the given context.

```python
class Faithfulness(MetricWithLLM):
    def _create_nli_prompt(self, row: t.Dict, statements: t.List[str]) -> PromptValue:
        assert self.llm is not None, "llm must be set to compute score"

        contexts = row["contexts"]
        # check if the statements are support in the contexts
        contexts_str: str = "\n".join(contexts)
        statements_str: str = json.dumps(statements)
        prompt_value = self.nli_statements_message.format(
            context=contexts_str, statements=statements_str
        )
        return prompt_value
```

instruction:

```
Your task is to judge the faithfulness of a series of statements based on a given context. For each statement you must return verdict as 1 if the statement can be directly inferred based on the context or 0 if the statement can not be directly inferred based on the context.
```

example:

```python
{
    "context": """John is a student at XYZ University. He is pursuing a degree in Computer Science. He is enrolled in several courses this semester, including Data Structures, Algorithms, and Database Management. John is a diligent student and spends a significant amount of time studying and completing assignments. He often stays late in the library to work on his projects.""",
    "statements": [
        "John is majoring in Biology.",
        "John is taking a course on Artificial Intelligence.",
        "John is a dedicated student.",
        "John has a part-time job.",
    ],
    "answer": StatementFaithfulnessAnswers.parse_obj(
        [
            {
                "statement": "John is majoring in Biology.",
                "reason": "John's major is explicitly mentioned as Computer Science. There is no information suggesting he is majoring in Biology.",
                "verdict": 0,
            },
            {
                "statement": "John is taking a course on Artificial Intelligence.",
                "reason": "The context mentions the courses John is currently enrolled in, and Artificial Intelligence is not mentioned. Therefore, it cannot be deduced that John is taking a course on AI.",
                "verdict": 0,
            },
            {
                "statement": "John is a dedicated student.",
                "reason": "The context states that he spends a significant amount of time studying and completing assignments. Additionally, it mentions that he often stays late in the library to work on his projects, which implies dedication.",
                "verdict": 1,
            },
            {
                "statement": "John has a part-time job.",
                "reason": "There is no information given in the context about John having a part-time job.",
                "verdict": 0,
            },
        ]
    ).dicts(),
}
```

### Step 3
Calculate the faithfulness score.

```python
class Faithfulness(MetricWithLLM):
    def _compute_score(self, answers: StatementFaithfulnessAnswers):
        # check the verdicts and compute the score
        faithful_statements = sum(
            1 if answer.verdict else 0 for answer in answers.__root__
        )
        num_statements = len(answers.__root__)
        if num_statements:
            score = faithful_statements / num_statements
        else:
            logger.warning("No statements were generated from the answer.")
            score = np.nan

        return score
```