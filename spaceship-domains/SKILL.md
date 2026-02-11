---
name: spaceship-domains
description: Manage domains via the Spaceship registrar API — register domains, configure DNS records, manage contacts and WHOIS privacy, transfer domains, set up nameservers, and sell domains on SellerHub. Use when asked to "manage domains", "register a domain", "configure DNS", "add DNS records", "set up nameservers", "check domain availability", "transfer a domain", "sell a domain", "set up email DNS", "manage WHOIS privacy", or any Spaceship domain registrar operation. Requires the Spaceship MCP server to be configured.
---

# Spaceship Domain Management

Manage domains through 48 Spaceship MCP tools across 8 categories: domain lookup, DNS records, contacts & privacy, nameservers, transfers, lifecycle operations, and SellerHub marketplace.

## Tool Loading

All Spaceship tools are deferred. Load them before use:

```
ToolSearch: "+spaceship <keyword>"
```

Examples: `+spaceship dns`, `+spaceship domain`, `+spaceship contact`, `+spaceship sellerhub`.

## Tool Categories

| Category | Tools | Key Operations |
|----------|-------|----------------|
| Domain Lookup | `check_domain_availability` | Check availability + pricing for 1–20 domains |
| Domain Info | `list_domains`, `get_domain` | List/inspect domains in account |
| Domain Settings | `set_auto_renew`, `set_transfer_lock`, `update_nameservers`, `get_auth_code` | Configure domain settings |
| Domain Lifecycle | `register_domain`, `renew_domain`, `restore_domain`, `transfer_domain`, `get_transfer_status`, `get_async_operation` | Registration, renewal, transfer (all async + financial) |
| DNS Management | `list_dns_records`, `save_dns_records`, `delete_dns_records`, `check_dns_alignment` | Bulk DNS operations |
| DNS Creators | `create_a_record`, `create_aaaa_record`, `create_cname_record`, `create_mx_record`, `create_txt_record`, `create_srv_record`, `create_ns_record`, `create_alias_record`, `create_caa_record`, `create_https_record`, `create_ptr_record`, `create_svcb_record`, `create_tlsa_record` | Create individual DNS records by type |
| Contacts & Privacy | `save_contact`, `get_contact`, `save_contact_attributes`, `get_contact_attributes`, `update_domain_contacts`, `set_privacy_level`, `set_email_protection` | Contact profiles, WHOIS privacy |
| Personal NS | `list_personal_nameservers`, `get_personal_nameserver`, `update_personal_nameserver`, `delete_personal_nameserver` | Vanity nameservers (glue records) |
| SellerHub | `list_sellerhub_domains`, `create_sellerhub_domain`, `get_sellerhub_domain`, `update_sellerhub_domain`, `delete_sellerhub_domain`, `create_checkout_link`, `get_verification_records` | Domain marketplace |

For complete parameter specifications, see [references/api-reference.md](references/api-reference.md).

## Common Workflows

### Check & Register a Domain

```
1. check_domain_availability({ domains: ["example.com"] })
   → Returns availability + pricing
2. save_contact({ firstName, lastName, email, address1, city, country, phone })
   → Returns { contactId }
3. register_domain({ domain: "example.com", contacts: { registrant: contactId } })
   → Returns { operationId } (async, financial)
4. get_async_operation({ operationId })
   → Poll until status is "completed"
5. set_privacy_level({ domain, level: "high", userConsent: true })
6. set_transfer_lock({ domain, locked: true })
```

### Configure DNS for a Website

```
1. list_dns_records({ domain: "example.com" })
   → See existing records
2. create_a_record({ domain, name: "@", address: "93.184.216.34" })
3. create_cname_record({ domain, name: "www", cname: "example.com" })
4. create_txt_record({ domain, name: "@", value: "v=spf1 include:_spf.google.com -all" })
5. check_dns_alignment({ domain, expectedRecords: [...] })
   → Verify all records are correct
```

### Set Up Email (Google Workspace)

