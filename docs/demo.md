# Using MiWu

[Back to overview](../README.md)

**Demo:** https://miwu-ai-store.myshopify.com/

An actual Shopify development store with a cloud-backed assistant. A storefront access gate or cloud model quota may affect access. No password, API key, merchant credentials or admin login is published. If store access is gated, the recorded examples below remain available without authentication.

## Start a conversation

1. Open the store and click **Ask MiWu** in the compact homepage entry or floating launcher.
2. Ask `Help me find a snowboard`. Inspect returned real images, product names and USD snapshot prices.
3. Ask `Tell me more about the second product`. Use a previous result to request more details.
4. Try a Chinese constraint request: `我想买蓝色滑雪板，预算不超过 750 美元；无法确认的条件请说明。` Inspect whether missing color evidence is explicitly disclosed. Do not treat returned alternatives as proven cheapest products.
5. Try `想买 The Complete Snowboard 滑雪板`. Use Chinese input in the English interface.
6. Open **View product** and check the storefront details. The product page is authoritative for current price and availability.

These are demonstration prompts, not guaranteed exact responses. Results depend on catalog contents and model decisions. Reference-resolution accuracy has known limitations documented in the evaluation.

![Current English sidebar and real product cards](../assets/shopify-sidebar.jpg)

## Recorded bilingual example

![User-tested Chinese request with explicit missing color evidence and product links](../assets/bilingual-evidence-boundary.png)

Captured on 2026-10-08 in the real storefront. The assistant declined to confirm the requested blue color, returned two alternatives below the stated budget, and showed real product cards. The screenshot demonstrates visible behavior, not a tool execution trace, comprehensive factuality or complete cheapest-product ranking.

## Comparison limitation observed in live testing

On 2026-10-08, the user requested a comparison of Complete Snowboard and Collection Snowboard: Hydrogen, including their price difference. The assistant returned five verified product facts and cards, without the requested comparison conclusion or calculation. This is an incomplete result, not a successful comparison demonstration. The fact-list format is consistent with fallback behavior, but screenshots alone do not establish the underlying tool or guardrail cause. Historical local Agent metrics remain separate from this cloud observation.

## Evidence and freshness

- This screenshot comes from the actual store, not a UI mockup. The sidebar is the current accepted design recorded on 2026-10-03.
- The screenshot shows Complete Snowboard at 699.95 USD and Hydrogen at 600.00 USD at capture time. These are historical snapshot prices, not offers.
- Chinese queries and comparison have historical integration acceptance records. The newer live comparison observation above limits what should be claimed about current comparison completion. Neither screenshot proves every feature or an exhaustive multilingual benchmark.
- This repository does not include a runnable clone, backend setup command or public administrative API.
