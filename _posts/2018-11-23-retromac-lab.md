---
layout: post
title:  "RetroMac"
sub_title: "Mac Pro Powered Home Lab"
image:  /assets/images/retromac/RetroMacSetup.png
date:   2018-11-23 12:00:00 -0600
categories: macpro vmware
---

Apple Mac Pro Cheese Graters make for a pretty sweet lego like substrate on which to run a Home Lab.  In this post we'll tease apart the ole Mac Pro and highlight its use as a Development and Virtualization Lab machine.

For those that don't know, Apple use to make expandable workstation/server class machines, completely opposite of the Apple boxes of today.  From 2009 to 2012 Apple sold Mac Pro Cheese Graters of various configurations and many are floating around for sale on Facebook Marketplace, Craigslist and Ebay.  The good thing about Mac Pros from this era is they can be modernized and deployed various ways in a Home Lab environment.  Let's dive in...

### What makes Mac Pro Cheese Graters relevant today?

1. Server Class chips that go zoom...Dual Intel Xeon CPUs (for example [3.46 GHz Intel Xeon X5690s](https://ark.intel.com/content/www/us/en/ark/products/52576/intel-xeon-processor-x5690-12m-cache-3-46-ghz-6-40-gt-s-intel-qpi.html))
1. Max out at 128 GB 1333 MHz Memory, that's a lot of headroom for Data, VMs and Containers!
1. Internal Expansion! 4 PCIe 2.0 slots (2x16 and 2x4), go NVMe, RAID controller, 10 GB nic?
1. Low cost when compared to latest and greatest Apple machines

### What makes the Cheese Graters smell like stinky cheese?

1. The internal SATA II/3GBs storage bus, native storage is glacial (PCIe expansion helps here)
1. macOS limited to Mojave as of Fall 2019, going beyond this is unsupported by Apple.
1. Bringing it current, which requires paving bits in the right order.
1. Power Consumption is 150-175W under nominal load (Sleeping is always an option tho)

## The Mac Pro

The Mac Pro 2009-2012 came in 2 basic configurations; [single-core](https://everymac.com/systems/apple/mac_pro/specs/mac-pro-six-core-3.33-mid-2010-westmere-specs.html) and [dual-core](https://everymac.com/systems/apple/mac_pro/specs/mac-pro-eight-core-2.4-mid-2010-westmere-specs.html).  All the Mac Pros used in my **RetroMac Lab** and one's I modernize for resale are dual-core.  The official 2009 models is a **4,1** and 2010 & 2012 models are **5,1**.  Other than the model designation and Firmware code the 2009-2010 are identical and thus why it's easy to [upgrade the Firmware](https://www.ifixit.com/Guide/How+to+Upgrade+the+Firmware+of+a+2009+Mac+Pro+41/98985) of a **4,1** to **5,1** which is required to unleash hardware goodness (NVMe support) and macOS Mojave.

**What's the net of this?**

That any 2009-2012 Mac Pro can be brought up to the latest Apple Firmware which allows for Mojave installation as well as other benefits such as booting from a PCIe attached NVMe drive.

**Basically...you get a beast of a Dev Box at a fraction of the cost of a new Apple machine.**

## Quick Compare to iMac Pro 2017

| Mac Pro (Mid 2010) | iMacPro 2017 |
| ------------------ | ------------ |
| ![RetroMac Setup](/assets/images/retromac/RetroMacSetup.png) | ![RetroMac Setup](/assets/images/retromac/iMacProSetup.png) |
| macOS Mojave 10.14.6 | macOS Mojave 10.14.6 |
| Boot ROM 144.0.0.0.0 | Boot ROM 1037.40.124.0.0 |
| MacPro5,1 | iMacPro1,1 |
| 2 x 3.46 GHz 6-core Intel Xeon X5690 | 3.2 GHz 8-core Intel Xeon W |
| 128 GB 1333 MHz DDR3 | 32 GB 2666 MHz DDR4 |
| 1 TB NVMe startup disk | 1 TB NVMe startup disk |
| Sapphire Pulse Radeon RX 580 8 GB | Radeon Pro Vega 56 8 GB |
| GB4 2900/24000 | GB4 5300/34000 |
| GB4 GPU Metal 134000 | GB4 GPU Metal 135000 |
| GB4 GPU OpenCL 131000 | GB4 GPU OpenCL 140000 |
| AJA Disk Test 1400 MB/s read | AJA Disk Test 3000 MB/s read |
| AJA Disk Test 1500 MB/s write | AJA Disk Test 2600 MB/s write |
| $1500 unit + $300 Display | $4200 unit |

## Modernization

Cheese Graters have great bones but there are things that must be improved to keep pace with current Apple machines.  It starts with paving the right firmware bits and in the end requires a supported Metal GPU.  Each modernization will differ but generally these are the basic steps needed to get a unit ready for Mojave.

### Basic Mojave upgrade steps

* Add a boot NVMe drive to a [PCIe adapter](http://www.lycom.com.tw/DT-120.htm) and plug into slot 2-4.
* Add at least 16 GB of DDR3 Memory
* Add an original GPU such as the GT 120 or Radeon 5770
* Make macOS High Sierra USB [bootable installer](https://support.apple.com/en-us/HT201372)
* Stick High Sierra bootable installer into USB port
* Boot up holding Option key to select High Sierra bootable installer as Startup disk
* High Sierra installer will prompt for Firmware update, follow instructions to update Firmware to MP51.0089.B00, which aligns with High Sierra 10.13.6
* After the Firmware install, close the installer and shutdown
* Remove the High Sierra bootable installer from USB port
* Open the Mac Pro, remove original GPU, add [Metal supported GPU](https://support.apple.com/en-us/HT208898) such as Radeon RX580
* Make macOS Mojave USB [bootable installer](https://support.apple.com/en-us/HT201372)
* Stick Mojave bootable installer into USB port
* Boot up, this make take a bit since it's reading Mojave installer from USB
* The Mojave installer will prompt for a Firmware update, follow instructions to update Firmware to 144.0.0.0.0, which aligns with Mojave 10.14.6
* After the Firmware install, continue Mojave installation and install osx onto the NVMe drive
* After Mojave installation completes, reboot and have fun :)


## References

* [Mac Pro 2009](https://support.apple.com/kb/sp506?locale=en_US)
* [Mac Pro 2009 Technician Guide](http://tim.id.au/laptops/apple/macpro/macpro_early2009.pdf)
* [Mac Pro 2010](https://support.apple.com/kb/sp589?locale=en_US)
* [Mac Pro 2010 Technician Guide](http://tim.id.au/laptops/apple/macpro/macpro_mid2010.pdf)
* [Apple Mac Pro 2012](https://support.apple.com/kb/sp652?locale=en_US)
* [Everymac Single Core Specs](https://everymac.com/systems/apple/mac_pro/specs/mac-pro-six-core-3.33-mid-2010-westmere-specs.html)
* [Everymac Dual Core Specs](https://everymac.com/systems/apple/mac_pro/specs/mac-pro-eight-core-2.4-mid-2010-westmere-specs.html)
* [Lycom DT-120 PCIe NVMe adapter](http://www.lycom.com.tw/DT-120.htm)
* [Create a Bootable Installer](https://support.apple.com/en-us/HT201372)
* [Metal Compatible GPUs](https://support.apple.com/en-us/HT208898)
* [Vice Mac Pro Article](https://www.vice.com/en_us/article/8xkq8k/mac-pro-upgrade-community)
* [Apple Insider Cheese Grater Article](https://appleinsider.com/articles/18/08/07/apples-mac-pro-cheese-grater-is-12-years-old-and-is-the-best-mac-ever-made)
* [Greg Gant's Excellent Mac Pro Upgrade Guide](http://blog.greggant.com/posts/2018/05/07/definitive-mac-pro-upgrade-guide.html)
* [iFixit Mac Pro Firmware Upgrade Guide](https://www.ifixit.com/Guide/How+to+Upgrade+the+Firmware+of+a+2009+Mac+Pro+41/98985)
