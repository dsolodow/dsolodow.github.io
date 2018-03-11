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
* An iApp in the F5 LTM cluster for ADFS
* An iApp in the F5 LTM cluster for ADFS proxy
