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

  - name: UseTracker
    type: bool
    description: If set we use the process tracker.

precondition: SELECT OS From info() where OS = 'windows'

sources:
  - query: |
        LET ProcList = SELECT * FROM if(condition=UseTracker,
        then={
          SELECT Pid, Ppid, NULL AS TokenIsElevated,
                 Username, Name, CommandLine, Exe, NULL AS Memory
          FROM process_tracker_pslist()
        }, else={
          SELECT * FROM pslist()
        })

        SELECT Pid, Ppid, TokenIsElevated, Name, CommandLine, Exe,
               hash(path=Exe) as Hash,
               authenticode(filename=Exe) AS Authenticode,
               Username, Memory.WorkingSetSize AS WorkingSetSize
        FROM ProcList
        WHERE Name =~ processRegex

```
