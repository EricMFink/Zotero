# This template is specifically for importing/sharing, using better 
# notes 'import from clipboard': copy the content and
# goto Zotero menu bar, click Tools->New Template from Clipboard.  
# Do not copy-paste this to better notes template editor directly.
name: "[item]OrderPreserving_ColorSemantic_annotations"
zoteroVersion: "8.0.4"
pluginVersion: "3.0.3"
savedAt: "2026-03-29T15:21:37.429Z"
content: |-
  // @ author pcschreiber1
  <h1>${topItem.getField('title') || "Untitled"}</h1>
  
  <strong>Authors:</strong> ${topItem.getCreators().map(v => v.lastName || "").join(", ") || "Unknown"}<br/>
  <strong>Year:</strong> ${topItem.getField('date') || "N/A"}
  
  <h2>Reason to read the paper/question to answer</h2>
  
  <h2>Summary</h2>
  ${{
    try {
      const abstract = topItem.getField('abstractNote');
      return abstract ? `<p>${abstract}</p>` : "<p><em>No summary available</em></p>";
    } catch (e) {
      Zotero.logError(e);
      return "<p><em>Error retrieving summary</em></p>";
    }
  }}$
  
  <h2>Notes</h2>
  
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
  
      return output || "<p><em>No annotations found</em></p>";
  
    } catch (e) {
      Zotero.logError(e);
      return "<p><em>Error processing annotations</em></p>";
    }
  }}$
