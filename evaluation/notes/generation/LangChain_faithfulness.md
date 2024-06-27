# LangChain: Faithfulness (Generated Answer <-> Retrieved Context, Question)
Reference: https://github.com/langchain-ai/langchain-benchmarks/blob/main/langchain_benchmarks/rag/evaluators.py

Given the question, evaluates how closely the generated answer adheres to the retrieved context, with higher scores indicating greater faithfulness.

## Prompt

**SYSTEM**

```
You are a helpful assistant.
```

**HUMAN**

```
[Instruction]
Please act as an impartial judge and evaluate the quality of the response provided by an AI assistant to the user question displayed below. For this evaluation, you should primarily consider the following criteria:
faithfulness: 
Score 1: The answer directly contradicts the information provided in the reference docs.
Score 3: The answer contains a mix of correct information from the reference docs and incorrect or unverifiable information not found in the docs.
Score 5: The answer is mostly aligned with the reference docs but includes extra information that, while not contradictory, is not verified by the docs.
Score 7: The answer aligns well with the reference docs but includes minor, commonly accepted facts not found in the docs.
Score 10: The answer perfectly aligns with and is fully entailed by the reference docs, with no extra information.
[Ground truth]
Reference docs:
<DOCS>
[<List of Documents>]
</DOCS>

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

**OUTPUT** (example)

```
The assistant's response provides a clear and concise explanation of what toolkits are in LangChain, along with specific examples of different toolkits and their functionalities. The examples given (SQLDatabaseToolkit, GitHubToolkit, and GmailToolkit) are accurate and align with the information provided in the reference documents. 

However, the response includes references like "[0]", "[1]", "[2]", and "[3]" which are not explained or linked to any specific part of the reference documents. This could be confusing to the reader as it implies there are citations or footnotes that are not actually provided.

Overall, the response is mostly aligned with the reference documents and includes minor, commonly accepted facts about the toolkits that are not explicitly detailed in the provided documents.

Rating: [[7]]
```
