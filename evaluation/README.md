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
| [RGB: Counterfactual Robustness](notes/additional_requirement/RGB_counterfactual-robustness.md) | Exact Match |
| [RGB: Information Integration](notes/additional_requirement/RGB_information-integration.md) | Exact Match |
| [RGB: Negative Rejection](notes/additional_requirement/RGB_negative-rejection.md) | Exact Match |
| [RGB: Noise Robustness](notes/additional_requirement/RGB_noise-robustness.md) | Exact Match |
| [BERTScore](notes/generation/BERTScore.md) | Semantic Similarity, Token-wise Accuracy |
| [BLEU](notes/generation/BLEU.md) | Token-wise Accuracy |
| [CDQA: F1-recall](notes/generation/CDQA_F1-recall.md) | Token-wise Accuracy |
| [LangChain: EmbeddingDistance](notes/generation/LangChain_embedding-distance.md) | Semantic Similarity |
| [MultiHop-RAG: Response Evaluation](notes/generation/MultiHop-RAG_response-evaluation.md) | Exact Match |
| [RAGAS: Answer Correctness](notes/generation/RAGAS_answer-correctness.md) | LLM-as-a-tool, Factual Consistency |
| [RAGAS: Answer Semantic Similarity](notes/generation/RAGAS_answer-semantic-similarity.md) | Semantic Similarity |
| [ROUGE](notes/generation/ROUGE.md) | Token-wise Accuracy |


## Generated Answer <-> Ground Truth Answer, Question
| Metric | Type |
| ------ | ---- |
| [LangChain: Accuracy](notes/generation/LangChain_accuracy.md) | LLM-as-a-judge |


## Generated Answer <-> Ground Truth Context
| Metric | Type |
| ------ | ---- |
| [CRUD: RAGQuestEval](notes/generation/CRUD_RAGQuestEval.md) | Token-wise Accuracy |


## Generated Answer <-> Retrieved Context
| Metric | Type |
| ------ | ---- |
| [ARES: Answer Faithfulness](notes/generation/ARES_answer-faithfulness.md) | LLM classifier |
| [RAGAS: Faithfulness](notes/generation/RAGAS_faithfulness.md) | LLM-as-a-tool, Factual Consistency |
| [ARES: Answer Relevance](notes/generation/ARES_answer-relevance.md) | LLM classifier |
| [TruLens: Groundedness](notes/generation/TruLens_groundedness.md) | LLM-as-a-judge |

## Generated Answer <-> Question
| Metric | Type |
| ------ | ---- |
| [Databricks: Comprehensiveness](notes/generation/Databricks_comprehensiveness.md) | LLM-as-a-judge |
| [Databricks: Correctness](notes/generation/Databricks_correctness.md) | LLM-as-a-judge |
| [RAGAS: Answer Relevance](notes/generation/RAGAS_answer-relevance.md) | Semantic Similarity |
| [TruLens: Answer Relevance](notes/generation/TruLens_answer-relevance.md) | LLM-as-a-judge |

## Generated Answer <-> Retrieved Context, Question
| Metric | Type |
| ------ | ---- |
| [LangChain: Faithfulness](notes/generation/LangChain_faithfulness.md) | LLM-as-a-judge |

## Retrieved Context
| Metric | Type |
| ------ | ---- |
| [Haystack: Diversity](notes/additional_requirement/Haystack_diversity.md) | Semantic Similarity |

## Retrieved Context <-> Ground Truth Context
| Metric | Type |
| ------ | ---- |
| [MultiHop-RAG: Retrieval Evaluation](notes/retrieval/MultiHop-RAG_retrieval-evaluation.md) | IR Metrics |

## Retrieved Context <-> Question
| Metric | Type |
| ------ | ---- |
| [ARES: Context Relevance](notes/retrieval/ARES_context-relevance.md) | LLM classifier |
| [RAGAS: Context Relevance](notes/retrieval/RAGAS_context-relevance.md) | LLM-as-a-judge |
| [TruLens: Context Relevance](notes/retrieval/TruLens_context-relevance.md) | LLM-as-a-judge |

## Retrieved Context <-> Ground Truth Answer
| Metric | Type |
| ------ | ---- |
| [RAGAS: Context Entity Recall](notes/retrieval/RAGAS_context-entities-recall.md) | LLM-as-a-tool, Entity Recall |

## Retrieved Context <-> Ground Truth Answer, Question
| Metric | Type |
| ------ | ---- |
| [RAGAS: Context Precision](notes/retrieval/RAGAS_context-precision.md) | LLM-as-a-judge, IR Metrics |
| [RAGAS: Context Recall](notes/retrieval/RAGAS_context-recall.md) | LLM-as-a-judge |


## Others
| Metric | Type |
| ------ | ---- |
| [Latency](notes/additional_requirement/latency.md) | System Efficiency |

## Tools
- [LangChain](notes/tools/LangChain.md)
- [RAGAS](notes/tools/RAGAS.md)
- [TruLens](notes/tools/TruLens.md)