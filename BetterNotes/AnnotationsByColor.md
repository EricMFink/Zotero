# This template is specifically for importing/sharing, using better 
# notes 'import from clipboard': copy the content and
# goto Zotero menu bar, click Tools->New Template from Clipboard.  
# Do not copy-paste this to better notes template editor directly.
name: "[item]Annotations by color"
zoteroVersion: "10.0.7"
pluginVersion: "3.3.3"
savedAt: "2026-10-31T04:39:59.098Z"
content: |-
  // @author efink
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
    	let output = `<h2>Annotations by color</h2>`;
    	for (const color in sharedObj.colors) {
    		const colorData = sharedObj.colors[color];
    		if (colorData.annotations.length > 0) {
    			output += `<h3><p style="background-color:${color}70;">${colorData.name}</p></h3>
    						${colorData.annotations.join("")}`;
    		}
    	}
    	return output;
    }}$
  // @afterloop-end
  