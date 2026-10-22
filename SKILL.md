---
name: quant-agent-shop
description: Pay-per-call web tools from Quant Agent Shop (quantagentshop.com), paid in USDC on Base over x402 with no account or API key. Use when an agent needs web search results, the readable content of a web page as Markdown, a short cited research answer, or a PDF made from Markdown, a brief or a web page, and it has an x402-capable wallet or client.
license: MIT
compatibility: Needs network access and a wallet holding USDC on Base (eip155:8453) with an x402 v2 client, for example @x402/fetch, @x402/mcp or the x402 Python package.
metadata:
  homepage: https://quantagentshop.com
  mcp: https://quantagentshop.com/mcp
---

# Quant Agent Shop

Single paid calls to web tools. Base URL: `https://quantagentshop.com`. Every route is `POST` with a JSON body and returns JSON. Failed calls are not charged.

## 1. Discover routes and prices (free)

- `GET https://quantagentshop.com/v1/sample` returns every current route with its path, price in USD and an example input and output. Read it first; the list grows over time.
- `GET https://quantagentshop.com/llms.txt` says when to use each route.
- `GET https://quantagentshop.com/openapi.json` has the full request and response schemas.

Routes at the time of writing:

| Route | USD | Use for |
|---|---|---|
| `/v1/web-search` | 0.004 | Links and snippets for a query (`{"query": "...", "num": 10}`) |
| `/v1/fetch-page` | 0.002 | Reading one public page as Markdown (`{"url": "...", "format": "markdown"}`) |
| `/v1/research-answer` | 0.009 | A cited 120 to 300 word answer (`{"question": "...", "max_sources": 3}`) |
| `/v1/pdf/from-markdown` | 0.003 | Markdown to A4 PDF (`{"markdown": "...", "title": "..."}`) |
| `/v1/pdf/from-brief` | 0.006 | A written document from a short brief, as PDF and Markdown (`{"brief": "..."}`) |
| `/v1/pdf/from-url` | 0.003 | A web page as a clean PDF (`{"url": "..."}`) |

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
