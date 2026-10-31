# This template is specifically for importing/sharing, using better 
# notes 'import from clipboard': copy the content and
# goto Zotero menu bar, click Tools->New Template from Clipboard.  
# Do not copy-paste this to better notes template editor directly.
name: "[item]Main-Item"
zoteroVersion: "10.0.7"
pluginVersion: "3.3.3"
savedAt: "2026-10-31T04:47:39.669Z"
content: |-
  
  // @use-refresh
  // @beforeloop-begin
  ${{
  	sharedObj.colors = {
  	  "#ffd400": { name: "General", annotations: [] },  // Yellow
  	  "#f19837": { name: "Issue", annotations: [] }, // Orange
      "#2ea8e5": { name: "Claim/Argument", annotations: [] }, // Blue
  	  "#5fb236": { name: "Analysis", annotations: [] }, // Green
  	  "#ff6666": { name: "Problem/Critique", annotations: [] }, // Red
      "#e56eee": { name: "Conclusion", annotations: [] }, // Magenta
      "#a28ae5": { name: "Other", annotations: [] }, // Purple
      "#aaaaaa": { name: "Meta", annotations: [] } // Grey  
  	};
  	return "";
  }}$
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
