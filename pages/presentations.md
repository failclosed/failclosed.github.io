---
layout: page
title: Speaking
permalink: /speaking
subtitle: Conference talks, slides, and materials
---

I speak at security conferences on offensive AI tooling, homelab security research, and practical red/blue team engineering. Interested in having me speak at yours? Reach out via [X](https://x.com/failclosed).

<div class="talk-card">
  <div class="talk-conference">BSides Cleveland 2026 &middot; Red Team Track &middot; September 26, 2026</div>
  <div class="talk-title">Pentesting with Claude Code</div>
  <p class="talk-subtitle">An AI agent through a full engagement — recon to exploitation to report.</p>
  <p>I told an AI coding agent to pentest my own home lab and see if it could escape a container to the hypervisor. This is what happened.</p>
  <p>The talk walks through a real authorized engagement against a self-owned Proxmox homelab: enumeration by reverse DNS, fingerprinting a live CVE (an unauthenticated arbitrary file read in Gitea, CVE-2026-59774), building a proof-of-concept where none existed publicly, and a read-only container-escape assessment, all with a full command-and-evidence log and a human approving every irreversible step. It also compares how three different Claude models (Sonnet 5, Opus 5, and Opus 4.8) handled the same engagement, and covers the eight reusable skills the agent produced along the way.</p>
  <div class="talk-links">
    <a href="https://github.com/4D5A/presentations/blob/main/BSides%20Cleveland%202026/BSides_Cleveland_presentation_github.pptx">Slides (PowerPoint)</a>
  </div>
</div>

<style>
.talk-card {
  max-width: 720px;
  margin: 24px 0;
  padding: 20px 24px;
  border-radius: 10px;
  background: #f8f8f8;
  box-shadow: 0 4px 8px rgba(0,0,0,0.12);
}
.talk-conference {
  font-size: 0.85em;
  text-transform: uppercase;
  letter-spacing: 0.03em;
  color: #777;
  margin-bottom: 6px;
}
.talk-title {
  font-size: 1.4em;
  font-weight: bold;
  color: #008AFF;
}
.talk-subtitle {
  font-style: italic;
  color: #555;
  margin-top: 4px;
}
.talk-links {
  margin-top: 14px;
  display: flex;
  gap: 16px;
}
.talk-links a {
  font-weight: bold;
}
</style>
