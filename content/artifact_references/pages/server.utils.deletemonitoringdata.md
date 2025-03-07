---
title: Server.Utils.DeleteMonitoringData
hidden: true
tags: [Server Artifact]
---

Velociraptor collects monitoring data from endpoints all the time.

Sometimes this data is no longer needed and we might want to free
up disk space.

This artifact searches the monitoring data for each client and
optionally removes data older than the specified timestamp.

**NOTE** This artifact will destroy all data irrevocably. Take
  care! You should always do a dry run first to see which flows
  will match before using the ReallyDoIt option.


```yaml
name: Server.Utils.DeleteMonitoringData
description: |
   Velociraptor collects monitoring data from endpoints all the time.

   Sometimes this data is no longer needed and we might want to free
   up disk space.

   This artifact searches the monitoring data for each client and
   optionally removes data older than the specified timestamp.

   **NOTE** This artifact will destroy all data irrevocably. Take
     care! You should always do a dry run first to see which flows
     will match before using the ReallyDoIt option.

type: SERVER

parameters:
   - name: DateBefore
     default: 2022-01-01
     type: timestamp
   - name: ArtifactRegex
     type: regex
     default: Generic.Client.Stats
   - name: HostnameRegex
     description: If specified only target these hosts
     type: regex
   - name: ReallyDoIt
     type: bool
     description: Do not actually delete until this is set!

sources:
  - query: |
        SELECT * FROM foreach(row={
            SELECT client_id,
                   os_info.hostname AS hostname
            FROM clients()
            WHERE hostname =~ HostnameRegex
        },
        query={
            SELECT FullPath,
                basename(path=dirname(path=FullPath)) AS ArtifactName, Size,
                timestamp(epoch=
                 split(string=basename(path=FullPath), sep="\\.")[0]) AS Timestamp,
                 if(condition=ReallyDoIt, then=file_store_delete(path=FullPath)) AS ReallyDoIt
            FROM glob(
               globs="/**.json*", accessor="fs",
               root="/clients/"+ client_id + "/monitoring")
            WHERE ArtifactName =~ ArtifactRegex
              AND Timestamp < DateBefore
        }, workers=10)

```
