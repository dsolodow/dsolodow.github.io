---
layout: single
title: New Work PC
excerpt: New job, new PC
tags: Off-Topic
classes: wide
---

One of the perks of a new job was a new computer. :smile:
My boss told me they're a Dell shop currently, and turned me loose on what I wanted to order.

I wanted a powerful laptop (I tend to run local VMs for testing), and prefer a 15" screen. It turned out that they already used the [Precision 5520][5520-link] for a few cases, like CAD and other design work. So after taking a look at the information for that model and being *really* impressed, I pulled up the info to customize one.

There are a lot of reviews and info on this model already, but the highlights are:

1. 15" LCD; either 1080 standard or 4k touch
2. Choice of i7 7th Generation or Xeon E3-1505
3. Up to 32 GB RAM
4. M.2 PCIe SSD
5. Height: 0.44" (11.1mm), Width: 14.05" (357mm), Depth: 9.26" (235.3mm), Weight: 3.93lbs (1.78kg)

I also ordered the Thunderbolt Dock [(TB16)][tb16-link] to go with it to hook up the pair of 24" 1080 monitors, as well as gigabit Ethernet, etc.

I like that the dock has a pair of USB3 ports on the front as well as a 3.5mm headphone jack, along with a nice complement of ports on the back.
It is rather annoying that while it has multiple display outputs on the back, they're all different kinds.

---
While I was waiting for the new PC to arrive, I made a customized Windows 10 USB stick to install from. This was both as a proof of a concept and to save me some work for the actual install. :smile:

Here's how I did it:

1. Download the Windows 10 1709 x64 ISO from the VLSC
2. Extract the ISO
3. Open the extract folder in [NTLite][ntlite-link] and mount install.wim
4. Remove the images from the wim except for Enterprise
5. Modify the installed Windows components (added WSL, Hyper-V, IIS Manager, etc.)
6. Pre-configured a few preferences and settings
7. Apply the current Windows 10 cumulative update and Adobe Flash update
8. Add the drivers for the model from the Dell site
9. Apply the changes to the WIM and save
10. Save to ISO (button in NTLite)
11. Write the ISO to USB for a UEFI install (shout-out to [Rufus])

After the hardware arrived, I made sure the BIOS was set for UEFI and the various virtualization and TPM settings were enabled.
Then booted the USB stick and did the install.
Turned out I missed a few drivers (some for the dock) and needed a firmware update or two as well.
Fortunately, Dell Command is an excellent and handy tool for dealing with those.

Then just had to load up a few extra things like RSAT, IIS Remote Manager, etc.

Will see how things go after time, but so far I've been pretty happy with the PC. :thumbsup:

[5520-link]: http://www.dell.com/en-us/work/shop/cty/pdp/spd/precision-15-5520-laptop#features
[tb16-link]:http://www.dell.com/en-us/shop/dell-business-thunderbolt-dock-tb16-with-240w-adapter/apd/452-bcnu/pc-accessories#overview
[ntlite-link]:https://www.ntlite.com/
[rufus-link]:https://rufus.akeo.ie/
