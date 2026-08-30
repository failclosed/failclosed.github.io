---
layout: post
title: "Diagramming the home lab — and keeping a version that is safe to share"
categories: [homelab, networking]
tags: [networking, vlan, documentation, diagram, haproxy, proxmox, dns, segmentation]
after-content: [disclaimer-notice.html]
---

## Introduction

For years the only complete map of my home lab lived in my head. That works right up until it doesn't — until you are trying to remember which VLAN a container sits in at 11pm, or until you want to show someone how the reverse proxy layer is wired without handing them a list of your internal addresses.

So I finally sat down and drew it. Two things came out of that exercise that I did not expect:

1. The diagram found problems the running network had been quietly hiding.
2. Drawing it *twice* — once with real addressing, once anonymized — turned out to be the whole point, not extra work.

This post covers where the source material comes from, how I structured the diagram, what it surfaced, and how I keep a shareable copy that doesn't leak the real thing.

## Don't draw from memory

The temptation is to open a diagramming tool and start dragging boxes around based on what you think is running. Don't. You will draw the network you *intended* to build, which is not the network you have.

Every node in my diagram traces back to one of these sources:

|Source|What it gives you|
|---|---|
|Gateway VLAN and DHCP configuration|The authoritative list of segments, their subnets, and which ones actually have leases|
|Reverse proxy backend configuration|Every published service, its backend address and port, TLS settings, and health checks|
|Reverse-DNS sweep of each subnet|Hosts that exist but that nobody remembered to document|
|Hypervisor guest inventories|Which VMs and containers live on which host, and which are powered off|
|Firewall rules between segments|Which of those "segments" are actually isolated versus nominally separate|

The reverse-DNS sweep is the one people skip, and it is the one that pays. In my case it turned up a hypervisor host on a second virtualization platform that had simply fallen out of my mental model. It had been running for months. It was in no notes anywhere.

The proxy configuration is the other high-yield source. A reverse proxy config is, functionally, a machine-readable inventory of everything you have decided to publish — backend address, port, hostname, and often the TLS posture too. Reading mine end to end told me I had 13 virtual hosts behind one proxy node and 9 behind another, several of which pointed at backends that were powered off.

## Structuring the diagram

I settled on a layout with three moving parts.

### Bands are segments

Each VLAN is a horizontal band, labelled with its subnet and its role. Hosts sit inside the band they belong to. This sounds obvious, but it forces an honest conversation: if a host's band doesn't match your intuition about where it lives, one of the two is wrong.

Grouping by segment also makes empty segments visible. I had three VLANs that were routed by the gateway and had no documented hosts at all — a gateway address and nothing behind it. That is either dead configuration to clean up or an inventory gap. Either way it was invisible until the segments got their own row.

### Nodes are hosts, with facts attached

Each node carries a short fact list rather than free text: address, listening ports, what it depends on, what depends on it. Constraining yourself to facts keeps the diagram from turning into prose, and it makes staleness obvious — a wrong port is a lot easier to spot than a wrong paragraph.

Critically, I mark nodes as **inferred** when I know they exist but haven't confirmed the details. My IoT segment has cameras, a printer, and a single-board computer on it. I know that. I haven't enumerated them properly. Marking them inferred is honest; leaving them off is a lie by omission, and drawing them as if confirmed is worse.

### Edges are typed

Not all connections mean the same thing, and a diagram where every line looks identical is a diagram nobody can read. I use distinct line styles for:

- **Physical uplink** — the actual cable path from the ISP handoff inward
- **Routed by gateway** — traffic that crosses segments and is therefore subject to inter-VLAN firewall policy
- **Reverse proxy backend** — a published service, labelled with the backend port
- **File storage** — a host mounting shares from the NAS
- **Offsite backup** — the one path that leaves the building
- **VPN** — remote peers, and where the tunnel terminates

Typing the edges is what makes the diagram answer questions instead of just decorating a page. "What happens if the NAS goes down?" becomes a matter of counting storage edges. "What is reachable from outside?" becomes a matter of following the uplink and the VPN edge.

### Multiple views over one dataset

The same node and edge data drives three views: a **logical** view showing segments and routing, a **services** view showing what is published through the proxies and on what ports, and a **storage** view showing which hosts depend on the NAS and where the offsite copy goes.

One dataset, three views, no chance of them drifting apart. This is the single strongest argument for building the diagram as data plus a renderer, rather than as hand-placed shapes.

## What the diagram found

