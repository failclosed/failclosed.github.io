---
layout: post
title: Checking Ubuntu Pro's compliance and hardening options
gh-repo: 4D5A
gh-badge: [follow]
categories: [blog]
tags: [Ubuntu, Ubuntu Pro, compliance, linux]
after-content: [disclaimer-notice.html]
---

Ubuntu Pro is Canonical's subscription that bundles Expanded Security Maintenance (extra CVE patching for both the base OS and a long list of common packages), Livepatch (kernel patches without a reboot), and a set of compliance/hardening tooling on top. It's free for personal use on up to 5 machines, which is worth knowing about if you run Ubuntu anywhere in your homelab.

To use it, first install the client:

```
sudo apt-get install ubuntu-advantage-tools
```

Once attached to a Pro subscription (`sudo pro attach`), check what's actually enabled:

```
sudo pro status --all
```

<img src="{{ 'assets/img/2025-01-04-checking-ubuntu-pro-compliance-and-hardening-options/pro-status-all.png' | relative_url }}" alt="pro status --all output" />

That's a lot of services, most of which are disabled by default and worth enabling individually rather than all at once (`esm-apps` and `esm-infra` are the two you probably want on right away).

Ubuntu Pro also has a compliance/hardening panel that surfaces FIPS 140-2 and the Ubuntu Security Guide (USG), which automates CIS benchmark and DISA-STIG profile hardening:

<img src="{{ 'assets/img/2025-01-04-checking-ubuntu-pro-compliance-and-hardening-options/compliance-hardening-panel.png' | relative_url }}" alt="Compliance and hardening panel showing FIPS 140-2 and USG" />

I tried enabling USG on Ubuntu 24.04 LTS (Noble Numbat) and hit a wall:

```
sudo pro enable usg
```

<img src="{{ 'assets/img/2025-01-04-checking-ubuntu-pro-compliance-and-hardening-options/pro-enable-usg-fails-1.png' | relative_url }}" alt="pro enable usg failing on Ubuntu 24.04" />

> Ubuntu Security Guide is not available for Ubuntu 24.04 LTS (Noble Numbat).
> Could not enable Ubuntu Security Guide.

So if you're planning around USG for CIS/DISA-STIG hardening, check Canonical's release support matrix for your specific Ubuntu version before you count on it. It wasn't available for 24.04 as of this writing.
