# MiWu · AI Shopping Assistant

**From natural-language shopping needs to evidence-grounded product discovery — embedded in a real Shopify store.**

[中文](README.zh-CN.md) · [Architecture](docs/architecture.md) · [Evaluation](docs/evaluation.md) · [Demo walkthrough](docs/demo.md) · [Engineering decisions](docs/engineering.md)

MiWu helps shoppers discover products, ask for details, and compare options without leaving a merchant's storefront. It connects retrieval and an LLM tool-calling Agent to real commerce data rather than relying on model knowledge to invent recommendations.

**This is a public project showcase, not an open-source implementation.** Core code, ranking implementation, prompts, business logic and operational configuration remain private. This repository contains product evidence, architecture and selected evaluation aggregates.

## See it in the store

[Open the Shopify demo](https://miwu-ai-store.myshopify.com/) → **Ask MiWu** → `Help me find a snowboard`.

The demo may be behind a storefront access gate. Credentials are not published; a guided walkthrough is the fallback. The link is an actual development store, not an availability guarantee. Initial integration acceptance and the original sidebar screenshot were recorded on **2026-10-03**. The bilingual evidence-boundary example below was captured during user testing on **2026-10-08**.

![MiWu sidebar in the real Shopify store, showing verified product cards](assets/shopify-sidebar.jpg)

*A restrained English sidebar with bilingual input, real product images, snapshot prices and same-store product links. The merchant's storefront remains the main experience.*

### Bilingual discovery with explicit evidence limits

![Chinese shopping request, explicit uncertainty about blue color, and real product cards](assets/bilingual-evidence-boundary.png)

*The shopper asks for a blue snowboard under $750. MiWu responds in Chinese, says the catalog cannot confirm color, and presents budget-compatible alternatives with product links. This demonstrates uncertainty handling and bilingual product discovery; it does not prove global cheapest-product ranking or image-based color recognition.*

## What was built

| Shopper need | Implemented behavior | Engineering behind it |
| --- | --- | --- |
| “Help me find a snowboard” | Search and recommend store products | Sparse + dense retrieval, hybrid ranking and reranking |
| “Tell me about the second one” | Resolve a product from the previous results | Session-scoped product references and validated tool arguments |
| “Compare the first two products” | Comparison tool implemented; complex live comparisons can fall back to a fact list | Native tool calling, product identity checks and evidence-constrained generation |
| “想买 The Complete Snowboard 滑雪板” | Accept Chinese input in an English storefront | Bilingual answer policy with unchanged retrieval/tool contracts |
| Open a recommended product | Follow an actual storefront product link | Storefront visibility and product-ID verification before rendering a card |

## Two validation tracks — kept separate

| Track | Evidence | What it demonstrates |
| --- | --- | --- |
| **Million-product retrieval foundation** | 1,002,822 corpus passages; 1,000 dev queries | Offline retrieval quality and memory-aware local engineering |
| **Real Shopify cloud MVP** | 17 products read; 15 indexed; actual storefront requests | End-to-end integration, deployment and product experience |

The small cloud catalog does **not** demonstrate million-product production serving. Offline results use local BGE/Qwen components; cloud acceptance uses API-backed models. Their quality and latency numbers are not interchangeable.

## Measured retrieval results

Historical Multi-CPR E-commerce dev evaluation, same 1,000-query benchmark:

| Method | Recall@10 | MRR@10 |
| --- | ---: | ---: |
| BM25 | 0.287 | 0.161193 |
| Dense | 0.485 | 0.292127 |
| Hybrid | 0.492 | 0.297857 |
| Hybrid + reranker | **0.501** | **0.314564** |

Hybrid + reranking improves Recall@10 by **21.4 percentage points** over BM25. Reranking adds **0.9 points** over hybrid alone; its MRR@10 improves by **5.61% relatively**. The reranker evaluates a bounded top-20 candidate pool, not the whole corpus.

![Recorded retrieval comparison](assets/retrieval-results.svg)

The historical 40-case native Agent evaluation achieved **82.5% tool-selection accuracy** and **75% reference-resolution accuracy**. Final checks found no candidate-external product IDs among the measured outputs. This is **not** 100% answer accuracy or a hallucination-free claim. [Read denominators, failures and scope](docs/evaluation.md); [inspect the curated numeric evidence](metrics/evaluation.json).

## Architecture

![Conceptual architecture of MiWu's Shopify and offline evaluation tracks](assets/architecture.svg)

Shopify Admin GraphQL → product adapter → unified product schema → retrieval/reranking → Agent tools → evidence checks → FastAPI → signed App Proxy → Theme App Extension sidebar.

The Agent can search products, read details and compare products. It operates under a bounded call budget. Rejected model output can fall back to verified structured product facts. The frontend only renders verified store products. [Architecture and trust boundaries](docs/architecture.md).

## Engineering depth

- **Memory-aware retrieval:** million-scale local vectors stored in float16 memory-mapped files; blockwise exact search avoided a memory limit encountered during an earlier HNSW experiment. Exact scanning still has a CPU cost.
- **Platform adaptation:** a unified product contract keeps Shopify fields and storefront transport separate from the retrieval/Agent core.
- **Constrained generation:** argument validation, product identity checks, limited evidence checks and controlled fallback reduce known failure modes without claiming comprehensive semantic correctness.
- **Small-server delivery:** an independent service and model-API deployment fit a 2-vCPU / 2-GB-class ECS shared with an existing application. No local large-model serving or million-corpus serving is claimed for this instance.
- **Product restraint:** a compact entry and sidebar support discovery inside the merchant's store. Comparison remains conversational; the rejected large workspace is not presented as the current UI.

**Stack:** Python · FastAPI · LangChain native tool calling · BM25 · BGE dense retrieval/reranker (offline) · Qwen/Ollama (local) · hosted model APIs (cloud) · Shopify Admin GraphQL/App Proxy/Theme App Extension · Ubuntu ECS · Nginx · systemd · HTTPS.

## What this demonstrates for AI engineering / FDE

Problem framing → data integration → retrieval evaluation → Agent contracts → constrained deployment → real storefront acceptance → product iteration from user feedback. Tradeoffs and observed failures are documented alongside results, rather than hidden behind a polished demo.

## Scope and next steps

This is a single-store MVP with historical end-to-end acceptance. In live user testing on 2026-10-08, a named two-product comparison returned a broader verified fact list without the requested price difference. Comparison completion and constraint satisfaction remain improvement areas; the screenshots are not a tool-call trace or a cloud Agent accuracy benchmark. Prices are synchronization snapshots; live availability is not guaranteed. Multi-tenant onboarding, load testing, scheduled/webhook synchronization and conversion measurement are not demonstrated. No fine-tuning, LoRA, multi-agent system or universal factuality guarantee is claimed.

To evaluate the project, start with the demo walkthrough, the architecture, and the metrics. Core-code walkthroughs would require a separate private review. This repository has no runnable backend and no secret-bearing setup files. [Publication boundary and asset attribution](NOTICE.md).
