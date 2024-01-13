## Stopping a Runaway VM
In order of increasing severity...

1st:
`sudo qm shutdown <vmid>`

2nd:
`sudo qm stop <vmid>`

3rd:
`sudo qm shutdown <vmid> --skiplock`

4th:
`sudo qm shutdown <vmid> --skiplock --forceStop`

Valerie:
> From reading the `man` page I believe the first two commands are just the regular Shutdown and Stop commands accessible from the GUI, but run as root. I watched command 4 notify me that it killed something with SIGTERM, after it failed to exit normally. I don't know if it will move on to SIGKILL, I couldn't find adequate documentation on this option.

## Mounting a VM filesystem

**Only do this with the VM off**

These instructions activate the find an unused loop device, activate the LV, and mount the necessary partition.

1. Make a new mount point
   
   `sudo mkdir /mnt/recover-lv-from-vm`

1. Display the Logical Volumes
   
   `sudo lvs`

1. Display the LV disk status, format is `/dev/VG-or-virtual-group/LV-or-logical-volume`
   
   `sudo lvdisplay /dev/pve/vm-100-disk-0`

1. Set the LV to active, which is necessary to mount it
   
   `sudo lvchange -ay /dev/pve/vm-100-disk-0`
   
1. Enumerate the first unused loop device
   
   `losetup -f`

1. Map the partitions on the LV onto the loop device, making block devices
   
   `sudo losetup -P /dev/loopN /dev/pve/vm-100-disk-0`
   
1. Enumerate the new block devices
   
   `lsblk /dev/loop0`
   
1. Mount the new loop-block device
   
   `sudo mount /mnt/recover-lv-from-vm`
