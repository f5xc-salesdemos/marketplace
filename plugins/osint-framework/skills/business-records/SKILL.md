---
name: business-records
description: >-
  Corporate filings, annual reports, SEC data, company research, and organizational intelligence.
  Use when the user mentions: company research, business lookup, SEC filing, corporate records,
  annual report, company information, OpenCorporates, Companies House, Crunchbase, corporate structure.
user-invocable: false
---

# Business Records

Corporate filings, annual reports, SEC data, company research,
employee profiles, and organizational intelligence across global
jurisdictions.

## Legal Notice

All tools use publicly available information only. Users must comply
with applicable laws and platform terms of service.

## Tools Reference

Read `skills/business-records/references/tools.md` for the complete
list of 28 free tools in this category.

## Web Resources

| Resource | URL | Best For |
|----------|-----|----------|
| SEC EDGAR | https://www.sec.gov/cgi-bin/browse-edgar | US public company filings (10-K, 10-Q, 8-K) |
| OpenCorporates | https://opencorporates.com/ | 200M+ companies across 200+ jurisdictions (API) |
| Companies House | https://beta.companieshouse.gov.uk/ | UK company registry, officers, filings (API) |
| Crunchbase | https://www.crunchbase.com/ | Startup funding, investors, acquisitions (API) |
| LittleSis | https://littlesis.org/ | Power relationship mapping (API) |
| Google Finance | https://www.google.com/finance/ | Stock data, financial summaries |
| AnnualReports.com | https://www.annualreports.com/ | Public company annual reports |
| Company Data Rex | https://www.cdrex.com/ | Cross-border EU company research |
| Europages | https://www.europages.co.uk/ | European B2B company directory (3M+ companies) |
| International Registries | https://www.gov.uk/government/publications/overseas-registries/overseas-registries | Official registry links by country |
| Buzzfile | https://www.buzzfile.com/ | US business profiles by SIC code |
| RecruitEm | https://recruiting.net/ | X-Ray search LinkedIn/GitHub profiles |
| EU VIES VAT | https://ec.europa.eu/taxation_customs/vies/ | EU VAT number validation (API) |
| Rusprofile | https://www.rusprofile.ru/ | Russian company registry data |

## Subcategories

- **Annual Reports** -- Public company financial reports and investor documents
- **SEC Filings** -- 10-K, 10-Q, 8-K, proxy statements, insider transactions
- **Company Profiles** -- Registration data, officers, addresses across jurisdictions
- **Employee / Executive Research** -- LinkedIn X-Ray, professional profiles, org charts
- **International Registries** -- UK, EU, Russian, Swiss, and global company registers
- **Corporate Relationships** -- Board memberships, political donations, power networks
- **Patents / Trademarks** -- Intellectual property associated with companies

## Investigation Workflow

1. **Identify the entity**: Search OpenCorporates for registration details across jurisdictions
2. **US public companies**: Query SEC EDGAR for financial filings (10-K, 10-Q, insider trades)
3. **UK companies**: Check Companies House for officers, filing history, charges
4. **Funding / startup**: Search Crunchbase for investment rounds, investors, acquisitions
5. **Annual reports**: Download from AnnualReports.com for financial narrative
6. **EU companies**: Use Company Data Rex, Europages, or VIES VAT validation
7. **Relationships**: Map power networks via LittleSis (board memberships, donors)
8. **Employee enumeration**: Use RecruitEm for LinkedIn X-Ray searches
9. **Cross-reference**: Pivot to `domain-recon` for company websites, `compliance-risk` for sanctions

## curl / API Patterns

### SEC EDGAR -- Full-Text Filing Search

```bash
curl -s "https://efts.sec.gov/LATEST/search-index?q=%22acme+corp%22&dateRange=custom&startdt=2023-01-01&enddt=2024-01-01&forms=10-K" \
  -H "User-Agent: CompanyName admin@company.com" \
  | jq '.hits.hits[] | {filing: ._source.file_description, date: ._source.file_date}'
```

### SEC EDGAR -- Company Filings by CIK

```bash
curl -s "https://data.sec.gov/submissions/CIK0000320193.json" \
  -H "User-Agent: CompanyName admin@company.com" \
  | jq '{name, cik, filings: .filings.recent | {forms: .form[:5], dates: .filingDate[:5]}}'
```

### OpenCorporates -- Company Search

```bash
curl -s "https://api.opencorporates.com/v0.4/companies/search?q=acme+corp&api_token=YOUR_TOKEN" \
  | jq '.results.companies[] | {name: .company.name, jurisdiction: .company.jurisdiction_code, status: .company.current_status}'
```

### Companies House UK -- Company Details

```bash
curl -s "https://api.company-information.service.gov.uk/company/00000001" \
  -H "Authorization: Basic $(echo -n 'YOUR_API_KEY:' | base64)" \
  | jq '{company_name, company_status, type, date_of_creation, registered_office_address}'
```

### Crunchbase -- Organization Lookup

```bash
curl -s "https://api.crunchbase.com/api/v4/entities/organizations/openai?user_key=YOUR_KEY" \
  | jq '{name: .properties.name, short_description: .properties.short_description, funding_total: .properties.funding_total}'
```

### EU VIES -- VAT Number Validation

```bash
curl -s "https://ec.europa.eu/taxation_customs/vies/rest-api/ms/DE/vat/123456789" \
  | jq '{valid, name, address}'
```

### LittleSis -- Entity Relationships

```bash
curl -s "https://littlesis.org/api/entities/search?q=goldman+sachs" \
  | jq '.data[] | {name: .attributes.name, type: .attributes.primary_ext, blurb: .attributes.blurb}'
```

## Cross-Category Pivots

| When you find... | Pivot to | Why |
|------------------|----------|-----|
| Company domain in filings | `domain-recon` | WHOIS, DNS, subdomains, technology stack |
| Officers or executives | `people-search` | Personal records, social profiles |
| Sanctions or regulatory flags | `compliance-risk` | OFAC, PEP screening, beneficial ownership |
| Company address | `geolocation` | Verify physical location, satellite imagery |
| Court cases in filings | `public-records` | Detailed dockets, opinions, outcomes |
| Intellectual property | `public-records` | USPTO patent details, Google Patents |

## OPSEC Notes

- Most tools are **passive** -- no target notification
- LinkedIn and XING are **active** -- profile views may be logged
- SEC EDGAR requires a User-Agent header with contact email (policy, not authentication)
- Companies House API requires free registration; usage is tracked
- Crunchbase API requires registration and has rate limits
- OpenCorporates free tier has limited API calls; consider caching results
- RecruitEm generates Google dork queries -- the Google search itself is logged
- Use research-dedicated accounts for registration-required services

## Delegation

### Tool Lookup

```
Agent(
  subagent_type="osint-framework:osint-researcher",
  description="Business Records tool search",
  prompt="Find OSINT tools for Business Records.\n
    Read skills/business-records/references/tools.md\n
    Return recommendations matching the user's specific need."
)
```

### Active Investigation

```
Agent(
  subagent_type="osint-framework:osint-investigator",
  description="Business Records investigation: [target]",
  prompt="Investigate using Business Records tools: [target]\n\n
    Primary: Read skills/business-records/references/tools.md\n
    Secondary: Read skills/compliance-risk/references/tools.md\n
    Execute available CLI tools, query web resources, report findings.\n
    Start with OpenCorporates and SEC EDGAR, then expand to jurisdiction-specific registries."
)
```
