# dualboot-dellxps
Notes on dual-booting the Dell XPS 13 (model 9343) laptop with Windows 10 and Ubuntu 16.04. This can actually be pretty easy (install Ubuntu alongside the Windows Boot Manager), but you end up with GRUB as your bootloader. The nice folks at Microsoft put all that effort into the 'tiny squares' design language, so why not use it? Also I'm stubborn as hell and prefer to do things the hard way.

So instead of using GRUB as the bootloader, I want to use the Windows Boot Manager. Make it so. (Besides being totally unnecessary, an aside on why this is likely an actively bad idea can be found [here](http://superuser.com/questions/499617/how-can-i-add-linux-to-the-new-windows-8-boot-manager).)

## Intro to booting
Let's start with some background on the UEFI boot process, and the UEFI boot process is bananas. I'm certainly no expert, but here's how I understand it (largely distilled from [this *excellent* post](https://blog.uncooperative.org/blog/2014/02/06/the-efi-system-partition/), which also refers to [*this excellent* post](https://www.happyassassin.net/2014/01/25/uefi-boot-how-does-that-actually-work-then/)).

The EFI System Partition is a FAT-formatted partition with a particular GPT partition type. It isn't explicitly named by a GPT GUID/label, and for good reason. This setup allows the preceding boot process (I have no idea what that process actually is; we'll assume it's magic and refer to it as the PBP) to look around and scan anything that *meets the ESP criteria* so it can use *that* as a potential ESP.

This is mostly irrelevant in this case because the EFI System Partition is already installed by Windows, and it's set as the first boot option, so there's no searching around for an appropriate ESP.

### The boot process, stepwise
So let's lay out the UEFI boot process in some detail.

1. The PBP goes through the boot options in order. By "boot options" I'm referring to a list of variables that look something like this: `ACPI(a0341d0,0)PCI(1f,2)SATA(0,0,0)HD(1,800,64000,12029cda-8961-470d-82ba-aeb17dba91a5)File(\EFI\fedora\shim.efi)`.  The `ACPI`, `PCI`, and `SATA` bits are things we don't have to care about, just references to hardware to initialize. We care about the `HD` and `File` pieces.
     1. `HD`: lists the
          * partition number - 1
          * partition offset - 800
          * partition size - 64000
          * partition GUID
               * NOTE: This `HD` partition doesn't have to be an "EFI System Partition" explicitly, it just has to be FAT-formatted.
     1. `File`: points to the specific file on the `HD` partition that will control the next phase of the boot process, so this is the file that we want to load up and pass control to.
1. As the PBP iterates over the boot options, it takes these steps:
     1. Initialize the hardware listed (if it's not already initialized)
     1. Search the disk for the `HD` partition
          1. If it finds `HD`, and `HD` is FAT-formatted, it looks for `File`
          1. If it doesn't find `HD`, it moves to the next boot option
1. If nothing in the boot options list works, the PBP spins up every fucking piece of hardware it can find, identifies the removable media stuff, and scans every device for an ESP that is FAT-formatted.
     1. If it finds an ESP, it looks for `\EFI\BOOT\BOOTX64.EFI` (or the appropriate file for the system)
1. If none of the removable media devices can boot, then the PBP resorts back to fixed media devices. Everything is pretty much already initialized, so the PBP just marches through these devices looking for an ESP, and then for `\EFI\BOOT\BOOTX64.efi`.

## So what about the XPS?
Dual-booting the XPS will consist of 3 main steps:

1. Install/tweak Windows
1. Install Ubuntu
1. Tweak UEFI boot settings

Gameplan: Installing Windows 10 is self explanatory/already done, but you'll need to make some adjustments. Disable "Secure Boot" in firmware (Ubuntu 16.04 won't work with Secure Boot enabled. 16.10 will, and 16.04.2--due out early 2017--will as well). Turn off "Fastboot" (maybe? look into hibernation issues).

* fastboot info [here](http://askubuntu.com/questions/452071/why-disable-fast-boot-on-windows-8-when-having-dual-booting) and [here](http://superuser.com/questions/211079/what-do-i-have-to-take-care-of-when-hibernating-both-ubuntu-and-windows-dual-bo).

**UPDATE**: fastboot is a UEFI setting, don't need to do anything there. You **should** disable fast startup under Control Panel > Power > Change power buttons. This is because Fast Startup uses a kind of hibernation that will maintain and freeze write ownership of the drive, and this won't play nice with another OS.

Shrink Windows partition (C:).

Then install Ubuntu. Create live USB. Install. Swap? Shared partition? (Hibernation and fastboot?) Where does `/boot` go, separate partition?


[add Ubuntu to Win10 boot manager](http://askubuntu.com/questions/690648/how-to-add-ubuntu-to-windows-10-boot-manager-but-it-will-be-in-another-hard-dri)

Add boot option to UEFI for Ubuntu.

Remove GRUB boot option for Windows?

## Windows 10 installation
Standard Windows 10 Home installations that comes with the laptop. But you have to shrink the 'C:' partition with Disk Management first.

The hard drive is GPT/UEFI (as opposed to MBR/BIOS). Windows creates 2 partitions before the OS: one ~450 MB partition for recovery; one ~100 MB EFI System Partition, or ESP. The `C:` partition serves as the partition for:

* boot
* page file
* crash dump
* os

<Add image of disk management before linux install here>
