---
layout: default
title: "Linux KVM Passthrough for Bluetooth Development"
author: "Abhishek Raj"
tags: Linux KVM Passthrough
---

### IOMMU Groups
For getting IOMMU Groups in Ubuntu(We need to detach all the devices in a IOMMU group):
```shell
for g in /sys/kernel/iommu_groups/*;
  do 
    echo "IOMMU Group ${g##*/}:";
    for d in $g/devices/*;
    do
    echo -e "\t$(lspci -nns ${d##*/})";
   done;
done;
```

For me there are 3 devices which we needs to be detached.
```
IOMMU Group 4:
	00:14.0 USB controller [0c03]: Intel Corporation Cannon Lake PCH USB 3.1 xHCI Host Controller [8086:a36d] (rev 10)
	00:14.2 RAM memory [0500]: Intel Corporation Cannon Lake PCH Shared SRAM [8086:a36f] (rev 10)
	00:14.3 Network controller [0280]: Intel Corporation Wireless-AC 9560 [Jefferson Peak] [8086:a370] (rev 10)
```

> Taken from Archlinux wiki: https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF

### Detaching
Get all the ids you want to passthrough(By matching from the above output):
```shell
lspci -n
```
For me these are the ids:
```
00:14.0 0c03: 8086:a36d (rev 10)
00:14.2 0500: 8086:a36f (rev 10)
00:14.3 0280: 8086:a370 (rev 10)
```

Now, detach the required devices:
```shell
virsh nodedev-dettach pci_8086_a36d
virsh nodedev-dettach pci_8086_a36f
virsh nodedev-dettach pci_8086_a370
```

Now since we are passing through a Bluetooth **PCI** Device we need to add the devices before starting the VM, I use virt-manager to add them. You can use the following https://www.linux-kvm.org/page/How_to_assign_devices_with_VT-d_in_KVM for doing it manually using virsh.

### Automating these things:
We can use the libvirt hooks to automate the detaching and reattaching the devices: https://libvirt.org/hooks.html.

We will be using the qemu hook.

Script `/etc/libvirt/hooks/qemu`:

```shell
#!/bin/bash

# $1 is the domain name and $2 is for the action step
if [[ $1 == "ubuntu" ]] && [[ $2 == "start" || $2 == "stopped" || $2 == "prepare" || $2 == "release" ]]
then
  if [[ $2 == "prepare" ]]
  then
    virsh nodedev-dettach pci_8086_a36d
    virsh nodedev-dettach pci_8086_a36f
    virsh nodedev-dettach pci_8086_a370
  elif [[ $2 == "release" ]]
  then
    virsh nodedev-reattach pci_8086_a36d
    virsh nodedev-reattach pci_8086_a36f
    virsh nodedev-reattach pci_8086_a370
  fi
fi
```
