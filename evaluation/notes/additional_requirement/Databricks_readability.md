# Databricks: Readability

- **Dimension:** Generated Answer
- **Reference:** [Best Practices for LLM Evaluation of RAG Applications](https://www.databricks.com/blog/LLM-auto-eval-best-practices-RAG)
- **Type:** LLM-as-a-judge
- **Comment:** Defining objective criteria for "Readability" is challenging.

It measures how readable is the answer, does it have redundant information or incomplete information that hurts the readability of the answer, grading scale: 0-3.

## Prompt

**SYSTEM**
```
Please act as an impartial judge and evaluate the quality of the provided answer which attempts to answer the provided question based on a provided context.

You'll be given a function grading_function which you'll call for each provided context, question and answer to submit your reasoning and score for the correctness, comprehensiveness and readability of the answer. 

Below is your grading rubric: 
```

**SPEC**

```
- Readability: How readable is the answer, does it have redundant information or incomplete information that hurts the readability of the answer.

  - Score 0: the answer is completely unreadable, e.g. fully of symbols that's hard to read; e.g. keeps repeating the words that it's very hard to understand the meaning of the paragraph. No meaningful information can be extracted from the answer.

  - Score 1: the answer is slightly readable, there are irrelevant symbols or repeated words, but it can roughly form a meaningful sentence that cover some aspects of the answer.

      - Example:

          - Question: How to use databricks API to create a cluster?

          - Answer: You you  you  you  you  you  will need a Databricks access token with the appropriate permissions. And then then you'll need to set up the request URL, then you can make the HTTP Request. Then Then Then Then Then Then Then Then Then

  - Score 2: the answer is correct and mostly readable, but there is one obvious piece that's affecting the readability (mentioning of irrelevant pieces, repeated words)

      - Example:

          - Question: How to terminate a databricks cluster

          - Answer: In the Databricks workspace, navigate to the "Clusters" tab.

          Find the cluster you want to terminate from the list of active clusters.

          Click on the down-arrow next to the cluster name to open the cluster details.

          Click on the "Terminate" button…………………………………..

          A confirmation dialog will appear. Click "Terminate" again to confirm the action.

  - Score 3: the answer is correct and reader friendly, no obvious piece that affect readability.
```