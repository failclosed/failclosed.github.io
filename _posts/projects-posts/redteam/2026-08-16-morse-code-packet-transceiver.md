---
layout: post
title: Morse Code Packet Transceiver
gh-repo: 4D5A/Red-Team-Projects
gh-badge: [follow, star]
categories: [redteam, projects]
tags: [Python, Network-Administration-Tools, infosec, Detection Engineering]
thumbnail-img: 'assets/img/2026-08-16-morse-code-packet-transceiver/morse-transceiver-gui-morse-session.png'
---

### Introduction

The Morse Code Packet Transceiver is a Python tool that sends a text message across a network one symbol at a time. Each dot, dash, and separator travels in its own ICMP echo request, TCP segment, or UDP datagram. A dot is a packet, a dash is a packet, and the gap between words is a packet.

It's a poor way to move data and an effective way to watch data move. I wanted a way to make payload-based data smuggling visible in a packet analyzer, because watching a message spell itself out one frame at a time explains the technique better than a diagram does. If you have ever tried to describe payload smuggling to someone who doesn't spend much time in a packet capture, this gives you something to point at.

I am filing it under Red Team because the underlying technique, placing content into protocol payloads that are not intended to carry it, is one that every network defender should be able to recognize.

<img src="{{ 'assets/img/2026-08-16-morse-code-packet-transceiver/morse-transceiver-gui-morse-session.png' | relative_url }}" alt="Morse Code Packet Transceiver GUI showing a completed ICMP session, transmit log on the left and decoded receive log on the right" />

### How it works

Every transmission is framed by sentinel packets:

```
SOT  ->  unit  unit  unit ...  ->  CHK  ->  EOT
```

1. The receiver runs a scapy sniffer and ignores everything until it sees a start-of-transmission sentinel.
2. It accumulates units until the end-of-transmission sentinel arrives, then decodes and displays the result.
3. Between the data and the EOT sits a CHK packet carrying a CRC32 of the message.

Anything outside that window is discarded, so unrelated traffic on the same protocol and port will not corrupt a session. The checksum is calculated over the canonically decoded text rather than the raw input, so both ends normalize the same way before hashing and a legitimate transmission always agrees with itself.

Four encodings are available, and they change what is placed on the wire rather than only how the log is displayed:

| Cipher | Wire form | Packets per character |
|---|---|---|
| Morse Code | dot, dash, separator | about 3.6 |
| Pigpen | Unicode glyphs | 1 |
| Atbash | Letters, A to Z reversed | 1 |
| None | Plaintext | 1 |

Morse is the loudest of the four. A fourteen character message becomes 48 symbol packets plus the three framing packets. At the default pacing of 0.3 seconds, that is roughly fifteen seconds of continuous traffic to move fourteen characters.

<img src="{{ 'assets/img/2026-08-16-morse-code-packet-transceiver/morse-transceiver-gui-pigpen-session.png' | relative_url }}" alt="The same tool running the Pigpen cipher, showing geometric glyph units on the wire and a verified CRC32 checksum" />

Both ends must be set to the same cipher. There is no negotiation in the protocol, so if you set them differently you'll get nonsense and a failed checksum, which is at least a loud failure rather than a silent one.

<img src="{{ 'assets/img/2026-08-16-morse-code-packet-transceiver/morse-transceiver-gui-checksum-mismatch.png' | relative_url }}" alt="A deliberate cipher mismatch between the transmit and receive sides, caught by the CRC32 check" />

### What this project is not

This is not a covert channel, and I would rather state that plainly than allow the Red Team label to imply otherwise.

A tool built to evade detection would pad payloads to resemble normal traffic, randomize its timing, avoid fixed markers, and move as few packets as possible. This project does the opposite of all four. It emits a fixed byte sequence at the start and end of every session, uses a hardcoded ICMP identifier, sends one packet per symbol on a metronome, and generates roughly fifty packets to move a short sentence. Every one of those choices was made for visibility.

That makes it useful for something more practical than evasion. It is a generator for traffic that your detections should catch, so you can use it to determine whether they do.

### Detection

There are two levels to work at.

#### Content matching

The default sentinels and the ICMP identifier are fixed values.

| Artifact | Value | Size |
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

That rule works, and it's close to worthless as a general detection, because every one of those values is configurable. The tool accepts custom sentinels on the command line and in the interface, so a signature keyed to the default strings will only catch the person who did not change them.

#### Behavioral indicators

What survives a sentinel change is the shape of the traffic, and the shape is unusual regardless of what the payloads contain. If you are writing a hunt rule, these are the properties worth keying on.

- **Payload sizes.** Every symbol packet carries a single byte. A one-byte ICMP echo request payload is unusual on any network. A normal Windows ping carries 32 bytes of padding and Linux carries 56, typically a timestamp followed by a fixed pattern.
- **Volume and regularity.** Dozens to hundreds of echo requests to a single destination, evenly spaced, is not what host-to-host ping traffic looks like.
- **Payload contents that are not padding.** Legitimate echo requests repeat the same padding every time, so a session where consecutive payloads differ is worth reviewing on its own.

```
alert icmp any any -> any any (msg:"Repeated single-byte ICMP echo payloads, possible per-symbol data channel"; \
  itype:8; dsize:1; threshold:type both, track by_src, count 30, seconds 60; \
  classtype:policy-violation; sid:1000002; rev:1;)
```

The second rule is the one worth keeping. It doesn't depend on Morse, or on this tool at all, and it will fire on anything that attempts to move data one small unit at a time through echo requests.

If you take one item from this project, take that distinction. Signatures pinned to a specific tool's constants are inexpensive to write and inexpensive to evade. Detections built on what the technique forces the traffic to look like are harder to write and considerably harder to work around, because an attacker cannot change them without giving up the result they wanted.

### Notes if you run it

You'll need raw socket privileges, so root on Linux and macOS, or Administrator plus Npcap on Windows. Loopback is detected automatically, so if you point it at a `127.x.x.x` address you get a self test that never leaves the machine. That's where I started when I was testing it.

Two implementation notes may save you some time.

Scapy dissects packet payloads based on port number. On UDP 53 it parsed the start sentinel as a DNS message, so reading `pkt[Raw].load` returned a single stray byte, the sentinel never matched, and the receiver sat in its idle state discarding everything while the transmit log appeared healthy. I found the same trap applies to ports 67, 68, 123, 161, 5353, and any other port that scapy has a protocol binding for. If you read the bytes directly off the transport layer instead of relying on the `Raw` layer, the receiver becomes port agnostic.

Running `sniff()` in short repeated calls to keep a stop button responsive opens and closes the capture handle on every iteration, and packets arriving in the gap are lost. I measured about 2.5 percent loss that way, which is more than enough to corrupt a message at random and produce checksum failures that appear to be a bug somewhere else. A single continuous `AsyncSniffer` corrected it.

### Repository

The project is catalogued in the [Red Team Projects](https://github.com/4D5A/Red-Team-Projects) repository along with the detection guidance above. The entry there is documentation only. The behavioral rule is the portion worth taking, because it applies to anything that moves data one small unit at a time.

The program is released under the Apache License 2.0 and requires scapy, which is licensed separately under the GPL v2 and is not distributed with it.
