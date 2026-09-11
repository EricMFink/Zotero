# This template is specifically for importing/sharing, using better 
# notes 'import from clipboard': copy the content and
# goto Zotero menu bar, click Tools->New Template from Clipboard.  
# Do not copy-paste this to better notes template editor directly.
# based on @pcschreiber1 https://github.com/windingwind/zotero-better-notes/discussions/1557
name: "[item]OrderPreserving_ColorSemantic_annotations"
zoteroVersion: "10.0.1"
pluginVersion: "1.0.1"
savedAt: "2026-08-26T16:00:00
content: |-
// @ author efink@elon.edu
// @use-refresh

// @use-markdown

- Title: ${topItem.getField("title")}
- Author(s): ${topItem.getCreators().map((au) => au.firstName + " " + au.lastName).join("; ")}
- Date: ${topItem.getField("date")}
- Item Type: ${topItem.itemType}
- Cite Key: ${topItem.citationKey ? topItem.citationKey : ""}
- Tags: ${topItem.getTags().map(tagObj=>tagObj.tag)}`
- PDF: ${{
	async function getPDFLink(item) {
	const att = await item.getBestAttachment();
	if (!att || !att.isPDFAttachment()) {
	  return "";
	}
	key = att.key;
	if (att.libraryID === 1) {
	  return `zotero://open/library/items/${key}`;
	} else {
	  groupID = Zotero.Libraries.get(att.libraryID).id;
	  return `zotero://open/groups/${groupID}/items/${key}`;
	}
	}
	sharedObj.getPDFLink = getPDFLink;
	return await getPDFLink(topItem);
	}}$


## Notes

Add notes here. 
  
## PDF Annotations
  
  ${{
    try {
  
      async function getAnnotation(item) {
        try {
          if (!item || !item.isAnnotation()) return null;
  
          let json = await Zotero.Annotations.toJSON(item);
          json.id = item.key;
          delete json.key;
  
          json.text = json.text || "";
          json.comment = json.comment || "";
          json.color = json.color || "";
  
          return json;
        } catch (e) {
          Zotero.logError(e);
          return null;
        }
      }
  
      const attachments = Zotero.Items.get(topItem.getAttachments() || [])
        .filter(i => i && i.isPDFAttachment && i.isPDFAttachment());
  
      let allAnnotations = [];
  
      for (const att of attachments) {
        try {
          const annots = att.getAnnotations ? att.getAnnotations() : [];
          const jsonAnnots = await Promise.all(annots.map(a => getAnnotation(a)));
          allAnnotations.push(...jsonAnnots.filter(Boolean));
        } catch (e) {
          Zotero.logError(e);
        }
      }
  
      allAnnotations.sort((a, b) => {
        if ((a.pageIndex ?? 0) !== (b.pageIndex ?? 0)) {
          return (a.pageIndex ?? 0) - (b.pageIndex ?? 0);
        }
        return (a.sortIndex ?? 0) - (b.sortIndex ?? 0);
      });
  
      let output = "";
  
      for (const annot of allAnnotations) {
  
        const color = (annot.color || "").toLowerCase();
  
        const page = annot.pageLabel
          ? ` (p. ${annot.pageLabel})`
          : (annot.pageIndex != null ? ` (p. ${annot.pageIndex + 1})` : "");
  
        // GREY → SECTION TITLE
        if (color === "#aaaaaa") {
          output += `<h3>${annot.text}${page}</h3>`;
          if (annot.comment) {
            output += `<p>{${annot.comment}}</p>`;
          }
        }
  
        // YELLOW → QUOTE + COMMENT
        else if (color === "#ffd400") {
          output += `<ul>
            <li>
              <blockquote>${annot.text}${page}</blockquote>
              ${annot.comment ? `<ul><li>{${annot.comment}}</li></ul>` : ""}
            </li>
          </ul>`;
        }
  
        // BLUE → COMMENT ONLY
        else if (color === "#2ea8e5") {
          if (annot.comment) {
            output += `<p>${annot.comment}${page}</p>`;
          }
        }
      }
  
      return output || "<p>No annotations found</p>";
  
    } catch (e) {
      Zotero.logError(e);
      return "<p>Error processing annotations</p>";
    }
  }}$
