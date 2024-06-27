# ARES: Answer Faithfulness

- **Dimension:** Generated Answer <-> Retrieved Context
- **Reference:** [ARES: An Automated Evaluation Framework for Retrieval-Augmented Generation Systems](https://arxiv.org/abs/2311.09476)
- **Type:** LLM classifier

Answer Faithfulness: Is the answer generated faithful to the retrieved passage, or does it contain hallucinated or extrapolated statements beyond the passage?

Fine-tune `DeBERTa-v3-Large` with a binary classifier head for Answer Faithfulness. Use a human preference validation set to assess model improvement after each epoch.

While similar to Answer Relevance, this metric focuses specifically on the faithfulness of the generated answer to the retrieved context. The key distinction lies in the input for prediction: it excludes the query, whereas Answer Relevance takes into account both the retrieved context and the query. 

## Limitations
- Fine tuning a LLM classifier for each criteria might not be the most efficient way, it requires certain amount of training data, although we could just use LLM to generate the positive and negative samples, it still requires much more data points compared with few shot prompting.
- Also, it requires a validation set with human annotation to assess the model improvement. If the validation set is too small, it might not be representative; while if it has more samples, it cost a lot of human effort.
- Don't mention the criteria changes very fast. It just take too much effort whenever we want to include an extra criteria.