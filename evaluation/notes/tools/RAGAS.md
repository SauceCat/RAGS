# RAGAS: Automated Evaluation of Retrieval Augmented Generation
Reference:
- Website: https://docs.ragas.io/en/stable/
- Paper: [RAGAS: Automated Evaluation of Retrieval Augmented Generation](https://arxiv.org/abs/2309.15217)

In the paper, RAGAS introduces a set of metrics for evaluating RAG systems across multiple dimensions that don't require human-annotated ground truth:
- **Context Relevance**: Retrieval effectiveness in finding relevant and focused context
- **Faithfulness**: LLM's ability to use these contexts accurately
- **Answer Relevance**: Overall generation quality

To verify the reliability of these ground-truth-free metrics, the authors created WikiEval, a small dataset of 50 test cases, designed to compare RAGAS metrics with human judgments. Their experiments demonstrated a strong correlation between RAGAS metrics and human assessments. 

The authors also prepared two baseline models for comparison:
- **GPT Score**: ChatGPT rates quality dimensions from 0-10 based on metric definitions.
- **GPT Ranking**: ChatGPT selects preferred answers/contexts based on quality metrics.

**Table 1**: Agreement with human annotators in pairwise comparisons of faithfulness, answer relevance and context relevance, using the WikEval dataset (accuracy).

<img src="../images/tools/RAGAS_alignment.png" width=400>

- **Faithfulness**: RAGAS predictions are highly accurate.
- **Answer Relevance**: Lower agreement, often due to subtle differences between candidate answers.
- **Context Relevance**: Most challenging to evaluate, with ChatGPT struggling to identify crucial information in longer contexts.

This analysis suggests RAGAS offers a promising approach for efficient, automated RAG system evaluation.

Besides, RAGAS also offers a bunch of metrics that require ground truth answers. They offer a more comprehensive assessment of RAG system performance, particularly in scenarios where high accuracy is crucial and resources are available for creating annotated datasets.
- Context Precision
- Context Recall
- Context Entities Recall
- Answer Semantic Similarity
- Answer Correctness