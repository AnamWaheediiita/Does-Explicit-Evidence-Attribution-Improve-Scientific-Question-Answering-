# Research Gap Matrix

## Project

**Does Explicit Evidence Attribution Improve Scientific Question Answering? A Controlled Pilot Study Using QASPER**

## 1. Purpose

This matrix compares relevant existing research with the proposed experimental investigation.

The objective is to identify what previous studies have established, what remains insufficiently examined in the literature reviewed, and how the present experiment addresses a specific research question.

The project does not claim that evidence-grounded question answering is a new research area. Instead, it investigates a controlled comparison of answer-only and evidence-attributed prompting under fixed retrieval conditions.

## 2. Literature Comparison

| Research | Main contribution | Relevance to this project | Remaining research opportunity |
|---|---|---|---|
| Dasigi et al. (2021), QASPER | Introduced a dataset of information-seeking questions and answers grounded in scientific papers, with supporting evidence annotations. | Provides the dataset and evidence annotations used in this experiment. | The dataset itself does not establish whether explicitly requiring a model to cite evidence improves its answers under otherwise identical retrieval conditions. |
| Menick et al. (2022), GopherCite | Investigated training language models to support answers with verified quotations. | Establishes the importance of supporting generated answers with evidence. | The present project studies an inference-time prompting intervention using a fixed compact model rather than training a model to produce verified quotations. |
| Imran and Solanky (2026), ResearchQA | Introduced a benchmark for citation-grounded question answering on scientific papers, including evaluation of factual accuracy and citation quality. | Provides recent context for evaluating scientific answers and their citations. | The reviewed benchmark evaluates systems under a citation contract. The present study asks a narrower controlled question: what changes when an evidence-attribution requirement is introduced versus omitted while keeping the model and retrieved passages fixed? |

## 3. Identified Research Gap

Prior work establishes the importance of scientific question answering, evidence-supported generation, and citation-grounded evaluation.

However, these research directions do not by themselves establish whether simply adding an explicit evidence-attribution instruction improves answer quality.

A useful controlled comparison is to hold the following components constant:

- The scientific question
- The retrieved passages
- The language model
- The decoding configuration
- The evaluation procedure

Only the prompting requirement for explicit evidence attribution is changed.

This project investigates that comparison using QASPER and a lightweight open-source model.

**Research gap statement:**

There is a need for controlled empirical analysis of how an explicit evidence-attribution requirement affects answer correctness and evidence support in scientific question answering when the underlying language model and retrieved context are held constant.

This is a narrowly scoped gap based on the literature reviewed, not a claim that no related experiment exists anywhere in the literature.

## 4. Research Questions

**Primary research question**

Under identical retrieval conditions, how does requiring explicit passage-level evidence attribution affect answer correctness and evidence support in scientific question answering?

**Secondary research questions**

1. Does mandatory evidence attribution reduce unsupported or incorrectly supported answers?
2. Does the effect differ depending on whether at least one gold evidence paragraph is retrieved?
3. How frequently does the model produce evidence identifiers without generating a substantive answer?

## 5. Research Gap to Experimental Design Mapping

| Identified issue | Experimental response | Measurement |
|---|---|---|
| Answers may be generated without explicit supporting evidence. | Compare answer-only and evidence-attributed prompting. | Token F1 and Exact Match |
| Retrieval failures can affect answer quality. | Freeze BM25 top-five retrieval results for both conditions. | Hit@5 and retrieval-based subgroup analysis |
| Generated citations may not correspond to actual evidence. | Compare cited paragraph IDs with mapped gold evidence IDs. | Citation precision, recall and F1 |
| Citation identifiers may be generated without a useful answer. | Inspect outputs after removing citation identifiers and bibliography tokens. | Citation-only output diagnostic |
| Overall scores may hide different failure patterns. | Analyze questions separately according to retrieval success. | Mean F1 and Exact Match by retrieval outcome |

## 6. Positioning of the Present Study

The contribution of this project is an empirical pilot comparison rather than a new model architecture, a new dataset, or a new retrieval algorithm.

Its main contributions are:

1. A controlled comparison of answer-only and evidence-attributed prompting.
2. Reuse of identical frozen BM25 retrieval results across both conditions.
3. Separate measurement of answer quality and evidence-attribution quality.
4. A retrieval-aware error analysis that examines where the model succeeds and fails.
5. A reproducible experimental record using an accessible dataset and a CPU-compatible model.

## 7. References

[1] I. Dasigi et al., “A Dataset of Information-Seeking Questions and Answers Anchored in Research Papers,” in *Proceedings of NAACL-HLT*, 2021. https://arxiv.org/abs/2105.03011

[2] J. Menick et al., “Teaching Language Models to Support Answers with Verified Quotes,” 2022. https://arxiv.org/abs/2203.11147

[3] Imran and Solanky, “ResearchQA: Benchmarking Citation-Grounded Question-Answering on Scientific Papers,” 2026. **Verify the complete author list and preprint identifier against the original paper before final submission.**

[4] S. Robertson and H. Zaragoza, “The Probabilistic Relevance Framework: BM25 and Beyond,” *Foundations and Trends in Information Retrieval*, vol. 3, no. 4, pp. 333–389, 2009.

## 8. Scope and Novelty Disclaimer

The study should be described as a controlled pilot investigation of an inference-time evidence-attribution instruction.

It should not claim:

- That citation-grounded question answering has not been studied before.
- That the proposed method is universally better than answer-only generation.
- That the results generalize to all language models.
- That a 50-question pilot establishes statistically significant superiority.

The contribution is the controlled comparison and the analysis of answer-quality and evidence-attribution failure modes under a fixed retrieval setup.
