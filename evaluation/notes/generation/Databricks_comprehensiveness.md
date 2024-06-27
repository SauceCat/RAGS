# Databricks: Comprehensiveness (Generated Answer <-> Question)
Reference: https://www.databricks.com/blog/LLM-auto-eval-best-practices-RAG

How comprehensive is the answer, does it fully answer all aspects of the question and provide comprehensive explanation and other necessary information, grading scale: 0-3.

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
- Comprehensiveness: How comprehensive is the answer, does it fully answer all aspects of the question and provide comprehensive explanation and other necessary information. Below are the details for different scores:

  - Score 0: typically if the answer is completely incorrect, then the comprehensiveness is also zero score.

  - Score 1: if the answer is correct but too short to fully answer the question, then we can give score 1 for comprehensiveness.

      - Example:

          - Question: How to use databricks API to create a cluster?

          - Answer: First, you will need a Databricks access token with the appropriate permissions. You can generate this token through the Databricks UI under the 'User Settings' option. And then (the rest is missing)

  - Score 2: the answer is correct and roughly answer the main aspects of the question, but it’s missing description about details. Or is completely missing details about one minor aspect.

      - Example:

          - Question: How to use databricks API to create a cluster?

          - Answer: You will need a Databricks access token with the appropriate permissions. Then you’ll need to set up the request URL, then you can make the HTTP Request. Then you can handle the request response.

      - Example:

          - Question: How to use databricks API to create a cluster?

          - Answer: You will need a Databricks access token with the appropriate permissions. Then you’ll need to set up the request URL, then you can make the HTTP Request. Then you can handle the request response.

  - Score 3: the answer is correct, and covers all the main aspects of the question
```