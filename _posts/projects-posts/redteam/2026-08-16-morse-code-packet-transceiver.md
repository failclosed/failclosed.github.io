---
layout: post
title: Morse Code Packet Transceiver
categories: [redteam, projects]
tags: [Python, Network-Administration-Tools, infosec, Detection Engineering]
thumbnail-img: 'assets/img/2026-08-16-morse-code-packet-transceiver/morse-transceiver-gui-morse-session.png'
after-content: [disclaimer-notice.html]
---

### Introduction

Every so often it is worth building the obvious bad idea on purpose, just to see what it looks like on the wire. This one sends a text message across a network one symbol at a time, with each dot, dash, and separator travelling in its own ICMP echo request, TCP segment, or UDP datagram. A dot is a packet. A dash is a packet. The gap between words is a packet.

It is a terrible way to move data and an excellent way to look at data moving. That is the entire point. If you have ever tried to explain payload-based data smuggling to someone who has not spent much time in a packet analyser, watching a message spell itself out one frame at a time does more work than a diagram.

I am filing it under Red Team because the underlying technique, stuffing arbitrary content into protocol payloads that are not supposed to carry it, is one every network defender should be able to recognise. But I want to be clear about what this tool is and is not, because that distinction is the interesting part.

<img src="{{ 'assets/img/2026-08-16-morse-code-packet-transceiver/morse-transceiver-gui-morse-session.png' | relative_url }}" alt="Morse Code Packet Transceiver GUI showing a completed ICMP session, transmit log on the left and decoded receive log on the right" />

### How it works

Every transmission is framed by sentinel packets:

```
SOT  ->  unit  unit  unit ...  ->  CHK  ->  EOT
```

The receiver runs a scapy sniffer and ignores everything until it sees a start-of-transmission sentinel. It then accumulates units until the end-of-transmission sentinel arrives, at which point it decodes and displays the result. Anything outside that window is discarded, so unrelated traffic on the same protocol and port does not corrupt a session.

Between the data and the EOT sits a CHK packet carrying a CRC32 of the message. That checksum is computed over the canonically decoded text rather than the raw input, so both ends normalise the same way before hashing and a legitimate transmission always agrees with itself.

Four encodings are supported, and they change what actually goes on the wire rather than just how the log is rendered:

| Cipher | Wire form | Packets per character |
|---|---|---|
| Morse Code | dot, dash, separator | about 3.6 |
| Pigpen | Unicode glyphs | 1 |
| Atbash | Letters, A to Z reversed | 1 |
| None | Plaintext | 1 |

Morse is by far the loudest. A fourteen character message becomes 48 symbol packets plus the three framing packets, and at the default 0.3 second pacing that is roughly fifteen seconds of continuous traffic to move fourteen characters.

<img src="{{ 'assets/img/2026-08-16-morse-code-packet-transceiver/morse-transceiver-gui-pigpen-session.png' | relative_url }}" alt="The same tool running the Pigpen cipher, showing geometric glyph units on the wire and a verified CRC32 checksum" />

Both ends have to agree on the cipher. There is no negotiation in the protocol, so a mismatch produces nonsense and fails the checksum, which is at least a loud failure rather than a silent one.

<img src="{{ 'assets/img/2026-08-16-morse-code-packet-transceiver/morse-transceiver-gui-checksum-mismatch.png' | relative_url }}" alt="A deliberate cipher mismatch between the transmit and receive sides, caught by the CRC32 check" />

### What this is not

This is not a covert channel, and I would rather say so plainly than let the Red Team label imply otherwise.

A tool built to evade detection would pad payloads to look like normal traffic, randomise its timing, avoid fixed markers entirely, and move as few packets as possible. This does the opposite of all four. It emits a fixed byte sequence at the start and end of every session, uses a hardcoded ICMP identifier, sends one packet per symbol on a metronome, and generates roughly fifty packets to move a short sentence. Every one of those choices was made for visibility.

That makes it useful for something more practical than evasion: it is a generator for traffic that a detection should catch, which means you can use it to find out whether yours does.

### Detection

If you want to catch this, or something built along the same lines, there are two levels to work at.

