# Databricks: Correctness (Generated Answer <-> Question)
Reference: https://www.databricks.com/blog/LLM-auto-eval-best-practices-RAG

Evaluates whether the answer correctly answer the question, grading scale: 0-3.

## Prompt

**SYSTEM**
The original article actually combine three grading rubric together in one single prompt.

```
Please act as an impartial judge and evaluate the quality of the provided answer which attempts to answer the provided question based on a provided context.

You'll be given a function grading_function which you'll call for each provided context, question and answer to submit your reasoning and score for the correctness, comprehensiveness and readability of the answer. 

Below is your grading rubric: 
```

**SPEC**

```
- Correctness: If the answer correctly answer the question, below are the details for different scores:

  - Score 0: the answer is completely incorrect, doesn't mention anything about the question or is completely contrary to the correct answer.

      - For example, when asked "How to terminate a databricks cluster", the answer is empty string, or content that's completely irrelevant, or sorry I don't know the answer.

  - Score 1: the answer provides some relevance to the question and answers one aspect of the question correctly.

      - Example:

          - Question: How to terminate a databricks cluster

          - Answer: Databricks cluster is a cloud-based computing environment that allows users to process big data and run distributed data processing tasks efficiently.

          - Or answer:  In the Databricks workspace, navigate to the "Clusters" tab. And then this is a hard question that I need to think more about it

  - Score 2: the answer mostly answer the question but is missing or hallucinating on one critical aspect.

      - Example:

          - Question: How to terminate a databricks cluster"

          - Answer: "In the Databricks workspace, navigate to the "Clusters" tab.

          Find the cluster you want to terminate from the list of active clusters.

          And then you'll find a button to terminate all clusters at once"

  - Score 3: the answer correctly answer the question and not missing any major aspect

      - Example:

          - Question: How to terminate a databricks cluster

          - Answer: In the Databricks workspace, navigate to the "Clusters" tab.

          Find the cluster you want to terminate from the list of active clusters.

          Click on the down-arrow next to the cluster name to open the cluster details.

          Click on the "Terminate" button. A confirmation dialog will appear. Click "Terminate" again to confirm the action."
```