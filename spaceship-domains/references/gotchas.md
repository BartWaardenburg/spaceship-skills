# Spaceship MCP — Critical Gotchas

Common pitfalls and their correct solutions when working with Spaceship domain tools.

---

## Async Operations Return Operation IDs, Not Results

`register_domain`, `renew_domain`, `restore_domain`, and `transfer_domain` are all asynchronous. They return `{ operationId }` immediately — the operation is **not complete** when the tool returns. You must poll `get_async_operation` until the status reaches a terminal state.

```
# WRONG — assuming the domain is registered after the call returns
register_domain({ domain: "example.com", years: 1 })
# Tool returns { operationId: "op-abc123" } — registration is still in progress!

# CORRECT — poll until complete
register_domain({ domain: "example.com", years: 1 })
→ { operationId: "op-abc123" }

get_async_operation({ operationId: "op-abc123" })
→ { status: "processing" }  # not done yet, poll again

get_async_operation({ operationId: "op-abc123" })
→ { status: "completed" }   # now it's done
```

---

## Financial Operations Cost Real Money

Registration, renewal, restore, and transfer all charge the account. **Always confirm pricing with the user before executing.** Use `check_domain_availability` to show pricing first.

```
# WRONG — registering without confirming cost
register_domain({ domain: "premium-name.com", years: 1 })

# CORRECT — check price, confirm with user, then register
check_domain_availability({ domains: ["premium-name.com"] })
→ { domain: "premium-name.com", available: true, pricing: { register: "$2,499.00" } }

# Show price to user, get explicit confirmation, THEN:
register_domain({ domain: "premium-name.com", years: 1 })
```

Restore operations are especially expensive — always warn the user about redemption grace period pricing.

---

## Privacy Level Requires User Consent

`set_privacy_level` requires `userConsent: true` — the API will reject the request without it. This is a legal requirement for WHOIS privacy changes.

```
# WRONG — missing userConsent
set_privacy_level({ domain: "example.com", level: "high" })
→ Error: userConsent is required

# WRONG — userConsent set to false
set_privacy_level({ domain: "example.com", level: "high", userConsent: false })
→ Error: consent must be true

# CORRECT
set_privacy_level({ domain: "example.com", level: "high", userConsent: true })
```

Always inform the user that enabling WHOIS privacy hides their contact information from public WHOIS lookups, and get their acknowledgment before setting `userConsent: true`.

---

## Auto-Renew Uses `enabled`, Not `autoRenew`

The `set_auto_renew` tool uses `{ enabled: bool }`, which differs from the `autoRenew` parameter used in `register_domain` and `transfer_domain`.

```
# WRONG — using autoRenew parameter name
set_auto_renew({ domain: "example.com", autoRenew: true })
→ Error: unexpected parameter

# CORRECT — use "enabled"
set_auto_renew({ domain: "example.com", enabled: true })
```

Note: `register_domain` and `transfer_domain` use `autoRenew: bool` as a parameter during creation. The naming inconsistency only applies to the standalone `set_auto_renew` tool.

---

## Nameserver Update Format

`update_nameservers` requires a `provider` field and a `nameservers` array. It replaces ALL current nameservers — there is no way to add or remove a single nameserver.

```
# WRONG — passing nameservers without provider
update_nameservers({ domain: "example.com", nameservers: ["ns1.cloudflare.com", "ns2.cloudflare.com"] })

# WRONG — passing a single nameserver as string
update_nameservers({ domain: "example.com", provider: "custom", nameservers: "ns1.cloudflare.com" })

# CORRECT — use provider + array of nameservers
update_nameservers({
  domain: "example.com",
  provider: "custom",
  nameservers: ["ns1.cloudflare.com", "ns2.cloudflare.com"]
})

# CORRECT — switch back to Spaceship's built-in DNS
update_nameservers({ domain: "example.com", provider: "basic" })
```

Maximum 6 nameservers. The `provider` is `"basic"` for Spaceship DNS or `"custom"` for external nameservers.

---

## DNS Delete Requires Name and Type

To delete DNS records, you must pass `{ name, type }` identifiers. Fetch existing records first with `list_dns_records` to get the exact name and type values.

```
# WRONG — trying to delete by address or value
delete_dns_records({ domain: "example.com", records: [{ address: "1.2.3.4" }] })

# WRONG — guessing the record name
delete_dns_records({ domain: "example.com", records: [{ name: "www", type: "A" }] })
# This might fail if the actual name stored is different

# CORRECT — fetch first, then delete with exact data
list_dns_records({ domain: "example.com" })
→ [{ name: "www", type: "CNAME", cname: "example.com", ttl: 3600 }, ...]

delete_dns_records({
  domain: "example.com",
  records: [{ name: "www", type: "CNAME" }]
})
```

---

## DNS Creators Replace All Records of Same Name+Type

Each `create_*_record` tool replaces **ALL** existing records that share the same name and type. This is by design and matches the behavior of `save_dns_records`.

```
# Existing state: two MX records at "@"
#   @ MX 1 aspmx.l.google.com
#   @ MX 5 alt1.aspmx.l.google.com

# WRONG — this REPLACES both existing MX records with just one
create_mx_record({ domain: "example.com", name: "@", priority: 10, exchange: "mail.example.com" })
# Result: only mail.example.com exists, Google MX records are gone!

# CORRECT — use save_dns_records to ADD to existing records
# First fetch existing records:
list_dns_records({ domain: "example.com" })

# Then include ALL desired MX records in one save:
save_dns_records({
  domain: "example.com",
  records: [
    { name: "@", type: "MX", preference: 1, exchange: "aspmx.l.google.com", ttl: 3600 },
    { name: "@", type: "MX", preference: 5, exchange: "alt1.aspmx.l.google.com", ttl: 3600 },
    { name: "@", type: "MX", preference: 10, exchange: "mail.example.com", ttl: 3600 }
  ]
})
```

