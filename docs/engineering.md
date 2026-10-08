# Engineering decisions and delivery tradeoffs

[Back to overview](../README.md)

| Constraint / failure | Decision | Benefit | Remaining tradeoff |
| --- | --- | --- | --- |
| Million-corpus HNSW experiment reached a memory safety limit | Store float16 vectors in memory-mapped files; use blockwise exact dense search | Completed the offline benchmark without requiring all vectors in RAM | CPU scan cost remains; no claim of ANN-scale production QPS |
| Lexical search misses semantic matches; dense search can lose exact identifiers | Combine sparse and dense retrieval, then rerank a bounded candidate pool | Measured gains over the BM25 baseline | Candidate omissions and reranking regressions remain |
| Model-generated product identities/prices can be wrong | Validate arguments, product IDs, supported facts and final evidence; use structured fallback | Prevent measured invalid candidate references from reaching final output | Heuristic checks do not guarantee all suitability or exclusion claims |
| Shopify-specific fields differ from the original corpus | Introduce an adapter and unified product contract | Reuse the retrieval/Agent core across transport and data sources | Small-store integration is not multi-tenant onboarding |
| Small ECS already hosts another application | Independent directory, environment, port and service; hosted model APIs | Deploy the real-store MVP without stopping the existing service | External API availability, quota and network latency |
| Product discovery needs to remain accessible within the merchant's store | Use a compact entry and right sidebar | Keep natural-language discovery accessible without taking over the store | Rich comparisons use conversational facts rather than a separate dashboard |

## Deployment behavior

FastAPI sits behind HTTPS and Shopify's signed App Proxy. Credentials stay in environment configuration. Request budgets, busy handling and manual retry constrain work. Product data is synchronized before querying, not claimed to be live inventory. Service health checks and real-store acceptance are separate from ranking evaluation.

Concrete server addresses, accounts, filesystem locations, service configuration, logs and credentials are intentionally absent. This document describes delivery decisions, not a redeployable infrastructure recipe.

## Design principles

- Ground product discovery in merchant catalog data.
- Measure ranking relevance, tool behavior, execution reliability and answer validity separately.
- Keep offline million-corpus evaluation separate from small-server cloud validation.
- Validate product identity and storefront visibility before displaying cards.
- Provide controlled handling for busy requests, missing facts, stale snapshots and model failures.
- Require additional validation before extending production-scale or business-outcome claims.

## Not implemented or not validated

No fine-tuning/LoRA, multi-agent orchestration or durable user memory. No large-scale production load test, webhook/scheduled-sync acceptance, cross-merchant onboarding or business outcome measurement is presented. Improving detail-tool selection, reference resolution and richer supported comparisons is a concrete next step.
