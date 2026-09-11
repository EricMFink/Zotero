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