This is the part that justified the afternoon.

**Management and clients share a segment.** My workstations, the hypervisor web interfaces, the NAS management interface, and the gateway's own admin UI were all reachable in the same VLAN. Every one of those is a high-value target, and every one was one compromised laptop away. Splitting management onto its own segment is the single biggest improvement available in my environment, and I had never noticed because I had never seen it drawn.

**One container host was carrying six published services.** Six separate applications, six ports, one host, one failure domain. Individually each deployment was a reasonable decision. Together they were the densest single point of failure in the service layer. That only becomes visible when the six edges are drawn converging on one box.

**Powered-off hosts are undefended, not safe.** My SIEM and my remote-access gateway were both powered down. A diagram that only shows running hosts would have quietly omitted both — including the fact that the thing meant to detect intrusions was not detecting anything.

**One uplink, one offsite copy.** No WAN failover, and a single external backup target. Both are entirely defensible choices for a home lab. Both are much easier to make deliberately once they are drawn.

**Internal addresses were resolving publicly.** My authoritative DNS correctly refuses zone transfers, but records were still handing out RFC 1918 addresses to anyone who asked for them by name. Not catastrophic, but free reconnaissance for anyone curious.

## Making a version that is safe to share

Here is the thing about a good network diagram: the better it is, the more dangerous it is to publish. A complete map of your segments, hosts, addresses, and published ports is exactly the document you would most want if you were attacking it.

So I keep two renderings from the same structure — the internal one with real addressing, and an anonymized one that is genuinely safe to hand to anyone.

### Substitute consistently, not randomly

The rule that matters: a given real value must map to exactly one fake value, everywhere, every time. If one address becomes `10.20.30.20` in one place and `10.20.30.21` in another, the diagram stops being true and you have destroyed the thing you were trying to share.

I keep an explicit substitution map and apply it mechanically. What gets substituted:

- **The domain**, everywhere it appears — including in service hostnames and certificate references
- **Hostnames**, replaced with role-based names (`proxy-01`, `hv-02`, `NAS-01`) rather than my actual naming scheme
- **All addressing**, remapped to a clean scheme that preserves structure — same number of segments, same subnet sizes, including the `/29` that isn't a `/24`
- **VLAN IDs and names**, relabelled to letters
- **Any account names, certificate subjects, or provider account identifiers**

### Preserve everything technical

What does *not* get substituted, because changing it would make the diagram useless:

- **Ports and protocols.** `tcp/8006`, `udp/51820`, `tcp/3000` — these are product defaults, not secrets, and they carry most of the meaning.
- **Topology.** The number of hops, which host proxies which, what depends on the NAS.
- **Relative structure.** If two services share a host in reality, they share a host in the anonymized version.
- **Findings and notes.** The observation that management shares a VLAN with clients is the valuable part. It survives anonymization intact.

### Watch for re-identification

Substituting addresses is the easy half. The half people miss:

- **Distinctive combinations.** A subnet size that appears exactly once, an unusual port, a service almost nobody runs — any of these can fingerprint an environment even with every address changed.
- **Third-party names.** Your ISP, your offsite storage provider, your hardware vendors. I generalize these to "ISP uplink" and "object storage."
- **Screenshots.** Cropping is not redaction. A single un-scrubbed screenshot undoes the entire exercise.
- **The file itself.** Metadata, revision history, and the substitution map must never ship with the shareable copy. Keep the map with the internal version.

I also label the anonymized rendering as anonymized, right in the header, with a note stating that no real internal addressing appears on the page. That is partly for readers and mostly for me — so that six months from now I do not confuse the two files.

## Keeping it honest

A diagram that is wrong is worse than no diagram, because you will trust it. Mine gets revisited whenever I add a VLAN, publish a new service through the proxy, or move a workload between hypervisors. Re-reading the proxy config and re-running a reverse-DNS sweep takes about ten minutes and catches most drift.

The inferred markers help here too. Anything still marked inferred is a standing to-do. When I finally enumerate the IoT segment properly, those markers come off, and the diagram gets one notch more true.

## Closing thought

I went into this expecting to produce documentation. What I actually produced was an audit. Drawing the network forced me to reconcile what I thought I had against what the configuration files and DNS actually said, and the gap between those two was where every finding lived.

If your home lab has grown past the point where you can hold it in your head — and it probably has — draw it. Draw it from configuration rather than memory, mark what you haven't verified, and build the shareable version at the same time rather than as an afterthought.
