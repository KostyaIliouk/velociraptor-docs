---
title: Windows.Sys.Drivers
hidden: true
tags: [Client Artifact]
---

Details for in-use Windows device drivers. This does not display installed but unused drivers.


```yaml
name: Windows.Sys.Drivers
description: |
  Details for in-use Windows device drivers. This does not display installed but unused drivers.

precondition:
      SELECT OS From info() where OS = 'windows'

sources:
  - name: SignedDrivers
    query: |
       SELECT *
       FROM wmi(
          query="select * from Win32_PnPSignedDriver",
          namespace="ROOT\\CIMV2")

  - name: RunningDrivers
    query: |
       SELECT *
       FROM wmi(
         query="select * from Win32_SystemDriver",
         namespace="ROOT\\CIMV2")

```
