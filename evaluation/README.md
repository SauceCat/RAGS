# RAG Evaluation
We group the evaluation methodologies according to the base for evaluation.

## Generated Answer
| Metric | Type |
| ------ | ---- |
| [Databricks: Readability](notes/additional_requirement/Databricks_readability.md) | LLM-as-a-judge |
| [RAGAS: Aspect Critique](notes/additional_requirement/RAGAS_aspect-critique.md) | LLM-as-a-judge |

## Generated Answer <-> Ground Truth Answer
| Metric | Type |
| ------ | ---- |
| [RECALL: Counterfactual Robustness](notes/additional_requirement/RECALL_counterfactual-robustness.md) | Exact Match |

## Retrieved Context
| Metric | Type |
| ------ | ---- |
| [Haystack: Diversity](notes/additional_requirement/Haystack_diversity.md) | Semantic Similarity |

## Others
| Metric | Type |
| ------ | ---- |
| [Latency](notes/additional_requirement/latency.md) | System Efficiency |





<img src="notes/additional_requirement/RECALL_counterfactual-robustness.md">