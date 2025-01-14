---
title: Windows.Sys.DiskInfo
hidden: true
tags: [Client Artifact]
---

Retrieve basic information about the physical disks of a system.

```yaml
name: Windows.Sys.DiskInfo
description: Retrieve basic information about the physical disks of a system.
sources:
  - precondition:
      SELECT OS From info() where OS = 'windows'
    query: |
        SELECT Partitions,
               Index as DiskIndex,
               InterfaceType as Type,
               PNPDeviceID,
               DeviceID,
               Size,
               Manufacturer,
               Model,
               Name,
               SerialNumber,
               Description
        FROM wmi(
           query="SELECT * from Win32_DiskDrive",
           namespace="ROOT\\CIMV2")

```
