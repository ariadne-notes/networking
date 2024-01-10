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
