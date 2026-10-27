<p align="center"><img src="logo.png" width="200" height="200" alt="Quant Agent Shop logo"></p>

# Quant Agent Shop

Quant Agent Shop ([quantagentshop.com](https://quantagentshop.com)) sells single API calls to AI agents: web search (Google web, news, places, shopping, patents, papers), page reading to Markdown, cited research answers, summaries and structured extraction, PDFs and OCR, sanctions screening, SEC filings, GLEIF company records, domain, email and phone checks, SEO audits and reports, local business lead lists, IP lookup and weather. Each call is paid in USDC on Base over the [x402](https://www.x402.org/) protocol, so there is no account, sign-up or API key. A call without payment gets HTTP 402 with the price; the agent's x402 client pays and retries. Failed calls are not charged. The same tools are available over remote MCP.

## Routes and prices

All 33 routes ([`/v1/sample`](https://quantagentshop.com/v1/sample) always has the live list with example inputs and outputs, free):

| Route | USD per call | Group | What it does |
|---|---|---|---|
| `POST /v1/lead-list` | 0.2 | Search | Local business lead list |
| `POST /v1/news-search` | 0.0099 | Search | News search |
| `POST /v1/paper-search` | 0.0099 | Search | Scholarly paper search |
| `POST /v1/patent-search` | 0.0099 | Search | Patent search |
| `POST /v1/places-search` | 0.0099 | Search | Local business search |
| `POST /v1/shopping-search` | 0.0099 | Search | Product search |
| `POST /v1/web-search` | 0.0099 | Search | Web search |
| `POST /v1/web-search-batch` | 0.0115 | Search | Batch Google web search |
| `POST /v1/fetch-page` | 0.0099 | Web pages and SEO | Page fetch |
| `POST /v1/page-change` | 0.0099 | Web pages and SEO | Page change check |
| `POST /v1/seo-audit` | 0.0099 | Web pages and SEO | On-page SEO audit of one URL |
| `POST /v1/seo-audit-pro` | 0.05 | Web pages and SEO | SEO audit with fixes |
| `POST /v1/seo-report` | 0.05 | Web pages and SEO | SEO audit report as PDF |
| `POST /v1/url-status` | 0.0099 | Web pages and SEO | URL status check |
| `POST /v1/ocr` | 0.0099 | Documents and PDFs | Image text (OCR) |
| `POST /v1/pdf/from-brief` | 0.0099 | Documents and PDFs | Brief to PDF |
| `POST /v1/pdf/from-markdown` | 0.0099 | Documents and PDFs | Markdown to PDF |
| `POST /v1/pdf/from-url` | 0.0099 | Documents and PDFs | Web page to PDF |
| `POST /v1/pdf-text` | 0.0099 | Documents and PDFs | PDF text |
| `POST /v1/qr` | 0.0099 | Documents and PDFs | QR code |
| `POST /v1/extract` | 0.0099 | Answers and extraction | Structured extraction |
| `POST /v1/research-answer` | 0.0099 | Answers and extraction | Cited research answer |
| `POST /v1/summarize` | 0.03 | Answers and extraction | Summary |
| `POST /v1/company-lookup` | 0.0099 | Compliance and company records | Company legal-entity lookup from the global LEI system (GLEIF) |
| `POST /v1/counterparty-check` | 0.03 | Compliance and company records | Counterparty check in one call |
| `POST /v1/sanctions-screen` | 0.0099 | Compliance and company records | Sanctions screening against the US Treasury OFAC SDN and Consolidated lists |
| `POST /v1/sec-filings` | 0.0099 | Compliance and company records | SEC EDGAR filings lookup |
| `POST /v1/domain-lookup` | 0.0099 | Lookups | Domain lookup in one call |
| `POST /v1/email-auth-check` | 0.0099 | Lookups | Email authentication check for a domain |
| `POST /v1/email-check` | 0.0099 | Lookups | Email address check without sending mail |
| `POST /v1/ip-geo` | 0.0099 | Lookups | IP address lookup for one public IPv4 or IPv6 address |
| `POST /v1/phone-check` | 0.0099 | Lookups | Phone number check |
| `POST /v1/weather` | 0.0099 | Lookups | Weather forecast for coordinates or a city name |

Prices run from US$0.0099 to US$0.2 a call. Payment: USDC on Base mainnet (`eip155:8453`), x402 version 2, `exact` scheme.

## Calling over HTTP with x402

1. Send the request with no payment. The server answers `402 Payment Required` with a `PAYMENT-REQUIRED` header (base64 JSON listing the accepted network, asset, amount and pay-to address).
2. Sign a payment for those terms with your wallet.
3. Send the same request again with the signed payment in the `PAYMENT-SIGNATURE` header. The response carries the result, and the settlement receipt in the `PAYMENT-RESPONSE` header.

Sign each payment with at least 90 seconds of validity (the standard 120 works). Each signed payment buys one call, and a payer can have one paid call in flight at a time.

See the price without paying:

```bash
curl -i -X POST https://quantagentshop.com/v1/fetch-page \
  -H "content-type: application/json" \
  -d '{"url": "https://www.x402.org/", "format": "markdown"}'
# HTTP/1.1 402 Payment Required
# payment-required: eyJ4NDAyVmVyc2lvbiI6Mi...
```

Pay and call with an x402 client, for example [`@x402/fetch`](https://www.npmjs.com/package/@x402/fetch):

```typescript
import { wrapFetchWithPaymentFromConfig } from "@x402/fetch";
import { ExactEvmScheme } from "@x402/evm";
import { privateKeyToAccount } from "viem/accounts";

const account = privateKeyToAccount(process.env.AGENT_WALLET_KEY as `0x${string}`);
const fetchWithPayment = wrapFetchWithPaymentFromConfig(fetch, {
  schemes: [{ network: "eip155:8453", client: new ExactEvmScheme(account) }],
});

const res = await fetchWithPayment("https://quantagentshop.com/v1/web-search", {
  method: "POST",
  headers: { "content-type": "application/json" },
  body: JSON.stringify({ query: "x402 facilitator fees", num: 10 }),
});
console.log(await res.json());
```

The wallet needs USDC on Base. Gas is paid by the facilitator, not the caller.

## Remote MCP

- Endpoint (streamable HTTP): `https://quantagentshop.com/mcp`
- Tools: one per route (33), named `quantagentshop_<route>` with dashes as underscores, for example `quantagentshop_sanctions_screen`, plus a free `quantagentshop_quote` that returns a route's price
- Payment follows the [x402 MCP transport](https://github.com/coinbase/x402/blob/main/specs/transports-v2/mcp.md): an unpaid call returns `isError: true` with the payment terms in `structuredContent`; the client retries with the signed payment in `_meta["x402/payment"]` and gets the receipt back in `_meta["x402/payment-response"]`.

Listing tools is free with any MCP client. Paying for a call needs a client that implements the x402 MCP transport. With [`@x402/mcp`](https://www.npmjs.com/package/@x402/mcp):

```typescript
import { createx402MCPClient } from "@x402/mcp";
import { ExactEvmScheme } from "@x402/evm/exact/client";
import { StreamableHTTPClientTransport } from "@modelcontextprotocol/sdk/client/streamableHttp.js";

const client = createx402MCPClient({
  name: "my-agent",
  version: "1.0.0",
  schemes: [{ network: "eip155:8453", client: new ExactEvmScheme(account) }],
  autoPayment: true,
});
await client.connect(new StreamableHTTPClientTransport(new URL("https://quantagentshop.com/mcp")));
const result = await client.callTool("quantagentshop_fetch_page", { url: "https://www.x402.org/" });
```

Config for hosts that use the common `mcpServers` format with a remote URL, such as Cursor (the host, or a payment-aware proxy in front of it, must handle x402 to complete paid calls):

```json
{
  "mcpServers": {
    "quantagentshop": {
      "url": "https://quantagentshop.com/mcp"
    }
  }
}
```

Gemini CLI uses `httpUrl` for streamable HTTP; this repo is also a Gemini CLI extension:

```bash
gemini extensions install https://github.com/TeaTime0906/quant-agent-shop
```

Agent Skills (skills.sh and compatible agents):

```bash
npx skills add TeaTime0906/quant-agent-shop
```

## Discovery

- [llms.txt](https://quantagentshop.com/llms.txt): routes, prices and when to use each
- [openapi.json](https://quantagentshop.com/openapi.json): request and response schemas, with `x-payment-info`
- [.well-known/x402](https://quantagentshop.com/.well-known/x402): paid resource list
- [MCP registry entry](https://registry.modelcontextprotocol.io/v0/servers?search=com.quantagentshop/web-tools): `com.quantagentshop/web-tools` (see [server.json](server.json))

## Files in this repo

- `SKILL.md`: Agent Skills instructions for discovering, paying for and calling the routes
- `gemini-extension.json`, `GEMINI.md`: Gemini CLI extension
- `.cursor-plugin/plugin.json`, `mcp.json`: Cursor plugin
- `server.json`: MCP registry manifest
- `logo.png`, `logo-1024.png`: logo

## Contact

johnlenontaken@gmail.com

## License

MIT, see [LICENSE](LICENSE). This repo holds documentation and client config; the service itself is hosted at quantagentshop.com.
