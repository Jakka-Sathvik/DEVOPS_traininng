# Day 15 – DNS Theory

Notes on how the Domain Name System works, and how AWS Route 53 implements it as a managed service.

---

## 1. What is DNS?

**DNS (Domain Name System)** is the internet's naming system. It translates human-readable **domain names** (like `www.amazon.com`) into machine-readable **IP addresses** (like `205.251.242.103`) that computers actually use to find and talk to each other over a network.

It's often called the **"phone book of the internet"** — you look up a name to get the number you actually need in order to make the connection.

**Key characteristics**

- **Hierarchical** – organized like a tree: Root → Top-Level Domain (TLD) → Domain → Subdomain
- **Distributed** – no single server stores all DNS data; millions of servers worldwide share the load
- **Cached** – answers are temporarily stored (per a record's TTL) at multiple layers so repeat lookups are fast

**How a lookup works, step by step**

```
User types "www.example.com" in the browser
        │
        ▼
Browser / OS cache ── already cached? ──▶ Yes → return IP, done
        │ No
        ▼
DNS Resolver (ISP, or a public resolver like 8.8.8.8 / 1.1.1.1)
        │
        ▼
Root Name Server ──▶ "Ask a .com TLD server"
        │
        ▼
TLD Name Server (.com) ──▶ "Ask example.com's name servers"
        │
        ▼
Authoritative Name Server (e.g., a Route 53 hosted zone) ──▶ "It's 192.0.2.1"
        │
        ▼
Resolver caches the answer and returns the IP to the browser
        │
        ▼
Browser opens a connection to 192.0.2.1
```

This whole round trip usually takes milliseconds, mostly because of caching — most lookups never actually reach the root or TLD servers.

---

## 2. Why do websites use domain names instead of IP addresses?

- **Human memorability** – `amazon.com` is far easier to remember and type than `205.251.242.103` (or an IPv6 address).
- **Infrastructure can change underneath** – Servers get migrated, scaled, or replaced, and their IP addresses change with them. Because the domain just points to a record that can be updated, users never have to learn a new address.
- **One name, many servers** – A single domain can resolve to multiple IPs for load balancing, redundancy, and failover — something a single static number can't do on its own.
- **Branding and trust** – A domain name is part of a company's identity; an IP address communicates nothing about who owns it or whether it's trustworthy.
- **Works across IPv4 and IPv6** – The domain abstracts away which protocol version is actually in use underneath (`A` vs `AAAA` records).
- **Enables virtual hosting & HTTPS** – Many websites can share one physical IP address, and SSL/TLS certificates are issued to domain names — which is what makes secure, named HTTPS connections possible in the first place.

---

## 3. What is Amazon Route 53?

**Amazon Route 53** is AWS's highly available, scalable **DNS and domain registration service**. The name is a nod to **port 53**, the standard port used for DNS traffic.

It bundles together three core capabilities:

| Capability | What it does |
|---|---|
| **Domain registration** | Buy, transfer, and manage domain names directly through AWS |
| **DNS management** | Host DNS records for a domain and answer queries via **Hosted Zones** |
| **Traffic routing & health checks** | Route traffic using various policies, and automatically route away from unhealthy endpoints |

**Routing policies available:**
- **Simple** – one domain → one resource (the default, no health checks)
- **Weighted** – split traffic across resources by assigned proportions (useful for A/B testing, canary/blue-green deploys)
- **Latency-based** – send users to the AWS region that gives them the best latency
- **Failover** – active-passive setup; automatically fail over to a standby resource
- **Geolocation** – route based on the user's geographic location
- **Geoproximity** – route based on the location of your resources, with the ability to shift traffic using a "bias"
- **Multi-value answer** – return up to 8 healthy records at random, for lightweight client-side load balancing
- **IP-based** – route based on the client's IP address ranges

**A few things that make Route 53 stand out:**
- A **100% availability SLA** — unusually strong compared to most cloud services.
- **Alias records**, a Route 53–specific record type that can point directly at AWS resources (S3 static websites, CloudFront, ELBs, API Gateway, etc.) at no extra charge, and — unlike a `CNAME` — can be used even at the zone apex (`example.com`, not just `www.example.com`).
- A global **anycast network** of DNS servers, so queries get answered from a location near the user.
- Newer additions such as **Route 53 Resolver** (hybrid DNS between on-prem networks and VPCs) and **Route 53 DNS Firewall** (blocking malicious domains at the resolver level).
- Simple pay-as-you-go pricing: a small monthly charge per hosted zone plus a cost per million queries, with no upfront commitment.

---

## 4. What is a Hosted Zone?

A **Hosted Zone** is a container inside Route 53 that holds all the DNS records for one domain (and its subdomains) — essentially, "the DNS settings for `example.com`."

When you create a hosted zone, Route 53 automatically creates two records for you:
- An **NS (Name Server) record** – lists the 4 authoritative name servers assigned to that zone. You point your domain's registrar at these.
- An **SOA (Start of Authority) record** – administrative metadata about the zone (primary name server, admin contact, refresh/retry timers, etc.)

From there, you add whatever records you need inside the zone — `www.example.com`, `api.example.com`, `mail.example.com`, and so on.

There are two types of hosted zone — **Public** and **Private** — covered in Question 7.

---

## 5. What is a DNS Record?

A **DNS record** (or "resource record") is a single entry inside a hosted zone that tells DNS how to answer a query for a name — essentially a mapping of *name → value*, plus a **record type** and a **TTL** (how long the answer may be cached).

**Common record types:**

| Type | Purpose | Example |
|---|---|---|
| `A` | Maps a name to an **IPv4** address | `example.com → 192.0.2.1` |
| `AAAA` | Maps a name to an **IPv6** address | `example.com → 2606:2800:220:1::1` |
| `CNAME` | Aliases one name to another domain name (not allowed at the zone apex) | `www.example.com → example.com` |
| `ALIAS` | *(Route 53–specific)* Aliases a name directly to an AWS resource, free of charge, allowed at the zone apex | `example.com → d123.cloudfront.net` |
| `MX` | Directs email to mail servers, with a priority | `example.com → 10 mail.example.com` |
| `TXT` | Arbitrary text — commonly used for domain verification, SPF/DKIM email authentication | `"v=spf1 include:_spf.google.com ~all"` |
| `NS` | Delegates a (sub)domain to a set of name servers | — |
| `SOA` | Administrative info about the zone | — |
| `PTR` | Reverse lookup — maps an IP address back to a domain name | — |
| `SRV` | Points to the host/port of a specific service | — |
| `CAA` | Restricts which Certificate Authorities may issue SSL/TLS certs for the domain | — |

---

## 6. What is a Domain Registrar?

A **Domain Registrar** is a company accredited by **ICANN** (the Internet Corporation for Assigned Names and Numbers) to sell and manage domain name registrations on behalf of customers.

**What a registrar actually does:**
- Lets you search for and purchase an available domain name
- Reserves it in the central **registry** for that TLD (e.g., Verisign runs the registry for `.com`/`.net`) for a term of 1–10 years, renewable
- Stores your WHOIS contact details
- Lets you point the domain at a set of name servers — i.e., choose who actually hosts its DNS

**Registrar vs. DNS host — an important distinction**

These are two separate roles, and they don't have to be the same company:
- The **registrar** is who you legally register the domain name through.
- The **DNS host** is whoever actually answers DNS queries for it (i.e., holds the hosted zone).

For example, you could register `example.com` with GoDaddy but update its name servers to point at **Route 53**, so AWS hosts and answers all the actual DNS queries while GoDaddy remains the registrar of record.

**Common registrars today:** Amazon Route 53, GoDaddy, Namecheap, Cloudflare Registrar, Squarespace Domains (Google Domains was sold to Squarespace in 2023 and is no longer a separate registrar), Porkbun, Dynadot, IONOS, Hostinger.

---

## 7. Difference between Public and Private Hosted Zones

| | **Public Hosted Zone** | **Private Hosted Zone** |
|---|---|---|
| Resolvable from | Anywhere on the public internet | Only inside the VPC(s) it's associated with |
| Typical use case | Public websites, public APIs | Internal microservices, internal DBs, admin tools |
| Requires a VPC | No | Yes — must be associated with one or more VPCs |
| Example domain | `example.com` | `db.internal.example.com` |
| Visible outside AWS | Yes | No — invisible to the public internet, even if someone guesses the name |

In short: a **public** hosted zone controls how the world reaches your domain; a **private** hosted zone controls how resources *inside your own VPC(s)* resolve names, and can even be associated with multiple VPCs — including across AWS accounts — to build shared internal DNS for a whole organization.

---

## 8. What is DNS propagation?

**DNS propagation** is the delay between when you change a DNS record and when that change is visible everywhere, worldwide.

**Why it isn't instant**

The change takes effect immediately at your authoritative name server (e.g., the moment you update a record in Route 53). But DNS is a heavily *cached* system — ISPs, public resolvers, operating systems, and browsers around the world may still be holding onto the old answer.

How long they hold onto it is controlled by the record's **TTL (Time To Live)**, measured in seconds:
- **Low TTL** (e.g., 60s) → faster propagation, but more repeat queries hitting your DNS servers
- **High TTL** (e.g., 24h) → better caching/performance, but changes take longer to be seen everywhere

**Typical propagation time:** usually well under an hour, though it's commonly quoted as anywhere from a few minutes up to **24–48 hours** in the worst case, since some resolvers cache more aggressively than they should.

**Best practice:** If you're planning a change (e.g., a server migration), **lower the TTL a day or so in advance**. Once old cached copies expire under the short TTL, your real change will propagate much faster when you actually make it.

---

## Key Takeaways

- DNS turns memorable domain names into the IP addresses computers actually need — it's the backbone that makes the internet usable.
- **Route 53** is AWS's managed DNS service *and* domain registrar, distinguished by its 100% SLA and free zone-apex `ALIAS` records.
- A **hosted zone** is the container for a domain's records; it's **public** for internet-facing resources or **private** for VPC-only resources.
- **DNS records** (A, CNAME, MX, TXT, etc.) are the individual instructions inside a hosted zone that answer specific kinds of queries.
- **Registrar** (who you bought the domain from) and **DNS host** (who actually answers queries for it) are separate roles, and can be different companies.
- **Propagation delay** exists because of caching at every layer of DNS, governed by each record's TTL.
