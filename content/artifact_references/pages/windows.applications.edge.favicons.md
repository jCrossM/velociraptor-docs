---
title: Windows.Applications.Edge.Favicons
hidden: true
tags: [Client Artifact]
---

Enumerate the users edge favicons.

Chrome Favicons are stored in the 'Favicons' SQLite database, within
the 'favicons', 'favicon_bitmaps' and 'icon_mapping' tables. Older
versions of Chrome stored Favicons in a 'Thumbnails' SQLite
database, within the 'favicons' table.


```yaml
name: Windows.Applications.Edge.Favicons
description: |
  Enumerate the users edge favicons.

  Chrome Favicons are stored in the 'Favicons' SQLite database, within
  the 'favicons', 'favicon_bitmaps' and 'icon_mapping' tables. Older
  versions of Chrome stored Favicons in a 'Thumbnails' SQLite
  database, within the 'favicons' table.

references:
  - https://www.foxtonforensics.com/browser-history-examiner/chrome-history-location

author: Phill Moore, @phillmoore

parameters:
  - name: faviconsGlob
    default: /AppData/Local/Microsoft/Edge/User Data/*/Favicons

  - name: faviconsQuery
    default: |
      SELECT favicons.id AS ID,
             favicon_bitmaps.icon_id AS IconID,
             favicon_bitmaps.image_data as _image,
             datetime( favicon_bitmaps.last_updated / 1000000 + ( strftime( '%s', '1601-01-01' ) ), 'unixepoch', 'localtime' ) AS LastUpdated,
             icon_mapping.page_url AS PageURL,
             favicons.url AS FaviconURL
             FROM favicons
             INNER JOIN icon_mapping
             INNER JOIN favicon_bitmaps
             ON icon_mapping.icon_id = favicon_bitmaps.icon_id
             AND favicons.id = favicon_bitmaps.icon_id
             ORDER BY favicons.id ASC
  - name: userRegex
    default: .
    type: regex

precondition: |
  SELECT OS From info() where OS = 'windows'

sources:
  - query: |
        LET favicons_files = SELECT * from foreach(
          row={
             SELECT Uid, Name AS User,
                    expand(path=Directory) AS HomeDirectory
             FROM Artifact.Windows.Sys.Users()
             WHERE Name =~ userRegex
          },
          query={
             SELECT User, OSPath, Mtime
             FROM glob(globs=faviconsGlob, root=HomeDirectory)
          })

        SELECT * FROM foreach(row=favicons_files,
          query={
            SELECT ID, IconID, LastUpdated, PageURL, FaviconURL,
                   upload(accessor="data",
                          file=_image,
                          name=format(format="Image%v.png", args=ID)) AS Image
            FROM sqlite(
              file=OSPath,
              query=faviconsQuery)
          })

column_types:
- name: Image
  type: preview_upload

```
