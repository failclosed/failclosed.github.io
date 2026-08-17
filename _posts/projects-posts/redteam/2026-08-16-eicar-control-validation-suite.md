---
layout: post
title: EICAR Control Validation Suite
categories: [redteam, projects]
tags: [PowerShell, Python, EICAR, IDS-IPS, TLS-Inspection, Detection Engineering, infosec]
thumbnail-img: 'assets/img/2026-08-16-eicar-control-validation-suite/console-ids-no-tls.png'
---

### Introduction

EICAR is the least impressive payload in existence. Sixty-eight printable ASCII characters, no execution, no persistence, no cleverness. Every anti-malware product on the market is expected to detect it, and detecting it proves nothing at all about how good an engine is.

That is exactly what makes it useful. EICAR does not measure whether your detection engine is smart. It measures whether it is looking. And the interesting question is never "did the scanner catch it", it is "which layers were even in a position to see it, and which quietly waved it through while reporting success."

This is a toolkit that answers that question properly: five artifact variants over two protocols, run through four network postures and against a real endpoint agent, with the network and host verdicts recorded separately so neither can hide behind the other.

I am filing it under Red Team because the finding it produces is an offensive one. In the most common enterprise posture I tested, every payload delivered over TLS reached the endpoint untouched, and the sensor logged that fact at a severity nobody alerts on.

<img src="{{ 'assets/img/2026-08-16-eicar-control-validation-suite/console-ids-no-tls.png' | relative_url }}" alt="IDS alert console showing critical BLOCK events for every cleartext request and informational NOT-INSPECTED events for every encrypted request carrying the same five files" />

### What is real here and what is not

Worth stating plainly, because screenshots of a security appliance blocking malware are exactly the sort of thing that should be treated with suspicion.

The gateway is a **simulation I wrote**. It is a Python proxy that genuinely scans response bodies, genuinely recurses into ZIP containers, genuinely blocks, and genuinely writes alerts. Every image in this post is a live render of that harness running, captured headlessly. None of it is a mockup. It is also not a commercial product, does not imitate one, and its appliance name is invented. Every page it serves is marked as a lab simulation.

The **endpoint results are real**, measured against Microsoft Defender on one Windows 10 host with real-time protection enabled and current signatures. One host is one data point, not a product review.

### How it works

Three modules, each of which runs standalone or as part of the suite.

The **egress module** pulls the four published artifacts from `secure.eicar.org` over both protocols. There is a trap here that invalidates the obvious approach: eicar.org answers plain HTTP with a 301 to HTTPS, so the payload never crosses the wire in cleartext when sourced upstream. A cleartext IDS test sourced from eicar.org is inconclusive by construction, and a pass obtained that way means nothing. The module records the upgrade explicitly rather than letting the client mask it.

The **controlled origin module** exists because of that trap. It serves byte-exact artifacts over genuine cleartext HTTP and over TLS from an origin you control, with `Serve` and `Fetch` modes so the origin and client can sit on opposite sides of the sensor. It also serves `eicar.exe`, the same 68 bytes under a PE extension, which EICAR does not publish upstream and which exercises extension and MIME policy independently of content signatures.

The **host module** characterises the endpoint agent across detection stages rather than asking a single yes or no: posture, extension handling, archive handling, extraction, on-demand scan, and AMSI.

Every verdict is recorded per layer, because collapsing them hides the finding:

| Verdict | Layer | Meaning |
|---|---|---|
| `Blocked` | Network | A device stopped or substituted the transfer, good |
| `Allowed` | Network | Delivered intact and hash verified, a gap |
| `Quarantined` | Host | Removed, read denied, or convicted per AV telemetry, good |
| `Persisted` | Host | Remained on disk and re-read byte for byte intact, a gap |

The case worth escalating is anything that is both `Allowed` and `Persisted`.

### Everything blocked, nothing blocked

