---
name: quant-agent-shop
description: Pay-per-call APIs from Quant Agent Shop (quantagentshop.com), paid in USDC on Base over x402 with no account or API key. Use when an agent needs web search (Google web, news, places, shopping, patents, papers), page reading to Markdown, cited research answers, summaries and structured extraction, PDFs and OCR, sanctions screening, SEC filings, GLEIF company records, domain, email and phone checks, SEO audits, IP lookup and weather, and it has an x402-capable wallet or client.
license: MIT
compatibility: Needs network access and a wallet holding USDC on Base (eip155:8453) with an x402 v2 client, for example @x402/fetch, @x402/mcp or the x402 Python package.
metadata:
  homepage: https://quantagentshop.com
  mcp: https://quantagentshop.com/mcp
---

# Quant Agent Shop

Single paid API calls. Base URL: `https://quantagentshop.com`. Every route is `POST` with a JSON body and returns JSON. Failed calls are not charged.

## 1. Discover routes and prices (free)

- `GET https://quantagentshop.com/v1/sample` returns every current route with its path, price in USD and an example input and output. Read it first; the list grows over time.
- `GET https://quantagentshop.com/llms.txt` says when to use each route.
- `GET https://quantagentshop.com/openapi.json` has the full request and response schemas.

Routes at the time of writing (31; `/v1/sample` is the live list):

| Route | USD | Use for |
|---|---|---|
| `/v1/news-search` | 0.01 | News search |
| `/v1/paper-search` | 0.01 | Scholarly paper search |
| `/v1/patent-search` | 0.01 | Patent search |
| `/v1/places-search` | 0.01 | Local business search |
| `/v1/shopping-search` | 0.01 | Product search |
| `/v1/web-search` | 0.01 | Web search |
| `/v1/web-search-batch` | 0.0115 | Batch Google web search |
| `/v1/fetch-page` | 0.01 | Page fetch |
| `/v1/page-change` | 0.01 | Page change check |
| `/v1/seo-audit` | 0.01 | On-page SEO audit of one URL |
| `/v1/seo-audit-pro` | 0.05 | SEO audit with fixes |
| `/v1/url-status` | 0.01 | URL status check |
| `/v1/ocr` | 0.01 | Image text (OCR) |
| `/v1/pdf/from-brief` | 0.01 | Brief to PDF |
| `/v1/pdf/from-markdown` | 0.01 | Markdown to PDF |
| `/v1/pdf/from-url` | 0.01 | Web page to PDF |
| `/v1/pdf-text` | 0.01 | PDF text |
| `/v1/qr` | 0.01 | QR code |
| `/v1/extract` | 0.01 | Structured extraction |
| `/v1/research-answer` | 0.01 | Cited research answer |
| `/v1/summarize` | 0.03 | Summary |
| `/v1/company-lookup` | 0.01 | Company legal-entity lookup from the global LEI system (GLEIF) |
| `/v1/counterparty-check` | 0.03 | Counterparty check in one call |
| `/v1/sanctions-screen` | 0.01 | Sanctions screening against the US Treasury OFAC SDN and Consolidated lists |
| `/v1/sec-filings` | 0.01 | SEC EDGAR filings lookup |
| `/v1/domain-lookup` | 0.01 | Domain lookup in one call |
| `/v1/email-auth-check` | 0.01 | Email authentication check for a domain |
| `/v1/email-check` | 0.01 | Email address check without sending mail |
| `/v1/ip-geo` | 0.01 | IP address lookup for one public IPv4 or IPv6 address |
| `/v1/phone-check` | 0.01 | Phone number check |
| `/v1/weather` | 0.01 | Weather forecast for coordinates or a city name |

Pick the cheapest route that answers the task: search for links, fetch-page to read a known URL, research-answer only when a sourced written answer is needed. PDFs come back as `pdf_base64`; decode before saving.

## 2. Pay with x402 and call over HTTP

1. POST the request without payment. Expect `402 Payment Required` and a `PAYMENT-REQUIRED` header: base64 JSON with `accepts` (scheme `exact`, network `eip155:8453`, USDC asset, `amount` in 6-decimal units, `payTo`).
2. Sign a payment for one entry in `accepts` with the wallet.
3. Repeat the same POST with the signed payment in the `PAYMENT-SIGNATURE` header. A 200 response carries the result and a `PAYMENT-RESPONSE` header with the settlement receipt.

An x402 client library does steps 1 to 3 for you, for example `wrapFetchWithPaymentFromConfig` from `@x402/fetch` with an `ExactEvmScheme` for `eip155:8453`.

Rules:
- Sign with at least 90 seconds of validity (the standard 120 works).
- One signed payment buys one call. Do not reuse a payment.
- One paid call at a time per payer; wait for a response before the next.
- A 4xx or 5xx response after payment is not charged. Read the JSON `error` and `detail` fields, fix the input and pay again.

## 3. Or call over remote MCP

- Server: `https://quantagentshop.com/mcp` (streamable HTTP). Tool names start with `quantagentshop_`, for example `quantagentshop_web_search`.
- An unpaid `tools/call` returns `isError: true` with the payment terms in `structuredContent` (same shape as the HTTP `PAYMENT-REQUIRED` body).
- Retry the call with the signed payment in `params._meta["x402/payment"]`. The receipt comes back in `_meta["x402/payment-response"]`.
- `@x402/mcp`'s `createx402MCPClient` with `autoPayment: true` handles this.

## Limits

- `fetch-page` and `pdf/from-url` read public pages only: no logins, no JavaScript rendering, first 1 MB.
- `pdf/from-markdown` takes up to 50,000 characters; images and raw HTML are not rendered.
- `pdf/from-brief` uses only the brief and general knowledge; it does not search the web.

Contact: johnlenontaken@gmail.com
