# This template is specifically for importing/sharing, using better 
# notes 'import from clipboard': copy the content and
# goto Zotero menu bar, click Tools->New Template from Clipboard.  
# Do not copy-paste this to better notes template editor directly.
name: "[item]Annotations by color → item"
zoteroVersion: "7.0.11"
pluginVersion: "2.2.5"
savedAt: "2025-01-20T21:20:08.061Z"
content: |-
// @author [fabianlatz](https://github.com/fabianlatz)
// @link [#1272](https://github.com/windingwind/zotero-better-notes/discussions/1272)
// @use-refresh
// @beforeloop-begin
  ${{
  	sharedObj.colors = {
  		"#ffd400": { name: "Important", annotations: [] },  // Yellow
  		"#a28ae5": { name: "Background information", annotations: [] }, // Purple
  		"#f19837": { name: "Other relevant sources", annotations: [] }, // Orange
  		"#5fb236": { name: "Advantages", annotations: [] }, // Green
  		"#ff6666": { name: "Challenges", annotations: [] }, // Red
  		"#aaaaaa": { name: "Practical application", annotations: [] }, // Grey
  		"#e56eee": { name: "Research gaps", annotations: [] }, // Magenta
  		"#2ea8e5": { name: "Other", annotations: [] } // Blue
  	};
  	return "";
  }}$
// @beforeloop-end
  
// @default-begin
// @use-markdown

- Source: ${(() => {
			const article = topItem.getField("publicationTitle");
			const bookSection = topItem.getField("bookTitle");
			const legislation = topItem.getField("code");
            const standard = topItem.getField("type");
            const legalCase = topItem.getField("court");
            if (article) {
              return `${topItem.getField('title')}, ${topItem.getField('volume')} ${topItem.getField('publicationTitle')} ${topItem.getField('pages')} (${topItem.getField('year')})`;
            } else if (legalCase) {
              return `${topItem.getField('caseName')}, ${topItem.getField('reporterVolume')} ${topItem.getField('reporter')} ${topItem.getField('firstPage')} (${topItem.getField('court')} ${topItem.getField('year')})`;
			}  else if (legislation) {
              return `${topItem.getField('nameofAct')}, ${topItem.getField('volume')} ${topItem.getField('code')} ${topItem.getField('section')}`;
            } else if (standard) {
              return `${topItem.getField('type')} ${topItem.getField('number')}`;
			} else if (bookSection) {
              return `${topItem.getField('title')}, in _${topItem.getField('bookTitle')}_ (${topItem.getField('year')})`;
            } else {
              return `${topItem.getField('title')} (${topItem.getField('year')})`;
            }
          })()}
  
  ${{
  	async function getAnnotationsByColor(_attachment, color) {
  		const annots = _attachment
  		.getAnnotations()
  		.filter((item) => item.annotationColor === color);
  		if (annots.length === 0) {
  			return "";
  		}
  		return Zotero.BetterNotes.api.convert.annotations2html(annots, {
  			noteItem: targetNoteItem,
  		});
  	}
  
  	const getIdentifier = () => {
  		const citationKey = topItem.getField("citationKey");
  		if (citationKey) {
  			return citationKey;
  		}
  		const creators = topItem.getCreators();
  		if (creators.length === 0) {
  			return "No author";
  		}
  		const year = topItem.getField("year");
  		return `${creators[0].lastName}, ${year}`;
  	};
  
  	const itemHeader = `<h2>${getIdentifier()} <br/>
  						<i>${topItem.getField("title")}</i></h2>`;
  
  	const attachments = Zotero.Items.get(topItem.getAttachments())
  		.filter((i) => i.isPDFAttachment() || i.isSnapshotAttachment() || i.isEPUBAttachment()
      );
  
  	await Promise.all(Object.keys(sharedObj.colors).map(async (color) => {
  		const annotationsPromises = attachments.map((attachment) => getAnnotationsByColor(attachment, color));
  		const renderedAnnotationsArray = await Promise.all(annotationsPromises);
  		const renderedAnnotations = renderedAnnotationsArray.filter(Boolean).join("");
  		if (renderedAnnotations) {
  			sharedObj.colors[color].annotations.push(itemHeader, renderedAnnotations);
  		}
  	}));
  
  	return "";
  }}$
// @default-end
  
// @afterloop-begin
  ${{
  	let output = `<h1>Annotations by color → item</h1>`;
  	for (const color in sharedObj.colors) {
  		const colorData = sharedObj.colors[color];
  		if (colorData.annotations.length > 0) {
  			output += `<h1><p style="background-color:${color}70;">${colorData.name}</p></h1>
  						${colorData.annotations.join("")}`;
  		}
  	}
  	return output;
  }}$
// @afterloop-end
