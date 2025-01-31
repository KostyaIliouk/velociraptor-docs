---
title: Windows.Attack.ParentProcess
hidden: true
tags: [Client Artifact]
---

Maps the Mitre Att&ck framework process executions into artifacts.

### References:
* https://www.sans.org/security-resources/posters/hunt-evil/165/download
* https://github.com/teoseller/osquery-attck/blob/master/windows-incorrect_parent_process.conf


```yaml
name: Windows.Attack.ParentProcess
description: |
  Maps the Mitre Att&ck framework process executions into artifacts.

  ### References:
  * https://www.sans.org/security-resources/posters/hunt-evil/165/download
  * https://github.com/teoseller/osquery-attck/blob/master/windows-incorrect_parent_process.conf

precondition: SELECT OS From info() where OS = 'windows'

parameters:
  - name: lookupTable
    type: csv
    default: |
       ProcessName,ParentRegex
       smss.exe,System
       runtimebroker.exe,svchost.exe
       taskhostw.exe,svchost.exe
       services.exe,wininit.exe
       lsass.exe,wininit.exe
       svchost.exe,services.exe
       cmd.exe,explorer.exe
       powershell.exe,explorer.exe
       iexplore.exe,explorer.exe
       firefox.exe,explorer.exe
       chrome.exe,explorer.exe

sources:
     - query: |
         // Build up some cached queries for speed.
         LET processes <= SELECT Name, Pid, Ppid, CommandLine, CreateTime, Exe
         FROM pslist()

         LET processes_lookup <= SELECT Name As ProcessName, Pid As ProcID
         FROM processes

         // Resolve the Ppid into a parent name using our processes_lookup
         LET resolved_parent_name = SELECT * FROM foreach(
           row={ SELECT * FROM processes},
           query={
             SELECT Name AS ActualProcessName,
                  ProcessName AS ActualParentName,
                  Pid, Ppid, CommandLine, CreateTime, Exe
             FROM processes_lookup
             WHERE ProcID = Ppid LIMIT 1
           })

         // Get the expected parent name from the table above.
         SELECT * FROM foreach(
           row=resolved_parent_name,
           query={
             SELECT ActualProcessName,
                    ActualParentName,
                    Pid, Ppid, CommandLine, CreateTime, Exe,
                    ParentRegex as ExpectedParentName
             FROM lookupTable
             WHERE ActualProcessName =~ ProcessName
               AND NOT ActualParentName =~ ParentRegex
          })

```
