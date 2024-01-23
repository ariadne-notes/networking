# Windows 10 P2V - Physical to Virtual

## My Setup

I am adding a compute node to an existing proxmox cluster.

I bought a used i7 Windows 10 machine with a 512 GB NVMe drive. On the outside are two [COA](https://en.wikipedia.org/wiki/Certificate_of_authenticity#COAs_for_software) stickers, one for Windows 10 Pro, and another for H&S Office 2019.

The current OS boots and the copy of Office works.

**Goal:** I want to keep this install of Windows 10 working, and copy the OS into Proxmox. I want to virtualize this OS.

This will give me a working licensed copy of Office.

## Theory

I just need to get the "data" onto the [VM.](https://en.wikipedia.org/wiki/Virtual_machine)

1. The physical machine and the VM need to be able to ping each other.
1. Installing the drivers ahead of time should make the OS bootable.
1. Copying the data should preserve the OS and applications.
1. Copying the partitions should make recovery easier.
1. Rebuilding the boot information should make the OS bootable.

A lot of this is to enable a clean "recovery" of the OS once it's copied over. My copy of Windows 10 relies on:

* [FAT32](https://en.wikipedia.org/wiki/File_Allocation_Table#FAT32)
* [NTFS](https://en.wikipedia.org/wiki/NTFS) - This filesystem should really only be checked using Microsoft's own tools.
* **BCD -** Boot Configuraiton Data
* [GPT](https://en.wikipedia.org/wiki/GUID_Partition_Table)
* [EFI](https://en.wikipedia.org/wiki/EFI_system_partition)
* [MSR](https://en.wikipedia.org/wiki/Microsoft_Reserved_Partition)

## Dataloss
**These tools cause dataloss.**

**A typo will destroy a filesystem.**

Before doing this, practice both making and recovering bare metal restores (BMRs) ... I used [Clonezilla.](https://clonezilla.org/)

BMR is usually *device-to-image*, or *image-to-device.*

[Here](https://clonezilla.org/clonezilla-live-doc.php) are the docs for using Clonezilla.

My Windows 10 BMR is 11GB stored as bzip2.

## If Possible Just Clone the Disk

I wanted to go from a larger drive (512GB) to a smaller drive (64GB). That meant instead of copying the devices, I needed to copy the partitions, *after* resizing them.

*drive-to-drive* cloning would be much easier.

## Download ISOs

Most of the time was spent inside of recovery OSes, working with unmounted filesystems.

[SystemRescue](https://www.system-rescue.org/) - Linux recovery media with NTFS support.

[Windows 10 Installation Media](https://www.microsoft.com/en-us/software-download/windows10) - This is also the recovery disk. It can be made on the host being virtualized. This is **needed** to fix, BCD (Boot Configuariton Data) and EFI problems.

[Clonezilla](https://clonezilla.org/) - A bare metal recovery tool.

## Preparing Windows 10 to be virtualized

My Windows 10 machine had some extras on it I didn't want to virtualize.

1. Create a restore image with Clonezilla

    This is the **failsafe** image, before touching anything. I saved mine to a samba share, but it can be saved anywhere it will fit that isn't on the device.

1. Turn off the hibernation file

   Via the command prompt as an administrator:
   
   `powercfg -h off`
   
1. Clean up the hard disk

   Into the search box type:
   
   `Disk Cleanup`
   
1. Set the virtual memory pagefile to 1024MB

   A file of this size is needed for coredumps, errors, and logging.
   
   Follow these [instructions.](https://www.tenforums.com/tutorials/77692-manage-virtual-memory-pagefile-windows-10-a.html)
   
1. (Optional) Run *WinDirStat* to look for odd or large files

   Delete or Uninstall them.

   [Windows Directory Statistics - WinDirStat](https://windirstat.net/)
   
1. Run `chkdsk` on C:

   Via the command prompt as an administrator:
   
   `chkdsk C: /R`
   
   `/R` - "Locates bad sectors and recovers readable information (implies /F, when /scan not specified)"
   
   Reboot
   
1. (Optional) - Create another restore point with Clonezilla

   This is the **cleaned** image, to save all the clean up work.
   
1. Boot GParted

   This is where it gets dangerous. GParted can be used to resize offline NTFS partitions.
   
1. Resize the "Basic data partition"

   My data partition was 410GiB. I resized it down to 48GiB. The data on the partition is 25GiB.
   
1. Move the "Recovery" partition

   I used the GUI to slide it over.
   
1. Save your work with GParted

   Click the green checkmark. This writes the changes to disk.
   
1. Boot into Windows 10

   Check to make sure the OS is still sane. Does the Internet work?

1. Run `chkdsk` again on C:

   This is done to make sure the filesystem is OK.

   Via the command prompt as an administrator:
   
   `chkdsk C: /R`
   
   `/R` - "Locates bad sectors and recovers readable information (implies /F, when /scan not specified)"
   
   Reboot
   
1. (Optional) - Create another restore point with Clonezilla

   This is the **prepared** image.
   
1. Boot into SystemRescue

   
## Creating the Virtual Machine

I used PVE - [Proxmox Virtual Environment](https://www.proxmox.com/en/proxmox-virtual-environment/overview) as my hypervisor. Any hypervisor should work.

I used the Proxmox GUI to assign the VM a hard disk of 64GB.

I boot the VM with SystemRescue, and make sure it can get a working IP address.

## Preparing the Hard Drive on the Virtual Machine

There are four partitions on my windows 10 machine. I want to copy them over-the-network using [netcat.](https://en.wikipedia.org/wiki/Netcat)

1. *Both* - Boot [SystemRescue](https://www.system-rescue.org/)

1. *Both* - Open GParted

1. *Destination* - Using GParted, recreate the partition structure on the new hard disk

   I used a mix of fdisk and the GUI for this.

   * Created a GPT Partition Table
   * Copied the partitions including the start and stop sectors, **exactly.**
   * Copied the flags
   
   I started with four partitions on both and ended with four partitions. They all fit on this smaller disk.
   
1. *Destination* - Turn off the firewall

   `systemctl stop iptables`
   
1. *Destination* - Get the IP Address

   `ip a`

1. *Destination* - Turn on the small service netcat

   This needs to be done for each partition, one at a time.
   
   `nc -l -p 19000 | bzip2 -d | dd of=/dev/sda1`
   
1. *Source* - Redirect dd into bzip into netcat, throw traffic at the Destination

   This needs to be done for each partition, one at a time.

   `dd bs=16M if=/dev/nvme0n1p1 | bzip2 -c | nc <ip_address> <port>`
   
## Windows 10 Recovery

I went from a NVMe drive to a IDE drive. I still needed to recover the bootdata.
   
1. *Destination* - Load the ISO for the Windows Recovery Environment.

   Click `Repair your computer`
   
   Click `Troubleshoot`
   
   Click `Command Prompt`
   
I followed [this guide](https://woshub.com/how-to-repair-uefi-bootloader-in-windows-8/) to repair the boot info.

1. Look at the new VM disk

   `diskpart`
   
   This leads to the `DISKPART>` prompt.
   
1. Verify the disk is GPT.

   Under "GPT" there should be a star.
   
1. Select Disk 0

   This is the only hard disk in this VM.
   
   `sel disk 0`

1. List the partitions and Volumes

   This is the windows equivalant to fdisk.
   
   `list partition`
   
   `list volume`
   
   This is my lab system.
   
   ```
   DISKPART> list partition 

      Partition ###   Type            Size        Offset
      -------------   --------------  ----------  -------
      Partition 1     System          100 MB      1024 KB
      Partition 2     Reserved        16 MB       101 MB
      Partition 3     Primary         46 GB       117 MB
   
   DISKPART> list volume 
   
      Volume ###  Ltr     Label       Fs      Type        Size        Status      Info 
      ----------  ---     ----------  -----   ----------  -------     ----------  -------
      Volume 0    D       ESD-ISO     UDF     CD-ROM      4667 MB     Healthy  
      Volume 1    C                   NTFS    Partition     46 GB     Healthy  
      Volume 2                        FAT32   Partition    100 MB     Healthy     Hidden
   ```
   
   There are the three required volumes.
   
   * **NTFS** - The data partition, apps and the OS
   * **EFI** - Extensible Firmware Interface. Where the modern boot system lives. Usually 100MB, FAT32 
   * **MSR** - Microsoft System Reserved. Usually 16MB formatted as "MSR". Used by Windows to help manage the file partitions

At this point,  I could just follow along with the Windows OS Hub article, to restore the BCD bootloader configuration.

### References

[Windows OS Hub - How to Repair EFI/GPT Bootloader on Windows 10 or 11](https://woshub.com/how-to-repair-uefi-bootloader-in-windows-8/)

[Microsoft - Disk cleanup in Windows](https://support.microsoft.com/en-us/windows/disk-cleanup-in-windows-8a96ff42-5751-39ad-23d6-434b4d5b9a68)

[Ten Forums - How to Manage Virtual Memory Pagefile in Windows 10](https://www.tenforums.com/tutorials/77692-manage-virtual-memory-pagefile-windows-10-a.html)

[Microsoft - BCD Boot Command Line Options ](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/bcdboot-command-line-options-techref-di?view=windows-10)

[Windows OS Hub - How to repair deleted EFI partition in windows 7](https://woshub.com/how-to-repair-deleted-efi-partition-in-windows-7/)
