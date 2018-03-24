---
title: "Upgrading ADFS to 2016"
date: 2018-03-11 10:54:00 -04:00
toc: yes
tags:
 - ADFS 
 - Procedural
excerpt: Upgrade ADFS 2012R2 to 2016
---

## Starting point

We started with the following setup:

* (2) Windows Server 2012R2 ADFS servers using WID
* (2) Windows Server 2012R2 WAP servers acting as ADFS proxys in the DMZ
* the servers above running as VMs on a vSphere 6.x cluster
* An iApp in the F5 LTM cluster for ADFS
* An iApp in the F5 LTM cluster for ADFS proxy

### Why upgrade?

1. Lots of new features in ADFS 2016 [Whats New][adfs2016-link]
2. New features in Web Application Proxy 2016 [Whats New][wap2016-link]
3. Strongly recommended for defense against password spray: [Article][article-link]




[adfs2016-link]: https://docs.microsoft.com/en-us/windows-server/identity/ad-fs/overview/whats-new-active-directory-federation-services-windows-server
[wap2016-link]:https://docs.microsoft.com/en-us/windows-server/remote/remote-access/web-application-proxy/web-application-proxy-windows-server
[article-link]:https://cloudblogs.microsoft.com/enterprisemobility/2018/03/05/azure-ad-and-adfs-best-practices-defending-against-password-spray-attacks/

