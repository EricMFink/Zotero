---
citekey: {{citationKey}}
title: {% if caseName %}{{caseName}}{% elif nameOfAct %}{{nameOfAct}}{% else %}"{{title | escape}}"{% endif %}
year: {{date | format("YYYY")}}
authors: {{authors}}
collection: {{collections[0].fullPath}} 
cssclasses: ["zotero"]
created: 2025-06-22
modified: 2025-06-22
tags: {% if allTags %}{{allTags}}{% endif %}
---

## {{bibliography}}

Zotero PDF Link: {{pdfZoteroLink}}

### Persistent Notes

{% persist "notes" %}{% if isFirstImport %}
Write notes here!
{% endif %}
{% endpersist %}

### In-text annotations

{% for annotation in annotations -%}
{%- if annotation.annotatedText -%}
{% if annotation.color %} <mark class="hltr-{{annotation.colorCategory | lower}}">"{{annotation.annotatedText | safe}}"</mark> {% else %} {{annotation.type | capitalize}} {% endif %}[Page {{annotation.pageLabel}}](zotero://open-pdf/library/items/{{annotation.attachment.itemKey}}?page={{annotation.pageLabel}}&annotation={{annotation.id}})
{%- endif %}
{% if annotation.comment %}
{{annotation.comment | safe}} [Page {{annotation.pageLabel}}](zotero://open-pdf/library/items/{{annotation.attachment.itemKey}}?page={{annotation.pageLabel}}&annotation={{annotation.id}})
{% endif %}
{%- if annotation.imageRelativePath %} 
![[{{annotation.imageRelativePath}}]]
{%- endif %}
{% if annotation.allTags %}
{{annotation.allTags}}
{% endif %}
{% endfor -%}
