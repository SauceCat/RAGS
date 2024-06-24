# RAGAS: Faithfulness
Reference: https://docs.ragas.io/en/stable/concepts/metrics/faithfulness.html

$$
\text{Faithfulness score} = {|\text{Number of claims in the generated answer that can be inferred from given context}| \over |\text{Total number of claims in the generated answer}|}
$$

## Comments

Extract sentences from the original context before LLM evaluation should be a more reliable approach. This method ensures:
- Preservation of the original text.
- Prevention of unintended alterations by the LLM, even if instructed not to change sentences.

It's better to shift from sentence-level to chunk-level evaluation. Sentence-level judgement heavily relies on chunking strategies, large chunks containing a single-sentence answer may receive unfairly low scores. 


## Code and Prompts

We first use an LLM to extract a set of statements, $S(a_s(q))$. The aim of this step is to decompose longer sentences into shorter and more focused assertions.

```python
LONG_FORM_ANSWER_PROMPT = Prompt(
    name="long_form_answer",
    output_format_instruction=_statements_output_instructions,
    instruction="Given a question, an answer, and sentences from the answer analyze the complexity of each sentence given under 'sentences' and break down each sentence into one or more fully understandable statements while also ensuring no pronouns are used in each statement. Format the outputs in JSON.",
    examples=[
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
    ],
    input_keys=["question", "answer", "sentences"],
    output_key="analysis",
    language="english",
)
```

For each statement $s_i$ in $S$, the LLM determines if $s_i$ can be inferred from $c(q)$ using a verification function $v(s_i, c(q))$. This verification step is carried out using the following prompt:

```python
NLI_STATEMENTS_MESSAGE = Prompt(
    name="nli_statements",
    instruction="Your task is to judge the faithfulness of a series of statements based on a given context. For each statement you must return verdict as 1 if the statement can be directly inferred based on the context or 0 if the statement can not be directly inferred based on the context.",
    output_format_instruction=_faithfulness_output_instructions,
    examples=[
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
        },
        {
            "context": """Photosynthesis is a process used by plants, algae, and certain bacteria to convert light energy into chemical energy.""",
            "statements": ["Albert Einstein was a genius."],
            "answer": StatementFaithfulnessAnswers.parse_obj(
                [
                    {
                        "statement": "Albert Einstein was a genius.",
                        "reason": "The context and statement are unrelated",
                        "verdict": 0,
                    }
                ]
            ).dicts(),
        },
    ],
    input_keys=["context", "statements"],
    output_key="answer",
    output_type="json",
    language="english",
)  # noqa: E501
```

The final faithfulness score, $F$, is then computed as $F = \frac{|V|}{|S|}$, where $|V|$ is the number of statements that were supported according to the LLM and $|S|$ is the total number of statements.
