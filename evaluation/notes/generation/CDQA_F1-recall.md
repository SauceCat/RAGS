# CDQA: F1-recall

- **Dimension:** Generated Answer <-> Ground Truth Answer
- **Reference:** [Let LLMs Take on the Latest Challenges! A Chinese Dynamic Question Answering Benchmark](https://arxiv.org/abs/2402.19248)
- **Type:** Token-wise Accuracy

F1-recall measures the overlap between model-generated responses and ground truth, focusing on the model's ability to reproduce key elements from the reference.

1. **Tokenization:** Both the generated text and ground truth are segmented into token lists using word segmentation tools.
2. **Calculation:** Determine the ratio of tokens in the model's output that also appear in the ground truth token list.
3. **Formula:** F1-recall = (Number of common tokens) / (Total number of tokens in ground truth)
