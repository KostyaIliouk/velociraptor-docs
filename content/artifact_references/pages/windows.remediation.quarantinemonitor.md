---
title: Windows.Remediation.QuarantineMonitor
hidden: true
tags: [Client Event Artifact]
---

An event query that will ensure the client is quarantined.

We re-calculate the quarantine every 10 minutes by default to
account for changes in DNS/connectivity details. When the query is
terminated, we undo the quarantine.


```yaml
name: Windows.Remediation.QuarantineMonitor
description: |
  An event query that will ensure the client is quarantined.

  We re-calculate the quarantine every 10 minutes by default to
  account for changes in DNS/connectivity details. When the query is
  terminated, we undo the quarantine.

type: CLIENT_EVENT

required_permissions:
  - EXECVE

parameters:
  - name: PolicyName
    default: "VelociraptorQuarantine"
  - name: RuleLookupTable
    type: csv
    default: |
        Action,SrcAddr,SrcMask,SrcPort,DstAddr,DstMask,DstPort,Protocol,Mirrored,Description
        Permit,me,,0,any,,53,udp,yes,DNS
        Permit,me,,0,any,,53,tcp,yes,DNS TCP
        Permit,me,,68,any,,67,udp,yes,DHCP
        Block,any,,,any,,,,yes,All other traffic
  - name: MessageBox
    description: |
        Optional message box notification to send to logged in users. 256
        character limit.
  - name: ReloadPeriod
    description: Reload the ipsec policy every this many seconds on the endpoint.
    default: "600"
    type: int

precondition:
  SELECT OS FROM info() WHERE OS = "windows"
     AND version(function="atexit") >= 0

sources:
  - query: |
      -- When the query is done we unset the policy.
      LET _ <= atexit(query={
         SELECT * FROM Artifact.Windows.Remediation.Quarantine(
           PolicyName=PolicyName, RemovePolicy=TRUE)
      })

      SELECT * FROM foreach(
        row={
           SELECT * FROM clock(period=ReloadPeriod, start=now())
           WHERE log(message="Setting quarantine policy")
        },
        query={
          SELECT * FROM Artifact.Windows.Remediation.Quarantine(
            PolicyName=PolicyName, RuleLookupTable=RuleLookupTable,
            MessageBox=MessageBox)
       })

```
