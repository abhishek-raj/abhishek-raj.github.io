---
layout: default
title: "Linux KVM Passthrough"
author: "Abhishek Raj"
tags: Linux KVM Passthrough
---

# For getting IOMMU Groups in Ubuntu
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
> Taken from Archlinux wiki: https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF
