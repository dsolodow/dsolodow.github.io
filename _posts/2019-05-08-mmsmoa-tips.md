---
layout: single
title: MMSMOA Tips and Tricks
excerpt: Tip (and a bit extra) shared at MMSMOA
tags:
- Conventions
- Off-Topic
- MMS
- VSCode
Categories: Tips & Tricks
---

## What's this about?

The ["Tips & Tricks Showcase"][session-link] session seemed like a must attend event (especially as it was the only one in the time slot :wink:)

And when they said there were some awesome Surface Laptops to giveaway, I figured, may as well get up and share something.

A big thanks to the community for being some encouraging and welcoming to a first time attendee and presenter (and for the extra 10 seconds of speaking time) :thumbsup:

## Enough buttering, get to the meat!

The tip that I shared was around the PowerShell Module [*EditorCommandServicesSuite*][module-link]. Just install it from the gallery with

```powershell
Install-Module EditorCommandServicesSuite -Scope CurrentUser
```

You'll also need to add the following to your VSCode PowerShell Profile so it auto-loads:

```powershell
Import-Module EditorServicesCommandSuite
Import-EditorCommand -Module EditorServicesCommandSuite
```

Once it's installed and loaded, you can activate by either:

- Selecting *PowerShell: Show Additional Commands from PowerShell Modules*
- pressing Shift-Alt-S while editing a PowerShell file

This provides a number of very useful capabilities, but my current favorite is **Convert Command to Splat Expression**
[Splatting][splatting-link] is a very handy way to make long PowerShell commands more readable, and more easily re-usable.

Unfortunately it has two drawbacks:

1. VSCode PowerShell auto-complete/intellisense doesn't help you with parameter names/values when splatting
2. I usually forget something in the syntax/formatting.

This function from the **EditorServicesCommandSuite** module solves both those problems; just write out the command as one long line, and then let it convert it to a splat for you.

## Didn't you also say something about PowerShell Profiles?

Yep! I didn't have time to go into it (didn't want the bear to growl at me!), but fortunately it's something I've previously written about: [PowerShell Profile Tips and Tricks]({% post_url 2014-10-21-powershell-profile-tips-tricks %})

That post is a bit old, but aside from not mentioning VSCode, it all still applies to Windows PowerShell, and almost all should apply to PowerShell Core.

I did however update the sample profile linked in it (and [here][profile-link] as well) to be more current.

Hopefully this has been useful, and if there is more you'd like me to write about, just click the **Suggest Topic** link at the top of the page, and tell me about it!

[session-link]:https://mms2019.sched.com/event/N6gG/tips-tricks-showcase
[module-link]:https://www.powershellgallery.com/packages/EditorServicesCommandSuite
[splatting-link]:https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_splatting?view=powershell-6
[profile-link]:https://github.com/dsolodow/IndyPoSH/blob/master/Profile.ps1