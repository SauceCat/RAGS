# RAGAS: Answer Correctness (Generated Answer <-> Answer)
Reference: https://docs.ragas.io/en/stable/concepts/metrics/answer_correctness.html

Answer correctness encompasses two critical aspects: semantic similarity between the generated answer and the ground truth, as well as factual similarity. These aspects are combined using a weighted scheme to formulate the answer correctness score. 

## Calculation
class: `AnswerCorrectness`

### Step 1
Calculate the factual correctness. It quantifies the factual overlap between the generated answer and the ground truth answer. This is done using the concepts of:
- TP (True Positive): Facts or statements that are present in both the ground truth and the generated answer.
- FP (False Positive): Facts or statements that are present in the generated answer but not in the ground truth.
- FN (False Negative): Facts or statements that are present in the ground truth but not in the generated answer.

```python
class AnswerCorrectness(MetricWithLLM, MetricWithEmbeddings):
    # same as faithfulness
    def _create_statements_prompt(self, question: str, text: str) -> PromptValue:
        assert self.sentence_segmenter is not None, "sentence_segmenter is not set"

        sentences = self.sentence_segmenter.segment(text)
        sentences = [
            sentence for sentence in sentences if sentence.strip().endswith(".")
        ]
        sentences = "\n".join([f"{i}:{x}" for i, x in enumerate(sentences)])
        prompt_value = self.long_form_answer_prompt.format(
            question=question, answer=text, sentences=sentences
        )
        return prompt_value
        
    async def _ascore(self, row: t.Dict, callbacks: Callbacks, is_async: bool) -> float:
        assert self.llm is not None, "LLM must be set"

        question = row["question"]
        statements = {}
        for item in ["answer", "ground_truth"]:
            p_value = self._create_statements_prompt(question, row[item])
            item_statement = await self.llm.generate(
                p_value, callbacks=callbacks, is_async=is_async
            )
            statements[item] = await _statements_output_parser.aparse(
                item_statement.generations[0][0].text,
                p_value,
                self.llm,
                self.max_retries,
            )
            statements[item] = (
                statements[item].dicts() if statements[item] is not None else []
            )
```

### Step 2
We can use the formula for the F1 score to quantify correctness based on the number of statements in each of these lists:

```python
class AnswerCorrectness(MetricWithLLM, MetricWithEmbeddings):
    async def _ascore(self, row: t.Dict, callbacks: Callbacks, is_async: bool) -> float:
        assert self.llm is not None, "LLM must be set"

        if not all([val == [] for val in statements.values()]):
            ground_truth = [
                statement
                for item in statements["ground_truth"]
                for statement in item["simpler_statements"]
            ]
            answer = [
                statement
                for item in statements["answer"]
                for statement in item["simpler_statements"]
            ]
            p_value = self.correctness_prompt.format(
                question=question,
                ground_truth=ground_truth,
                answer=answer,
            )
            is_statement_present = await self.llm.generate(
                p_value, callbacks=callbacks, is_async=is_async
            )
            result_text = is_statement_present.generations[0][0].text

            answers = await _output_parser.aparse(
                result_text, p_value, self.llm, self.max_retries
            )
            if answers is None:
                return np.nan

            f1_score = self._compute_statement_presence(answers)
        else:
            f1_score = 1.0

    def _compute_statement_presence(
        self, prediction: AnswerCorrectnessClassification
    ) -> float:
        tp = len(prediction.TP)
        fp = len(prediction.FP)
        fn = len(prediction.FN)
        score = tp / (tp + 0.5 * (fp + fn)) if tp > 0 else 0
        return score
```

instruction:

```
Given a ground truth and an answer statements, analyze each statement and classify them in one of the following categories:

- TP (true positive): statements that are present in answer that are also directly supported by the one or more statements in ground truth,
- FP (false positive): statements present in the answer but not directly supported by any statement in ground truth,
- FN (false negative): statements found in the ground truth but not present in answer.

Each statement can only belong to one of the categories. Provide a reason for each classification.
```

example:

```python
{
    "question": """What powers the sun and what is its primary function?""",
    "answer": [
        "The sun is powered by nuclear fission, similar to nuclear reactors on Earth.",
        "The primary function of the sun is to provide light to the solar system.",
    ],
    "ground_truth": [
        "The sun is powered by nuclear fusion, where hydrogen atoms fuse to form helium.",
        "This fusion process in the sun's core releases a tremendous amount of energy.",
        "The energy from the sun provides heat and light, which are essential for life on Earth.",
        "The sun's light plays a critical role in Earth's climate system.",
        "Sunlight helps to drive the weather and ocean currents.",
    ],
    "classification": AnswerCorrectnessClassification.parse_obj(
        {
            "TP": [
                {
                    "statement": "The primary function of the sun is to provide light to the solar system.",
                    "reason": "This statement is somewhat supported by the ground truth mentioning the sun providing light and its roles, though it focuses more broadly on the sun's energy.",
                }
            ],
            "FP": [
                {
                    "statement": "The sun is powered by nuclear fission, similar to nuclear reactors on Earth.",
                    "reason": "This statement is incorrect and contradicts the ground truth which states that the sun is powered by nuclear fusion.",
                }
            ],
            "FN": [
                {
                    "statement": "The sun is powered by nuclear fusion, where hydrogen atoms fuse to form helium.",
                    "reason": "This accurate description of the sun’s power source is not included in the answer.",
                },
                {
                    "statement": "This fusion process in the sun's core releases a tremendous amount of energy.",
                    "reason": "This process and its significance are not mentioned in the answer.",
                },
                {
                    "statement": "The energy from the sun provides heat and light, which are essential for life on Earth.",
                    "reason": "The answer only mentions light, omitting the essential aspects of heat and its necessity for life, which the ground truth covers.",
                },
                {
                    "statement": "The sun's light plays a critical role in Earth's climate system.",
                    "reason": "This broader impact of the sun’s light on Earth's climate system is not addressed in the answer.",
                },
                {
                    "statement": "Sunlight helps to drive the weather and ocean currents.",
                    "reason": "The effect of sunlight on weather patterns and ocean currents is omitted in the answer.",
                },
            ],
        }
    ).dict(),
}
```

### Step 3
Once we have the semantic similarity, we take a weighted average of the semantic similarity and the factual similarity calculated above to arrive at the final score.

```python
class AnswerCorrectness(MetricWithLLM, MetricWithEmbeddings):
    async def _ascore(self, row: t.Dict, callbacks: Callbacks, is_async: bool) -> float:
        if self.weights[1] == 0:
            similarity_score = 0.0
        else:
            assert self.answer_similarity is not None, "AnswerSimilarity must be set"

            similarity_score = await self.answer_similarity.ascore(
                row, callbacks=callbacks, is_async=is_async
            )

        score = np.average(
            [f1_score, similarity_score],
            weights=self.weights,
        )

        return float(score)
```