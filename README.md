# Does Explicit Evidence Attribution Improve Scientific Question Answering?

### A Controlled Pilot Study Using QASPER

## 1. Overview

Scientific question answering systems can generate fluent answers about research papers, but a fluent answer is not necessarily a well-supported answer.

This project investigates whether explicitly requiring a language model to identify supporting passages improves the quality and reliability of answers to questions about scientific papers.

A controlled pilot experiment compares two prompting conditions using the same language model and the same frozen retrieval results:

- **Condition A — Answer-only:** The model generates an answer without being required to identify supporting passages.
- **Condition B — Evidence-attributed:** The model is instructed to provide an answer with supporting paragraph identifiers.

The experiment uses the QASPER scientific question-answering dataset, BM25 passage retrieval, and the lightweight FLAN-T5-small language model.

## 2. Research Question

**Primary research question:**

Under identical retrieval conditions, how does requiring explicit passage-level evidence attribution affect answer correctness and evidence support in scientific question answering?

**Secondary questions:**

1. Does mandatory evidence attribution reduce unsupported or incorrectly supported answers?
2. Does the effect differ depending on whether a gold evidence passage is retrieved?
3. How frequently does the model produce evidence identifiers without a substantive answer?

## 3. Research Gap

Previous research has explored scientific question answering, evidence-supported generation, and citation-grounded evaluation.

This project investigates a narrower experimental question: what happens when an explicit evidence-attribution requirement is added to a fixed model while the retrieved context remains unchanged?

The focus is on separating answer quality from evidence-attribution quality rather than assuming that the presence of a citation guarantees a correct answer.

See [`gap_matrix.md`](gap_matrix.md) for the literature comparison and research-gap analysis.

## 4. Dataset

