# Spaceship MCP — Common DNS Patterns & Recipes

Step-by-step tool calls for common domain and DNS configurations.

---

## Table of Contents

- [Website Hosting Setup](#website-hosting-setup)
- [Google Workspace Email](#google-workspace-email)
- [Microsoft 365 Email](#microsoft-365-email)
- [Vercel Deployment](#vercel-deployment)
- [Netlify Deployment](#netlify-deployment)
- [Cloudflare Proxy](#cloudflare-proxy)
- [SSL/TLS with CAA Records](#ssltls-with-caa-records)
- [Domain Verification Patterns](#domain-verification-patterns)
- [SPF + DMARC Email Security](#spf--dmarc-email-security)

---

## Website Hosting Setup

Basic website hosting with an A record for the root domain and a CNAME for `www`.

### Step 1: Set the root A record

```
create_a_record({
  domain: "example.com",
  name: "@",
  address: "93.184.216.34",
  ttl: 3600
})
```

### Step 2: Point www to the root domain

```
create_cname_record({
  domain: "example.com",
  name: "www",
  cname: "example.com",
  ttl: 3600
})
```

### Step 3: Verify the setup

```
check_dns_alignment({
  domain: "example.com",
  expectedRecords: [
    { type: "A", name: "@", address: "93.184.216.34" },
    { type: "CNAME", name: "www", cname: "example.com" }
  ]
})
```

### With IPv6 support

Add an AAAA record alongside the A record:

```
create_aaaa_record({
  domain: "example.com",
  name: "@",
  address: "2606:2800:220:1:248:1893:25c8:1946",
  ttl: 3600
})
```

---

## Google Workspace Email

Full Google Workspace setup with MX, SPF, DKIM, and DMARC.

### Step 1: Add Google MX records

Use `save_dns_records` to set all MX records in a single call (avoids the replace behavior of `create_mx_record`):

```
save_dns_records({
  domain: "example.com",
  records: [
    { name: "@", type: "MX", preference: 1, exchange: "aspmx.l.google.com", ttl: 3600 },
    { name: "@", type: "MX", preference: 5, exchange: "alt1.aspmx.l.google.com", ttl: 3600 },
    { name: "@", type: "MX", preference: 5, exchange: "alt2.aspmx.l.google.com", ttl: 3600 },
    { name: "@", type: "MX", preference: 10, exchange: "alt3.aspmx.l.google.com", ttl: 3600 },
    { name: "@", type: "MX", preference: 10, exchange: "alt4.aspmx.l.google.com", ttl: 3600 }
  ]
})
```

### Step 2: Add SPF record

```
create_txt_record({
  domain: "example.com",
  name: "@",
  value: "v=spf1 include:_spf.google.com ~all"
})
```

### Step 3: Add DKIM record

Get the DKIM value from Google Admin Console > Apps > Google Workspace > Gmail > Authenticate email.

```
create_txt_record({
  domain: "example.com",
  name: "google._domainkey",
  value: "v=DKIM1; k=rsa; p=MIIBIjANBgkqhki..."
})
```

### Step 4: Add DMARC record

```
create_txt_record({
  domain: "example.com",
  name: "_dmarc",
  value: "v=DMARC1; p=quarantine; rua=mailto:dmarc-reports@example.com; pct=100"
})
```

### Step 5: Verify the full setup

```
check_dns_alignment({
  domain: "example.com",
  expectedRecords: [
    { type: "MX", name: "@", preference: 1, exchange: "aspmx.l.google.com" },
    { type: "MX", name: "@", preference: 5, exchange: "alt1.aspmx.l.google.com" },
    { type: "MX", name: "@", preference: 5, exchange: "alt2.aspmx.l.google.com" },
    { type: "MX", name: "@", preference: 10, exchange: "alt3.aspmx.l.google.com" },
    { type: "MX", name: "@", preference: 10, exchange: "alt4.aspmx.l.google.com" },
    { type: "TXT", name: "@", value: "v=spf1 include:_spf.google.com ~all" },
    { type: "TXT", name: "google._domainkey", value: "v=DKIM1; k=rsa; p=MIIBIjANBgkqhki..." },
    { type: "TXT", name: "_dmarc", value: "v=DMARC1; p=quarantine; rua=mailto:dmarc-reports@example.com; pct=100" }
  ]
})
```

---

## Microsoft 365 Email

Full Microsoft 365 / Exchange Online setup with MX, SPF, autodiscover, and DMARC.

### Step 1: Add Microsoft MX record

Replace `<tenant>` with your Microsoft 365 tenant name (e.g., `contoso-com` for contoso.com — dots become dashes).

```
create_mx_record({
  domain: "example.com",
  name: "@",
  priority: 0,
  exchange: "example-com.mail.protection.outlook.com"
})
```

### Step 2: Add SPF record

```
create_txt_record({
  domain: "example.com",
  name: "@",
  value: "v=spf1 include:spf.protection.outlook.com -all"
})
```

### Step 3: Add autodiscover CNAME

Required for Outlook client auto-configuration:

```
create_cname_record({
  domain: "example.com",
  name: "autodiscover",
  cname: "autodiscover.outlook.com"
})
```

### Step 4: Add DKIM CNAMEs

Microsoft uses CNAME-based DKIM. The selector values are found in the Microsoft 365 admin center under Settings > Domains.

```
save_dns_records({
  domain: "example.com",
  records: [
    { name: "selector1._domainkey", type: "CNAME", cname: "selector1-example-com._domainkey.example.onmicrosoft.com", ttl: 3600 },
    { name: "selector2._domainkey", type: "CNAME", cname: "selector2-example-com._domainkey.example.onmicrosoft.com", ttl: 3600 }
  ]
})
```

### Step 5: Add DMARC record

```
create_txt_record({
  domain: "example.com",
  name: "_dmarc",
  value: "v=DMARC1; p=quarantine; rua=mailto:dmarc-reports@example.com; pct=100"
})
```

### Step 6: Add SIP and Teams SRV records (optional)

For Skype for Business / Teams federation:

```
save_dns_records({
  domain: "example.com",
  records: [
    { name: "_sip._tls", type: "SRV", priority: 100, weight: 1, port: 443, target: "sipdir.online.lync.com", service: "_sip", protocol: "_tls", ttl: 3600 },
    { name: "_sipfederationtls._tcp", type: "SRV", priority: 100, weight: 1, port: 5061, target: "sipfed.online.lync.com", service: "_sipfederationtls", protocol: "_tcp", ttl: 3600 }
  ]
})
```

### Step 7: Verify

```
check_dns_alignment({
  domain: "example.com",
  expectedRecords: [
    { type: "MX", name: "@", preference: 0, exchange: "example-com.mail.protection.outlook.com" },
    { type: "TXT", name: "@", value: "v=spf1 include:spf.protection.outlook.com -all" },
    { type: "CNAME", name: "autodiscover", cname: "autodiscover.outlook.com" },
    { type: "TXT", name: "_dmarc", value: "v=DMARC1; p=quarantine; rua=mailto:dmarc-reports@example.com; pct=100" }
  ]
})
```

---

## Vercel Deployment

Point a domain to Vercel for hosting a Next.js or other Vercel-deployed app.

### Option A: Root domain + www (recommended)

```
# Step 1: Point root to Vercel's IP
create_a_record({
  domain: "example.com",
  name: "@",
  address: "76.76.21.21",
  ttl: 3600
})

# Step 2: Point www to Vercel DNS
create_cname_record({
  domain: "example.com",
  name: "www",
  cname: "cname.vercel-dns.com",
  ttl: 3600
})
```

### Option B: Subdomain only

```
create_cname_record({
  domain: "example.com",
  name: "app",
  cname: "cname.vercel-dns.com",
  ttl: 3600
})
```

### Verify

```
check_dns_alignment({
  domain: "example.com",
  expectedRecords: [
    { type: "A", name: "@", address: "76.76.21.21" },
    { type: "CNAME", name: "www", cname: "cname.vercel-dns.com" }
  ]
})
```

After setting DNS, add the domain in Vercel's project settings under Domains. Vercel handles SSL automatically.

---

## Netlify Deployment

Point a domain to Netlify for hosting.

### Option A: Root domain with ALIAS + www CNAME

For root domains, Netlify recommends an ALIAS record (if supported) or their load balancer IP:

```
# Step 1: Root domain — use ALIAS record
create_alias_record({
  domain: "example.com",
  name: "@",
  aliasName: "apex-loadbalancer.netlify.com",
  ttl: 3600
})

# Step 2: www subdomain
create_cname_record({
  domain: "example.com",
  name: "www",
  cname: "example-site.netlify.app",
  ttl: 3600
})
```

### Option A (alternative): Root domain with A record

If ALIAS is not preferred, use Netlify's load balancer IP:

```
create_a_record({
  domain: "example.com",
  name: "@",
  address: "75.2.60.5",
  ttl: 3600
})
```

### Option B: Subdomain only

```
create_cname_record({
  domain: "example.com",
  name: "blog",
  cname: "example-blog.netlify.app",
  ttl: 3600
})
```

After setting DNS, add a custom domain in Netlify's site settings. Netlify provisions SSL via Let's Encrypt automatically.

---

## Cloudflare Proxy

Delegate DNS to Cloudflare by updating nameservers. Once delegated, DNS is managed in Cloudflare's dashboard.

### Step 1: Update nameservers to Cloudflare

Cloudflare assigns specific nameservers when you add a site. Use the assigned values:

```
update_nameservers({
  domain: "example.com",
  provider: "custom",
  nameservers: [
    "ivy.ns.cloudflare.com",
    "leon.ns.cloudflare.com"
  ]
})
```

### Step 2: Verify nameserver change

```
get_domain({ domain: "example.com" })
# Check that nameservers show the Cloudflare values
```

After delegation, all DNS management happens through Cloudflare. Spaceship DNS tools will no longer affect the domain's resolution until nameservers are switched back.

### Switching back to Spaceship DNS

```
update_nameservers({
  domain: "example.com",
  provider: "basic"
})
```

---

## SSL/TLS with CAA Records

CAA (Certificate Authority Authorization) records control which CAs can issue certificates for your domain.

### Let's Encrypt only

Restrict certificate issuance to Let's Encrypt:

```
save_dns_records({
  domain: "example.com",
  records: [
    { name: "@", type: "CAA", flag: 0, tag: "issue", value: "letsencrypt.org", ttl: 3600 },
    { name: "@", type: "CAA", flag: 0, tag: "issuewild", value: "letsencrypt.org", ttl: 3600 }
  ]
})
```

### Let's Encrypt + Sectigo

Allow both Let's Encrypt and Sectigo:

```
save_dns_records({
  domain: "example.com",
  records: [
    { name: "@", type: "CAA", flag: 0, tag: "issue", value: "letsencrypt.org", ttl: 3600 },
    { name: "@", type: "CAA", flag: 0, tag: "issue", value: "sectigo.com", ttl: 3600 },
    { name: "@", type: "CAA", flag: 0, tag: "issuewild", value: "letsencrypt.org", ttl: 3600 },
    { name: "@", type: "CAA", flag: 0, tag: "issuewild", value: "sectigo.com", ttl: 3600 }
  ]
})
```

### With violation reporting

Add an `iodef` record to receive reports when unauthorized CA issuance is attempted:

```
create_caa_record({
  domain: "example.com",
  name: "@",
  flag: 0,
  tag: "iodef",
  value: "mailto:security@example.com"
})
```

### Google Trust Services (for Google-managed certs)

```
save_dns_records({
  domain: "example.com",
  records: [
    { name: "@", type: "CAA", flag: 0, tag: "issue", value: "pki.goog", ttl: 3600 },
    { name: "@", type: "CAA", flag: 0, tag: "issuewild", value: "pki.goog", ttl: 3600 }
  ]
})
```

### Verify CAA records

```
check_dns_alignment({
  domain: "example.com",
  expectedRecords: [
    { type: "CAA", name: "@", flag: 0, tag: "issue", value: "letsencrypt.org" },
    { type: "CAA", name: "@", flag: 0, tag: "issuewild", value: "letsencrypt.org" }
  ]
})
```

---

## Domain Verification Patterns

Common domain verification methods for third-party services.

### Google Search Console

Google provides a TXT record value in the Search Console verification page:

```
create_txt_record({
  domain: "example.com",
  name: "@",
  value: "google-site-verification=AbCdEfGhIjKlMnOpQrStUvWxYz1234567890"
})
```

### Generic TXT domain verification

Many services (GitHub Pages, Postmark, Mailgun, etc.) ask you to add a TXT record:

```
create_txt_record({
  domain: "example.com",
  name: "@",
  value: "service-verification=abc123xyz"
})
```

### Subdomain CNAME verification

Some services verify via CNAME at a specific subdomain:

```
create_cname_record({
  domain: "example.com",
  name: "_verify",
  cname: "verify.service.com"
})
```

### Spaceship SellerHub verification

SellerHub uses account-level verification records:

```
# Step 1: Get verification records (account-level, no parameters)
get_verification_records()
→ { options: [{ records: [{ name: "_spaceship-verify", type: "TXT", value: "abc123" }] }] }

# Step 2: Create the record on your domain
create_txt_record({
  domain: "example.com",
  name: "_spaceship-verify",
  value: "abc123"
})
```

### Multiple TXT records at the root

When you need multiple TXT records at `@` (SPF + verification + DMARC), use `save_dns_records` to avoid overwriting. Fetch existing records first:

```
# Step 1: Fetch current TXT records
list_dns_records({ domain: "example.com" })
# Note all existing TXT records at "@"

# Step 2: Save all TXT records together (existing + new)
save_dns_records({
  domain: "example.com",
  records: [
    { name: "@", type: "TXT", value: "v=spf1 include:_spf.google.com ~all", ttl: 3600 },
    { name: "@", type: "TXT", value: "google-site-verification=AbCdEf...", ttl: 3600 },
    { name: "@", type: "TXT", value: "postmark-verification=abc123", ttl: 3600 }
  ]
})
```

---

## SPF + DMARC Email Security

Protect your domain against email spoofing even if you don't send email.

### Domain that doesn't send email

If the domain is not used for email, publish a reject-all SPF and strict DMARC:

```
# Step 1: SPF — reject all senders
create_txt_record({
  domain: "example.com",
  name: "@",
  value: "v=spf1 -all"
})

# Step 2: DMARC — reject all failures
create_txt_record({
  domain: "example.com",
  name: "_dmarc",
  value: "v=DMARC1; p=reject; rua=mailto:dmarc-reports@example.com"
})

# Step 3: Empty MX — signal no mail is accepted
create_mx_record({
  domain: "example.com",
  name: "@",
  priority: 0,
  exchange: "."
})
```

### SPF for a single provider

```
# Google Workspace
create_txt_record({
  domain: "example.com",
  name: "@",
  value: "v=spf1 include:_spf.google.com ~all"
})

# Microsoft 365
create_txt_record({
  domain: "example.com",
  name: "@",
  value: "v=spf1 include:spf.protection.outlook.com -all"
})

# Amazon SES
create_txt_record({
  domain: "example.com",
  name: "@",
  value: "v=spf1 include:amazonses.com -all"
})
```

### SPF for multiple providers

Combine multiple `include` directives in a single TXT record (max 10 DNS lookups):

```
create_txt_record({
  domain: "example.com",
  name: "@",
  value: "v=spf1 include:_spf.google.com include:amazonses.com include:sendgrid.net -all"
})
```

### DMARC policies explained

```
# Monitoring only — receive reports but don't enforce
create_txt_record({
  domain: "example.com",
  name: "_dmarc",
  value: "v=DMARC1; p=none; rua=mailto:dmarc-reports@example.com"
})

# Quarantine — failed emails go to spam
create_txt_record({
  domain: "example.com",
  name: "_dmarc",
  value: "v=DMARC1; p=quarantine; rua=mailto:dmarc-reports@example.com; pct=100"
})

# Reject — failed emails are blocked entirely (strictest)
create_txt_record({
  domain: "example.com",
  name: "_dmarc",
  value: "v=DMARC1; p=reject; rua=mailto:dmarc-reports@example.com; pct=100"
})
```

### Recommended rollout order

1. Start with `p=none` to collect reports without affecting delivery
2. Review reports for a few weeks to identify legitimate senders
3. Move to `p=quarantine` with `pct=25` to test on a subset
4. Gradually increase `pct` to 100
5. Move to `p=reject` once confident all legitimate senders are covered by SPF/DKIM

### Subdomain DMARC policy

Control DMARC for subdomains separately using the `sp` tag:

```
create_txt_record({
  domain: "example.com",
  name: "_dmarc",
  value: "v=DMARC1; p=quarantine; sp=reject; rua=mailto:dmarc-reports@example.com"
})
```

This quarantines failures from the root domain but rejects failures from all subdomains.