```
1. create_mx_record({ domain, name: "@", priority: 1, exchange: "aspmx.l.google.com" })
2. create_mx_record({ domain, name: "@", priority: 5, exchange: "alt1.aspmx.l.google.com" })
3. create_mx_record({ domain, name: "@", priority: 5, exchange: "alt2.aspmx.l.google.com" })
4. create_txt_record({ domain, name: "@", value: "v=spf1 include:_spf.google.com ~all" })
5. create_txt_record({ domain, name: "_dmarc", value: "v=DMARC1; p=quarantine; rua=mailto:..." })
6. create_txt_record({ domain, name: "google._domainkey", value: "v=DKIM1; k=rsa; p=..." })
```

### Set Up Email (Microsoft 365)

```
1. create_mx_record({ domain, name: "@", priority: 0, exchange: "<tenant>.mail.protection.outlook.com" })
2. create_txt_record({ domain, name: "@", value: "v=spf1 include:spf.protection.outlook.com -all" })
3. create_cname_record({ domain, name: "autodiscover", cname: "autodiscover.outlook.com" })
4. create_txt_record({ domain, name: "_dmarc", value: "v=DMARC1; p=quarantine; rua=mailto:..." })
```

### Transfer a Domain to Spaceship

```
1. save_contact({ ... })
   → Create contact profile first
2. transfer_domain({ domain, authCode: "EPP-CODE", contacts: { registrant: contactId } })
   → Returns { operationId } (async, financial)
3. get_transfer_status({ domain })
   → Check transfer progress
4. get_async_operation({ operationId })
   → Poll until complete
```

### Sell a Domain on SellerHub

```
1. create_sellerhub_domain({ domain, binPrice: { amount: "5000", currency: "USD" }, binPriceEnabled: true })
2. get_verification_records()
   → Get DNS verification records (account-level)
3. create_txt_record({ domain, name, value })
   → Add the verification TXT record
4. create_checkout_link({ domain, type: "buyNow" })
   → Returns shareable purchase URL
```

### Set Up Vanity Nameservers

```
1. update_personal_nameserver({ domain, host: "ns1.example.com", ips: ["93.184.216.34"] })
2. update_personal_nameserver({ domain, host: "ns2.example.com", ips: ["93.184.216.35"] })
3. update_nameservers({ domain, provider: "custom", nameservers: ["ns1.example.com", "ns2.example.com"] })
```

## Critical Gotchas

- **Async operations**: `register_domain`, `renew_domain`, `restore_domain`, `transfer_domain` all return `{ operationId }` — poll with `get_async_operation` until complete
- **Financial operations**: Registration, renewal, restore, and transfer cost money — always confirm with the user before executing
- **Privacy requires consent**: `set_privacy_level` needs `{ level, userConsent: true }` — consent is mandatory
- **Auto-renew format**: Uses `{ enabled: bool }`, not `{ autoRenew: bool }`
- **Nameserver format**: `update_nameservers` uses `{ provider, nameservers: [...] }`
- **DNS delete needs full data**: Fetch records with `list_dns_records` first, then pass full record data to `delete_dns_records`
- **DNS creators replace**: Each `create_*_record` tool replaces ALL existing records with the same name+type
- **Contacts use string IDs**: `save_contact` returns `{ contactId }`, use these IDs in `register_domain`, `update_domain_contacts`
- **SellerHub pricing**: Uses `{ amount: string, currency: string }` format (amount is a string, not number)
- **Checkout links**: Require `{ type: "buyNow", domain }` — domain must already be listed on SellerHub
- **Verification records**: `get_verification_records` is account-level (no domain parameter)
- **Rate limits**: 5 requests per 300 seconds per domain for most write endpoints
- **Domain deletion**: Intentionally NOT exposed — too destructive

## Instructions

1. **Identify the action** from the user's request
2. **Load required tools** via `ToolSearch` with `+spaceship <keyword>`
3. **Confirm destructive/financial operations** before executing (register, renew, restore, transfer, delete DNS)
4. **Follow the appropriate workflow** above
5. **Poll async operations** for lifecycle operations
6. **Report results** clearly — show domain status, DNS records, pricing, or operation results in readable format

If `$ARGUMENTS` is provided, interpret it as the domain name or action to perform.
