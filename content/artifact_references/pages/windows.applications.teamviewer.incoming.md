---
title: Windows.Applications.TeamViewer.Incoming
hidden: true
tags: [Client Artifact]
---

Parses the TeamViewer Connections_incoming.txt log file.

When inbound logging enabled, this file will show all inbound TeamViewer
connections.


```yaml
name: Windows.Applications.TeamViewer.Incoming
description: |
   Parses the TeamViewer Connections_incoming.txt log file.

   When inbound logging enabled, this file will show all inbound TeamViewer
   connections.

author: Matt Green - @mgreen27

reference:
  - https://attack.mitre.org/techniques/T1219/
  - https://www.systoolsgroup.com/forensics/teamviewer/


type: CLIENT
parameters:
  - name: FileGlob
    default: C:\Program Files (x86)\TeamViewer\Connections_incoming.txt
  - name: DateAfter
    description: "search for events after this date. YYYY-MM-DDTmm:hh:ss Z"
    type: timestamp
  - name: DateBefore
    description: "search for events before this date. YYYY-MM-DDTmm:hh:ss Z"
    type: timestamp
  - name: TeamViewerIDRegex
    description: "Regex of TeamViewer ID"
    default: .
    type: regex
  - name: SourceHostRegex
    description: "Regex of source host"
    default: .
    type: regex
  - name: UserRegex
    description: "Regex of user"
    default: .
    type: regex
  - name: SearchVSS
    description: "Add VSS into query."
    type: bool


sources:
  - query: |
      -- Build time bounds
      LET DateAfterTime <= if(condition=DateAfter,
        then=DateAfter, else="1600-01-01")
      LET DateBeforeTime <= if(condition=DateBefore,
        then=DateBefore, else="2200-01-01")

      -- expand provided glob into a list of paths on the file system (fs)
      LET fspaths <= SELECT FullPath FROM glob(globs=expand(path=FileGlob))

      -- function returning list of VSS paths corresponding to path
      LET vsspaths(path) = SELECT FullPath
        FROM Artifact.Windows.Search.VSS(SearchFilesGlob=path)

      LET parse_log(FullPath) = SELECT FullPath,
          parse_string_with_regex(
            string=Line,
            regex="^(?P<TeamViewerID>^\\d+)\\s+"+
              "(?P<SourceHost>.+)\\s" +
              "(?P<StartTime>\\d{2}-\\d{2}-\\d{4}\\s\\d{2}:\\d{2}:\\d{2})\\s" +
              "(?P<EndTime>\\d{2}-\\d{2}-\\d{4}\\s\\d{2}:\\d{2}:\\d{2})\\s" +
              "(?P<User>.+)\\s+" +
              "(?P<ConnectionType>[^\\s]+)\\s+" +
              "(?P<ConnectionID>.+)$") as Record
        FROM parse_lines(filename=FullPath)
        WHERE Line
          AND Record.TeamViewerID =~ TeamViewerIDRegex
          AND Record.SourceHost =~ SourceHostRegex
          AND Record.User =~ UserRegex

      -- function returning IOC hits
      LET logsearch(PathList) = SELECT * FROM foreach(
            row=PathList,
            query={
               SELECT *, timestamp(epoch=Record.StartTime,
                                format="02-01-2006 15:04:05") AS StartTime,
                      timestamp(epoch=Record.EndTime,
                                format="02-01-2006 15:04:05") AS EndTime
               FROM parse_log(FullPath=FullPath)
               WHERE StartTime < DateBeforeTime
                    AND StartTime > DateAfterTime
                    AND EndTime < DateBeforeTime
                    AND EndTime > DateAfterTime
            })

      -- include VSS in calculation and deduplicate with GROUP BY by file
      LET include_vss = SELECT * FROM foreach(row=fspaths,
            query={
                SELECT *
                FROM logsearch(PathList={
                        SELECT FullPath FROM vsspaths(path=FullPath)
                    })
                GROUP BY Record
              })

      -- exclude VSS in logsearch`
      LET exclude_vss = SELECT * FROM logsearch(PathList={SELECT FullPath FROM fspaths})


      -- return rows
      SELECT
        Record.TeamViewerID as TeamViewerID,
        Record.SourceHost as SourceHost,
        StartTime,
        EndTime,
        Record.User as User,
        Record.ConnectionType as ConnectionType,
        Record.ConnectionID as ConnectionID,
        FullPath
      FROM if(condition=SearchVSS,
            then=include_vss,
            else=exclude_vss)

```
