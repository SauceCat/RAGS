# Best Practice

## Retrieval
### Context <-> Question
LLM binary relevance judgement. (TruLens)
Check whether each context chunk is required to answer the question. (RAGAS)

### Retrieved Context <-> Ground Truth Answer
RAGAS: Context Precision

Assesses if relevant items from ground truth are ranked higher in the retrieved contexts. Ideal scenario: All relevant chunks appear at top ranks.

Level: context chunk

---


RAGAS: Context Recall

Evaluates if each sentence in the ground truth answer can be traced back to the retrieved context. Ideal scenario: Every sentence in the ground truth answer is attributable to the retrieved context.

Level: ground truth sentence

Better first extract statements from answers and then do the judgement.

---

RAGAS: Context entities recall

Measures the proportion of ground truth entities present in the retrieved context. Particularly useful for fact-based applications (e.g., tourism help desks, historical Q&A).

Level: entities

quite interesting, worth considering.

---

## Generation
### Generated Answer <-> Retrieved Context
Break answer into claims or statements, then verify whether these claims are supported by the context.

groundedness: (TruLens groundedness)
1. Identify and extract the actual claims from the response.
2. Evaluate the groundedness of these specific claims against the provided context.

RAGAS: faithfulness (no need gt)
1. Break the generated answer into individual statements.
2. For each of the generated statements, verify if it can be inferred from the given context.

### Generated Answer <-> Question
RAGAS: Answer Relevance
Assess the relevance of the generated answer to the original question.

### Generated Answer <-> Answer
RAGAS: Answer semantic similarity

The concept of Answer Semantic Similarity pertains to the assessment of the semantic resemblance between the generated answer and the ground truth. 

## Additional Requirement
RAGAS: Aspect Critique (Generated Answer)


# Extra Knowledge

RAGAS's prompt management and formatting seems quite interesting.