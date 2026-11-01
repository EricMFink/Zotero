# This template is specifically for importing/sharing, using better 
# notes 'import from clipboard': copy the content and
# goto Zotero menu bar, click Tools->New Template from Clipboard.  
# Do not copy-paste this to better notes template editor directly.
name: "[ExportMDFileHeaderV2]"
zoteroVersion: "10.0.6"
pluginVersion: "3.3.3"
savedAt: "2026-10-14T22:23:11.090Z"
content: |-
  ${{
    let header = {};
    header.citekey = noteItem.parentItem
      ? noteItem.parentItem.getField("citationKey")
      : "";
    header.tags = noteItem.parentItem.getTags().map(tagObj=>tagObj.tag);
    return JSON.stringify(header);
  }}$

