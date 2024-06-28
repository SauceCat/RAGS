# LangChain: Accuracy

- **Dimension:** Generated Answer <-> GroundTruth Answer, Question
- **Reference:** [Evaluating RAG Architectures on Benchmark Tasks](https://langchain-ai.github.io/langchain-benchmarks/notebooks/retrieval/comparing_techniques.html)
- **Type:** LLM-as-a-judge

Given the context, evaluates the level of alignment between the generated answer and the ground truth answer.

## Prompt

**SYSTEM**

```
You are a helpful assistant.
```

**HUMAN**

```
[Instruction]
Please act as an impartial judge and evaluate the quality of the response provided by an AI assistant to the user question displayed below. For this evaluation, you should primarily consider the following criteria:
accuracy: 
Score 1: The answer is incorrect and unrelated to the question or reference document.
Score 3: The answer shows slight relevance to the question or reference document but is largely incorrect.
Score 5: The answer is partially correct but has significant errors or omissions.
Score 7: The answer is mostly correct with minor errors or omissions, and aligns with the reference document.
Score 10: The answer is correct, complete, and perfectly aligns with the reference document.

If the reference answer contains multiple alternatives, the predicted answer must only match one of the alternatives to be considered correct.
If the predicted answer contains additional helpful and accurate information that is not present in the reference answer, it should still be considered correct.
[Ground truth]
Toolkits are collections of tools that are designed to be used together for specific tasks and have convenience loading methods.
Begin your evaluation by providing a short explanation. Be as objective as possible. After providing your explanation, you must rate the response on a scale of 1 to 10 by strictly following this format: "[[rating]]", for example: "Rating: [[5]]".

[Question]
whats a toolkit

[The Start of Assistant's Answer]
Toolkits in LangChain are collections of tools that allow agents to interact with various services or data. Examples include:

- **SQLDatabaseToolkit:** Helps agents query and interact with SQL databases [0].
- **GitHubToolkit:** Enables agents to manage issues, pull requests, and comments on GitHub [1][2].
- **GmailToolkit:** Allows agents to read, send, update, and delete emails in Gmail [3][2].
[The End of Assistant's Answer]
```

**OUTPUT**

```
The assistant's answer provides specific examples of toolkits in LangChain, which is helpful and relevant. However, it does not directly address the general definition of a toolkit as a collection of tools designed to be used together for specific tasks and having convenience loading methods. The examples given are accurate and relevant to LangChain but do not fully capture the broader definition required by the question.

Rating: [[7]]
```

## Limitations
- Defining objective criteria for "Accuracy" is challenging.
- A 0-10 scale for evaluation can be overly granular and potentially problematic. The drawbacks include:
    - Ambiguity: Defining clear boundaries between adjacent scores (e.g., 7 vs. 8) can be challenging.
    - Consistency: Maintaining consistent scoring across different evaluators or even for a single evaluator over time can be difficult with so many options.
    - Unnecessary Precision: Such fine-grained distinctions may not provide meaningful additional information in many cases.

  A simpler scale could be more effective and reliable:
    - 3-point scale: Low, Medium, High
    - 5-point scale: Very Poor, Poor, Average, Good, Excellent
    - Binary scale: Satisfactory / Unsatisfactory
  
  These simplified scales can offer clearer distinctions between categories, improve consistency in scoring, and still provide meaningful evaluations without the complexity of a 0-10 system.