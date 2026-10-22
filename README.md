<p align="center"><img src="logo.png" width="200" height="200" alt="Quant Agent Shop logo"></p>

# Quant Agent Shop

Quant Agent Shop ([quantagentshop.com](https://quantagentshop.com)) sells single calls to web tools for AI agents: web search, page reading to Markdown, cited research answers and PDFs. Each call is paid in USDC on Base over the [x402](https://www.x402.org/) protocol, so there is no account, sign-up or API key. A call without payment gets HTTP 402 with the price; the agent's x402 client pays and retries. Failed calls are not charged. The same tools are available over remote MCP.

## Routes and prices

Current list (more routes are being added; [`/v1/sample`](https://quantagentshop.com/v1/sample) always has the live list with example inputs and outputs, free):

| Route | Price per call (USD) | What it does |
|---|---|---|
| `POST /v1/web-search` | 0.004 | Up to 20 ranked Google web results with title, URL, snippet and date |
| `POST /v1/fetch-page` | 0.002 | A public page's main content as clean Markdown or plain text |
| `POST /v1/research-answer` | 0.009 | A 120 to 300 word answer with numbered citations and the source URLs read |
| `POST /v1/pdf/from-markdown` | 0.003 | Markdown rendered to an A4 PDF (base64 in JSON) |
| `POST /v1/pdf/from-brief` | 0.006 | A 400 to 900 word document written from a short brief, as PDF plus Markdown |
| `POST /v1/pdf/from-url` | 0.003 | A web page's main content as a clean A4 PDF |

Payment: USDC on Base mainnet (`eip155:8453`), x402 version 2, `exact` scheme.

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
- Tools: `quantagentshop_web_search`, `quantagentshop_fetch_page`, `quantagentshop_research_answer`, `quantagentshop_pdf_from_markdown`, `quantagentshop_pdf_from_brief`, `quantagentshop_pdf_from_url`
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
