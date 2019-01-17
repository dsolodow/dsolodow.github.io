---
title: Exchange Schema Upgrade
date: 2018-07-17 11:00:00 -04:00
tags:
 - Exchange
categories: Troubleshooting
excerpt: Bringing Exchange schema up to date
classes: wide
---
# What and why

One of the things I discovered at new $WORK was that we have all our mailboxes in Exchange Online, and are using ADFS and directory synchronization.

Pretty common and normal.
The catch?
*No on-premise Exchange server.*

This isn't supported, and prevents you from modifying most of the Exchange properties on your objects. [Reference][supported-link] :frowning:

The first step was to find out what version of Exchange had last been present and get the schema upgraded to Exchange 2016.

I ran the following and then looked up the result on [this site][schemaVersion-link]

```powershell
Get-ADObject "CN=ms-Exch-Schema-Version-Pt,$((Get-ADRootDSE).schemaNamingContext)" -Property RangeUpper
```

This told me the last Exchange versions installed was 2007 SP1.

This would require a few upgrades in succession to get to 2016 as 2007 SP1 and 2016 can't coexist or direct upgrade.

## The plan

Since we were getting ready to decommission our last Windows 2008 DC, I decided to use it for the upgrade.

The plan was:

1. Make the 2008 DC the schema master
2. Add my admin account to *Schema Admins*
3. Disable outbound replication from the 2008 DC
4. Apply Exchange 2010 schema upgrade and domain prep
5. Apply Exchange 2013 schema upgrade and domain prep
6. Verify things went ok
7. Re-enable outbound replication from 2008 DC

The idea here was that if something went badly with the upgrade, I could kill the 2008 DC and seize the schema master role without hosing the forest.

## The reality

Steps 1..4 went basically as expected.

However I ran into a few snags for the Exchange 2013 schema:

First, Exchange 2013 setup (including /prepareAD) won't run on Server 2008.

So I had to create a temporary pair of 32 bit AD subnets for the 2008 DC and my PC, and put those two subnets with a temporary AD site.

This was necessary because the schema upgrade expects the schema master to be in the same site.

The next snag was the schema upgrade threw an error on the pre-req checks. :frowning:
According to the logs, there was still an Exchange 2003 server in the environment.

This turned out to be due to the Exchange 2000/3 server not having been properly decommissioned upon upgrade to Exchange 2007. Judging by what I saw, it looked like Exchange hadn't uninstalled as there were still objects in the Configuration partition such as the Exchange Server object, Recipient Update Service, etc.

Once these were removed via ADSIEdit, the Exchange 2013 schema upgrade completed without issue. :thumbsup:

## Next steps

The Exchange 2016 schema upgrade required a Windows 2008 R2 domain and forest functional levels, so I wasn't able to apply that schema upgrade yet. So it was necessary to:

1. Re-enable outbound replication from the 2008 domain controller
2. Verify replication completed successfully
3. Transfer Schema Master FSMO role to another DC
4. Demote Windows 2008 domain controller
5. Raise forest and domain functional levels (was able to go to 2012 R2)
6. Run Exchange 2016 schema upgrade on new Schema Master

## Guess what they forgot to do?

The Exchange 2016 schema upgrade failed saying there was still an Exchange 2007 server in the environment.
A bit of checking in ADSIEdit showed the Exchange Server object for the Exchange 2007 server. Once this was removed, the Exchange 2016 schema upgrade completed successfully.

With all this done, it was now possible to actually (*finally*) install Exchange Server 2016.

Fortunately that went without hiccup and we were in business. :smile:

[schemaVersion-link]:https://eightwone.com/references/schema-versions/
[supported-link]:https://blogs.technet.microsoft.com/exchange/2012/12/05/decommissioning-your-exchange-2010-servers-in-a-hybrid-deployment/
