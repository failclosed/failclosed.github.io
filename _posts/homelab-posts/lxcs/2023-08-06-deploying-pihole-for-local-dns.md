---
layout: post
title: Deploying Pi-Hole for local DNS
categories: [homelab, lxcs]
tags: [proxmox, linux, container, pi-hole]
after-content: [disclaimer-notice.html]
---

Pi-hole has good documentation, and I encourage you to read it. For now, here is a short post on the steps to deploy Pi-hole and some issues to avoid.

Pi-hole can be deployed on a Raspberry Pi, as a virtual machine, or as a Proxmox Linux container (LXC). In this example, I am going to deploy Pi-hole as an LXC.

## Create a Proxmox LXC

### 1. Create the LXC

According to Pi-hole's [documentation](https://docs.pi-hole.net/main/prerequisites/), Pi-hole requires the following:

* Minimum of 2 GB of disk space (4 GB is recommended)
* 512 MB of RAM

Below is a sample Pi-hole LXC configuration.

#### General
Hostname: pi-hole
Unprivileged container: checked
Nesting: unchecked

#### Template
Template: debian-12-standard_12.2-1_amd64.tar.zst

#### Disks
Disk size (GiB): 8

#### CPU
Cores: 1

#### Memory
Memory (MiB): 512
Swap (MiB): 512

#### Network
This is going to be specific to your environment, but generally your Pi-hole LXC needs one network interface. The LXC should be connected to the VLAN where Pi-hole is going to reply to DNS queries. You should also assign a static IP address to the LXC's network interface.

#### DNS
The values for the DNS domain and DNS servers that are entered on the LXC's DNS tab are those that the LXC's OS will use to communicate with your network and the Internet. These settings are separate from the DNS settings you will configure on Pi-hole related to replying to DNS queries or forwarding them to external DNS servers.

#### Confirm
Review your LXC's settings. If they look good, check the "Start after created" checkbox and click "Finish".

## Install Pi-hole (LXC)
Once the LXC has been created and started, it is time to install Pi-hole.

### 1. Connect to the LXC's Console and login as root.

Best practice is to create a separate user and use that user. For this example, we will use root.

### 2. Optionally, disable IPv6 for the LXC if you do not use it.

```nano /etc/sysctl.conf```

At the bottom of /etc/sysctl.conf, add these entries to disable IPv6.

~~~
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
~~~

```CTRL + X```

```SHIFT + Y```

```ENTER```

### 3. Verify your LXC can connect to the Internet

```ping 8.8.8.8```

```dig google.com```

If you receive errors after running either of those commands, you should stop and troubleshoot those problems before making any other changes.

### 4. Update packages

```apt update && sudo apt upgrade -y```

### 5. Optionally, install curl

```apt install curl -y```

## Install Pi-hole (application)

### 1. Use curl and bash to download and install Pi-hole.

```curl -sSL https://install.pi-hole.net | bash```

Or use wget and bash to download and install Pi-hole.

### 2. Select an upstream DNS Provider. 

In this example, I chose "Google (ECS, DNSSEC)".

### 3. Add the suggested ad block list.

### 4. Enable query logging.

### 5. Select a privacy mode for FTL. 

I chose "Show everything," but depending on where you are, you may be required to (or want to) select "Hide domains," "Hide domains and clients," or "Anonymous mode."

These additional privacy modes do not offer the Pi-hole administrator or network owner any further privacy but may offer some degree of privacy protection to other people connecting to your network and/or whose devices use your Pi-hole as their DNS server.

### 6. Configure your devices to use Pi-hole as their DNS server.

On this page, there are multiple important messages.

Make sure you document the URI to your Pi-hole's web interface and your Admin Webpage login password. If you lose your Admin Webpage login password, you can reset it from the LXC's console with `pihole -a -p`.

## Login and setup Pi-hole

1. Browse to https://\<yourIPaddress>/admin/login.
2. Login using the password you set in the setup wizard.

## Configure devices to use your Pi-hole server as their DNS server

The easiest way to point your whole network at Pi-hole is to update the DNS server setting on your router's DHCP configuration to your Pi-hole's static IP address, so every device that gets an address from DHCP automatically uses it. If you'd rather configure DNS per-device instead, most operating systems let you set a static DNS server in their network adapter settings.
