# Architecture & trust boundaries

[Back to overview](../README.md)

![Architecture](../assets/architecture.svg)

## Storefront request path

1. A Theme App Extension provides a compact homepage entry and a right-side shopping assistant.
2. Same-store requests pass through Shopify App Proxy to the FastAPI gateway over HTTPS.
3. The gateway checks the proxy signature, shop identity and request freshness before invoking the commerce core.
4. Native tool calling selects product search, product detail or product comparison. Tool arguments and session references are validated before execution.
5. Search uses sparse/dense retrieval and reranking. Detail/comparison read catalog facts; they do not invent products or require rerunning search for an already identified product.
6. Generation is constrained to returned evidence. Product identity and limited factual checks run before the reply; a conservative structured fallback may replace rejected model output.
7. The sidebar verifies the returned product against the storefront product JSON before displaying its image, title, snapshot price and link.

The diagram communicates component responsibilities, not proprietary ranking rules, prompt templates, tuning settings or deployment topology.

## Data boundary

The Shopify adapter maps platform-specific products and variants to a unified product contract. Catalog synchronization and indexing are separate from shopper-facing requests. Draft/archived records were excluded in the recorded integration. An ACTIVE Admin status alone is not a guarantee of storefront publication; the frontend checks visibility separately.

## Two execution profiles

| Responsibility | Million-corpus local experiments | Small-catalog Shopify cloud MVP |
| --- | --- | --- |
| Data | Multi-CPR E-commerce passages | Real Shopify test products |
| Retrieval | BM25 + local BGE dense/reranker | Reused retrieval flow with API-backed embedding/reranking |
| Generation | Local Qwen via Ollama | Hosted Qwen-compatible API |
| Serving | Local evaluation/demo | FastAPI behind Shopify App Proxy |
| Evidence | Historical ranking and Agent experiments | Recorded end-to-end store acceptance |

Local corpus throughput, local Agent quality and cloud interaction latency must not be combined into a single benchmark.

## Limits

Sessions hold result references rather than durable user memory. The current deployment bounds inference concurrency and can return a busy response. Product prices and inventory are snapshots. Evidence checking is deterministic identity validation plus limited heuristics; it is not a general-purpose semantic verification model.
