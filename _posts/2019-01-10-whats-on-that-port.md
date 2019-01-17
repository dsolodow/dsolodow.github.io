---
title: What's on that port?
date: 2019-01-10 17:25:00 -05:00
tags:
 - Security
 - PowerShell
excerpt: Why are these PCs listening on port 80?
classes: wide
---
# What's on that port?

Today, the network admin at $WORK contacted me, saying while running a ZenMap on part of one of the offices, he found several Windows PCs in the AP and Payroll departments that were listening on 80/tcp. This gave him the willies, so he reached out to me to find out WTF. :anguished:

Once the IPs of the PCs were converted to names, it was time to investigate.

First, let's make sure they're still online and listening as expected:

```powershell
$PCs = 'PC1','PC2','PC3'
$PCS | Foreach-object {Test-NetConnection -ComputerName $_ -CommonPort HTTP}
```

(Had to use the Foreach as Test-NetConnection doesn't allow an array for ComputerName) :persevere:

This showed that yes, all three were online, reachable and accepting connections for inbound HTTP traffic.
Now let's see what process is listening on the port:

```powershell
Invoke-Command -ComputerName $PCs -ScriptBlock {netstat -ano | findstr :80}
```

![Table of TCP ports, status and PID]({{site.url}}/assets/images/whats-on-that-port/process.jpg)

On each computer, the process listening on port 80 had PID 4 (far right column). On a Windows machine, PID 4 is SYSTEM, aka the kernel.
This meant that it wasn't a user process listening on port 80, so the odds of it being malware or the like went down __*dramatically*__. :relieved:
It *also* meant that it was almost definitely from the Windows built-in, kernel level HTTP listener (http.sys)

Maybe they had something like IIS, MSMQ, etc. installed?

```powershell
Invoke-Command -ComputerName $pcs -Credential $admin -ScriptBlock {
    Get-WindowsOptionalFeature -Online |
        Where-Object {($_.FeatureName -like '*web*' -or $_.FeatureName -like '*http*') -and $_.state -eq "Enabled"}
}
```

Nope, no hits there. :frowning:

A bit of Googling for 'http.sys show http listener' revealed the handy command **netsh http show urlacl**, so...

```powershell
Invoke-Command -ComputerName $pcs -ScriptBlock {netsh http show urlacl | findstr :80}
```

![Image of URL strings]({{site.url}}/assets/images/whats-on-that-port/listener.jpg)

Those look like GUIDs. To the Google!

And one of the first hits was: [How can I determine why the System process is listening on port 80?](https://blogs.msdn.microsoft.com/oldnewthing/20180703-00/?p=99145)

This indicated BranchCache, which is harmless but maybe undesirable now.
