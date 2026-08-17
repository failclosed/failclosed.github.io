---
layout: post
title: EICAR Control Validation Suite
gh-repo: 4D5A/Red-Team-Projects
gh-badge: [follow, star]
categories: [redteam, projects]
tags: [PowerShell, Python, EICAR, IDS-IPS, TLS-Inspection, Detection Engineering, infosec]
thumbnail-img: 'assets/img/2026-08-16-eicar-control-validation-suite/console-ids-no-tls.png'
---

### Introduction

EICAR is the least impressive payload in existence. Sixty-eight printable ASCII characters, no execution, and no persistence. Every anti-malware product is expected to detect it, and detecting it proves nothing about how good an engine is.

That's what makes it useful. EICAR does not measure whether your detection engine is smart, it measures whether it is looking. I wanted to know which layers of my own setup were in a position to see a malicious download at all, so I built a suite that runs five artifact variants over two protocols, through four network postures, against a real endpoint agent, and records the network and host verdicts separately.

The short version is that in the most common enterprise posture I tested, every payload delivered over TLS reached the endpoint untouched, and the sensor recorded that fact at a severity nobody alerts on.

<img src="{{ 'assets/img/2026-08-16-eicar-control-validation-suite/console-ids-no-tls.png' | relative_url }}" alt="IDS alert console showing critical BLOCK events for every cleartext request and informational NOT-INSPECTED events for every encrypted request carrying the same five files" />

I want to be clear about one point before anything else. The gateway in these screenshots is a simulation I wrote. It genuinely scans response bodies, recurses into ZIP containers, blocks, and writes alerts, and every image here is a live render of it running. It's not a commercial product and it doesn't imitate one. The endpoint results are real, measured against Microsoft Defender on one Windows 10 computer with real-time protection enabled and current signatures. One host is one data point rather than a product review.

### The results

Three inspection policies, five artifacts each, over both protocols. The only number that matters is how many payloads reached the endpoint.

| Inspection policy | Cleartext HTTP | TLS |
|---|---|---|
| No inspection | 5/5 delivered | 5/5 delivered |
| IDS/IPS, no TLS interception | 0/5 delivered | 5/5 delivered |
| IDS/IPS with TLS interception | 0/5 delivered | 0/5 delivered |

The middle row is the point of the whole project. That sensor has a 100% block rate on everything it inspected, so its dashboard is green. It also passed every payload that arrived over TLS, which in any environment resembling the modern web is all of them. It's not a broken sensor. It is a working sensor pointed at the wrong five percent of the traffic, and the metric it reports cannot distinguish between those two situations.

<img src="{{ 'assets/img/2026-08-16-eicar-control-validation-suite/cli-matrix-curl.png' | relative_url }}" alt="Full test matrix run from curl showing five blocked cleartext downloads followed by five delivered TLS downloads under the no-interception policy" />

Look at the severity column in the first screenshot. The blocks are `critical` and the misses are `informational`. Informational events are what gets filtered out of a SOC view, rolled into a daily digest, or never alerted on at all. The sensor is telling you that it is blind, in the one severity class nobody reads.

If you turn interception on, the same requests resolve differently.

<img src="{{ 'assets/img/2026-08-16-eicar-control-validation-suite/console-ids-tls.png' | relative_url }}" alt="The same alert console with TLS interception enabled, every row now a critical BLOCK event" />

The difference between those two screenshots is not detection capability. The signature was loaded and matching the entire time. The difference is visibility.

### What the suite tests

There are three modules, and you can run each one on its own or all of them together.

1. **Egress** pulls the four published artifacts from `secure.eicar.org` over both protocols.
2. **Controlled origin** serves byte-exact artifacts over genuine cleartext HTTP and TLS from an origin you control, with `Serve` and `Fetch` modes so the two ends can sit on opposite sides of the sensor.
3. **Host AV** characterizes the endpoint agent across detection stages: posture, extension handling, archive handling, extraction, on-demand scan, and AMSI.

There is a trap in the first module that you should know about before building anything similar. eicar.org answers plain HTTP with a 301 redirect to HTTPS, so the payload never crosses the wire in cleartext when you source it upstream. A cleartext IDS test sourced from eicar.org is inconclusive by construction, and a pass obtained that way means nothing. That's why the second module exists.

Verdicts are recorded per layer, because collapsing them hides the finding.

| Verdict | Layer | Meaning |
|---|---|---|
| `Blocked` | Network | A device stopped or substituted the transfer, good |
| `Allowed` | Network | Delivered intact and hash verified, a gap |
| `Quarantined` | Host | Removed, read denied, or convicted per AV telemetry, good |
| `Persisted` | Host | Remained on disk and re-read byte for byte intact, a gap |

The case worth escalating is anything that is both `Allowed` and `Persisted`.

### Archives behave differently

Two of the five artifacts are containers, EICAR inside a ZIP and that ZIP inside another ZIP. Container handling is where implementations diverge, so it is worth testing rather than assuming.

