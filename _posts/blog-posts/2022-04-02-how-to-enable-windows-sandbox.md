---
layout: post
title: How to enable Windows Sandbox
gh-repo: 4D5A
gh-badge: [follow]
categories: [blog]
tags: [Virtualization, Windows 10, Windows 11, Windows Sandbox]
after-content: [disclaimer-notice.html]
---

Windows 10 and Windows 11 (Pro and Enterprise versions) include a great feature called Windows Sandbox.

If you aren't familiar with operating system virtualization, you can read my overview of virtualization in [What is Virtualization]({{ site.baseurl }}{% post_url blog-posts/2022-03-26-what-is-virtualization %}).

Windows Sandbox provides a disposable virtual environment that will discard all changes in the virtual environment after you close the session. This gives you the flexibility to test changes in the registry or test software in a non-persistent environment without needing to create a full virtual machine, install an operating system for your virtual machine, or configure networking (there are pros and cons to this, and I will discuss that later).

To enable Windows Sandbox, you need to make sure your physical machine supports virtualization and that your physical machine's BIOS has virtualization turned on. There are also some minimum hardware requirements to use Windows Sandbox<sup>1</sup>, but for most people those will not be a problem.

If you want to use Windows Sandbox in a virtual machine and the host operating system is running Windows, you will need to enable nested virtualization. To do that, you will need to open PowerShell as an Administrator and run the following command<sup>2</sup>:

~~~
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
~~~

To enable Windows Sandbox using Windows' UI, you would click Start, type "Turn Windows features on and off," and click "Turn Windows features on and off."

<img src="{{ 'assets/img/2022-04-02-how-to-enable-windows-sandbox/turn-windows-features-on-or-off.png' | relative_url }}" alt='Turn Windows features on or off' />

You will need to scroll down until you see the Windows feature named Windows Sandbox, check the checkbox next to that Windows feature, and click the OK button.

<img src="{{ 'assets/img/2022-04-02-how-to-enable-windows-sandbox/windows-features.png' | relative_url }}" alt='Windows Features' />

You will need to restart your computer to finish installing the Windows feature.

If you prefer to install Windows Sandbox using PowerShell, you can open PowerShell as an Administrator and run the following command<sup>3</sup>:

~~~
Enable-WindowsOptionalFeature -FeatureName "Containers-DisposableClientVM" -All -Online
~~~

After you restart your computer, you can click the Start button and type Windows Sandbox. Clicking the item for Windows Sandbox will start it with its default configuration.

<img src="{{ 'assets/img/2022-04-02-how-to-enable-windows-sandbox/windows-sandbox.png' | relative_url }}" alt='Windows Sandbox' />

I will discuss how to customize Windows Sandbox in a future article. For now, enjoy experimenting with Windows Sandbox (but don't use it to analyze suspicious programs using the default configuration).

[1] [https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-sandbox/windows-sandbox-overview#prerequisites](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-sandbox/windows-sandbox-overview#prerequisites)

[2] [https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-sandbox/windows-sandbox-overview#installation](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-sandbox/windows-sandbox-overview#installation)

[3] [https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-sandbox/windows-sandbox-overview#installation](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-sandbox/windows-sandbox-overview#installation)
