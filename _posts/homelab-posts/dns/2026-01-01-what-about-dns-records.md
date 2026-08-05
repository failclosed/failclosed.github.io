---
layout: post
title: What about DNS records?
categories: [homelab, dns]
tags: [dns]
after-content: [disclaimer-notice.html]
---

There are numerous types of DNS records. The most common types are:

- Start of Authority (SOA) Records
- Name Server (NS) Records
- Forward (A) Records
- Reverse (PTR) Records (i.e. Pointer Records)
- Mail Exchanger (MX) Records
- Canonical Name (CNAME) Records
- Text (TXT) Records
- Service (SRV) Records
- IPv6 Forward (AAAA) Records

For resolving domain names and fully qualified domain names (FQDNs) to IPv6 IP addresses, the additional DNS record type AAAA was created. If a domain name resolves to a server that serves content using both IPv4 and IPv6 IP addresses, it should have both A and AAAA DNS records, with the A record resolving to the IPv4 IP address and the AAAA record resolving to the IPv6 IP address.

## Start of Authority (SOA) Records

Every DNS zone has one SOA record. It names the zone's primary name server, the administrator's contact, and the timers secondary name servers use to know when to check in for updates.

## Name Server (NS) Records

NS Records determine which Name Server hosts the DNS Records (i.e. the Zone File) for the domain name.

## Forward (A) Records

A Records resolve domain names to IPv4 IP addresses.

## Reverse (PTR) Records (i.e. Pointer Records)

PTR Records resolve IP addresses to names. PTR records are still important, but had a more critical role when organizations hosted their own email servers because ISPs often blocked email sent from IP addresses that did not have a valid PTR record. Now PTR records are often used for identifying what an IP address is used for, so if someone queries a DNS server for a specific IP address's PTR record, they will receive a name which should indicate who owns the server and what it does.

## Mail Exchanger (MX) Records

MX Records tell email servers where emails for a specific domain name should be sent. Each time you send an email, the server you are using queries a DNS server for the MX record of the domain you are attempting to send your email to. The MX record tells your email server what email server is responsible for receiving emails sent to the recipient's domain.

Here is a simplified example. Bob owns the domain name example.com. Alice owns the domain name example.net.

Sender: bob@example.com
Recipient: alice@example.net

### DNS Records for example.com

example.com A 127.0.0.1
mail.example.com A 127.0.0.2
example.com MX mail.example.com
2.0.0.127.in-addr.arpa PTR mail.example.com

### DNS Records for example.net

example.net A 127.50.60.1
mail.example.net A 127.50.62.2
example.net MX mail.example.net
2.60.50.127.in-addr.arpa PTR mail.example.net

When Bob attempts to send an email to alice@example.net, his email server (mail.example.com) sends a DNS query to its DNS server for the MX Record for example.net. The DNS server responds that mail.example.net is the MX Record for example.net. Bob's email is then sent from mail.example.com to mail.example.net.

Alice's server (mail.example.net) receives the email sent from bob@example.com. The server may handle spam filtering, use RBLs (Real-Time Black Lists), [SPF (Sender Policy Framework)]({{ site.baseurl }}{% post_url blog-posts/2022-04-23-configuring-sender-policy-framework-in-exchange-online %}), [DomainKeys Identified Mail (DKIM)]({{ site.baseurl }}{% post_url blog-posts/2022-04-23-how-to-configure-dkim-in-microsoft-365 %}), [Domain-based Message Authentication, Reporting and Conformance (DMARC)]({{ site.baseurl }}{% post_url blog-posts/2022-05-23-implementing-dmarc-for-enhanced-email-security-a-guide-for-systems-administrators %}), or other filtering to determine if it will accept the email. If the server decides to accept the email and alice@example.net is a real email address on the server, then the email gets delivered. Alice's email account may have additional filters like a Junk Email folder or custom filters she added that may place the email in a specific folder, move it to the Junk Email folder, or delete it.

## Canonical Name (CNAME) Records

CNAME Records are aliases. Instead of pointing straight at an IP address, a CNAME points at another domain name, and the resolver follows it to whatever that name resolves to. Handy if `www.example.com` and `blog.example.com` should both follow the same host without maintaining separate A records for each.

## Text (TXT) Records

TXT Records hold arbitrary text tied to a domain name. These days they're mostly used for machine-readable stuff: SPF, DKIM, and DMARC records are all TXT records, and so are most domain-ownership verification tokens (Google Workspace, Microsoft 365, etc. all use them).

## Service (SRV) Records

SRV Records advertise the hostname and port a service runs on, in the form `_service._protocol.domain`. Active Directory's LDAP and Kerberos services use these, as does SIP and XMPP, so clients can find the right server without it being hardcoded, and admins can set priority/weight values for load balancing or failover.

## IPv6 Forward (AAAA) Records

AAAA Records resolve domain names to IPv6 IP addresses, the IPv6 equivalent of an A record.
