# Spaceship MCP Tools — API Reference

Complete parameter specifications for all 48 tools. All `domain` parameters: string, min 4 chars, max 255 chars. All `ttl` parameters: number, min 60, max 86400, default 3600.

## Table of Contents

- [Domain Lookup & Info](#domain-lookup--info)
- [Domain Settings](#domain-settings)
- [Domain Lifecycle (Async/Financial)](#domain-lifecycle)
- [DNS Management](#dns-management)
- [DNS Record Creators](#dns-record-creators)
- [Contacts & Privacy](#contacts--privacy)
- [Personal Nameservers](#personal-nameservers)
- [SellerHub Marketplace](#sellerhub-marketplace)
- [Analysis](#analysis)

---

## Domain Lookup & Info

### `check_domain_availability`
Check availability and pricing for 1–20 domains.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domains` | string[] | yes | Domain names to check (1–20 items) |

Returns: Array of `{ domain, available, result, pricing }`.

### `list_domains`
List all domains in the account.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `fetchAll` | boolean | no | true | Auto-paginate all results |
| `take` | number | no | 100 | Items per page (1–100) |
| `skip` | number | no | 0 | Offset for pagination |
| `orderBy` | enum | no | — | `"name"`, `"-name"`, `"registrationDate"`, `"-registrationDate"`, `"expirationDate"`, `"-expirationDate"`, `"unicodeName"`, `"-unicodeName"` |

### `get_domain`
Get detailed info (dates, auto-renew, privacy, nameservers, contacts).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | yes | Domain name |

---

## Domain Settings

### `set_auto_renew`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | yes | Domain name |
| `enabled` | boolean | yes | Enable/disable auto-renewal |

### `set_transfer_lock`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | yes | Domain name |
| `locked` | boolean | yes | Lock/unlock transfers |

### `update_nameservers`
Replaces ALL current nameservers.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `domain` | string | yes | — | Domain name |
| `provider` | enum | no | `"custom"` | `"basic"` (Spaceship DNS) or `"custom"` (external) |
| `nameservers` | string[] | no | [] | NS hostnames (max 6, required for "custom") |

### `get_auth_code`
Get EPP/authorization code for outbound transfer.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | yes | Domain name |

Returns: `{ authCode, expires }`.

### `set_email_protection`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | yes | Domain name |
| `contactForm` | boolean | yes | true = show contact form, false = show raw email |

---

## Domain Lifecycle

All lifecycle tools are **async** and **financial**. They return `{ operationId }` — poll with `get_async_operation`.

### `register_domain`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `domain` | string | yes | — | Domain to register |
| `years` | number | no | 1 | Registration period (1–10) |
| `autoRenew` | boolean | no | true | Enable auto-renewal |
| `privacyLevel` | enum | no | `"high"` | `"high"` or `"public"` |
| `contacts` | object | no | — | `{ registrant?, admin?, tech?, billing? }` — all string contact IDs |

### `renew_domain`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | yes | Domain to renew |
| `years` | number | yes | Renewal period (1–10) |
| `currentExpirationDate` | string | yes | ISO 8601 date (e.g. "2025-12-31") — must match actual expiry |

### `restore_domain`
Restore from redemption grace period (expensive).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | yes | Domain to restore |

### `transfer_domain`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `domain` | string | yes | — | Domain to transfer in |
| `authCode` | string | no | — | EPP code (required for most TLDs) |
| `autoRenew` | boolean | no | true | Enable auto-renewal |
| `privacyLevel` | enum | no | `"high"` | `"high"` or `"public"` |
| `contacts` | object | no | — | Same contact ID structure as register |

### `get_transfer_status`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | yes | Domain to check |

### `get_async_operation`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operationId` | string | yes | Operation ID from lifecycle tools |

---

## DNS Management

### `list_dns_records`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `domain` | string | yes | — | Domain name |
| `fetchAll` | boolean | no | true | Auto-paginate |
| `take` | number | no | 500 | Items per page (1–500) |
| `skip` | number | no | 0 | Offset |
| `orderBy` | enum | no | — | `"type"`, `"-type"`, `"name"`, `"-name"` |

### `save_dns_records`
Bulk upsert — replaces ALL records with same name+type combination.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | yes | Domain name |
| `records` | array | yes | Array of DNS record objects (min 1). See DNS record fields below. |

**DNS record fields** (used in `save_dns_records`):

| Field | Type | Used By | Description |
|-------|------|---------|-------------|
| `name` | string | all | Subdomain name ("@" for root) |
| `type` | string | all | A, AAAA, ALIAS, CAA, CNAME, HTTPS, MX, NS, PTR, SRV, SVCB, TLSA, TXT |
| `ttl` | number | all | TTL in seconds (60–86400, default 3600) |
| `address` | string | A, AAAA | IP address |
| `aliasName` | string | ALIAS | Alias target |
| `flag` | number | CAA | 0 or 128 |
| `tag` | string | CAA | "issue", "issuewild", "iodef" |
| `value` | string | TXT, CAA | Text/CA value |
| `cname` | string | CNAME | Canonical name |
| `svcPriority` | number | HTTPS, SVCB | 0–65535 |
| `targetName` | string | HTTPS, SVCB | Target hostname |
| `svcParams` | string | HTTPS, SVCB | SvcParams string |
| `exchange` | string | MX | Mail server |
| `preference` | number | MX | Priority 0–65535 |
| `nameserver` | string | NS | Nameserver hostname |
| `pointer` | string | PTR | Pointer target |
| `service` | string | SRV | e.g. "_sip" |
| `protocol` | string | SRV, TLSA | e.g. "_tcp" |
| `priority` | number | SRV | 0–65535 |
| `weight` | number | SRV | 0–65535 |
| `port` | number/string | SRV / HTTPS,SVCB,TLSA | Port number or string |
| `target` | string | SRV | Target hostname |
| `scheme` | string | HTTPS, SVCB, TLSA | e.g. "_https" |
| `usage` | number | TLSA | 0–255 |
| `selector` | number | TLSA | 0–255 |
| `matching` | number | TLSA | 0–255 |
| `associationData` | string | TLSA | Cert data (hex) |

### `delete_dns_records`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | yes | Domain name |
| `records` | array | yes | `[{ name: string, type: string }]` — record identifiers |

---

## DNS Record Creators

All creators share: `domain` (required), `name` (required, "@" for root), `ttl` (optional, default 3600). Each replaces ALL existing records with same name+type.

### `create_a_record`
| Extra Params | Type | Required | Description |
|-------------|------|----------|-------------|
| `address` | string | yes | IPv4 address |

### `create_aaaa_record`
| Extra Params | Type | Required | Description |
|-------------|------|----------|-------------|
| `address` | string | yes | IPv6 address |

### `create_cname_record`
| Extra Params | Type | Required | Description |
|-------------|------|----------|-------------|
| `cname` | string | yes | Canonical name target |

### `create_mx_record`
| Extra Params | Type | Required | Description |
|-------------|------|----------|-------------|
| `priority` | number | yes | 0–65535 (lower = higher priority) |
| `exchange` | string | yes | Mail server hostname |

### `create_txt_record`
| Extra Params | Type | Required | Description |
|-------------|------|----------|-------------|
| `value` | string | yes | TXT value (SPF, DKIM, verification, etc.) |

### `create_srv_record`
| Extra Params | Type | Required | Description |
|-------------|------|----------|-------------|
| `priority` | number | yes | 0–65535 |
| `weight` | number | yes | 0–65535 |
| `port` | number | yes | 1–65535 |
| `target` | string | yes | Target hostname |

### `create_ns_record`
| Extra Params | Type | Required | Description |
|-------------|------|----------|-------------|
| `nameserver` | string | yes | Nameserver hostname |

### `create_alias_record`
| Extra Params | Type | Required | Description |
|-------------|------|----------|-------------|
| `aliasName` | string | yes | Alias target (CNAME at zone apex) |

### `create_caa_record`
| Extra Params | Type | Required | Description |
|-------------|------|----------|-------------|
| `flag` | number | yes | 0 (non-critical) or 128 (critical) |
| `tag` | string | yes | "issue", "issuewild", or "iodef" |
| `value` | string | yes | CA domain or reporting URL |

### `create_https_record`
| Extra Params | Type | Required | Description |
|-------------|------|----------|-------------|
| `svcPriority` | number | yes | 0 = alias, 1+ = service mode |
| `targetName` | string | yes | Target ("." for same name) |
| `svcParams` | string | no | e.g. "alpn=h2,h3" |
| `port` | string | no | e.g. "_443" |
| `scheme` | string | no | e.g. "_https" |

### `create_ptr_record`
| Extra Params | Type | Required | Description |
|-------------|------|----------|-------------|
| `pointer` | string | yes | Pointer target hostname |

### `create_svcb_record`
| Extra Params | Type | Required | Description |
|-------------|------|----------|-------------|
| `svcPriority` | number | yes | 0 = alias, 1+ = service mode |
| `targetName` | string | yes | Target hostname |
| `svcParams` | string | no | SvcParams string |
| `port` | string | no | Port string |
| `scheme` | string | no | Scheme label |

### `create_tlsa_record`
| Extra Params | Type | Required | Description |
|-------------|------|----------|-------------|
| `port` | string | yes | e.g. "_443" |
| `protocol` | string | yes | e.g. "_tcp" |
| `usage` | number | yes | 0=CA, 1=EE-PKIX, 2=TA, 3=EE |
| `selector` | number | yes | 0=full cert, 1=public key |
| `matching` | number | yes | 0=exact, 1=SHA-256, 2=SHA-512 |
| `associationData` | string | yes | Hex certificate data |
| `scheme` | string | no | Scheme label |

---

## Contacts & Privacy

### `save_contact`
Create/update a reusable contact profile.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `contactId` | string | no | Omit when creating new |
| `firstName` | string | yes | First name |
| `lastName` | string | yes | Last name |
| `organization` | string | no | Company name |
| `email` | string | yes | Email address |
| `address1` | string | yes | Street address |
| `address2` | string | no | Apt, suite, etc. |
| `city` | string | yes | City |
| `country` | string | yes | ISO 3166-1 alpha-2 (e.g. "US", "NL") |
| `stateProvince` | string | no | State/province |
| `postalCode` | string | no | ZIP/postal code |
| `phone` | string | yes | E.164 format (e.g. "+1.5551234567") |
| `phoneExt` | string | no | Extension |
| `fax` | string | no | Fax E.164 |
| `faxExt` | string | no | Fax extension |
| `taxNumber` | string | no | Tax ID |

Returns: `{ contactId }` only.

### `get_contact`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `contactId` | string | yes | Contact ID |

### `save_contact_attributes`
TLD-specific attributes (e.g. `.us` requires `appPurpose`, `nexusCategory`).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `attributes` | object | yes | Flat key-value pairs (e.g. `{ type: "us", appPurpose: "P1", nexusCategory: "C11" }`) |

### `get_contact_attributes`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `contactId` | string | yes | Contact ID |

### `update_domain_contacts`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | yes | Domain name |
| `registrant` | string | no | Registrant contact ID |
| `admin` | string | no | Admin contact ID |
| `tech` | string | no | Tech contact ID |
| `billing` | string | no | Billing contact ID |
| `attributes` | string[] | no | Contact attribute IDs |

### `set_privacy_level`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | yes | Domain name |
| `level` | enum | yes | `"high"` (WHOIS hidden) or `"public"` (visible) |
| `userConsent` | boolean | yes | **Must be true** — consent is mandatory |

---

## Personal Nameservers

Vanity/glue record nameservers (e.g. ns1.yourdomain.com).

### `list_personal_nameservers`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | yes | Parent domain |

### `get_personal_nameserver`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | yes | Parent domain |
| `host` | string | yes | NS hostname (e.g. "ns1.example.com") |

### `update_personal_nameserver`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | yes | Parent domain |
| `host` | string | yes | NS hostname |
| `ips` | string[] | yes | IP addresses (min 1) |

### `delete_personal_nameserver`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | yes | Parent domain |
| `host` | string | yes | NS hostname to delete |

---

## SellerHub Marketplace

### `list_sellerhub_domains`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `fetchAll` | boolean | no | true | Auto-paginate |
| `take` | number | no | 100 | Items per page (1–100) |
| `skip` | number | no | 0 | Offset |

### `create_sellerhub_domain`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | yes | Domain to list for sale |
| `displayName` | string | no | Display name |
| `description` | string | no | Listing description |
| `binPriceEnabled` | boolean | no | Enable buy-it-now price |
| `binPrice` | object | no | `{ amount: string, currency: string }` |
| `minPriceEnabled` | boolean | no | Enable minimum offer price |
| `minPrice` | object | no | `{ amount: string, currency: string }` |

### `get_sellerhub_domain`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | yes | Domain name |

### `update_sellerhub_domain`
Same parameters as `create_sellerhub_domain`.

### `delete_sellerhub_domain`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | yes | Domain to remove from SellerHub |

### `create_checkout_link`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | string | yes | Domain (must be listed on SellerHub) |
| `type` | enum | yes | `"buyNow"` (only supported type) |
| `basePrice` | object | no | Optional price override `{ amount: string, currency: string }` |

### `get_verification_records`
Account-level, no parameters. Returns `{ options: [{ records: [...] }] }`.

---

## Analysis

### `check_dns_alignment`
Compare expected records against actual Spaceship DNS.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `domain` | string | yes | — | Domain name |
| `expectedRecords` | array | yes | — | Expected DNS records (discriminated union by `type`) |
| `includeTtlInMatch` | boolean | no | false | Include TTL in comparison |
| `includeUnexpectedOfTypes` | string[] | no | A, AAAA, CNAME, MX, TXT, SRV | Record types to flag as unexpected |

**Expected record format** — each record is `{ type, name, ...typeSpecificFields }`. Fields match the DNS record creators above.
