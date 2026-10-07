# ApiOne MCP server

B2B company, people and email enrichment for AI agents, with cited sources, built for companies that Western databases cover poorly: GCC and MENA (UAE, Saudi Arabia, Qatar, Kuwait, Bahrain, Oman, Egypt, Morocco).

- Endpoint: `https://apione.store/mcp` (Streamable HTTP, MCP 2025-06-18)
- Auth: `X-API-Key` header. Free key with 100 credits, no card: https://apione.store
- Tool discovery (`initialize`, `tools/list`) works without a key
- Every enriched field carries its sources in `_meta.sources`
- No data, no charge: misses, errors and answers without live evidence are free

This repository holds documentation and examples only. The service itself is a hosted product and is not open source.

## Tools

| Tool | What it does | Credits |
|---|---|---|
| `apione_company_enrich` | Firmographics for a domain: industry, size, HQ, tech stack, growth signals, funding stage | 5 |
| `apione_company_search` | Find companies by country, city, industry, keyword, size or funding stage (up to 25 results) | 3 per result |
| `apione_people_enrich` | Title, seniority, department, skills for a person | 5 |
| `apione_people_search` | Find people by company, title, country | 3 per result |
| `apione_email_find` | Ranked work-email candidates for a person at a domain | 3 |
| `apione_email_verify` | Deliverability, MX validity, catch-all detection | 1 |
| `apione_tech_detect` | CRM, analytics, infrastructure, frameworks | 4 |
| `apione_signals` | Hiring surges, funding, tech adoption, leadership changes | 3 |
| `apione_hiring` | Live open roles from the careers page and public job boards | 3 |
| `apione_name_match` | Do two company names (Arabic or English) refer to the same name? | 1 |
| `apione_name_normalize` | Strip legal forms, return core name, match key and spelling variants | 1 |

Typical latency: company enrichment 5 to 9 s (live-sourced), people about 1.5 s, email find about 2.5 s, verify 1 to 2 s. Full reference: https://apione.store/docs

## Connect

Cursor (`~/.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "apione": {
      "url": "https://apione.store/mcp",
      "headers": { "X-API-Key": "YOUR_KEY" }
    }
  }
}
```

Claude Desktop, via the `mcp-remote` bridge (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "apione": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://apione.store/mcp", "--header", "X-API-Key: YOUR_KEY"]
    }
  }
}
```

Any other MCP client: point it at `https://apione.store/mcp` and send the `X-API-Key` header.

## Example: matching Arabic and English company names

Request:

```bash
curl -s https://apione.store/mcp \
  -H "X-API-Key: $APIONE_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"apione_name_match","arguments":{"name_a":"شركة الفطيم ذ.م.م","name_b":"Al-Futtaim Trading LLC"}}}'
```

Result (trimmed):

```json
{
  "name_a": { "core_name": "الفطيم", "legal_forms_removed": ["ذ م م", "شركه"] },
  "name_b": { "core_name": "futtaim", "legal_forms_removed": ["llc", "trading"] },
  "score": 1,
  "verdict": "same",
  "method": ["normalize", "legal_form_strip", "cross_script_skeleton", "transliteration_variants"],
  "note": "Compares names only. Two different companies can share a name; confirm with the domain before merging records.",
  "_meta": { "endpoint": "companies/match", "credits_used": 1 }
}
```

`apione_name_normalize` on `شركة الفطيم للتجارة ذ.م.م` returns core name `الفطيم`, match key `ftym` and Latin variants `Al-Futtaim`, `Al Futtaim`, `Alfuttaim`.

## Good to know

- Search tools are billed per result returned (3 credits each, up to 25 results). Set `count` to cap spend.
- When a value cannot be verified, the field is `null` and a genuine miss is not charged. Treat `null` as "unknown", not "none".
- Name matching compares names, not entities. Confirm with the domain before merging records.
- Benchmark of the name matching tools on 50 labelled pairs, with its failures: [benchmarks/](benchmarks/) and https://apione.store/blog/arabic-english-company-name-matching
- Pricing, status and policies: https://apione.store (docs, pricing, https://apione.store/status).

## Support and security

Questions and bugs: open an issue. Security reports: see [SECURITY.md](SECURITY.md). Please do not post API keys in issues.

ApiOne (apione.store) is not affiliated with apione.org.
