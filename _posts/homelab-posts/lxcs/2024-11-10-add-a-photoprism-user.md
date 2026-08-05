---
layout: post
title: Add a PhotoPrism user
categories: [homelab, lxcs]
tags: [container, linux, photoprism, proxmox]
after-content: [disclaimer-notice.html]
---

If you're running PhotoPrism through Docker (see [Deploy PhotoPrism in a Container]({{ site.baseurl }}{% post_url homelab-posts/lxcs/2024-11-10-deploy-photoprism-in-a-container %})), you can add additional users from the `photoprism` CLI inside the container rather than through the web UI:

```
docker compose exec photoprism photoprism users add -p YOURPASSWORD -n YOURNAME -m YOUREMAILADDRESS YOURUSERNAME
```

Replace `YOURPASSWORD`, `YOURNAME`, `YOUREMAILADDRESS`, and `YOURUSERNAME` with the new user's actual details.
