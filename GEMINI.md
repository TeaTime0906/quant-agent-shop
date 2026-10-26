# Quant Agent Shop

The `quantagentshop` MCP server (https://quantagentshop.com/mcp) gives 31 paid tools. Each call costs from US$0.01 in USDC on Base over x402. Failed calls are not charged.

Tools:
- `quantagentshop_news_search` (0.01 USD): News search.
- `quantagentshop_paper_search` (0.01 USD): Scholarly paper search.
- `quantagentshop_patent_search` (0.01 USD): Patent search.
- `quantagentshop_places_search` (0.01 USD): Local business search.
- `quantagentshop_shopping_search` (0.01 USD): Product search.
- `quantagentshop_web_search` (0.01 USD): Web search.
- `quantagentshop_web_search_batch` (0.0115 USD): Batch Google web search.
- `quantagentshop_fetch_page` (0.01 USD): Page fetch.
- `quantagentshop_page_change` (0.01 USD): Page change check.
- `quantagentshop_seo_audit` (0.01 USD): On-page SEO audit of one URL.
- `quantagentshop_seo_audit_pro` (0.05 USD): SEO audit with fixes.
- `quantagentshop_url_status` (0.01 USD): URL status check.
- `quantagentshop_ocr` (0.01 USD): Image text (OCR).
- `quantagentshop_pdf_from_brief` (0.01 USD): Brief to PDF.
- `quantagentshop_pdf_from_markdown` (0.01 USD): Markdown to PDF.
- `quantagentshop_pdf_from_url` (0.01 USD): Web page to PDF.
- `quantagentshop_pdf_text` (0.01 USD): PDF text.
- `quantagentshop_qr` (0.01 USD): QR code.
- `quantagentshop_extract` (0.01 USD): Structured extraction.
- `quantagentshop_research_answer` (0.01 USD): Cited research answer.
- `quantagentshop_summarize` (0.03 USD): Summary.
- `quantagentshop_company_lookup` (0.01 USD): Company legal-entity lookup from the global LEI system (GLEIF).
- `quantagentshop_counterparty_check` (0.03 USD): Counterparty check in one call.
- `quantagentshop_sanctions_screen` (0.01 USD): Sanctions screening against the US Treasury OFAC SDN and Consolidated lists.
- `quantagentshop_sec_filings` (0.01 USD): SEC EDGAR filings lookup.
- `quantagentshop_domain_lookup` (0.01 USD): Domain lookup in one call.
- `quantagentshop_email_auth_check` (0.01 USD): Email authentication check for a domain.
- `quantagentshop_email_check` (0.01 USD): Email address check without sending mail.
- `quantagentshop_ip_geo` (0.01 USD): IP address lookup for one public IPv4 or IPv6 address.
- `quantagentshop_phone_check` (0.01 USD): Phone number check.
- `quantagentshop_weather` (0.01 USD): Weather forecast for coordinates or a city name.

The live list and prices are at https://quantagentshop.com/v1/sample.

How payment works: a call without payment returns an error result whose content is the x402 payment terms (`x402Version`, `accepts` with network `eip155:8453`, USDC amount and `payTo`). The call goes through only when it is retried with a signed payment in `_meta["x402/payment"]`, which needs an x402-capable MCP client or wallet. If the terms come back and no wallet is set up, tell the user the price and that the tool needs x402 payment; do not retry in a loop.

Use the cheapest tool that answers the task: search for links, fetch_page to read a known URL, research_answer only when a sourced written answer is needed, sanctions_screen or counterparty_check before paying a new counterparty.
