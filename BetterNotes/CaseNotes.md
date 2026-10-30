# This template is specifically for importing/sharing, using better

# notes 'import from clipboard': copy the content and

# goto Zotero menu bar, click Tools->New Template from Clipboard.

# Do not copy-paste this to better notes template editor directly.

name: "[item]LegalCaseNotes"
zoteroVersion: "10.0.6"
pluginVersion: "1.0.0"
savedAt: "2026-10-31T17:30:00.947Z"
content: |-
  // @ author efink@elon.edu

  // @use-markdown

  ${(() => {
      const legalCase = topItem.getField("court");{
          return `${topItem.getField('caseName')}, ${topItem.getField('reporterVolume')} ${topItem.getField('reporter')} ${topItem.getField('firstPage')} (${topItem.getField('court')} ${topItem.getField('year')})`;
      }
      })()}

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
            output += `<blockquote>${annot.text}${page}</blockquote>
                ${annot.comment ? `<p>{${annot.comment}}</p>` : ""}`;
          }
    
          // YELLOW → QUOTE + COMMENT
          else if (color === "#ffd400") {
            output += `<blockquote>${annot.text}${page}</blockquote>
                ${annot.comment ? `<p>{${annot.comment}}</p>` : ""}`;
          }
    
          // MAGENTA → QUOTE + COMMENT
          else if (color === "#e56eee") {
            output += `<blockquote>${annot.text}${page}</blockquote>
                ${annot.comment ? `<p>{${annot.comment}}</p>` : ""}`;
          }
    
          // GREEN → QUOTE + COMMENT
          else if (color === "#5fb236") {
            output += `<blockquote>${annot.text}${page}</blockquote>
                ${annot.comment ? `<p>{${annot.comment}}</p>` : ""}`;
          }
    
          // RED → COMMENT ONLY
          else if (color === "#ff6666") {
            output += `<blockquote>${annot.text}${page}</blockquote>
                ${annot.comment ? `<p>{${annot.comment}}</p>` : ""}`;
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
