# Research Decisions and Experimental Log

## Project Title

**Does Explicit Evidence Attribution Improve Scientific Question Answering? A Controlled Pilot Study Using QASPER**

## 1. Research Objective

This project investigates whether requiring a language model to explicitly identify supporting passages improves the quality and reliability of scientific question answering.

The experiment compares two prompting conditions while keeping the underlying model and retrieved passages fixed:

- **Condition A — Answer-only:** The model generates an answer without being required to provide evidence passage identifiers.
- **Condition B — Evidence-attributed:** The model is instructed to provide an answer supported by explicit passage identifiers.

The primary objective is to measure the effect of this evidence-attribution requirement on answer correctness and evidence support.

## 2. Research Decisions

### Decision 1: Dataset Selection

**Decision:** Use the QASPER dataset.

**Reason:** QASPER contains questions and answers grounded in scientific research papers. Its annotations include answer types and supporting evidence, making it suitable for studying scientific question answering and evidence attribution.

**Verification:** The dataset was successfully loaded from its Hugging Face Parquet files.

Dataset statistics:

| Split | Papers | Questions |
|---|---:|---:|
| Train | 888 | 2,593 |
| Validation | 281 | 1,005 |
| Test | 416 | 1,451 |
| Total | 1,585 | 5,049 |

**License:** CC BY 4.0, as specified in the dataset card.

### Decision 2: Dataset Loading Method

**Decision:** Load the dataset through its Parquet files rather than the original dataset loading script.

**Reason:** The installed Hugging Face `datasets` version no longer supports the legacy QASPER loading script. Loading the available Parquet files allowed the experiment to proceed without relying on the deprecated script.

### Decision 3: Evidence Mapping

**Decision:** Map annotated evidence to the original paper paragraphs using normalized text matching.

**Method:** Text was lowercased and whitespace was normalized before checking whether an evidence statement appeared within a paragraph.

**Training-split validation:**

- Evidence statements examined: 4,209
- Matched evidence statements: 3,691
- Evidence statement matching rate: 87.69%
- Evidence-containing annotations: 2,308
- Annotations with at least one matched evidence statement: 2,161
- Annotation-level matching rate: 93.63%

**Limitation:** Exact substring matching can fail when annotations refer to figures, tables, or text with different formatting. Therefore, unmatched evidence is treated as a limitation of the mapping procedure rather than proof that the annotation is invalid.

### Decision 4: Retrieval Baseline

**Decision:** Use BM25 as the retrieval method.

**Reason:** BM25 is computationally inexpensive, deterministic for a fixed corpus and query, and suitable for a CPU-based experimental environment.

**Configuration:** Retrieve up to five paragraphs from the corresponding scientific paper for each question.

**Validation results:**

| Metric | Result |
|---|---:|
| Evaluated questions | 888 |
| Skipped questions without mapped evidence | 117 |
| Hit@1 | 21.06% |
| Hit@3 | 43.92% |
| Hit@5 | 60.14% |
| MRR@5 | 0.3457 |

Hit@k measures whether at least one mapped gold-evidence paragraph appears among the top-k retrieved paragraphs. MRR@5 measures the reciprocal rank of the first relevant paragraph within the top five, assigning zero when none is retrieved.

### Decision 5: Language Model Selection

**Decision:** Use `google/flan-t5-small`.

**Reason:** The model could be downloaded and executed in the available Google Colab CPU environment without requiring model fine-tuning or a GPU.

**Configuration:**

- Execution environment: Google Colab
- Hardware: CPU
- Maximum input length: 512 tokens
- Maximum generated output: 100 tokens
- Decoding: Greedy decoding (`do_sample=False`)

No model fine-tuning was performed.

### Decision 6: Experimental Conditions

**Decision:** Compare two prompting conditions using the same model and frozen retrieval results.

**Condition A — Answer-only:** The model receives the question and retrieved paragraphs and is asked to provide an answer.

**Condition B — Evidence-attributed:** The model receives the same question and retrieved paragraphs but is additionally instructed to identify supporting passages using their paragraph IDs.

The intention is to isolate the effect of the evidence-attribution instruction rather than the effect of changing the retriever or model.

### Decision 7: Test Sample Selection

**Decision:** Select 50 eligible questions from the official QASPER test split using random seed 42.

- Eligible questions identified: 1,309
- Questions selected: 50
- Questions were required to have usable non-unanswerable reference answers and at least one mapped evidence paragraph.
- Paper IDs were retained for traceability.

Answer-type counts are multi-label because one question may contain annotations of more than one type:

