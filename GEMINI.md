# Quant Agent Shop

The `quantagentshop` MCP server (https://quantagentshop.com/mcp) gives paid web tools. Each call costs a fraction of a cent in USDC on Base over x402. Failed calls are not charged.

Tools:
- `quantagentshop_web_search` (0.004 USD): ranked Google web results for a query.
- `quantagentshop_fetch_page` (0.002 USD): a public page's main content as Markdown or text.
- `quantagentshop_research_answer` (0.009 USD): a 120 to 300 word answer with numbered citations.
- `quantagentshop_pdf_from_markdown` (0.003 USD): Markdown to an A4 PDF (base64).
- `quantagentshop_pdf_from_brief` (0.006 USD): a document written from a short brief, as PDF and Markdown.
- `quantagentshop_pdf_from_url` (0.003 USD): a web page as a clean PDF.

The live list and prices are at https://quantagentshop.com/v1/sample.

How payment works: a call without payment returns an error result whose content is the x402 payment terms (`x402Version`, `accepts` with network `eip155:8453`, USDC amount and `payTo`). The call goes through only when it is retried with a signed payment in `_meta["x402/payment"]`, which needs an x402-capable MCP client or wallet. If the terms come back and no wallet is set up, tell the user the price and that the tool needs x402 payment; do not retry in a loop.

Use the cheapest tool that answers the task: search for links, fetch_page to read a known URL, research_answer only when a sourced written answer is needed.