The easy level is content matching. The default sentinels and the ICMP identifier are fixed values:

| Artefact | Value | Size |
|---|---|---|
| SOT sentinel | `\xfe\xfeMORSE:SOT\xfe\xfe` | 13 bytes |
| EOT sentinel | `\xfe\xfeMORSE:EOT\xfe\xfe` | 13 bytes |
| CHK prefix | `\xfe\xfeMORSE:CHK:` | 22 bytes with checksum |
| ICMP identifier | `0xBEEF` (48879) | |
| TCP/UDP source port | 9999 | |

```
alert icmp any any -> any any (msg:"Morse Packet Transceiver SOT sentinel"; \
  itype:8; content:"|fe fe|MORSE:SOT|fe fe|"; \
  classtype:policy-violation; sid:1000001; rev:1;)
```

That rule works, and it is also close to worthless as a general detection, because every one of those values is configurable. The tool accepts custom sentinels on the command line and in the interface, so a signature keyed to the default strings catches only the person who did not change them.

The durable level is behavioural. What survives a sentinel change is the shape of the traffic, and the shape is unusual regardless of what the payloads contain:

- **Payload sizes.** Every symbol packet carries a single byte. A one-byte ICMP echo request payload is strange on any network. A normal Windows ping carries 32 bytes of padding, and Linux carries 56, typically a timestamp followed by a fixed pattern.
- **Volume and regularity.** Dozens to hundreds of echo requests to a single destination, evenly spaced, is not what host-to-host ping traffic looks like.
- **Payload contents that are not padding.** The whole premise is that the payload varies meaningfully between packets. Legitimate echo requests repeat the same padding every time, so a session where consecutive payloads differ is worth a look on its own.

```
alert icmp any any -> any any (msg:"Repeated single-byte ICMP echo payloads, possible per-symbol data channel"; \
  itype:8; dsize:1; threshold:type both, track by_src, count 30, seconds 60; \
  classtype:policy-violation; sid:1000002; rev:1;)
```

The second rule is the one worth keeping. It does not care about Morse, or about this tool at all, and it will fire on anything that tries to move data one small unit at a time through echo requests.

The wider lesson is the usual one. Signatures pinned to a specific tool's constants are cheap to write and cheap to evade. Detections built on what the technique forces the traffic to look like are harder to write and considerably harder to get around, because the attacker cannot change them without giving up the thing they were trying to do.

### Things worth knowing if you run it

It needs raw socket privileges, so root on Linux and macOS, Administrator plus Npcap on Windows. Loopback is detected automatically, so pointing it at a `127.x.x.x` address gives you a self test that never leaves the machine, which is the sensible place to start.

Two implementation notes that cost me time and might save you some.

First, scapy dissects packet payloads based on port number. On UDP 53 it parsed the start sentinel as a DNS message, so reading `pkt[Raw].load` returned a single stray byte, the sentinel never matched, and the receiver sat in its idle state discarding everything while the transmit log looked perfectly healthy. The same trap applies to 67, 68, 123, 161, 5353, and any other port scapy has a protocol binding for. Reading the bytes directly off the transport layer instead of relying on the `Raw` layer makes the receiver port agnostic.

Second, running `sniff()` in short repeated calls to keep a stop button responsive opens and closes the capture handle on every iteration, and packets arriving in the gap are lost outright. I measured about 2.5 percent loss that way, which is more than enough to corrupt a message at random and produce checksum failures that look like a bug somewhere else entirely. A single continuous `AsyncSniffer` fixed it.

### Authorized use only

This tool transmits raw packets and captures network traffic. Use it only on networks, systems, and devices that you own, or for which you hold prior written authorization from the owner. It is intended for security research and education, academic use, authorized penetration testing and security assessments conducted under a written agreement, and diagnostics on your own infrastructure.

Unauthorized access to, use of, interception of traffic from, or interference with computer systems or networks is prohibited and may constitute a criminal offence under the Computer Fraud and Abuse Act, the Computer Misuse Act 1990, and comparable legislation elsewhere. Determining whether a given use is lawful is the responsibility of the user.

The program is released under the Apache License 2.0 and requires scapy, which is licensed separately under the GPL v2 and is not distributed with it.
