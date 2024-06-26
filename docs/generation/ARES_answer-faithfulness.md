# ARES: Answer Faithfulness (Generated Answer <-> Retrieved Context)
Reference: [ARES: An Automated Evaluation Framework for Retrieval-Augmented Generation Systems](https://arxiv.org/abs/2311.09476)

Fine-tune `DeBERTa-v3-Large` with a binary classifier head for Answer Faithfulness: Is the answer generated faithful to the retrieved passage, or does it contain hallucinated or extrapolated statements beyond the passage? Use a human preference validation set to assess model improvement after each epoch.

While similar to Answer Relevance, this metric focuses specifically on the faithfulness of the generated answer to the retrieved context. The key distinction lies in the input for prediction: it excludes the query, whereas Answer Relevance takes into account both the retrieved context and the query. 