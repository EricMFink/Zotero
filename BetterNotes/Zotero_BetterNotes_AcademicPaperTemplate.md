# https://github.com/river-ya/Academic-Paper-Note-Template-for-Zotero-Better-Notes/blob/main/paper_note_template_en.md

${await (async () => {
  const getParentItem = (currentItem) => {
    if (!currentItem) return currentItem;
    const parentID =
      currentItem.parentItemID ||
      currentItem.parentID ||
      (typeof currentItem.getSource === "function" ? currentItem.getSource() : null);
    return parentID ? Zotero.Items.get(parentID) : currentItem.parentItem || currentItem;
  };

  const getRegularItem = (currentItem) => {
    let regularItem = currentItem;
    for (let i = 0; i < 3; i++) {
      if (!regularItem) break;
      const isChildItem =
        (typeof regularItem.isAttachment === "function" && regularItem.isAttachment()) ||
        (typeof regularItem.isAnnotation === "function" && regularItem.isAnnotation()) ||
        (typeof regularItem.isNote === "function" && regularItem.isNote());
      if (!isChildItem) break;

      const parent = getParentItem(regularItem);
      if (!parent || parent.id === regularItem.id) break;
      regularItem = parent;
    }
    return regularItem;
  };

  const inputItem = (typeof topItem !== "undefined" && topItem) ? topItem : items[0];
  const item = getRegularItem(inputItem);

  const esc = (value = "") => String(value)
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;");

  const getField = (field) => item.getField(field) || "";
  const itemType = (() => {
    try {
      const typeName = Zotero.ItemTypes.getName(item.itemTypeID) || "";
      return typeName
        .replace(/([a-z])([A-Z])/g, "$1 $2")
        .replace(/^./, (character) => character.toUpperCase());
    } catch (error) {
      return "";
    }
  })();
  const date = (getField("date").split("T")[0] || getField("year") || "").trim();
  const title = getField("title");
  const titleTranslation = getField("titleTranslation");
  const noteTitle = date
    ? `(${date}) ${title}${titleTranslation ? ` (${titleTranslation})` : ""}`
    : `${title}${titleTranslation ? ` (${titleTranslation})` : ""}`;

  const creators = item.getCreators();
  const authors = creators
    .slice(0, 10)
    .map((v) => `${v.firstName || ""} ${v.lastName || ""}`.trim())
    .filter(Boolean)
    .join("; ") + (creators.length > 10 ? "; et al." : creators.length ? ";" : "");

  const renderPublicationTags = () => {
    if (!Zotero.ZoteroStyle?.api?.renderCell) return "";
    const space = "  ";
    return Array.prototype.map.call(
      Zotero.ZoteroStyle.api.renderCell(item, "publicationTags").childNodes,
      (e) => {
        e.innerText = space + e.innerText + space;
        return e.outerHTML;
      }
    ).join(space);
  };

  const renderJournalAndRanking = () => {
    const journal = getField("publicationTitle") || getField("conferenceName") || getField("publisher");
    const publicationTags = renderPublicationTags();
    const ranking = publicationTags || "<span style=\"color:#7b858d;\">No ranking data available. Install or enable Zotero Style and refresh the journal data.</span>";
    return `<b>Journal: </b><span style="color:#526f7d">${esc(journal)}</span><br><b>Ranking: </b>${ranking}`;
  };

  const renderJournalLine = () => {
    const volume = getField("volume");
    const issue = getField("issue");
    const pages = getField("pages");
    const year = getField("date").split("-")[0] || getField("year");
    let line = "";
    if (volume) {
      line += volume;
      if (issue) line += `(${issue})`;
      line += pages ? `: ${pages},` : ",";
    } else if (issue) {
      line += `(${issue})`;
      line += pages ? `: ${pages},` : ",";
    } else {
      line += pages ? `${pages},` : "";
    }
    line += year ? ` ${year}.` : "";
    return line;
  };

  const renderLocalLink = () => {
    const attachments = (typeof item.isAttachment === "function" && item.isAttachment())
      ? [item]
      : Zotero.Items.get(item.getAttachments());
    const pdfs = attachments.filter((i) => i.isPDFAttachment());
    const attachment = pdfs[0] || attachments[0];
    if (!attachment) return "<b>Local Link: </b>";
    return `<b>Local Link: </b><a href="zotero://open-pdf/0_${attachment.key}">${esc(attachment.getFilename())}</a>`;
  };

  const renderDoiOrUrl = () => {
    const doi = getField("DOI");
    const url = getField("url");
    if (doi) return `<b>DOI: </b><a href="https://doi.org/${esc(doi)}">${esc(doi)}</a>`;
    if (url) return `<b>URL: </b><a href="${esc(url)}">${esc(url)}</a>`;
    return "<b>DOI/URL: </b>";
  };

  const parseComment = (comment = "") => {
    const parsed = comment.match(/^(.*?)🔤(.*?)🔤(.*?)$/s);
    return parsed
      ? {
          comment: `${parsed[1].trim()}${parsed[3].trim()}`,
          translation: `🔤${parsed[2].trim()}🔤`
        }
      : { comment: comment.trim(), translation: "" };
  };

  const renderAnnotation = async (annotation) => {
    const isNoteType = ["note", "text"].includes(annotation.annotationType);
    const htmlContent = await Zotero.BetterNotes.api.convert.annotations2html(
      [annotation],
      {
        noteItem: targetNoteItem,
        ignoreComment: !isNoteType
      }
    );

    if (isNoteType) return htmlContent;

    const parsedComment = parseComment(annotation.annotationComment || "");
    const extra = `${parsedComment.translation ? `<br>${parsedComment.translation}` : ""}${parsedComment.comment ? `<br><b>${esc(parsedComment.comment)}</b>` : ""}`;
    if (!extra) return htmlContent;
    return htmlContent.includes("</p>")
      ? htmlContent.replace("</p>", `${extra}</p>`)
      : `${htmlContent}<p>${extra}</p>`;
  };

  const attachments = (typeof item.isAttachment === "function" && item.isAttachment())
    ? [item]
    : Zotero.Items.get(item.getAttachments())
        .filter((i) => i.isPDFAttachment() || i.isSnapshotAttachment() || i.isEPUBAttachment());
  const annotatedAttachmentCount = attachments
    .filter((attachment) => attachment.getAnnotations().length > 0)
    .length;

  const renderTaggedAnnotations = async (tagNames) => {
    const wantedTags = Array.isArray(tagNames) ? tagNames : [tagNames];
    const chunks = [];
    const seenAnnotationIds = new Set();

    for (const attachment of attachments) {
      const annotations = attachment.getAnnotations().filter((annotation) => {
        const annotationTags = annotation.getTags().map((tagObj) => tagObj.tag);
        return wantedTags.some((tagName) => annotationTags.includes(tagName));
      });

      const rendered = [];
      for (const annotation of annotations) {
        if (seenAnnotationIds.has(annotation.id)) continue;
        seenAnnotationIds.add(annotation.id);
        rendered.push(await renderAnnotation(annotation));
      }

      if (rendered.length) {
        const documentHeader = annotatedAttachmentCount > 1
          ? `<h4>📄 For Document: <a href="zotero://select/items/0_${attachment.key}">${esc(attachment.getField("title") || attachment.getFilename())}</a></h4>`
          : "";
        chunks.push(`${documentHeader}${rendered.join("")}`);
      }
    }

    return chunks.join("");
  };

  const tagged = {
    content: await renderTaggedAnnotations([
      "Research Question / Gap",
      "Background/Institution / Context"
    ]),
    innovations: await renderTaggedAnnotations("Innovation"),
    shortcomings: await renderTaggedAnnotations("Shortage/Shortcomings/Drawbacks"),
    data: await renderTaggedAnnotations("Data"),
    method: await renderTaggedAnnotations([
      "Identification/ Method",
      "Assumption"
    ]),
    experiment: await renderTaggedAnnotations([
      "Mechanism",
      "Heterogeneity"
    ]),
    conclusion: await renderTaggedAnnotations([
      "Finding",
      "Conclusion"
    ])
  };

  const abstractTranslation = getField("abstractTranslation");
  const abstract = abstractTranslation
    ? `<b>Translated Abstract: </b><i>${esc(abstractTranslation)}</i>`
    : `<b>Abstract: </b><i>${esc(getField("abstractNote"))}</i>`;
  const itemTags = item.getTags()
    .map((tagObj) => tagObj.tag)
    .filter((tag) => tag.startsWith("#"))
    .map((tag) => `${esc(tag)} ,`)
    .join(" ");

  const sectionHeading = (number, title) => `
    <h2 style="color:#334a5e; background-color:#e8eef2; border-left:5px solid #607d8b; padding:9px 12px; margin-top:28px; margin-bottom:14px;">
      <span style="color:#78909c; font-size:0.82em; margin-right:8px;">${number}</span>${title}
    </h2>`;
  const subHeading = (title) => `
    <h3 style="color:#3f596c; border-bottom:1px solid #c7d1d8; padding:5px 2px; margin-top:20px; margin-bottom:10px;">
      ${title}
    </h3>`;
  const tip = (text) => `
    <blockquote style="color:#7b858d; background-color:#f5f7f8; border-left:3px solid #c4cdd3; padding:8px 12px; margin:8px 0 14px 0; font-size:0.92em;">
      <span style="color:#98a2a9; font-weight:bold;">Analytical Prompt · </span>${text}
    </blockquote>`;
  const blank = '<p style="min-height:1.2em;"></p>';
  const statusBadge = (label) => `<span style="display:inline-block; color:#6f7e87; background-color:#eef2f4; border:1px solid #d2dbe0; padding:3px 9px; margin:2px 6px 2px 0; font-size:0.9em;">□ ${label}</span>`;

  return `
<!-- Title -->
<div style="border-top:5px solid #526b7a; border-bottom:1px solid #bcc8cf; background-color:#f1f4f6; padding:18px 16px 14px 16px; margin-bottom:16px;">
  <h1 style="color:#263d4b; margin:0 0 7px 0; line-height:1.35;">${esc(noteTitle)}</h1>
  <p style="color:#7b8992; margin:0; font-size:0.92em; letter-spacing:0.04em;">RESEARCH NOTE · LITERATURE REVIEW</p>
</div>

<!-- Paper Information Card -->
<div style="border:1px solid #cbd5da; background-color:#f7f9fa; margin:0 0 18px 0;">
  <div style="color:#445d6d; background-color:#e8eef2; border-bottom:1px solid #cbd5da; padding:8px 11px; font-weight:bold;">Paper Information Card</div>
  <table style="width:100%; border-collapse:collapse; color:#43515a;">
    <tr>
      <td style="width:20%; color:#667983; padding:7px 10px; border-bottom:1px solid #dbe2e6;"><b>Document Type</b></td>
      <td style="padding:7px 10px; border-bottom:1px solid #dbe2e6;">${esc(itemType)}</td>
    </tr>
    <tr>
      <td style="color:#667983; padding:7px 10px; border-bottom:1px solid #dbe2e6;"><b>Research Domain</b></td>
      <td style="padding:7px 10px; border-bottom:1px solid #dbe2e6;"><span style="color:#98a2a9;">Specify the relevant keywords, research task, or disciplinary area:</span>&nbsp;</td>
    </tr>
    <tr>
      <td style="color:#667983; padding:7px 10px;"><b>Reading Status</b></td>
      <td style="padding:7px 10px;">
        ${statusBadge("Abstract Reviewed")}${statusBadge("Rapid Review")}${statusBadge("In-depth Reading")}${statusBadge("Reproduced")}
      </td>
    </tr>
  </table>
</div>

<!-- Meta Data -->
<table style="width:100%; border-collapse:collapse; color:#37474f; margin-bottom:20px;">
  <tr>
    <td style="width:20%; color:#60727d; background-color:#e9eef1; padding:8px 10px; border-bottom:1px solid #d3dce1;"><b>Authors</b></td>
    <td style="background-color:#f7f9fa; padding:8px 10px; border-bottom:1px solid #d3dce1;">${esc(authors)}</td>
  </tr>
  <tr>
    <td style="color:#60727d; background-color:#e9eef1; padding:8px 10px; border-bottom:1px solid #d3dce1;"><b>Source</b></td>
    <td style="background-color:#f7f9fa; padding:8px 10px; border-bottom:1px solid #d3dce1;">
      ${renderJournalAndRanking()}<br><b>Bibliographic Details: </b>${esc(renderJournalLine())}
    </td>
  </tr>
  <tr>
    <td style="color:#60727d; background-color:#e9eef1; padding:8px 10px; border-bottom:1px solid #d3dce1;"><b>Journal Tags</b></td>
    <td style="background-color:#f7f9fa; padding:8px 10px; border-bottom:1px solid #d3dce1;">${renderPublicationTags()}</td>
  </tr>
  <tr>
    <td style="color:#60727d; background-color:#e9eef1; padding:8px 10px; border-bottom:1px solid #d3dce1;"><b>Resources</b></td>
    <td style="background-color:#f7f9fa; padding:8px 10px; border-bottom:1px solid #d3dce1;">${renderLocalLink()}<br>${renderDoiOrUrl()}</td>
  </tr>
  <tr>
    <td style="color:#60727d; background-color:#e9eef1; padding:8px 10px; border-bottom:1px solid #d3dce1;"><b>Abstract</b></td>
    <td style="background-color:#f7f9fa; padding:10px; border-bottom:1px solid #d3dce1; line-height:1.65;">${abstract}</td>
  </tr>
  <tr>
    <td style="color:#60727d; background-color:#e9eef1; padding:8px 10px; border-bottom:1px solid #d3dce1;"><b>Tags</b></td>
    <td style="background-color:#f7f9fa; padding:8px 10px; border-bottom:1px solid #d3dce1;">${itemTags}</td>
  </tr>
  <tr>
    <td style="color:#60727d; background-color:#e9eef1; padding:8px 10px;"><b>Note Date</b></td>
    <td style="color:#687780; background-color:#f7f9fa; padding:8px 10px;">${new Date().toLocaleString()}</td>
  </tr>
</table>

<!-- Research Core -->
${sectionHeading("01", "Research Core")}
${subHeading("Transferable Approaches")}
${tip("Identify transferable elements in the research strategy, figure and table design, argumentative structure, and scholarly writing.")}
${blank}
${subHeading("Original Contributions")}
${tagged.innovations || blank}
${subHeading("Limitations and Shortcomings")}
${tagged.shortcomings || blank}
${subHeading("Implications and Further Directions")}
${tip("Why was this experimental design selected? Are more appropriate alternatives available? To what extent can the conclusions be generalized to other settings? Can the acknowledged limitations be reformulated as new research questions? How might the problem be approached differently?")}
${blank}
${subHeading("Potentially Citable Passages")}
${blank}

<!-- Research Content -->
${sectionHeading("02", "Research Content")}
${subHeading("Research Context and Problem Formulation")}
${tip("Summarize the motivation, state of the field, limitations of prior research, and the specific research gap, and clarify how the study seeks to address that gap.")}
${tagged.content || blank}
${subHeading("Methodology")}
${tip("Specify the theoretical framework, identification strategy, principal assumptions, comparative methods, and the performance differences between the proposed method and established alternatives.")}
${tagged.method || blank}
${subHeading("Core Formulas or Principles")}
${tip("Document the essential equations, notation, objective functions, theoretical foundations, conditions of applicability, and principal assumptions. In addition to transcribing each formula, explain the problem it is intended to resolve.")}
${blank}
${subHeading("Model Architecture and Workflow")}
${tip("Describe the complete workflow in terms of inputs, preprocessing, principal components, training objectives, inference procedures, and outputs. Indicate the data flow and dependencies among components.")}
${blank}
${subHeading("Data")}
${tip("Record the data sources, collection procedures, sample size, key variables, preprocessing methods, representativeness, and potential sources of bias.")}
${tagged.data || blank}
${subHeading("Experiments and Results")}
${tagged.experiment || blank}
${subHeading("Discussion")}
${tip("What mechanisms may account for the observed results? Has the research gap been substantively addressed? Assess the robustness, limits of generalizability, and potential practical or theoretical significance of the conclusions.")}
${blank}
${subHeading("Conclusion")}
${tagged.conclusion || blank}
${subHeading("Related Literature")}
${tip("Organize relevant publications by theoretical foundations, baseline methods, closely related studies, contradictory evidence, and subsequent work. Where possible, include Zotero item links and state explicitly how each publication relates to the present study. If the present paper is difficult to interpret, consult its foundational references first.")}
${blank}

<!-- Personal Summary -->
${sectionHeading("03", "Critical Synthesis")}
${tip("Distinguish carefully among the authors’ claims, the evidence presented in support of those claims, and your own assessment. Sustained critical evaluation is fundamental to rigorous scholarship.")}
${subHeading("Questions and Critical Assessment")}
${blank}
${subHeading("Issues Requiring Further Investigation")}
${blank}
`;
})()}
