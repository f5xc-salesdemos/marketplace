---
name: people-search
description: >-
  Person lookup, identity verification, background research, and people
  finder tools. Use when the user mentions: people search, person lookup,
  find person, background check, identity, people finder, reverse face
  search, genealogy, registry search, white pages.
user-invocable: false
---

# People Search

Person lookup, identity verification, background research, and people
finder tools using public records and aggregation services.

## Legal Notice

All tools use publicly available information only. Users must comply
with applicable laws and platform terms of service.

## Tools Reference

Read `skills/people-search/references/tools.md` for the complete
list of 18 free/freemium tools in this category.

## Web Resources

| Tool | URL | Pricing | Best For |
|------|-----|---------|----------|
| **General People Search** | | | |
| ThatsThem | thatsthem.com | Freemium | Free multi-input people lookups (name, email, phone, IP) |
| PeekYou | peekyou.com | Free | Social media profile aggregation by name |
| Webmii | webmii.com | Free | Web visibility scoring and online presence |
| Snitch.name | snitch.name | Free | Cross-platform social media discovery (~40 networks) |
| Lullar | com.lullar.com | Free | Username/email profile enumeration (148+ platforms) |
| Yasni | yasni.com | Free | Name-based search with professional context |
| IDCrawl | idcrawl.com | Free | Aggregated social + public records search |
| AnyWho | anywho.com | Free | US white pages and phone directory |
| Addresses.com | addresses.com | Freemium | US people and address lookups (detailed reports are **paywalled** via Intelius) |
| FaceCheckID | facecheck.id | Freemium | Reverse face image search and identity matching |
| InfoFlow (Chilean) | infoflow.cloud | Freemium | Chilean public records (RUN, vehicles, companies). Requires **registration** |
| **Genealogy & Family** | | | |
| FamilySearch.org | familysearch.org | Free | Free genealogy with billions of historical records |
| Ancestry.com | ancestry.com | Freemium | Largest genealogy platform (most features **paywalled**) |
| **Registries** | | | |
| The Knot | theknot.com | Free | Wedding registry search by couple name |
| Registry Finder | registryfinder.com | Free | Cross-retailer gift registry search |
| My Registry | myregistry.com | Free | Universal cross-store registry search |
| Amazon Registry | amazon.com/registries | Free | Amazon gift registry and wish list search |
| The Bump | registry.thebump.com | Free | Baby registry search by parent name |

**Paywalled tools note**: Addresses.com and Ancestry.com provide limited free results but require payment for full reports. InfoFlow requires registration. FaceCheckID offers limited free searches.

## Subcategories

- **General People Search** -- Broad name/email/phone/address lookups across aggregated public data (ThatsThem, PeekYou, IDCrawl, AnyWho, Webmii, Yasni, Snitch.name, Lullar)
- **Background Checks** -- Deeper identity and record lookups, often freemium or paywalled (Addresses.com via Intelius, Ancestry.com, InfoFlow)
- **Reverse Face Search** -- Match a face photo to online profiles (FaceCheckID)
- **Genealogy** -- Historical records, family trees, and lineage (FamilySearch.org, Ancestry.com)
- **Registries** -- Gift and event registries revealing relationships and life events (The Knot, Registry Finder, My Registry, Amazon Registry, The Bump)

## Delegation

### Tool Lookup

```
Agent(
  subagent_type="osint-framework:osint-researcher",
  description="People Search tool search",
  prompt="Find OSINT tools for People Search.\n
    Read skills/people-search/references/tools.md\n
    Return recommendations matching the user's specific need."
)
```

### Active Investigation

```
Agent(
  subagent_type="osint-framework:osint-investigator",
  description="People Search investigation: [target]",
  prompt="Investigate person: [target]\n\n
    Primary: Read skills/people-search/references/tools.md\n
    Secondary: Read skills/username-recon/references/tools.md\n
    Secondary: Read skills/social-networks/references/tools.md\n
    Execute available CLI tools, query web resources, report findings.\n
    Start with free tools before freemium services."
)
```

## Investigation Workflow

1. **Name Search**: Start with free aggregators (ThatsThem, PeekYou, IDCrawl) for initial hits
2. **Web Presence**: Check Webmii for visibility score and Snitch.name for social media profiles
3. **Username Pivot**: If a username is found, pivot to `username-recon` for cross-platform enumeration
4. **Social Networks**: Deep-dive into discovered social profiles via `social-networks`
5. **Email Pivot**: If email found, verify and check breaches via `email-recon`
6. **Face Search**: If a photo is available, use FaceCheckID for reverse face matching
7. **Registry Search**: Check wedding/baby registries for relationship and life event data
8. **Genealogy**: For historical or family context, use FamilySearch.org (free) before Ancestry.com
9. **Public Records**: Cross-reference with `public-records` for court, property, and government data

## Cross-Category Pivots

| When you find... | Pivot to |
|------------------|----------|
| Username or handle | `username-recon` -- Cross-platform username enumeration |
| Social media profiles | `social-networks` -- Platform-specific deep investigation |
| Email address | `email-recon` -- Verification, breach data, domain analysis |
| Phone number | `phone-recon` -- Carrier lookup, reverse phone, caller ID |
| Court or government records needed | `public-records` -- Court filings, property, voter records |

## OPSEC Notes

- Most tools (13 of 18) are **passive** with no registration required
- FaceCheckID, InfoFlow, and Ancestry.com are **active** -- they log queries and may require accounts
- ThatsThem, PeekYou, and IDCrawl are free but may display your search to the subject in some cases (people search notification features)
- Registry searches (The Knot, Amazon, The Bump) are fully passive and reveal relationship data without alerting the subject
- Genealogy sites may expose your research account name to other users of shared family trees
- People search aggregators frequently contain outdated or incorrect data; always cross-reference
- Be aware of legal restrictions: FCRA-regulated background check data requires permissible purpose
