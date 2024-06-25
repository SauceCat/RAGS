# RAGAS: Answer Relevance (Generated Answer <-> Question)
Reference: https://docs.ragas.io/en/stable/concepts/metrics/answer_relevance.html

Assess the relevance of the generated answer to the original question using an indirect method:

1. Generate multiple questions that the answer could respond to.
2. Calculate the cosine similarity between each generated question and the original question.
3. Take the average of these similarity scores.

This approach evaluates relevance by considering how well the generated answer aligns with questions similar to the original, rather than directly comparing the answer to the question itself.

## Calculation
class: `AnswerRelevancy`

### Step 1
Reverse-engineer `n` variants of the question from the generated answer using a Large Language Model (LLM). For instance, for the first answer, the LLM might generate the following possible questions:

- Question 1: "In which part of Europe is France located?"
- Question 2: "What is the geographical location of France within Europe?"
- Question 3: "Can you identify the region of Europe where France is situated?"

```python
class AnswerRelevancy(MetricWithLLM, MetricWithEmbeddings):
    def _create_question_gen_prompt(self, row: t.Dict) -> PromptValue:
        ans, ctx = row["answer"], row["contexts"]
        return self.question_generation.format(answer=ans, context="\n".join(ctx))
```

instruction:

```
Generate a question for the given answer and Identify if answer is noncommittal. Give noncommittal as 1 if the answer is noncommittal and 0 if the answer is committal. A noncommittal answer is one that is evasive, vague, or ambiguous. For example, "I don't know" or "I'm not sure" are noncommittal answers
```

example (noncommittal):

```python
{
    "answer": """Albert Einstein was born in Germany.""",
    "context": """Albert Einstein was a German-born theoretical physicist who is widely held to be one of the greatest and most influential scientists of all time""",
    "output": AnswerRelevanceClassification.parse_obj(
        {
            "question": "Where was Albert Einstein born?",
            "noncommittal": 0,
        }
    ).dict(),
}
```

example (committal):

```python
{
    "answer": """I don't know about the  groundbreaking feature of the smartphone invented in 2023 as am unaware of information beyond 2022. """,
    "context": """In 2023, a groundbreaking invention was announced: a smartphone with a battery life of one month, revolutionizing the way people use mobile technology.""",
    "output": AnswerRelevanceClassification.parse_obj(
        {
            "question": "What was the groundbreaking feature of the smartphone invented in 2023?",
            "noncommittal": 1,
        }
    ).dict(),
}
```

### Step 2
Calculate the mean cosine similarity between the generated questions and the actual question.

```python
class AnswerRelevancy(MetricWithLLM, MetricWithEmbeddings):
    def calculate_similarity(
        self: t.Self, question: str, generated_questions: list[str]
    ):
        assert self.embeddings is not None
        question_vec = np.asarray(self.embeddings.embed_query(question)).reshape(1, -1)
        gen_question_vec = np.asarray(
            self.embeddings.embed_documents(generated_questions)
        ).reshape(len(generated_questions), -1)
        norm = np.linalg.norm(gen_question_vec, axis=1) * np.linalg.norm(
            question_vec, axis=1
        )
        return (
            np.dot(gen_question_vec, question_vec.T).reshape(
                -1,
            )
            / norm
        )

    def _calculate_score(
        self, answers: t.Sequence[AnswerRelevanceClassification], row: t.Dict
    ) -> float:
        question = row["question"]
        gen_questions = [answer.question for answer in answers]
        committal = np.any([answer.noncommittal for answer in answers])
        if all(q == "" for q in gen_questions):
            logger.warning(
                "Invalid JSON response. Expected dictionary with key 'question'"
            )
            score = np.nan
        else:
            cosine_sim = self.calculate_similarity(question, gen_questions)
            score = cosine_sim.mean() * int(not committal)

        return score
```
