# Engineering decisions and delivery tradeoffs

[Back to overview](../README.md)

| Constraint / failure | Decision | Benefit | Remaining tradeoff |
| --- | --- | --- | --- |
| Million-corpus HNSW experiment reached a memory safety limit | Store float16 vectors in memory-mapped files; use blockwise exact dense search | Completed the offline benchmark without requiring all vectors in RAM | CPU scan cost remains; no claim of ANN-scale production QPS |
| Lexical search misses semantic matches; dense search can lose exact identifiers | Combine sparse and dense retrieval, then rerank a bounded candidate pool | Measured gains over the BM25 baseline | Candidate omissions and reranking regressions remain |
| Model-generated product identities/prices can be wrong | Validate arguments, product IDs, supported facts and final evidence; use structured fallback | Prevent measured invalid candidate references from reaching final output | Heuristic checks do not guarantee all suitability or exclusion claims |
| Shopify-specific fields differ from the original corpus | Introduce an adapter and unified product contract | Reuse the retrieval/Agent core across transport and data sources | Small-store integration is not multi-tenant onboarding |
| Small ECS already hosts another application | Independent directory, environment, port and service; hosted model APIs | Deploy the real-store MVP without stopping the existing service | External API availability, quota and network latency |
| A dramatic homepage/workspace competed with the merchant's store | Restore a compact entry and right sidebar | Keep natural-language discovery accessible without taking over the store | Rich comparisons use conversational facts rather than a separate dashboard |

## Deployment behavior

FastAPI sits behind HTTPS and Shopify's signed App Proxy. Credentials stay in environment configuration. Request budgets, busy handling and manual retry constrain work. Product data is synchronized before querying, not claimed to be live inventory. Service health checks and real-store acceptance are separate from ranking evaluation.

Concrete server addresses, accounts, filesystem locations, service configuration, logs and credentials are intentionally absent. This document describes delivery decisions, not a redeployable infrastructure recipe.

## What to discuss in an AI engineer / FDE interview

- Why a working merchant data boundary matters more than a chatbot-only demo.
- How to distinguish ranking relevance, tool behavior, execution reliability and final-answer validity.
- Why offline million-corpus evaluation and tiny-server cloud serving require different claims.
- How the product-ID and storefront-visibility boundary prevents showing invented or inaccessible product cards.
- What happens on busy requests, missing facts, stale snapshots and failed model calls.
- Which missing evidence would be needed before claiming multi-store production readiness, conversion gains or latency improvements.

## Not implemented or not validated

No fine-tuning/LoRA, multi-agent orchestration or durable user memory. No large-scale production load test, webhook/scheduled-sync acceptance, cross-merchant onboarding or business outcome measurement is presented. Improving detail-tool selection, reference resolution and richer supported comparisons is a concrete next step.
