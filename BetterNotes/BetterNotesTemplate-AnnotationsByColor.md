# This template is specifically for importing/sharing, using better 
# notes 'import from clipboard': copy the content and
# goto Zotero menu bar, click Tools->New Template from Clipboard.  
# Do not copy-paste this to better notes template editor directly.
name: "[item]AnnotationsByColor"
zoteroVersion: "10.0.7"
pluginVersion: "3.3.3"
savedAt: "2026-10-31T21:36:08.360Z"
content: |-
  // @author efink
  // @use-markdown
  // @use-refresh
  
  ${{
      sharedObj.colors = {
          "#ffd400": { name: "Background/Facts", annotations: [] },  // Yellow
          "#f19837": { name: "Issue", annotations: [] }, // Orange
          "#2ea8e5": { name: "Claim/Argument", annotations: [] }, // Blue
          "#5fb236": { name: "Analysis", annotations: [] }, // Green
          "#ff6666": { name: "Theory/Rule", annotations: [] }, // Red
          "#e56eee": { name: "Conclusion/Holding", annotations: [] }, // Magenta
          "#a28ae5": { name: "Concern/Problem", annotations: [] }, // Purple
          "#aaaaaa": { name: "Meta", annotations: [] } // Grey  
      };
      return "";
  }}$
  
   ${(() => {
      const article = topItem.getField("publicationTitle");
      const bookSection = topItem.getField("bookTitle");
      const legalCase = topItem.getField("court");
      if (article) {
      	return `${topItem.getField('title')}, ${topItem.getField('volume')} ${topItem.getField('publicationTitle')} ${topItem.getField('pages')} (${topItem.getField('year')})`;
      } else if (legalCase) {
      	return `${topItem.getField('caseName')}, ${topItem.getField('reporterVolume')} ${topItem.getField('reporter')} ${topItem.getField('firstPage')} (${topItem.getField('court')} ${topItem.getField('year')})`;
      } else if (bookSection) {
      	return `${topItem.getField('title')}, in _${topItem.getField('bookTitle')}_ (${topItem.getField('year')})`;
      } else {
      	return `${topItem.getField('title')} (${topItem.getField('year')})`;
      }
    })()}
      
    ${{
      let author=topItem.getCreators().slice(0, 10).map((v) => v.firstName + " " + v.lastName).join("; ") + (topItem.getCreators().length > 10 ? "; et al." : ";");
      let names = String(author);
      if (names.endsWith(';')) {
      names = names.slice(0, -1);
      }
      names=names.split(/;\s*/)
      let wrappedNames = names.map(name => `${name.trim()}`);
      return wrappedNames.join(', ');
    }}$
  
  ## Notes
  
  _Add notes here._
    
  ## PDF Annotations by Color
  
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
  
    ${{
    	let output = ``;
    	for (const color in sharedObj.colors) {
    		const colorData = sharedObj.colors[color];
    		if (colorData.annotations.length > 0) {
    			output += `<h3><p style="background-color:${color}70;">${colorData.name}</p></h3>
    						${colorData.annotations.join("")}`;
    		}
    	}
    	return output;
    }}$