Three inspection policies, five artifacts each, over both protocols. The only number that matters is how many payloads reached the endpoint.

| Inspection policy | Cleartext HTTP | TLS |
|---|---|---|
| No inspection | 5/5 delivered | 5/5 delivered |
| IDS/IPS, no TLS interception | **0/5 delivered** | **5/5 delivered** |
| IDS/IPS with TLS interception | 0/5 delivered | 0/5 delivered |

The middle row is the entire point of the project.

That sensor has a one hundred percent block rate on everything it inspected. Its dashboard is green. It also passed every single payload that arrived over TLS, which in any environment resembling the modern web is all of them. This is not a broken sensor. It is a correctly functioning sensor pointed at the wrong five percent of the traffic, and the metric it reports cannot distinguish that from success.

<img src="{{ 'assets/img/2026-08-16-eicar-control-validation-suite/cli-matrix-curl.png' | relative_url }}" alt="Full test matrix run from curl showing five blocked cleartext downloads followed by five delivered TLS downloads under the no-interception policy" />

No evasion, no obfuscation, default user agent. The same run through PowerShell's `Invoke-WebRequest` produces identical results, which is worth confirming rather than assuming, since gateways sometimes treat clients differently by user agent.

Now look at the severity column in that first console screenshot again. The blocks are `critical`. The misses are `informational`. Informational events are exactly what gets filtered out of a SOC view, rolled into a daily digest, or never alerted on at all. The sensor is telling the truth about its own blindness, in the one severity class nobody reads.

Turn interception on and the same requests resolve very differently.

<img src="{{ 'assets/img/2026-08-16-eicar-control-validation-suite/console-ids-tls.png' | relative_url }}" alt="The same alert console with TLS interception enabled, every row now a critical BLOCK event" />

The delta between those two screenshots is not detection capability. The signature was always loaded, always correct, always matching. The delta is visibility.

### The archive wrinkle

Two of the five artifacts are containers, EICAR inside a ZIP and that ZIP inside another ZIP. Container handling is where implementations genuinely diverge, so it is worth testing rather than assuming.

<img src="{{ 'assets/img/2026-08-16-eicar-control-validation-suite/block-page-nested-zip.png' | relative_url }}" alt="Interception page for a nested ZIP archive showing the match path recorded through both container layers" />

A sensor that stops at depth one passes that file. One that recurses catches it. The only way to know which you have is to send it.

The endpoint told a more interesting story, and this part is measurement rather than simulation. Defender convicted the raw 68-byte file in roughly two seconds under every extension I tried, including `.com`, `.txt`, `.exe`, `.dll` and `.js`. Detection is content based, so renaming a file buys an attacker nothing. That is a good result and worth saying plainly.

The same payload inside a ZIP sat on disk untouched past twenty seconds with no detection telemetry at all. Extraction of the single layer archive was then denied outright, and an explicit on-demand scan convicted everything. Archive members are inspected on access and on demand, not on write.

That is not a bug. Deep scanning every container on write is expensive and most products trade it away deliberately, and the payload still cannot reach execution undetected. The residual risk is dwell time and propagation rather than execution: a malicious archive can sit on a file share, replicate into a backup set, and sync onward for as long as nobody opens it. That belongs on file servers and mail gateways, not on the endpoint.

It is also the reason the toolkit polls three signals instead of one. My first implementation checked whether the file was still on disk after a few seconds and reported EICAR as undetected, which was flatly wrong. Defender had convicted every artifact at the exact write timestamps and simply remediates asynchronously, so a convicted file stays visible in the namespace for a while. A naive presence check produces confident false findings. The host check now polls disk presence, read-back success and AV telemetry concurrently until one of them fires, and the read-back is the decisive signal, because a file that cannot be read cannot be executed regardless of whether its directory entry has been unlinked yet.

### Detecting this

Signature detection for EICAR itself is trivial, and I want to write the rule out because how it has to be written is instructive.

