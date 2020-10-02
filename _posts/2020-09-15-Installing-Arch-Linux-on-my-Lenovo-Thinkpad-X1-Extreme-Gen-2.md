---
title: "Installing Arch Linux on my Lenovo Thinkpad X1 Extreme Gen 2 (In Progress)"
date: 2020-09-15T15:34:30-04:00
toc: true
toc_sticky: true
---

Recently I switched from my Microsoft Surface Book 2 to a Thinkpad X1 Extreme Gen 2, in no small part due to the former's compatibility issues with Linux. While a [custom kernel](https://github.com/linux-surface/linux-surface) does exist to make Linux work with Surface devices, the resulting setup remained too unstable for me to be comfortable using it as my daily driver, so by popular recommendation I'm making the switch to a Thinkpad. 

Having run Fedora and Ubuntu in the past and subsequently improved my Linux skills by a wide margin, I'm looking at Arch as my next OS to try. I like the idea of building my OS from the ground up, and the Arch Wiki seems to be a fantastic resource to draw from, even featuring a [dedicated page](https://wiki.archlinux.org/index.php/Lenovo_ThinkPad_X1_Extreme_(Gen_2)) for the X1 Extreme Gen 2. This post serves to document my process of learning both how to install and configure Arch Linux with a desktop environment (I'll be using GNOME) and making it compatibile with all of my hardware, which according to the Wiki should be possible for every part of this laptop, including the fingerprint reader and Nvidia GeForce® GTX 1650.

## Pre-Installation Setup

### Updating Firmware

Since I am going to be dual-booting Windows along with Arch, there are a couple preparation steps I will be doing on Windows: first, based on advice from the Arch Wiki page, update my Thinkpad's BIOS to the latest version (1.34 at the time of writing) using Levovo's built-in Vantage software. This can be done on Linux as well (see the Arch wiki page for this device), but I personally think it's much easier to do on Windows, and it's highly recommended to have an up-to-date BIOS before going through the Arch installation process. Even if you aren't planning to dual-boot, I would still recommend doing this step before you get rid of Windows.

### Bootloader

Once you've updated the BIOS and other firmware, during which time the laptop will likely restart a few times, the last step is to prepare the _bootloader_. This is what points the laptop to Arch on startup and will let you choose between Arch and Windows each time you boot up the machine. There are a lot of bootloaders out there, but I've decided to use rEFInd, which I liked because it seems easier to install, uninstall, and customize alongside a Windows dual boot than its more popular counterpart, Grub.

I'll be following [this](https://www.rodsbooks.com/refind/installing.html#windows) guide from the official rEFInd website, but as I added a few extra steps to suit my needs, I'll be outlining the whole process here. First, we'll need to download [rEFInd]() itself, [shim](http://www.codon.org.uk/~mjg59/shim-signed/) for letting rEFInd work with Secure Boot, and finally an optional [theme](https://github.com/andersfischernielsen/rEFInd-minimal-black/) to give rEFInd a sleeker look. Save/extract all these folders to your desktop. 

To prep the installation for install, locate the "refind" folder in the folder for the initial rEFInd download and move this to the desktop. This is the folder that contains the actual rEFInd bootloader. From the same directory you found that folder in, move the refind.cer file into "refind". Now open the refind folder and create a new folder called "themes". Move the theme you downloaded into refind/themes (if you downloaded the theme from Github, you have to rename that folder to remove the "-master" that will be appended to the end of the folder name). Next, from within Windows, open the command prompt by searching for "cmd" from the start menu and click "Run as Administrator". Since Arch can use the same EFI directory as Windows (i.e., the partition on the hard drive where the bootloader lives), we can install rEFInd to the existing EFI partition that will already be on the Windows machine. To access this partition, first run 
```bash
mountvol S: /S
```
Then switch to your Desktop (or wherever directory you put rEFInd):
```bash
cd \path\to\Desktop
```

## Installing Arch


I'll start by following the general [Arch install guide](https://wiki.archlinux.org/index.php/Installation_guide) from the Arch wiki.wifi setup, gdisk setup, refind (windows way)

## Post-Installation Steps

### Signing kernel for Secure Boot

## Troubleshooting
### GPU Lost on Suspend or Lock
Run 
```bash
prime-offload
optimus --switch <graphics mode>
```