**Dataset:** QASPER  
**Source:** [Hugging Face — allenai/qasper](https://huggingface.co/datasets/allenai/qasper)  
**Original paper:** [A Dataset of Information-Seeking Questions and Answers Anchored in Research Papers](https://arxiv.org/abs/2105.03011)  
**License:** CC BY 4.0, as stated in the dataset card.

Dataset statistics:

| Split | Papers | Questions |
|---|---:|---:|
| Train | 888 | 2,593 |
| Validation | 281 | 1,005 |
| Test | 416 | 1,451 |
| Total | 1,585 | 5,049 |

The dataset includes scientific papers, questions, answer annotations, answer types, and supporting evidence.

The original dataset loading script was not used because the installed Hugging Face `datasets` version did not support the legacy loading script. The available Parquet files were used instead.

## 5. Methodology

The experimental pipeline consists of the following stages:

1. Load the QASPER dataset.
2. Inspect answer annotations and supporting evidence.
3. Map evidence statements to original paper paragraphs using normalized text matching.
4. Build a BM25 retriever for each scientific paper.
5. Select 50 eligible questions from the official test split using random seed 42.
6. Retrieve up to five paragraphs for each question.
7. Freeze the retrieval results.
8. Generate answers using Condition A and Condition B.
9. Evaluate answer quality and evidence attribution separately.
10. Perform paired comparison and retrieval-based error analysis.

### Experimental configuration

| Component | Configuration |
|---|---|
| Dataset | QASPER |
| Retrieval method | BM25 |
| Retrieved passages | Up to 5 |
| Language model | `google/flan-t5-small` |
| Execution environment | Google Colab |
| Hardware | CPU |
| Maximum input length | 512 tokens |
| Maximum generated tokens | 100 |
| Decoding | Greedy (`do_sample=False`) |
| Test sample | 50 questions |
| Random seed | 42 |

No model fine-tuning was performed.

### Controlled comparison

Both conditions use the same questions, model, and frozen retrieval results. The main intended difference is the instruction to provide explicit evidence attribution in Condition B.

## 6. Results

### Answer quality

| Metric | Condition A: Answer-only | Condition B: Evidence-attributed |
|---|---:|---:|
| Mean token F1 | 0.1229 | 0.1132 |
| Exact Match | 0.06 | 0.08 |
| Mean inference time | 0.6823 s | 0.6569 s |

### Retrieval performance

| Metric | Result |
|---|---:|
| Test questions | 50 |
| Questions with at least one gold passage in top five | 29 |
| Test-sample Hit@5 | 58% |

### Evidence attribution

| Metric | Result |
|---|---:|
| Answers with recognized citations | 24/50 |
| Total cited passage IDs | 24 |
| Citations matching mapped gold evidence | 5 |
| Citations found in retrieved top five | 23 |
| Overall citation precision | 0.1000 |
| Overall citation recall | 0.0495 |
| Overall citation F1 | 0.0617 |

### Paired comparison

- Mean token F1 difference (B − A): −0.0097
- Condition B higher: 7 questions
- Condition A higher: 5 questions
- Tied: 38 questions
- 95% paired bootstrap confidence interval: [−0.071, 0.0403]

The confidence interval includes zero. The results therefore do not establish that either prompting condition is superior in answer quality.

A notable failure mode was the generation of passage identifiers without a substantive answer.

## 7. Key Observations

- Explicit evidence attribution did not improve mean token F1 in this pilot.
- Condition B achieved a slightly higher Exact Match rate, but the overall difference was small.
- Only five of the 24 recognized cited passages matched the mapped gold evidence.
- Most recognized citations appeared in the retrieved top-five passages, suggesting that citation selection and evidence identification remained challenging even when the passage was available.
- Retrieval success was associated with higher answer F1 in both conditions.
- The model sometimes produced only a paragraph identifier instead of a meaningful answer.

These findings highlight the importance of evaluating retrieval, answer generation, and evidence attribution separately.

## 8. Repository Structure

```text
QASPER-Evidence-Attribution/
│
├── README.md
├── research_decisions.md
├── gap_matrix.md
├── requirements.txt
│
├── data/
│   └── README.md
│
├── notebooks/
│   └── research_experiment.ipynb
│
├── results/
│   ├── g3_test_sample.json
│   ├── g3_frozen_retrieval.json
│   ├── g3_condition_A.json
│   ├── g3_condition_B.json
│   ├── g3_evaluation_results.csv
│   ├── g3_summary_results.csv
│   ├── g3_deep_error_analysis.csv
│   └── g3_retrieval_error_analysis.csv
│
└── paper/
    ├── main.tex
    ├── IEEE_QASPER_Evidence_Attribution_Paper.docx
    └── figures/
```

The structure above describes the intended organization. Only files actually committed to the repository should be treated as available.

## 9. Reproducibility

The experiment was developed using Google Colab and CPU-compatible open-source components.

To reproduce the experiment:

1. Open the notebook in Google Colab.
2. Install the dependencies listed in `requirements.txt`.
3. Load the QASPER dataset from its accessible Parquet files.
4. Run the notebook cells in order.
5. Generate and save the retrieval outputs.
6. Execute both prompting conditions using the frozen retrieval results.
7. Run the evaluation and error-analysis cells.

The JSON and CSV result files preserve the experimental sample, retrieved passages, model outputs, and evaluation results.

## 10. Limitations

This is a small pilot study rather than a large-scale benchmark evaluation.

Important limitations include:

- Only 50 test questions were evaluated.
- Questions without usable mapped evidence were excluded from the selected sample.
- Evidence mapping uses normalized substring matching, which may miss valid evidence.
- FLAN-T5-small has limited generation capacity and sometimes produces incomplete or irrelevant responses.
- Token F1 and Exact Match are lexical metrics and do not fully capture semantic correctness.
- Prompt lengths differ between conditions, which may affect truncation under the shared input limit.
- Results should not be generalized to larger language models without further experiments.

## 11. Research Documentation

Additional project documentation:

- [`research_decisions.md`](research_decisions.md) — Research rationale, methodology decisions, measured results, and limitations.
- [`gap_matrix.md`](gap_matrix.md) — Comparison with related research and identification of the research gap.

## 12. References

[1] P. Dasigi, J. L. Liu, A. Marasović, N. A. Smith, and M. Gardner, “A dataset of information-seeking questions and answers anchored in research papers,” in Proc. NAACL-HLT, 2021, pp. 5454–5474.
[2] J. Menick et al., “Teaching language models to support answers with verified quotes,” arXiv:2203.11147, 2022.
[3] [Verify author initials], Imran and [Verify author initials] Solanky, “ResearchQA: Benchmarking citation-grounded question-answering on scientific papers,” preprint, 2026. Verify complete authorship and preprint identifier before submission.
[4] S. Robertson and H. Zaragoza, “The probabilistic relevance framework: BM25 and beyond,” Foundations and Trends in Information Retrieval, vol. 3, no. 4, pp. 333–389, 2009.
[5] P. Lewis et al., “Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks,” in Advances in Neural Information Processing Systems, vol. 33, 2020.
[6] T. Gao, H. Yen, J. Yu, and D. Chen, “Enabling Large Language Models to Generate Text with Citations,” in Proc. 2023 Conf. Empirical Methods in Natural Language Processing (EMNLP), 2023, pp. 6465–6488, doi: 10.18653/v1/2023.emnlp-main.398.
[7] C. D. Manning, P. Raghavan, and H. Schütze, Introduction to Information Retrieval. Cambridge, U.K.: Cambridge University Press, 2008.
[8] H. W. Chung et al., “Scaling Instruction-Finetuned Language Models,” Journal of Machine Learning Research, vol. 25, no. 70, pp. 1–53, 2024.
[9] P. Rajpurkar, J. Zhang, K. Lopyrev, and P. Liang, “SQuAD: 100,000+ Questions for Machine Comprehension of Text,” in Proc. 2016 Conf. Empirical Methods in Natural Language Processing (EMNLP), 2016, pp. 2383–2392, doi: 10.18653/v1/D16-1264.
[10] T. Baumgärtner, T. Briscoe, and I. Gurevych, “PeerQA: A Scientific Question Answering Dataset from Peer Reviews,” in Proc. 2025 Conf. North American Chapter of the Association for Computational Linguistics (NAACL-HLT), 2025, pp. 508–544.
[11] S. Singh, N. Sarkar, and A. Cohan, “SciDQA: A Deep Reading Comprehension Dataset over Scientific Papers,” in Proc. 2024 Conf. Empirical Methods in Natural Language Processing (EMNLP), 2024, pp. 20908–20923.
[12] X. Ye, R. Sun, S. Arik, and T. Pfister, “Effective Large Language Model Adaptation for Improved Grounding and Citation Generation,” in Proc. 2024 Conf. North American Chapter of the Association for Computational Linguistics (NAACL-HLT), 2024, pp. 6237–6251.
[13] L. Gao, Z. Dai, P. Pasupat, A. Chen, A. T. Chaganty, Y. Fan, V. Zhao, N. Lao, H. Lee, D.-C. Juan, and K. Guu, “RARR: Researching and Revising What Language Models Say, Using Language Models,” in Proc. 61st Annual Meeting of the Association for Computational Linguistics (ACL), 2023, pp. 16477–16508.
[14] A. Abolghasemi, L. Azzopardi, S. H. Hashemi, M. de Rijke, and S. Verberne, “Evaluation of Attribution Bias in Generator-Aware Retrieval-Augmented Large Language Models,” in Findings of the Association for Computational Linguistics: ACL 2025, 2025, pp. 21105–21124.
[15] T. Schreieder, T. Schopf, and M. Färber, “Attribution, Citation, and Quotation: A Survey of Evidence-based Text Generation with Large Language Models,” in Proc. 64th Annual Meeting of the Association for Computational Linguistics (ACL), 2026, pp. 30956–31000.
[16] X. Liu, Y. Wang, P. Shi, B. Xue, X. Ke, S. Cai, K. Choi, D. Wu, F. Shi, and K. Czarnecki, “LitTraceQA: A Benchmark for Multi-Stage Grounding and Verification in Scientific Question Answering,” in Proc. 1st Workshop on Grounding Language Models (GroundLM), 2026.

## 13. Conclusion

This project presents a controlled pilot investigation of explicit evidence attribution in scientific question answering.

The results show that requiring a compact language model to produce evidence identifiers does not automatically ensure that its answers are more correct or better supported.

The experiment emphasizes the need to evaluate answer correctness, retrieval quality, and evidence attribution as separate components of a scientific question-answering system.

---

**Project status:** Experimental pilot completed; results and limitations documented.
