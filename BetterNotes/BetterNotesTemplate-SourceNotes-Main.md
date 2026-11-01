# This template is specifically for importing/sharing, using better 
# notes 'import from clipboard': copy the content and
# goto Zotero menu bar, click Tools->New Template from Clipboard.  
# Do not copy-paste this to better notes template editor directly.
name: "[item]SourceNotes-Main"
zoteroVersion: "10.0.7"
pluginVersion: "3.3.3"
savedAt: "2026-10-31T20:34:55.499Z"
content: |-
  // @beforeloop-begin
  // @beforeloop-end
  
  // @default-begin
  // @use-markdown
  # @${topItem.getField('citationKey')}
  // @default-end
  
  // @afterloop-begin
  ${await Zotero.BetterNotes.api.template.runItemTemplate("[item]SourceNotes", {
  	itemIds: items.map(item=>item.id),
  	targetNoteId: targetNoteItem?.id,
  	dryRun: _env.dryRun
  })}
  // @afterloop-end

