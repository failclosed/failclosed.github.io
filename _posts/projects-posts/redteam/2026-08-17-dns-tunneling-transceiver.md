---
layout: post
title: DNS Tunneling Transceiver
gh-repo: 4D5A/Red-Team-Projects
gh-badge: [follow, star]
categories: [redteam, projects]
tags: [Python, DNS, Network-Administration-Tools, infosec, Detection Engineering]
---

### Introduction

DNS is the channel that is allowed to leave when nothing else is. A host with no route to the internet, sitting behind a proxy that inspects everything, will still happily resolve names, because something has to or the network stops working. That is why DNS tunneling has outlived every generation of egress control that was meant to kill it.

The DNS Tunneling Transceiver is a Python tool that moves a file out of a network one DNS label at a time. It chunks the file, encodes each chunk into a subdomain, and sends one query per chunk to a zone it controls. A receiver acting as the authoritative side of that zone logs each query, decodes the label, and reassembles the file. It is a poor way to move data and a good way to watch data move.

I am filing it under Red Team because the underlying technique, riding stolen data out inside DNS query names, is the single most common real-world exfiltration path and the one every network defender should be able to recognize in their own resolver logs.

### How it works

Every transfer is framed by handshake queries under an attacker-controlled zone:

```
<seq>.<base32-chunk>.exfil.lab.example   ->   A / TXT
```

1. A `SOT` query announces the filename and total size.
2. Numbered data queries carry the body, one chunk per query.
3. An `EOT` query closes the transfer, carrying a CRC32 of the reassembled file so both ends agree they saw the same bytes.

The receiver ignores everything until it sees a start-of-transmission label and discards anything outside that window, so unrelated lookups for the same zone will not corrupt a session. Data comes back on a return channel when you want one: the receiver answers `TXT` queries with base32-encoded response data, which the transmitter reads out of the answer section.

Four transport profiles are available, and they change what actually goes on the wire rather than only how the log renders:

| Profile | Wire form | Relative volume |
|---|---|---|
| A-record, metronome | one base32 chunk per `A` query, fixed interval | baseline |
| TXT request/response | larger return capacity on a rare record type | baseline out, higher back |
| Base16 labels | hex encoding instead of base32 | about 2x |
| Jittered timing | the only profile that tries to blend in at all | baseline |

At roughly 30 usable bytes per label after encoding and overhead, a one-megabyte file is on the order of thirty-five thousand queries to a single zone. That number is the point, not a flaw.

### What this project is not

This is not a covert channel, and I would rather say that plainly than let the Red Team label imply otherwise.

A tool built to evade detection would pack far more data into each query, rotate across many zones, hide its labels inside otherwise-normal-looking names, and pace itself against the host's real DNS behaviour. This project does the opposite of all four. It uses a fixed subdomain prefix, a hardcoded handshake, one small chunk per query, and a default metronome that a baseline detection will catch on the first run. Every one of those choices was made for visibility.

That makes it useful for something more practical than evasion. It is a generator for the traffic your detections should catch, so you can find out whether they do.

### Detection

Two levels, and the gap between them is the reason this project exists.

#### Content matching

The default prefix and handshake tokens are fixed strings sitting in the query name.

| Artifact | Value |
|---|---|
| Zone | `exfil.lab.example` |
| Start / end tokens | `sot`, `eot` |
| Return record type | `TXT` |
| Default query interval | 0.3 s |

```
alert dns any any -> any any (msg:"DNS Tunneling Transceiver handshake label"; \
  dns.query; content:"exfil"; nocase; pcre:"/\b(sot|eot)\b/i"; \
  classtype:policy-violation; sid:1000020; rev:1;)
```

That rule works, and it is close to worthless as a general detection, because every one of those values is configurable. Point the tool at a different zone with different tokens and the signature only catches the person who did not change them.

#### Behavioral indicators

What survives a rename is the shape of the traffic, and the shape is forced by the technique. To move a byte out over DNS you have to ask a question the resolver has never been asked before, because a cache exists to answer repeated questions and exfil is the act of never repeating one. Every chunk is therefore a fresh cache miss and a recursive lookup that your own resolver writes into its query log on the way out.

- **Unique-label cardinality per registered domain.** A benign zone is answered from cache after the first few lookups. An exfil zone shows a continuously climbing count of never-before-seen subdomains. Jitter, re-encoding, and record-type swaps do nothing to this.
- **Query volume and rate to one second-level domain**, especially one with little history or reputation. Thirty-five thousand lookups to a single zone is not a browsing pattern.
- **Label length and character distribution.** Base32 and base16 labels sit near the maximum length with near-uniform character frequency and high entropy. Human and CDN names do not.
- **NXDOMAIN ratio per zone**, and unusual `TXT` or `NULL` answer volume on the return path.

The durable signal is not a packet signature at all. It is a query on telemetry you already have, from Pi-hole's FTL database, Unbound with logging, or Zeek's `dns.log`:

```
-- per registered domain over a rolling window
SELECT   registered_domain,
         COUNT(*)                  AS queries,
         COUNT(DISTINCT subdomain) AS unique_labels,
         AVG(LENGTH(subdomain))    AS avg_label_len
FROM     dns_queries
WHERE    ts > now() - INTERVAL '5 minutes'
GROUP BY registered_domain
HAVING   unique_labels > 100 AND avg_label_len > 30
ORDER BY unique_labels DESC;
```

That query does not care about this tool, or about DNS tunneling by any particular name. It fires on anything that has to keep asking new questions to move data, which is every exfiltration-over-DNS technique there is. This is the same move the EICAR suite in this collection makes: the most durable detection is built on telemetry your own infrastructure is forced to produce, because there is nothing in it for the adversary to change.

If you take one thing from this project, take that distinction. Signatures pinned to a tool's constants are cheap to write and cheap to evade. Detections built on what a technique forces the resolver to record are harder to write and much harder to get around.

### Notes if you run it

Run the whole thing inside an isolated lab VLAN, pointed at a zone you actually control, with the receiver as its authoritative side. Do not aim it at a domain you do not own, and do not aim it at your production resolver unless you have told whoever watches it first.

The interesting exercise is not watching the file reassemble. It is opening your resolver's query log afterwards and checking whether the per-zone cardinality and volume would have flagged on their own, without you knowing the prefix in advance. Then turn on the jittered profile, switch the labels to base16, and confirm the content signature dies while the cardinality query does not. That contrast is the whole lesson.

### Repository

The project is catalogued in the [Red Team Projects](https://github.com/4D5A/Red-Team-Projects) repository along with the detection guidance above. The entry there is documentation only. The behavioral query is the portion worth taking, because it applies to anything that moves data out through DNS one label at a time.

Released for lawful and authorized use only. Use it against zones and systems you own or are explicitly authorized in writing to test. Unauthorized exfiltration or interception of traffic from systems you do not own may be a criminal offence, and determining whether a given use is lawful is the responsibility of the user.