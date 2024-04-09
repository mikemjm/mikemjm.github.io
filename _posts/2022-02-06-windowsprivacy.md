---
title: Windows Phoning Home?
date: 2022-02-06 12:00:00 -500
categories: [security,networking,privacy]
tags: [security,networking,privacy]
---

![featured_image](/assets/windowsprivacy/pssst.png)

### How often does Windows actually call home, and what can we do about it?

Unless you’ve been living as a hermit, you know about how privacy is now one of the main concerns people have with their internet connected devices. Our phones, computers, and other household smart devices are continuously phoning home, providing usage statistics, diagnostic information, etc. Some of us expect this, most of us are oblivious, and some are paranoid and lock their devices down. You can call me paranoid.

Microsoft has been the focus recently about how intrusive Windows 10 can be. It comes pre-packaged with apps that many don’t want, it forces unexpected updates, and it captures everything you type or say. They even admitted it!

***“When you interact with your Windows device by speaking, [handwriting], or typing, Microsoft collects speech, inking, and typing information – including information about your Calendar and People [contacts]…”***

![installation_face](/assets/windowsprivacy/10_installation_face.png)

#### The Experiment:

Wanting to see for myself how often Windows 10 phones home, I decided to conduct an experiment. I downloaded the latest installer from Microsoft and installed it inside a virtual machine. I setup the network adapter to send DNS requests to my Raspberry Pi’s DNS server to be captured.

![dns_settings](/assets/windowsprivacy/dns_settings.png)

Next I cleared the DNS logs on my Raspberry Pi and let the virtual machine sit idle for 30 minutes. Below is a fraction of the captured requests.

![windows_dns](/assets/windowsprivacy/windows_dns.png)

I did this same test again, this time with my main PC which is running Arch Linux loaded with hundreds of applications. The only requests being made were to check whether or not my PC had an internet connection.

![linux_dns](/assets/windowsprivacy/linux_dns.png)

It’s clear to me that Windows is a lot more chatty than my Linux PC. So what can Windows users do to help minimize unnecessary communication? There are a few tools available that automatically remove applications and tweak settings to limit the amount of data collection. This particular tool seems to be the most popular, O&O ShutUp10++. However, this tool is free and closed source. Be wary of programs like these, because “free” usually means you are the product. The creators make money by collecting information from you, such as usage statistics, and host device information. They then sell this information to 3rd party advertising agencies.