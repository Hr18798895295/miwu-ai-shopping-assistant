# Evaluation: measured results, denominators and limitations

[Back to overview](../README.md) · [Numeric evidence](../metrics/evaluation.json) · [Retrieval CSV](../metrics/retrieval.csv)

These are recorded historical experiments, curated on 2026-10-08. No new model evaluation was run to produce this repository. Sources were checked against the private aggregate reports; their SHA-256 fingerprints are recorded in the public JSON. Selected aggregates are inspectable, but the private evaluator/data/logs are required for independent reproduction.

## Retrieval

Multi-CPR E-commerce: **1,002,822 passages, 1,000 dev queries**, evaluated locally on CPU. Train-subset tuning and dev queries were recorded as disjoint. Qrels were used for tuning/evaluation, not for online recommendation or ranking fabrication.

| Method | Recall@10 | MRR@10 | Recall@100 | Recall@1000 |
| --- | ---: | ---: | ---: | ---: |
| BM25 | 0.287 | 0.161193 | 0.512 | 0.715 |
| Dense (raw query) | 0.485 | 0.292127 | 0.757 | 0.901 |
| Hybrid | 0.492 | 0.297857 | 0.765 | 0.909 |
| Hybrid + reranker | 0.501 | 0.314564 | Not reported | Not reported |

Recall@K is the evaluator's query hit rate: fraction of dev queries with at least one annotated relevant item in the top K. MRR@10 averages reciprocal rank of the first relevant hit inside the top 10; a miss contributes zero. This is ranking relevance against dataset labels, not a suitability, purchase or conversion metric.

The reranker only reorders the top-20 candidate pool. Its Recall@100/1000 is not reported. Candidates missing a relevant item cannot be recovered by reranking. Historical rank-change counts include 206 improvements, 198 degradations and 423 queries with the ground-truth item outside the pool; the reranker is not uniformly beneficial.

## Native tool-calling Agent

40 local CPU target cases using **qwen3:4b-instruct**:

| Metric | Recorded result | Denominator / interpretation |
| --- | ---: | --- |
| Tool selection | 82.5% | 33 correct decisions / 40 target cases, including non-shopping cases |
| Argument validity | 100% | Actually requested tool calls; omitted calls are not successes |
| Reference resolution | 75% | 12 / 16 reference cases |
| Execution success | 100% | No execution exception; not semantic-answer correctness |
| Final invalid product-ID rate | 0% | 0 invalid formatted references / 69 final references |
| Final candidate-external product rate | 0% | 0 candidate-external explicit mentions / 69 final mentions |
| Non-shopping tool avoidance | 100% | Small predefined non-shopping subset |

The recorded final checker blocked 12 answers. Before final checks, invalid-ID rate was 2.74% and candidate-external product rate was 3.33%; denominators differ, so these are not one unified hallucination rate. No tool was chosen in 7 shopping/detail cases where one was expected. Detail-tool selection was only 54.55%; it remains an improvement target.

Unsupported-fact checks covered five predefined missing-fact cases, not all answer claims. Two of those cases omitted required tools. A zero final heuristic violation rate is therefore not a universal factuality score. Manual inspection still found accessories/category mismatch and insufficiently supported comparisons.

Historical Agent latency: mean **22.93 s**, P50 **22.66 s**, P95 **52.78 s** on local CPU. Preceding searches used to set up reference cases were excluded; this is not 40 full cold-start conversations or a cloud latency SLA.

## Earlier intent-first RAG experiments

These snapshots describe an earlier pipeline, not the current deployed Agent's accuracy:

| Layer | Cases | Selected findings |
| --- | ---: | --- |
| Intent | 50 | Schema parsing 100%; product-type extraction 74.42%; exclusion extraction 50%; no-extra-guess rate 34% |
| RAG | 30 | 24 completed, 6 parse failures; completion 80%; retrieval success 52.17%; evidence-citation rate 50% |

RAG retrieval success uses all expected-shopping cases, including routing failures. Shopping misrouting was 21.74%. Safe outputs after checks can improve validity while reducing deliverable coverage; all these numbers belong together.

## Shopify integration acceptance

Recorded on **2026-10-03**: 17 products read, 15 indexed, two draft/archived products excluded; real English/Chinese requests and product comparison passed through the store. The local regression snapshot ran 61 tests: 59 passed and 2 optional-dependency skips. Those tests are not 61 user-quality benchmarks.

Cloud chat model last verified: qwen3.8-flash. The 15-product catalog, cloud models and integration requests were not rerun through the 1,000-query offline benchmark. There is no measured production concurrency, multi-store isolation benchmark or conversion uplift in this repository.
