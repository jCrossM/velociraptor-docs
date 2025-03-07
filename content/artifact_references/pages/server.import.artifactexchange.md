---
title: Server.Import.ArtifactExchange
hidden: true
tags: [Server Artifact]
---

This artifact will automatically import the latest
artifact exchange bundle into the current server.


```yaml
name: Server.Import.ArtifactExchange
description: |
   This artifact will automatically import the latest
   artifact exchange bundle into the current server.

type: SERVER

required_permissions:
- SERVER_ADMIN

parameters:
   - name: ExchangeURL
     default: https://github.com/Velocidex/velociraptor-docs/raw/gh-pages/exchange/artifact_exchange.zip
   - name: Prefix
     description: Add artifacts with this prefix
     default: Exchange.

sources:
  - query: |
        LET X = SELECT artifact_set(prefix=Prefix, definition=Definition) AS Definition
        FROM foreach(row={
          SELECT Content FROM http_client(
             remove_last=TRUE,
             tempfile_extension=".zip", url=ExchangeURL)
        }, query={
          SELECT read_file(accessor="zip", filename=OSPath) AS Definition
          FROM glob(
             globs='/**/*.yaml',
             root=pathspec(
                DelegateAccessor="auto",
                DelegatePath=Content),
             accessor="zip")
        })

        SELECT Definition.name AS Name,
               Definition.description AS Description,
               Definition.author AS Author
        FROM X

```