This is especially important for MX records (where you typically have multiple) and TXT records (SPF, DKIM, DMARC, verification records can coexist at the same name).

---

## Contacts Use String IDs

`save_contact` returns `{ contactId }` as a string. These string IDs are used as references in `register_domain`, `transfer_domain`, and `update_domain_contacts`.

```
# WRONG — passing contact data inline to register_domain
register_domain({
  domain: "example.com",
  contacts: { registrant: { firstName: "John", lastName: "Doe", ... } }
})

# CORRECT — create contact first, then reference by ID
save_contact({
  firstName: "John",
  lastName: "Doe",
  email: "john@example.com",
  address1: "123 Main St",
  city: "Springfield",
  country: "US",
  phone: "+1.5551234567"
})
→ { contactId: "ct-abc123" }

register_domain({
  domain: "example.com",
  contacts: { registrant: "ct-abc123", admin: "ct-abc123", tech: "ct-abc123", billing: "ct-abc123" }
})
```

You can reuse the same `contactId` for all four contact roles, or use different contacts for each.

---

## SellerHub Pricing Uses String Amounts

SellerHub pricing fields use `{ amount: string, currency: string }` — the `amount` must be a **string**, not a number.

```
# WRONG — amount as number
create_sellerhub_domain({
  domain: "premium.com",
  binPriceEnabled: true,
  binPrice: { amount: 5000, currency: "USD" }
})

# CORRECT — amount as string
create_sellerhub_domain({
  domain: "premium.com",
  binPriceEnabled: true,
  binPrice: { amount: "5000", currency: "USD" }
})
```

This applies to both `binPrice` and `minPrice` in `create_sellerhub_domain` and `update_sellerhub_domain`, as well as `basePrice` in `create_checkout_link`.

---

## Checkout Links Require SellerHub Listing First

`create_checkout_link` only works for domains already listed on SellerHub. The `type` must be `"buyNow"`.

```
# WRONG — creating checkout link for unlisted domain
create_checkout_link({ domain: "example.com", type: "buyNow" })
→ Error: domain not listed on SellerHub

# CORRECT — list on SellerHub first, then create link
create_sellerhub_domain({
  domain: "example.com",
  binPriceEnabled: true,
  binPrice: { amount: "10000", currency: "USD" }
})

create_checkout_link({ domain: "example.com", type: "buyNow" })
→ { url: "https://..." }
```

---

## Verification Records Are Account-Level

`get_verification_records` takes **no parameters** — it returns verification records for the entire account, not a specific domain.

```
# WRONG — passing a domain parameter
get_verification_records({ domain: "example.com" })
→ Error: unexpected parameter

# CORRECT — no parameters
get_verification_records()
→ { options: [{ records: [{ name: "...", type: "TXT", value: "..." }] }] }
```

After getting the verification records, you still need to create them on the specific domain using the DNS creator tools.

---

## Rate Limits

The Spaceship API enforces rate limits of **5 requests per 300 seconds per domain** for most write endpoints. Plan your operations to stay within this limit.

```
# WRONG — rapid-fire DNS changes
create_a_record({ domain: "example.com", name: "@", address: "1.1.1.1" })
create_cname_record({ domain: "example.com", name: "www", cname: "example.com" })
create_mx_record({ domain: "example.com", name: "@", priority: 1, exchange: "mx1.example.com" })
create_mx_record({ domain: "example.com", name: "@", priority: 5, exchange: "mx2.example.com" })
create_txt_record({ domain: "example.com", name: "@", value: "v=spf1 ..." })
create_txt_record({ domain: "example.com", name: "_dmarc", value: "v=DMARC1; ..." })
# 6th call may be rate-limited!

# BETTER — batch with save_dns_records to minimize API calls
save_dns_records({
  domain: "example.com",
  records: [
    { name: "@", type: "A", address: "1.1.1.1", ttl: 3600 },
    { name: "www", type: "CNAME", cname: "example.com", ttl: 3600 },
    { name: "@", type: "MX", preference: 1, exchange: "mx1.example.com", ttl: 3600 },
    { name: "@", type: "MX", preference: 5, exchange: "mx2.example.com", ttl: 3600 },
    { name: "@", type: "TXT", value: "v=spf1 ...", ttl: 3600 },
    { name: "_dmarc", type: "TXT", value: "v=DMARC1; ...", ttl: 3600 }
  ]
})
# Single API call covers everything
```

When setting up complex DNS configurations, prefer `save_dns_records` for bulk operations to stay within rate limits.

---

## Domain Deletion Is Not Exposed

There is **no delete domain tool** in the Spaceship MCP server. This is intentional — domain deletion is irreversible and too destructive to expose through an AI assistant.

If a user asks to delete a domain, explain that this must be done manually through the Spaceship control panel. Offer alternatives like:
- Letting the domain expire by disabling auto-renew: `set_auto_renew({ domain, enabled: false })`
- Transferring the domain to another registrar
- Listing it for sale on SellerHub
