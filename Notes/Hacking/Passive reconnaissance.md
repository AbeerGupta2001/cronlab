### WHOIS

WHOIS is a query/response protocol defined in [RFC 3912 (opens in new tab)](https://www.ietf.org/rfc/rfc3912.txt). WHOIS servers listen on **TCP port 43** and provide registration details for domain names. The **domain registrar** maintains these records for the domains they lease.

From a WHOIS response, the following details may be available (when not redacted):

- **Registrar**: The company (e.g., Namecheap, GoDaddy) that registered the domain.
- **Registrant contact information**: Name, organisation, address, phone, and email. However, **privacy services** (standard since GDPR 2018) usually replace this with "Withheld for Privacy" or similar.
- **Dates**: Creation (registration), Updated (last change), and Expiration (renewal deadline).
- **Name servers**: The DNS servers authoritative for the domain.
- **Status codes**: For example, `clientTransferProhibited` indicates the domain is locked against unauthorised transfers.
- **Abuse contacts**: The registrar's email and phone for reporting issues.

### Why prefer dig over nslookup?

This task introduces two DNS query tools: `nslookup` and `dig`. Both query DNS, but `dig` (historically a backronym for "Domain Information Groper") is the modern, preferred option. It provides cleaner output, displays TTL values by default (showing how long records are cached), and is more reliable for complex queries and scripting. `nslookup` is covered here for compatibility, since you will encounter it in older documentation and on Windows systems, but `dig` should be your default tool.

### nslookup

**nslookup** (Name Server Lookup) is the older of the two tools.

Syntax:

- `nslookup DOMAIN_NAME` performs a simple lookup using your default resolver.
- `nslookup -type=TYPE DOMAIN_NAME [SERVER]` specifies a record type and an optional DNS server.

Common DNS record types:

|Query type|Result|
|---|---|
|A|IPv4 address(es) for the domain|
|AAAA|IPv6 address(es) for the domain|
|CNAME|Canonical Name: an alias that points one domain name to another|
|MX|Mail Servers: the servers responsible for handling email for the domain|
|SOA|Start of Authority: the primary name server, admin email, and zone serial number|
|TXT|Text Records: arbitrary text, commonly used for SPF, DKIM, DMARC, and domain verification|

Example (IPv4 addresses via Cloudflare's resolver):

Terminal

`user@TryHackMe$ nslookup -type=A tryhackme.com 1.1.1.1`

These IPs are often anycast (Cloudflare in this case). For penetration testing, each IP may host different services, so check whether they fall within scope.

**MX example** (mail servers):

Terminal

`user@TryHackMe$ nslookup -type=MX tryhackme.com`

The number before each server indicates priority: lower values mean higher priority. In this case, Google Workspace handles email, which is common for many organisations and typically well-patched.

### dig

`dig` is the modern, preferred DNS query tool.

Syntax: `dig [@SERVER] DOMAIN_NAME [TYPE]`

Example (MX records via Cloudflare):

Terminal

`user@TryHackMe$ dig @1.1.1.1 tryhackme.com MX`


**Command quick-reference**:

|Purpose|Command-line Example|
|---|---|
|Lookup WHOIS record|`whois tryhackme.com`|
|Lookup DNS A records (legacy)|`nslookup -type=A tryhackme.com`|
|Lookup DNS MX records at specific server (legacy)|`nslookup -type=MX tryhackme.com 1.1.1.1`|
|Lookup DNS TXT records (legacy)|`nslookup -type=TXT tryhackme.com`|
|Lookup DNS A records (recommended)|`dig tryhackme.com A`|
|Lookup DNS MX records at specific server (recommended)|`dig @1.1.1.1 tryhackme.com MX`|
|Lookup DNS TXT records (recommended)|`dig tryhackme.com TXT`|
|Passive subdomain discovery (browser-based)|Visit https://crt.sh and search `%.tryhackme.com`|
