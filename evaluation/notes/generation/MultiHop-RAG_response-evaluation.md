# MultiHop-RAG: Response Evaluation

- **Dimension:** Generated Answer <-> Ground Truth Answer
- **Reference:** [MultiHop-RAG: Benchmarking Retrieval-Augmented Generation for Multi-Hop Queries](https://arxiv.org/pdf/2401.15391)
- **Type:** Exact Match

In this experiment, we evaluate the quality of generated responses under two different settings. 
- In the first setting, we employ the best-performing retrieval model, namely `voyage-02` with `bge-reranker-large`, to retrieve the top-K texts
and then feed them into the LLM. 
- In the second setting, we use the ground-truth evidence associated with each query as the retrieved text for the LLM. This setting represents a ceiling performance for testing the LLM's response capabilities, as it utilizes the actual evidences.
