---
citekey: {{citationKey}}
title: {% if caseName %}{{caseName}}{% elif nameOfAct %}{{nameOfAct}}{% else %}"{{title}}"{% endif %}{% if date %}
year: {{date | format("YYYY")}}{% endif %}{% if authors %}
authors: {{authors}}{% endif %}
collection: {{collections[0].fullPath}} 
tags: {% if allTags %}{{allTags}}{% endif %}
cssclasses: ["zotero"]
created: 2025-07-14
modified: 2025-10-11
---

{{bibliography}}

### Persistent Notes

{% persist "notes" %}{% if isFirstImport %}
Write notes here!
{% endif %}
{% endpersist %}

### In-text annotations

{% for annotation in annotations -%}
{%if annotation.annotatedText -%}
{% if annotation.color %} "{{annotation.annotatedText | safe}}" {% else %} {{annotation.type | capitalize}} {% endif %}[Page {{annotation.pageLabel}}](zotero://open-pdf/library/items/{{annotation.attachment.itemKey}}?page={{annotation.pageLabel}}&annotation={{annotation.id}})
{%endif %}
{% if annotation.comment %}
{{annotation.comment | safe}} [Page {{annotation.pageLabel}}](zotero://open-pdf/library/items/{{annotation.attachment.itemKey}}?page={{annotation.pageLabel}}&annotation={{annotation.id}})
{% endif %}
{%if annotation.imageRelativePath %} 
![[{{annotation.imageRelativePath}}]]
{%endif %}
{% if annotation.allTags %}
{{annotation.allTags}}
{% endif %}
{% endfor -%}
