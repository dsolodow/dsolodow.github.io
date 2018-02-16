---
title: PowerShell Profile Tips & Tricks
date: '2014-10-21T11:52:00.001-04:00'
tags:
- PowerShell
categories: Conceptual
modified_time: '2014-10-21T12:05:46.502-04:00'
blogger_id: tag:blogger.com,1999:blog-23185081170317196.post-8761384466274846516
blogger_orig_url: http://theevolvingadmin.blogspot.com/2014/10/powershell-profile-tips-tricks.html
excerpt: When, why and how to use a PowerShell profile
classes: wide
---

# What it is

A PowerShell profile is just a normal PowerShell script with a specific name and location. This means it's subject to your current ExecutionPolicy, so you'll either need to sign your profile or use RemoteSigned.
It's a similar concept as say autoexec.bat or .bashrc; when you launch the PowerShell console, or open a new tab in the ISE, the host process looks for and executes the applicable profile.
There are multiple possible profiles, and you **can** have several of them, although I prefer (and suggest) not doing so unless you have an overriding need as it makes it more complicated.
The Scripting Guy has an excellent [post](https://blogs.technet.microsoft.com/heyscriptingguy/2013/01/04/understanding-and-using-powershell-profiles/) that explains in excellent detail, but here's the short version:

* There is a **CurrentUser** set of profiles which live in your Documents\\WindowsPowerShell folder
* There is a **AllUsers** set of profiles which live in $PSHOME
* Each set can have an **AllHosts** profile, as well as a host specific profile for each host (generally console and ISE)
* If you have multiple profiles, the most specific one runs last (and thus "wins")
* None of the profiles exist by default, they have to be created.
* The profile scripts do **not** get invoked automatically by remote session.

There is a built-in variable ($profile) that makes life easier. Here's a command to show you where the profiles would live:

```powershell
$profile | Get-Member –MemberType NoteProperty
```

# What it's for

Here's the real meat of the story. :smile:

Since the profiles are standard PowerShell scripts, anything you can do in a PowerShell script can be in your profile. Just remember though, that the profile scripts run **every** time you launch PowerShell, so you don't want them to take a long time to run.

You also do **not** want them to require any user input as PowerShell won't finish loading until that input is provided. *This is especially painful if you have PowerShell logon/logoff or start/shutdown script defined in your GPOs.*
Anything you find yourself running/typing **every** time you launch the console or ISE should probably go in your profile.

## The most common and useful things to put in your profile are

* Functions (help [about_Functions](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_functions))
* Aliases (help [about_Aliases](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_aliases))
* PSDrives (help [About_Providers](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_providers))
* PSDefaultParameterValues (help [about\_Parameters\_Default_Values](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_parameters_default_values))

My profile started out originally because I didn't want to have to launch each of the different, separate, PowerShell "consoles" that each snap-in or tool seemed to create. After all, why go launch [PowerCLI](http://www.vmware.com/go/PowerCLI) or Exchange Management Shell when I can just create a function in my profile that adds those things to my current session?

Another very useful case are modules/snap-ins that have to be run from a server with a specific role such as XenApp or SharePoint. The SharePoint PowerShell snap-in has to be run from a SharePoint server that is part of the farm you're working with.

So rather than have to create a new session each time you want to access it (or worse RDP to the server to run PowerShell!) I've created a function that uses [_implicit remoting_](http://blogs.technet.com/b/heyscriptingguy/archive/2013/09/08/remoting-the-implicit-way.aspx) to import the snap-in.

I've posted a generalized and heavily commented version of my own profile script on GitHub: [Profile.ps1](https://github.com/dsolodow/IndyPoSH/blob/master/Profile.ps1)

Feel free to read through it, post questions or comments in the Issues or Discussion pages for the project. Also feel free to borrow snippets or sections out of it that you want to use for yourself.

## Other ideas

The Scripting Guy has a series of posts about "What's in your profile" with information from various PFEs, PowerShell MVPs, members of the PowerShell team, etc. Links below.

* [What's in your profile (PFEs)](https://blogs.technet.microsoft.com/heyscriptingguy/2014/05/23/whats-in-your-powershell-profile-microsoft-pfes-favorites/)
* [What's in your profile (PowerShell team)](https://blogs.technet.microsoft.com/heyscriptingguy/2014/05/24/whats-in-your-powershell-profile-powershell-team-favorites/)
* [What's in your profile (MVPs)](https://blogs.technet.microsoft.com/heyscriptingguy/2014/05/22/whats-in-your-powershell-profile-powershell-mvps-favorites/)
* [What's in your profile (User favorites Part 1)](https://blogs.technet.microsoft.com/heyscriptingguy/2014/05/20/whats-in-your-powershell-profile-here-are-some-users-favorites/)
* [What's in your profile (User favorites Part 2)](https://blogs.technet.microsoft.com/heyscriptingguy/2014/05/21/whats-in-your-powershell-profile-users-favorites-part-2/)
* [What's in your profile (Scripting Guy)](https://blogs.technet.microsoft.com/heyscriptingguy/2014/05/19/whats-in-your-powershell-profile-here-are-a-few-of-my-favorites/)

## Rules, best practices, no-nos

There aren't a lot of hard, fast rules but here are the ones I know of as well as some good practices/suggestions:

* Don't make your profiles require user input
* Be careful about system/version dependent features/cmdlets
* Make sure you either sign your script or run PowerShell with the **RemoteSigned** executionPolicy
* Don't make your profile script something that takes a long time to run as you'll run it a lot
* _It's a PowerShell script, treat it accordingly. This means comment it, format it appropriately, put it in version control, etc._
  [http://windowsitpro.com/blog/my-12-powershell-best-practices](http://windowsitpro.com/blog/my-12-powershell-best-practices)
* Do share cool tricks or ideas with your co-workers, the PowerShell community, blog readers, etc.
* Let your profile grow organically; when in doubt [K.I.S.S.](https://en.wikipedia.org/wiki/KISS_principle)
* Remember: help [about_profiles](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_profiles)
