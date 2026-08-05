---
layout: post
title: Mount an SMB share hosted on a QNAP from Proxmox
categories: [homelab, lxcs]
tags: [proxmox, linux, qnap, smb]
after-content: [disclaimer-notice.html]
---

If you have a QNAP NAS on your network, you can mount one of its SMB shares directly as Proxmox storage, so any LXC or VM can use it without configuring the mount itself. From the Proxmox shell:

```
pvesm add cifs qnap --server 192.168.8.8 --share home --username username --password password --subdir /backups --smbversion 2.1
```

Replace `192.168.8.8` with your QNAP's IP address, and `home`/`/backups` with the actual share and subdirectory you want to mount. Use a dedicated share user with the least privilege necessary rather than an admin account.

Since the command above includes the password in plain text, clear it from your shell history afterward so it doesn't sit there indefinitely:

```
history -c
history -w
```
