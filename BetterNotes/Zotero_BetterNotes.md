---
title: Zotero Better Notes
created: 2025-06-21
modified: 2025-06-22
tags:
  - zotero
---

## Templates

### Export MD File Name

To export note using BBT Citation Key as file name, replace default with this:

```
@${(noteItem.parentItem && noteItem.parentItem.getField("citationKey")) || noteItem.key}.md
```

### Export MD File Header

To include BBT Citation Key in YAML header, add this line:

```
${{
  let header = {};_env.dryRun
  header.citekey = noteItem.parentItem
    ? noteItem.parentItem.getField("citekey")
    : "";
  header.title = noteItem.parentItem
    ? noteItem.parentItem.getField("title")
    : "";
  header.opinion = noteItem.parentItem
    ? noteItem.parentItem.getField("caseName")
    : "";
  header.statute = noteItem.parentItem
    ? noteItem.parentItem.getField("nameOfAct")
    : "";
  header.type = noteItem.parentItem.itemType;
  header.collections = (
    await Zotero.Collections.getCollectionsContainingItems([
      (noteItem.parentItem || noteItem).id,
    ])
  ).map((c) => c.name);
  header.tags = noteItem.getTags().map((_t) => _t.tag);
  return JSON.stringify(header);
}}$

```

### Source Notes with PDF Annotations

Uses two files: 

1. SourceNotes 

`Zotero_BetterNotes_Template_SourceNotes.md`

This is the note template, with a header for document metadata, a section for notes, and a section of extracted PDF annotations.

2. Main-Item

`Zotero_BetterNotes_Template_Main-Item.md`

This is the template for creating new notes. It incorporates Template 1 (i.e. SourceNotes), adding a top-level header (based on the Zotero item citation key) that will automatically be used as the file name. This is needed because the `// @use-refresh` code in Template 1 (which enables the "update content from Template" feature in Zotero) adds a code block at the top, so automatic file naming wouldn't work. 