```
alert http any any -> any any (msg:"EICAR test file in cleartext HTTP response body"; \
  flow:established,to_client; file_data; \
  content:"X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR"; \
  content:"-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*"; distance:0; within:35; \
  classtype:policy-violation; sid:1000010; rev:1;)
```

That is split across two `content:` matches with a `distance`/`within` constraint rather than one. Functionally it is the same match. It is written that way so the rule file does not itself contain the contiguous signature and get quarantined by the AV running on the sensor you are trying to deploy it to. The same problem shapes the toolkit, where the string is assembled from two literals in source for exactly that reason. The bytes on the wire and on disk are always the unmodified 68.

That rule works, and as a general detection it is close to worthless. Nothing that is actually trying will ever send you EICAR. It fires on control validation tests and on nothing else, which makes it useful as a positive control for your pipeline and useless as a threat detection.

The durable signal is not a payload signature at all. It is the coverage ratio:

```
count of NOT-INSPECTED events / count of inspected events, per egress path, over time
```

That number is the one thing in this whole exercise that generalises. It does not care about EICAR, or about any particular malware family, and it cannot be evaded by an attacker, because it measures your own sensor's admission that it could not see. If it is climbing, or if it dwarfs your inspected volume, that ratio is the finding, and no block-rate metric computed over inspected traffic will ever surface it.

The wider lesson is the usual one. A block rate measured only over the traffic you inspected is a measure of your inspection, not of your coverage. Detections built on what your own infrastructure is forced to admit are harder to fool than detections built on what the payload happens to look like.

### Things worth knowing if you run it

EICAR is harmless but it is not quiet. It will light up your AV console, your SIEM, and somebody's on-call queue. Get authorization, and tell the monitoring team first, unless a blind detection test is the actual objective, in which case tell whoever authorized it so the alerts are attributable to you. Every entry point in the toolkit requires an explicit `-Authorized` flag before it generates or transfers anything, for that reason.

The single-host convenience mode runs the origin and client on the same machine. Traffic to a local address is short-circuited by the network stack and will not reach an external tap, span port or inline IPS, so use it to validate the toolkit and the host path, not to certify a network sensor. Results from that mode are labelled accordingly in the report. For a real inline test, run `-Mode Serve` on one side of the sensor and `-Mode Fetch` on the other.

Test output is sensitive. The reports contain hostnames, agent versions, signature versions and exclusion configuration, which is useful reconnaissance in the wrong hands. `results/` is gitignored in the repository and should stay that way in yours.

Two implementation notes that cost me time. ZIP local headers embed a modification timestamp, so two hosts generating the same logical archive produce different bytes, and an origin-to-client integrity comparison reports the payload as altered in transit when nothing altered it. Pinning the entry timestamp makes generation deterministic. Separately, `ConvertFrom-Json` under PowerShell 5.1 emits a JSON array as a single object rather than enumerating it, so wrapping the pipeline in `@()` yields a one-element array containing the whole set, and every count downstream silently collapses to one.

The suite targets Windows PowerShell 5.1 with no external modules. The simulation needs Python 3.8 or later. Report export uses pandoc for HTML and DOCX and Edge for PDF, and missing export dependencies are skipped with a warning rather than failing the run. The Linux runner covers the egress and host tests for non-Windows test hosts.

### Repository

This project is catalogued in the Red Team Projects repository, together with the detection material from this post:

**[github.com/4D5A/Red-Team-Projects](https://github.com/4D5A/Red-Team-Projects)**

The entry there is documentation only. The coverage ratio is the part worth taking, and it needs none of this code, because it comes from telemetry your sensors already produce. Anyone who wants the suite itself can rebuild it from the methodology above.

If that is useful to you, starring or watching the repository is the easiest way to catch new entries as they are written up.

The EICAR test file is published by the European Institute for Computer Antivirus Research and is unmodified in every use here.
