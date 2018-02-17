---
title: Write-Blog "Dear IIS, please tell my *why* it failed." -UsefulContent $true
date: '2013-05-16T20:13:00.002-04:00'
tags:
- security
- passwords
- iis
modified_time: '2013-05-16T20:13:12.931-04:00'
blogger_id: tag:blogger.com,1999:blog-23185081170317196.post-501679293012065415
blogger_orig_url: http://theevolvingadmin.blogspot.com/2013/05/write-blog-dear-iis-please-tell-my-why.html
excerpt: Running down the cause of an unhelpful IIS error
---

# Write-Blog "Dear IIS, please tell my *why* it failed." -UsefulContent $true

## So I got this error...

After seeing it mentioned in a thread about multi-user password vaults I decided to take a look at the product [PasswordState](http://www.clickstudios.com.au/passwordstate.html). The capabilities looked like a good fit what the IS department at $Work was looking for; it's a web based app and it's an ASP.NET app that runs on IIS and works with MSSQL. These counted on the plus side of the ledger for us as we have in house expertise with that platform.

They offer a free license good for up to 5 users (with no expiration) and larger licenses are under $30 a person. And to make it even better, they have excellent documentation for installing and configuring the application.

So why am I talking about it and an error? Well I ran into an issue during the post install configuration.
The install was straight-forward and the configuration was handled by hitting the web page for the app. Everything went fine until the penultimate step; selecting a user for the first SecurityAdmin. It gave me an error message about not being able to query Active Directory to get the list of users.

The message it gave was very straight-forward and pointed me to their documentation on the problem. The issue was that by default new Application Pools in IIS 7+ run as **"ApplicationPoolIdentity"** which is a specialized local user account unique to the application pool. This makes a lot of sense from a "least user" perspective. More information on this account can be found at [IIS.Net](http://www.iis.net/learn/manage/configuring-security/application-pool-identities).

Their documentation correctly and helpfully pointed out that these accounts can't talk to Active Directory, and to resolve this issue the Application Pool should be reconfigured to run as a domain user account. They continued with step by step directions on how to make this change.

Once I made this change, recycled the AppPool and reloaded the web page, I received one of the more non-specific errors I've seen:

***The page cannot be displayed because an internal server error has occurred.***

Well, it's accurate but not at all useful. So now it was time to dig down and find out what's going on...

The first step was getting it to tell me something more meaningful so I find out why it didn't like the new identity account.

**"Internal Server Error"** translates to an **HTTP 500** error, and by default IIS shows very generic messages for these errors to clients. This is because the information provided by detailed error pages would be very useful not just to troubleshooters but to trouble causers (aka attackers).

A quick change in IIS Manager (details [here](http://www.iis.net/configreference/system.webserver/httperrors)) and a refresh of the page showed a much more helpful message:

***Either a required impersonation level was not provided, or the provided impersonation level is invalid. (0x80070542)***

Now that's useful! This is because the account being used as the Application Pool Identity didn't posses the right "impersonate a client after authentication" on the web server.

By default when you install IIS on a server it creates a local group called IIS_IUSRS that is granted this right as well as permissions to the Inetpub directory tree.

Once the domain account was added to this group and the AppPool restarted, the web app loaded properly and the installation completed successfully.

Microsoft has a handy KB article on the default permissions and user rights for IIS7+: [KB981949](http://support.microsoft.com/kb/981949)

This is a pretty common issue with IIS applications (including classic ASP) that require domain accounts but is easy enough to solve. One way to avoid it is to create a domain group for AppPool identities, and that group can be granted the necessary user rights by GPO on the IIS servers and being added to the local IIS_IUSRS groups on them.
