// @use-refresh
// @beforeloop-begin
  ${{
	sharedObj.colors = {
		"#ffd400": { name: "Highlights", annotations: [] },  // Yellow
		"#f19837": { name: "Case Facts", annotations: [] }, // Orange
		"#5fb236": { name: "Case Holding", annotations: [] }, // Green
		"#e56eee": { name: "Case Analysis", annotations: [] },  // Magenta
		"#ff6666": { name: "Case Issues", annotations: [] }, // Red
		"#2ae8e5": { name: "Comments", annotations: [] }, // Blue
		"#a28ae5": { name: "Problems", annotations: [] }, // Purple
		"#aaaaaa": { name: "Other", annotations: [] }, // Grey
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
 - Author(s): ${{
  	let author=topItem.getCreators().slice(0, 10).map((v) => v.firstName + " " + v.lastName).join("; ") + (topItem.getCreators().length > 10 ? "; et al." : ";");
	let names = String(author);
	if (names.endsWith(';')) {
	names = names.slice(0, -1);
	}
	names=names.split(/;\s*/)
	let wrappedNames = names.map(name => `${name.trim()}`);
	return wrappedNames.join(', ');
	}}$
- Item Type: ${topItem.itemType}
- Cite Key: ${topItem.getField("citationKey")}
- PDF: ${(() => {
	const attachments = Zotero.Items.get(topItem.getAttachments());
	const pdf = attachments.filter((i) => i.isPDFAttachment());
	if (pdf && pdf.length > 0) {
		return `<a href="zotero://open-pdf/library/items/${pdf[0].key}">${pdf[0].getFilename()}</a>`;
	} else if (attachments && attachments.length > 0) {
		return `<a href="zotero://open-pdf/library/items/${attachments[0].key}">${attachments[0].getFilename()}</a>`;
	} else {
		return ``;
	}
	})()}
- Zotero link: [${topItem.key}](zotero://select/items/1_${topItem.key}) 

## PDF Annotations

  ${{
	async function getAnnotationsByColor(_attachment, color) {
		const annots = _attachment
		.getAnnotations();
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
  
  	const itemHeader = ``;
  
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
		let output = ``;
		for (const color in sharedObj.colors) {
			const colorData = sharedObj.colors[color];
			if (colorData.annotations.length > 0) {
				output += `${colorData.annotations.join("")}`;
			}
			}
			return output;
	}}$
// @afterloop-end