| Answer type | Count |
|---|---:|
| Extractive | 40 |
| Free-form | 26 |
| Yes/no | 7 |

These counts therefore do not add up to 50.

### Decision 8: Frozen Retrieval Protocol

**Decision:** Store the retrieved passages before generating answers.

The same stored top-five retrieval results were supplied to both experimental conditions. This prevents differences in retrieved passages from becoming an additional experimental variable.

Test-sample retrieval result:

- Questions evaluated: 50
- Questions with at least one mapped gold passage in the top five: 29
- Test-sample Hit@5: 58%

### Decision 9: Evaluation Metrics

The following metrics were selected:

**Answer quality**
- Mean token-level F1
- Exact Match rate
- Mean inference time

**Evidence attribution**
- Citation precision
- Citation recall
- Citation F1

**Statistical comparison**
- Paired difference in token F1
- Paired bootstrap confidence interval

The answer metrics are custom lexical comparisons against the available QASPER reference answers. They should not be described as official QASPER benchmark scores.

### Decision 10: Error Analysis

**Decision:** Examine performance according to whether BM25 retrieved at least one mapped gold passage.

This analysis helps distinguish retrieval-related failures from answer-generation and evidence-attribution failures.

A further analysis examined cases where the evidence-attributed condition improved or reduced answer F1.

## 3. Experimental Results

| Metric | Condition A | Condition B |
|---|---:|---:|
| Mean token F1 | 0.1229 | 0.1132 |
| Exact Match | 0.06 | 0.08 |
| Mean inference time | 0.6823 s | 0.6569 s |

Paired comparison:

- Mean token F1 difference (B − A): −0.0097
- B performed better: 7 questions
- A performed better: 5 questions
- Tied: 38 questions
- 95% paired bootstrap confidence interval: [−0.071, 0.0403]

The confidence interval includes zero. Therefore, the observed difference does not establish a reliable improvement or decline in answer quality.

### Evidence Attribution Results

| Metric | Result |
|---|---:|
| Answers with recognized citations | 24/50 |
| Total cited passage IDs | 24 |
| Citations matching mapped gold evidence | 5 |
| Citations appearing in retrieved top five | 23 |
| Overall citation precision | 0.1000 |
| Overall citation recall | 0.0495 |
| Overall citation F1 | 0.0617 |

A notable failure mode was the generation of passage identifiers without a substantive answer. The diagnostic classified all 24 outputs with recognized citations as citation-only after removing identifiers and bibliography tokens.

## 4. Important Limitations

1. The experiment uses a small sample of 50 questions and should be interpreted as a pilot study.
2. The selected sample excludes questions without usable mapped evidence and therefore does not represent every QASPER question type.
3. Evidence mapping relies on normalized substring matching and may miss valid evidence.
4. The compact FLAN-T5-small model frequently generates incomplete, irrelevant, or identifier-only responses.
5. The answer evaluation uses lexical overlap and may not fully capture semantic correctness.
6. The evidence-attributed prompt may have a different token length from the answer-only prompt. Both use a 512-token input limit, so truncation may not be perfectly identical.
7. Evidence annotations from multiple annotators were pooled during sample construction.
8. Results should not be generalized to larger language models or all scientific QA systems.

## 5. Final Research Interpretation

In this pilot experiment, requiring explicit evidence attribution did not improve mean token F1. Condition B had a slightly lower mean token F1 but a slightly higher exact-match rate. The paired bootstrap interval included zero.

More importantly, evidence attribution was unreliable: only five of the 24 recognized cited passages matched the mapped gold evidence. The model also frequently generated passage identifiers without a substantive answer.

These findings suggest that adding a citation requirement to a compact language model is not sufficient by itself to ensure grounded scientific answers. Retrieval quality, answer generation, and evidence selection need to be evaluated separately.

## 6. Reproducibility Artifacts

The following files preserve the main experimental outputs:

- `g3_test_sample.json`
- `g3_frozen_retrieval.json`
- `g3_condition_A.json`
- `g3_condition_B.json`
- `g3_evaluation_results.csv`
- `g3_summary_results.csv`
- `g3_deep_error_analysis.csv`
- `g3_retrieval_error_analysis.csv`

These files should be included in the repository under an appropriate results directory.

## 7. Research Integrity

All reported numerical results in this document are based on the recorded experiment outputs. No missing measurements have been estimated or fabricated.

Any future changes to the dataset, prompts, retrieval configuration, model, or evaluation procedure should be documented separately rather than silently replacing the original experimental results.
