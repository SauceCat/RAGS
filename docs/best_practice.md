# Best Practice

## Retrieval
### Context <-> Question
LLM binary relevance judgement. (TruLens)
Check whether each context chunk is required to answer the question. (RAGAS)

### Retrieved Context <-> Ground Truth Answer

RAGAS: Context Recall
1. Break the ground truth answer into individual statements.
2. For each of the ground truth statements, verify if it can be attributed to the retrieved context.
3. Context recall = number of GT sentences that can be attributed to context / total number of GT sentences

RAGAS: Context entities recall
1. Identify entities in the ground truth answer.
2. Check how many of these entities are present in the retrieved context.
3. Context entities recall = number of GT entities present in context / total number of GT entities.

## Generation
### Generated Answer <-> Retrieved Context
Break answer into claims or statements, then verify whether these claims are supported by the context.

groundedness: (TruLens groundedness)
1. Identify and extract the actual claims from the response.
2. Evaluate the groundedness of these specific claims against the provided context.

RAGAS: faithfulness (no need gt)
1. Break the generated answer into individual statements.
2. For each of the generated statements, verify if it can be inferred from the given context.




### Answer <-> Question
