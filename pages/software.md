---
layout: page
title: Resources
permalink: /software
subtitle: Software, GPTs, and eBooks for IT and cybersecurity professionals
---

FailClosed builds tools for IT and cybersecurity professionals — currently a suite of free custom GPTs and an eBook, with more software planned. Free items are marked **Free**; anything sold separately is marked **Buy** and links out to where it's actually purchased.

<div class="software-grid">

  <a class="software-card" href="https://chatgpt.com/g/g-67b206d82c3081918141e76fca506290-source-code-license-assistant">
    <span class="software-badge software-badge-free">Free</span>
    <div class="software-card-title">Source Code License Assistant</div>
    <div class="software-card-desc">Helps developers select the best open-source licenses based on project type, intended use, and compatibility. It even scans existing repositories for compliance issues.</div>
  </a>

  <a class="software-card" href="https://chatgpt.com/g/g-67b147bbb5bc8191a8f8c49b2a56bfdc-smtp-tutor">
    <span class="software-badge software-badge-free">Free</span>
    <div class="software-card-title">SMTP Tutor</div>
    <div class="software-card-desc">Provides a deep dive into SMTP fundamentals, aids in troubleshooting email delivery and configuration, and shares best practices for securing your SMTP servers.</div>
  </a>

  <a class="software-card" href="https://chatgpt.com/g/g-67b141e2f99081919ee147b58fb93091-shadow-paw">
    <span class="software-badge software-badge-free">Free</span>
    <div class="software-card-title">Shadow Paw</div>
    <div class="software-card-desc">A master ninja cat hacker who delivers swift and stealthy answers to network security questions.</div>
  </a>

  <a class="software-card" href="https://chatgpt.com/g/g-67b13ecd6d908191b8a6cbf80e54c1e2-dns-security-analyzer">
    <span class="software-badge software-badge-free">Free</span>
    <div class="software-card-title">DNS Security Analyzer</div>
    <div class="software-card-desc">Detects potential DNS hijacking, misconfigurations, and privacy risks while guiding you through the implementation of DNSSEC and other protective measures.</div>
  </a>

</div>

<!-- To add a paid entry: copy a card above, swap software-badge-free for software-badge-buy
     with text "Buy $X", and point the href at the purchase page (Gumroad/Payhip/store/etc). -->

## Guides & eBooks

<div class="software-grid">

  <a class="software-card" href="/gate?file=download1">
    <span class="software-badge software-badge-free">Free</span>
    <div class="software-card-title">Email Security</div>
    <div class="software-card-desc">A downloadable guide to SPF, DKIM, DMARC, and defending against spoofed mail.</div>
  </a>

</div>

<style>
.software-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 20px;
  margin: 24px 0;
}
.software-card {
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
.software-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 16px rgba(0,0,0,0.2);
  text-decoration: none;
}
.software-badge {
  display: inline-block;
  font-size: 0.75em;
  font-weight: bold;
  padding: 2px 10px;
  border-radius: 999px;
  margin-bottom: 10px;
  text-transform: uppercase;
  letter-spacing: 0.03em;
}
.software-badge-free {
  background: #d4edda;
  color: #1e7e34;
}
.software-badge-buy {
  background: #fff3cd;
  color: #8a6516;
}
.software-card-title {
  font-weight: bold;
  margin-bottom: 8px;
  color: #008AFF;
}
.software-card-desc {
  font-size: 0.95em;
  color: #555;
}
</style>