<img src="{{ 'assets/img/2026-08-16-eicar-control-validation-suite/block-page-nested-zip.png' | relative_url }}" alt="Interception page for a nested ZIP archive showing the match path recorded through both container layers" />

A sensor that stops at depth one passes that file and a sensor that recurses catches it. The only way to know which one you have is to send it.

The endpoint side is where this became interesting, and that part is measurement rather than simulation. Defender convicted the raw 68-byte file in roughly two seconds under every extension I tried, including `.com`, `.txt`, `.exe`, `.dll` and `.js`. Detection is content based, so renaming a file gains an attacker nothing.

The same payload inside a ZIP sat on disk untouched past twenty seconds with no detection telemetry at all. Extraction of the single layer archive was then denied outright, and an explicit on-demand scan convicted everything. Archive members are inspected on access and on demand rather than on write.

That's not a defect. Deep scanning every container on write is expensive and most products trade it away deliberately, and the payload still cannot reach execution undetected. The residual risk is dwell time and propagation rather than execution, because a malicious archive can sit on a file share, replicate into a backup set, and sync onward for as long as nobody opens it. That belongs on your file servers and mail gateways rather than on the endpoint.

It is also the reason the suite polls three signals instead of one. My first version checked whether the file was still on disk after a few seconds and reported EICAR as undetected, which was flatly wrong. Defender had convicted every artifact at the exact write timestamps and simply remediates asynchronously, so a convicted file remains visible in the namespace for a while. If you build something similar, a naive presence check will produce confident false findings. The host check now polls disk presence, read-back success, and AV telemetry concurrently until one of them fires, and the read-back is the decisive signal, because a file you cannot read cannot be executed regardless of whether its directory entry has been unlinked yet.

### Detection

Signature detection for EICAR is trivial, but how you have to write the rule is instructive.

```
alert http any any -> any any (msg:"EICAR test file in cleartext HTTP response body"; \
  flow:established,to_client; file_data; \
  content:"X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR"; \
  content:"-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*"; distance:0; within:35; \
  classtype:policy-violation; sid:1000010; rev:1;)
```

That is split across two `content:` matches with a `distance` and `within` constraint rather than one. Functionally it is the same match. It is written that way so the rule file does not contain the contiguous signature itself and get quarantined by the AV running on the sensor you are deploying it to. The same problem shapes the toolkit, where the string is assembled from two literals in source for the same reason. The bytes on the wire and on disk are always the unmodified 68.

That rule works, and as a general detection it is close to worthless. Nothing that is actually trying will ever send you EICAR. It's a positive control for your pipeline and nothing more.

The durable signal is not a payload signature at all. It is the coverage ratio.

```
count of NOT-INSPECTED events / count of inspected events, per egress path, over time
```

That is the one measurement here that generalizes. It doesn't depend on EICAR or on any particular malware family, and an attacker cannot evade it, because it measures your own sensor's admission that it could not see. If it is climbing, or if it dwarfs your inspected volume, that ratio is the finding. No block rate computed over inspected traffic will surface it.

A block rate measured only over the traffic you inspected is a measure of your inspection rather than of your coverage.

### Notes if you run it

EICAR is harmless but it's not quiet. It will light up your AV console, your SIEM, and somebody's on-call queue. Get authorization, and tell the monitoring team first unless a blind detection test is the actual objective, in which case tell whoever authorized it so that the alerts are attributable to you. Every entry point in the suite requires an explicit `-Authorized` flag before it generates or transfers anything.

The single-host convenience mode runs the origin and client on the same computer. Traffic to a local address is short-circuited by the network stack and will not reach an external tap, span port, or inline IPS, so use it to validate the toolkit and the host path rather than to certify a network sensor. For a real inline test, run `-Mode Serve` on one side of the sensor and `-Mode Fetch` on the other.

Test output is sensitive, and I keep it out of anything I publish. The reports contain hostnames, agent versions, signature versions, and exclusion configuration, which is useful reconnaissance in the wrong hands. Keep `results/` out of any repository you publish.

Two implementation notes may save you some time. ZIP local headers embed a modification timestamp, so two computers generating the same logical archive produce different bytes, and an origin-to-client integrity comparison will report the payload as altered in transit when nothing altered it. Pinning the entry timestamp makes generation deterministic. Separately, `ConvertFrom-Json` under PowerShell 5.1 emits a JSON array as a single object rather than enumerating it, so if you wrap the pipeline in `@()` you get a one-element array containing the whole set and every count downstream silently collapses to one.

The suite targets Windows PowerShell 5.1 with no external modules, and the simulation needs Python 3.8 or later. Report export uses pandoc for HTML and DOCX and Edge for PDF, and missing dependencies are skipped with a warning rather than failing the run.

### Repository

The project is catalogued in the [Red Team Projects](https://github.com/4D5A/Red-Team-Projects) repository along with the detection guidance above. The entry there is documentation only. The coverage ratio is the portion worth taking and it requires none of this code, because it comes from telemetry your sensors already produce.

The EICAR test file is published by the [European Institute for Computer Antivirus Research](https://www.eicar.org/download-anti-malware-testfile/) and is unmodified in every use here.
