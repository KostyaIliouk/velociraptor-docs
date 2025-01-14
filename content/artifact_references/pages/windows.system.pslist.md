---
title: Windows.System.Pslist
hidden: true
tags: [Client Artifact]
---

List processes and their running binaries.


```yaml
name: Windows.System.Pslist
description: |
  List processes and their running binaries.

parameters:
  - name: processRegex
    default: .
    type: regex

precondition: SELECT OS From info() where OS = 'windows'

sources:
  - query: |
        SELECT Pid, Ppid, TokenIsElevated, Name, CommandLine, Exe,
               hash(path=Exe) as Hash,
               authenticode(filename=Exe) AS Authenticode,
               Username, Memory.WorkingSetSize AS WorkingSetSize
        FROM pslist()
        WHERE Name =~ processRegex

```
