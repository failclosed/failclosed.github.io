---
layout: page
title: Resources
permalink: /software
subtitle: Software, GPTs, Claude Skills, and eBooks for IT and cybersecurity professionals
---

FailClosed builds tools for IT and cybersecurity professionals. Free items are marked **Free**; anything sold separately is marked **Buy** and links out to where it's actually purchased.

## Free Custom GPTs

<div class="resource-grid">

  <a class="resource-card" href="https://chatgpt.com/g/g-67b206d82c3081918141e76fca506290-source-code-license-assistant">
    <span class="resource-badge resource-badge-free">Free</span>
    <div class="resource-card-title">Source Code License Assistant</div>
    <div class="resource-card-desc">Helps developers select the best open-source licenses based on project type, intended use, and compatibility. It even scans existing repositories for compliance issues.</div>
  </a>

  <a class="resource-card" href="https://chatgpt.com/g/g-67b147bbb5bc8191a8f8c49b2a56bfdc-smtp-tutor">
    <span class="resource-badge resource-badge-free">Free</span>
    <div class="resource-card-title">SMTP Tutor</div>
    <div class="resource-card-desc">Provides a deep dive into SMTP fundamentals, aids in troubleshooting email delivery and configuration, and shares best practices for securing your SMTP servers.</div>
  </a>

  <a class="resource-card" href="https://chatgpt.com/g/g-67b141e2f99081919ee147b58fb93091-shadow-paw">
    <span class="resource-badge resource-badge-free">Free</span>
    <div class="resource-card-title">Shadow Paw</div>
    <div class="resource-card-desc">A master ninja cat hacker who delivers swift and stealthy answers to network security questions.</div>
  </a>

  <a class="resource-card" href="https://chatgpt.com/g/g-67b13ecd6d908191b8a6cbf80e54c1e2-dns-security-analyzer">
    <span class="resource-badge resource-badge-free">Free</span>
    <div class="resource-card-title">DNS Security Analyzer</div>
    <div class="resource-card-desc">Detects potential DNS hijacking, misconfigurations, and privacy risks while guiding you through the implementation of DNSSEC and other protective measures.</div>
  </a>

</div>

<!-- To add a paid entry: copy a card above, swap resource-badge-free for resource-badge-buy
     with text "Buy $X", and point the href at the purchase page (Gumroad/Payhip/store/etc). -->

## Claude Skills

Reusable [Claude Code](https://claude.com/claude-code) skills for authorized penetration testing engagements — enumeration through report writing. Public source: [github.com/4D5A/AI-Code/tree/main/skills](https://github.com/4D5A/AI-Code/tree/main/skills/).

<div class="resource-grid">

  <div class="resource-card resource-card-static">
    <span class="resource-badge resource-badge-skill">Skill</span>
    <div class="resource-card-title">pentest-enumeration</div>
    <div class="resource-card-desc">Low-noise host/subnet/VLAN/service discovery for an authorized engagement, without hammering shared infrastructure.</div>
  </div>

  <div class="resource-card resource-card-static">
    <span class="resource-badge resource-badge-skill">Skill</span>
    <div class="resource-card-title">pentest-fingerprinting-cve</div>
    <div class="resource-card-desc">Fingerprints real software and versions on discovered hosts, separating confirmed CVEs from mere patch-currency risk.</div>
  </div>

  <div class="resource-card resource-card-static">
    <span class="resource-badge resource-badge-skill">Skill</span>
    <div class="resource-card-title">pentest-key-systems</div>
    <div class="resource-card-desc">Identifies the crown-jewel systems in an environment — the ones that yield the broadest access if breached — to prioritize targets.</div>
  </div>

  <div class="resource-card resource-card-static">
    <span class="resource-badge resource-badge-skill">Skill</span>
    <div class="resource-card-title">pentest-offensive-tooling</div>
    <div class="resource-card-desc">Builds safe, staged proof-of-concept exploits against authorized targets when no public PoC exists yet.</div>
  </div>

  <div class="resource-card resource-card-static">
    <span class="resource-badge resource-badge-skill">Skill</span>
    <div class="resource-card-title">pentest-system-hardening</div>
    <div class="resource-card-desc">Converts findings into concrete, verifiable step-by-step remediation and hardening guidance.</div>
  </div>

  <div class="resource-card resource-card-static">
    <span class="resource-badge resource-badge-skill">Skill</span>
    <div class="resource-card-title">pentest-diagramming</div>
    <div class="resource-card-desc">Turns an engagement into clear network-topology and attack-path diagrams for reports, portable to DOCX/PDF.</div>
  </div>

  <div class="resource-card resource-card-static">
    <span class="resource-badge resource-badge-skill">Skill</span>
    <div class="resource-card-title">pentest-report-writing</div>
    <div class="resource-card-desc">Turns engagement notes into a client-ready report — HTML, PDF, and DOCX from one source.</div>
  </div>

  <div class="resource-card resource-card-static">
    <span class="resource-badge resource-badge-skill">Skill</span>
    <div class="resource-card-title">pentest-report-anonymization</div>
    <div class="resource-card-desc">Sanitizes a pentest report for external sharing, scrubbing domains, IPs, hostnames, and identifiers while preserving every finding.</div>
  </div>

</div>

## Guides & eBooks

<div class="resource-grid">

  <a class="resource-card" href="/gate?file=download1">
    <span class="resource-badge resource-badge-free">Free</span>
    <div class="resource-card-title">Email Security</div>
    <div class="resource-card-desc">A downloadable guide to SPF, DKIM, DMARC, and defending against spoofed mail.</div>
  </a>

</div>

<style>
.resource-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 20px;
  margin: 16px 0 32px;
}
.resource-card {
  position: relative;
  display: block;
  padding: 18px 20px;
  border-radius: 10px;
  background: #f8f8f8;
  box-shadow: 0 4px 8px rgba(0,0,0,0.12);
  text-decoration: none;
  color: inherit;
  transition: transform 0.2s, box-shadow 0.2s;
}
.resource-card:not(.resource-card-static):hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 16px rgba(0,0,0,0.2);
  text-decoration: none;
}
.resource-card-static {
  cursor: default;
}
.resource-badge {
  display: inline-block;
  font-size: 0.75em;
  font-weight: bold;
  padding: 2px 10px;
  border-radius: 999px;
  margin-bottom: 10px;
  text-transform: uppercase;
  letter-spacing: 0.03em;
}
.resource-badge-free {
  background: #d4edda;
  color: #1e7e34;
}
.resource-badge-buy {
  background: #fff3cd;
  color: #8a6516;
}
.resource-badge-skill {
  background: #d6e4ff;
  color: #1e3a8a;
}
.resource-card-title {
  font-weight: bold;
  margin-bottom: 8px;
  color: #008AFF;
}
.resource-card-desc {
  font-size: 0.95em;
  color: #555;
}
</style>
